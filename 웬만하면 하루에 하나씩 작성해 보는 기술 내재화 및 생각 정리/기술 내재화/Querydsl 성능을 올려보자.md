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
>  
```kotlin
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
    fun existOrder(userId: Int): Boolean
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
```
