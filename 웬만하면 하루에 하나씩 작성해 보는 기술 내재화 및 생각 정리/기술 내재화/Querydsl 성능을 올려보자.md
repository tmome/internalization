# 실제로 사용해본 Querydsl 성능 올리기 
> - extends / implements 제거
>   - 불필요한 코드를 제거할 수 있고 공통으로 사용이 가능해진다.
> - 동적쿼리는 BooleanExpression
>   - 메소드로 만들었기 떄문에 해당 값이 null로 리턴이 될 경우 조건 제거
> - exist 메소드 사용 금지
>   - 첫번째 값이 발견 될 경우 중지되는데 반면에 count는 발견된다 하더라도 끝까지 모든 조건들을 체크하게 됌
>   - queryDsl의 exist는 실제로는 sql의 exist를 사용하지 않고 count를 내부적으로 사용해 성능 저하가 될 수 있음
>   - limit 1로 조회를 제한을 두고 return이 null이 때문에 null 체크 해야 됌
> - Cross Join 회피
>   - 나올 수 있는 모든 대상에 대해서 조회를 하기 때문에 성능이 좋을 수 없다.
>   - ORM를 사용하다 보면 실제로 join 문을 사용하지 않더라도 묵시적 Join이 발생하면서 cross join 발생
>   - 명시적 Join으로 회피
> - Entity 보다는 Dto 우선
>   - Entity 조회시 Hibernate 캐시 불필요한 컬럼 조회 OneToOne N+1 쿼리등 단순 조회 기능에서는 성능 이슈 요소가 많다.
>   - Entity 조회
>     - 실시간으로 Entity 변경이 필요한 경우
>   - Dto 조회
>     - 고강도 성능 개선 또는 대량의 데이터 조회가 필요한 경우
>   - Select 컬럼에 entity 자제
>     - Order 와 User 가 OneToOne 관계라면 Order에 User Entity를 조회 시 매 건마다 조회가 되는 이슈가 있음
>       - OneToOne은 Lazy Loading이 되지 않는다 N+1문제가 무조건 발생 
>   - distinct 가 있다면 select에 선언된 Entity 컬럼 전체가 distinct의 대상이 되기 더 성능이 안좋아 진다.
> - Group By 최적화
>   - MySQL에서 Group By를 실행하면 Filesort가 필수로 발생 (Index가 아닌 경우)
>     - order by null을 사용하면 Filesort가 제거된다.
>     - Querydsl은 지원하지 않는다 직접 구현해야함
>     - 정렬이 필요하더라도, 조회 결과가 많지 않는다면 어플리케이션 레벨에서 정렬하는게 비용적으로 더 값싸다
>       - DB보단 WAS를 사용하는게 더 낫다(WAS 는 상대적으로 더 많기 때문에)
>     - 페이징일 경우, order by null을 사용하지 못한다.
>     - v8.0부턴 해당 문제들에 대한 성능 최적화가 된것으로 알고 있음
> - 커버링 인덱스
>   - 쿼리를 충족시키는데 필요한 모든 컬럼을 갖고 있는 인덱스
>   - JPQL은 from 절 서브쿼리를 지원하지 않는다.
>   - 키로 빠르게 조회하고, 조회된 키로 in 절로 후속 조회한다.
> - 일괄 Update 최적화
>   - 무분별한 DirtyChecking은 피하자
>   - Querydsl.update 호출
>     - 하이버네이트 캐시는 일괄 업데이트시 캐시 갱신이 안되서 Cache Eviction이 필요하다. 
>   - bulk Insert 시 batch, nativeQuery를 이용하는게 성능에 더 유리하다.

```kotlin
import java.beans.Expression

/*
Spring Bean 으로 JPAQueryFactory 및 SQLQueryFactory 를 등록시켜줌으로써 Persistence layer 에서 queryFactory만 주입하여 사용이 가능하게 된다.
 */
@Configuration
class QueryDslConfig(
    @PersistenceContext
    private val entityManager: EntityManager,
) {

    @Bean
    fun jpaQueryFactory(): JPAQueryFactory {
        return JPAQueryFactory(entityManager)
    }

    @Bean
    fun sqlQueryFactory(
        @Qualifier(DbConfig.ROUTING_DATA_SOURCE) dataSource: DataSource,
    ): SQLQueryFactory {
        val sqlTemplates = MySQLTemplates.builder().build()
        val queryDslConfiguration = com.querydsl.sql.Configuration(sqlTemplates)
        return SQLQueryFactory(queryDslConfiguration, SpringConnectionProvider(dataSource))
    }
}

//정책적으로 적당한 예시가 아닐 수 있지만 참고용 
interface OrderRepository : JpaRepository<OrderEntity, Int>, OrderQueryDslRepository

interface OrderQueryDslRepository {
    fun existOrder(userId: Int?): Boolean
}

class OrderQueryDslRepositoryImpl(
    private val jpaQueryFactory: JPAQueryFactory
) : OrderQueryDslRepository {

    private val order = QOrderEntity.orderEntity
    private val user = QUserEntity.userEntity

    override fun existOrder(userId: Int?): Boolean {
        return jpaQueryFactory
            .selectOne()
            .from(order)
            .innerJoin(order.user, user)
            .where(getUserIdPredicate(userId))
            .fetchFirst() != null
    }

    // 정책적으로 사실 필수값이겠지만 예시를 위해 작성
    private fun getUserIdPredicate(userId: Int?): BooleanExpression? {
        return userId?.let {
            order.user.id.eq(it)
        }
    }

    //조회 컬럼을 최소화 하자
    fun getOrders(userId: Int): List<Order> {
        return jpaQueryFactory.select(
            Projections.fields(
                Order::class.java,
                order.userId, //TODO : 이미 알고 있는 값은 아래와 같이 치환 가능
                Expressions.asNumber(userId).as("userId"),
            )
        ).from(order)
            .where(order.userId.eq(userId))
            .limit(10)
            .orderBy(order.creatAt.desc())
            .fetch()
    }
}

//OrderByNull 직접구현 
class OrderByNull private constructor() : OrderSpecifier<Void>(Order.ASC, NullExpression.defaultExpression<Void>(), Default) {
    companion object {
        val DEFAULT = OrderByNull()
    }
}
```
