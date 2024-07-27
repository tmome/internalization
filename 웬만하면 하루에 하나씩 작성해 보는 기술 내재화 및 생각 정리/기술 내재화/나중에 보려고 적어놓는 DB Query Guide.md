# 나중에 보려고 적어놓는 DB Query Guide
## 쿼리

기본적으로 `@Query` 애너테이션이나 QueryDSL, 메서드 쿼리를 사용하여 작성하도록 한다. 네이티브 쿼리는 사용하지 않도록 한다.

단, 배치 모듈에서는 복잡한 쿼리가 요구되는 경우가 생기므로 네이티브 쿼리 사용을 허용한다.

## 쿼리 코멘트

QueryDSL을 사용하여 작성하는 경우 `JPAQueryCommentAspect`에 의해 쿼리 코멘트가 자동 적용된다.

쿼리 메서드를 사용하는 경우에는 다음과 같이 쿼리 코멘트를 넣도록 한다.

```kotlin
@QueryHints(
    QueryHint(name = AvailableHints.HINT_COMMENT, value = "SampleRepository::findById")
)
fun findById(id: Int): Long
```

## 롱(long) 트랜잭션 유발 지양

트랜잭션 구간내에서 시간이 오래 걸리는 작업을 하지 않도록 한다. 시간이 오래 걸리지만 트랜잭션과 상관이 없다면 트랜잭션 바깥으로 이동하는 것이 좋다.
트랜잭션과 상관이 있다면, 오래 걸리는 작업의 타임아웃을 적절하게 튜닝하는 것이 좋다. 만약 오래걸리는 작업을 방치하면 대기로 인해 커넥션 풀을 모두
소모하게 될 가능성이 생긴다.

## 페이징

offset limit 방식의 페이징은 전체 조회 결과가 많은 경우 페이지 뒤로 갈수록 속도 저하가 심해진다. 이런 경우에는 페이징 방식보다는 next 값을 참조하는
형태의 페이징 방식을 사용하는 것이 좋다.

## DB 데드락

DB 데드락을 유발할 가능성이 없는지 확인하도록 한다. 예를 들어 동시에 요청된 트랜잭션에서 여러 항목을 insert 나 update 시
서로 다른 트랜잭션에서 항목의 처리 순서가 다를 경우 데드락 발생 가능성이 높아진다. 이런 경우에는 다른 트랜잭션에서도 동일한 순서를 가질 수 있도록
순서 보장을 하는 것이 좋다.

## N+1 조회 지양

N+1 조회가 일어나지 않도록 한다. 라운드트립 비용은 비싸기 때문에 되도록 한번에 조회할 수 있는 방법을 선택하도록 한다.
쿼리의 경우 `IN` 절을 활용하거나, API의 경우 벌크형태의 조회 API가 필요할 수 있다.

## JPA 쿼리 메소드

쿼리 메소드는 네이밍이 필요 이상으로 길어져 가독성을 해치기 쉽다. 그렇게 길게 작성되어지는 경우 쿼리가 조회하는 정보가 무엇인지를 네이밍에 반영하도록 하자.
예를들면 다음과 같다.

```kotlin
// before
return petItemWeighingBreedStatRepository.findFirstByPetItemWeighingBreedStatPk_BreedCodeAndPetItemWeighingBreedStatPk_AgeGroupAndPetItemWeighingBreedStatPk_AgeWeekAndPetItemWeighingBreedStatPk_StatDateBetweenOrderByPetItemWeighingBreedStatPk_StatDateDesc(
            breedCode,
            petAgeGroup,
            ageWeek,
            startDate,
            endDate
        )

// after
return petItemWeighingBreedStatRepository.findLatestWeightStatByBreedAndAge(...)
```

## 트랜잭션

DB 커넥션은 writer와 reader로 분리되어 있다. 서비스 레이어 로직 내부에서 변경(CUD)이 일어난다면 writer 커넥션을 사용해야 하고, 조회(R)만 일어난다면
reader 커넥션을 사용해야 한다. 두 가지 커넥션을 혼합해서 사용하지 않도록 한다.

이러한 커넥션 제어를 트랜잭션 애너테이션을 통해 할 수 있도록 구성되어 있다. 트랜잭션 애너테이션은 서비스 레이어 수준에만 적용하도록 한다.
다음과 같이 writer 커넥션과 reader 커넥션을 지정할 수 있다.

```kotlin
// writer
@Transactional

// reader
@Transactional(readOnly = true)
```

간혹 트랜잭션 애너테이션을 명시하지 않은 서비스 레이어 내부 메소드 A에서 같은 서비스 레이어의 트랜잭션 애너테이션이 달린 B 메소드를 `this` 참조로 호출하는
경우를 보는데, 이 경우 트랜잭션이 의도하는 대로 동작하지 않을 수 있다. 트랜잭션 애너테이션은 프록시 객체 호출에 대해서만 동작하므로 `this` 참조 호출을 통해
호출하는 경우 트랜잭션이 동작하지 않을 수 있다. 이 경우에는 프록시 객체(스프링 빈 인스턴스)를 통해 호출하는 방식으로 만들어 주도록 한다.

## 인덱스

퀴리는 `explain`을 통해 `filesort`가 일어나지 않는지 확인하도록 한다. `filesort`의 의미는 인덱스를 타지 않는다는 의미이다.

인덱스를 생성해야 하는 고려 대상이 되는 경우는 다음과 같다.

- where 조건절에 포함된 조건
- order by 절에 포함된 조건
- group by 절에 포함된 조건

이와함께 다음 조건 역시 만족해야 한다.

- 선택도가 좋아야 한다(데이터가 유니크에 가까울수록 좋다)
- 데이터가 충분히 많아야 한다. 조건에 따라 다르지만 일반적으로 100건 미만은 풀스캔이 더 빠르다.

주의할 부분은 다음과 같다.

- `order by` 나 `group by`의 경우 컬럼 출현 순서 그대로 복합 인덱스를 생성해야 한다. 개별로 생성하면 안된다.

인덱스 네이밍 규칙은 다음과 같다.

```sql
    // idx_{테이블 명}_{세 자리 순번}
    KEY `idx_sample_001` (`started_at`),
    KEY `idx_sample_002` (`ended_at`),
    KEY `idx_sample_003` (`created_at`),
    KEY `idx_sample_004` (`updated_at`),
    UNIQUE KEY `uidx_sample_005` (`user_id`) -- 유니크 인덱스
```

* Online DDL 참고
    * https://dev.mysql.com/doc/refman/5.7/en/innodb-online-ddl-operations.html#online-ddl-index-operations

## 네이티브 쿼리 스타일

예외적으로 배치 애플리케이션의 경우 네이티브 쿼리 사용을 허용한다.

들여쓰기는 다음과 같은 형태로 구역을 나눠 하도록 한다.

```sql
```sql
SELECT
    ...
FROM
    ...
WHERE
    ...
ORDER BY
    ...
GROUP BY
    ...
HAVING
    ...
```

MySQL 키워드, 함수는 대문자로 표현하도록 한다.

```sql
SELECT
    MAX(table1.col3)
FROM
    table1
    INNER JOIN table2
        ON table1.col1 = table2.col2
WHERE
    table2.condition1 = 'condition1'
    AND table2.condition2 IN ('condition2', 'condition3')
GROUP BY
    table1.col3
```

네이티브 쿼리에도 쿼리 호출 지점을 파악할 수 있도록 쿼리 코멘트를 달아주도록 한다.

```sql
-- ${this::class.java.simpleName}::${메소드 이름}
SELECT ...
```

## 벌크 인서트

벌크 인서트는 `querydsl-sql` 을 사용한다. `QueryDslConfig` 에 정의된 sqlQueryFactory 빈을 주입받아 사용하도록 한다.
사용 예는 다음과 같다.

```kotlin
class SampleQueryDslRepositoryImpl(
    private val jpaQueryFactory: JPAQueryFactory,
    private val sqlQueryFactory: SQLQueryFactory,
    private val objectMapper: ObjectMapper,
) : SampleQueryDslRepository {

    ...

    override fun bulkInsert(entities: List<SampleEntity>): List<Int> {
        if (entities.isEmpty()) return listOf()

        return sqlQueryFactory.insert(
            QSampleRelation.sample,
        ).also { insert ->
            entities.forEach { item ->
                insert.populate(
                    SampleRelationDto().apply {
                        createdAt = Timestamp.from(item.createdAt)
                        errorCnt = item.errorCnt
                        errorMessage = item.errorMessage
                        groupingKey = item.groupingKey
                        id = item.id
                        mobileNumber = item.mobileNumber
                        parameter = objectMapper.writeValueAsString(item.parameter)
                        queuedAt = Timestamp.from(item.queuedAt)
                        result = item.result?.let { objectMapper.writeValueAsString(item.result) }
                        status = item.status?.name
                        tryAt = item.tryAt?.let { Timestamp.from(item.tryAt) }
                        tryCnt = item.tryCnt
                        type = item.type?.name
                        updatedAt = item.updatedAt?.let { Timestamp.from(item.updatedAt) }
                        userId = item.user?.id
                        vendor = item.vendor?.name
                    },
                ).addBatch()
            }
        }.executeWithKeys().use { rs ->
            val keys = mutableListOf<Int>()
            while (rs.next()) {
                keys.add(rs.getInt(1))
            }
            if (keys.size != entities.size) {
                throw IllegalStateException("Key size does not match with input size.")
            }
            keys
        }
    }
}
```

`querydsl-sql` 을 이용한 벌크 인서트는 미리 Q클래스와 Dto클래스를 제너레이팅 해줘야 한다. 제네레이팅에 대한 설정은
`commons-domain` 모듈에 `build.gradle.kts` 에 정의되어 있다. `tableNamePattern` 에 제너레이팅을 원하는 테이블명을
기입한다음 `queryDslMetadataExport` 태스크를 실행해 주면 `src/main/java/com.sample.domain.relation` 에 제너레이팅 된다.
단, MySQL 컨테이너를 실행중인 상태여야 한다.

보면 알겠지만, Jpa 엔티티를 직접적으로 사용하지 못하기 때문에 **Jpa 엔티티에 애너테이션이나 리스너를 붙여 자동 동작하도록 되어 있는 부분이 있다면**
해당 부분이 동작할 수 있도록 벌크 인서트 로직에서 추가적으로 신경써줘야 한다.

그외 `JdbcTemplate` 이나 `EntityQl` 도 대안이 될 수 있겠지만 `JdbcTemplate` 는 생쿼리를 직접 만들어야 한다는 점, `EntityQl` 은
신규 `jakarta.persistence` 인터페이스를 지원하지 않아(2020년 이후로 업데이트 되지 않는 듯) 사용하지 못한다.

## 쿼리 매개변수 바인딩

네이티브 쿼리를 사용하는 경우 다음과 같이 문자열 붙이기로 매개변수 값을 바인딩하지 않도록 한다.
매개변수 바인딩은 항상 라이브러리가 제공하는 매개변수 바인딩 기능을 사용하도록 한다.

```kotlin
// BAD - SQL 인젝션에 취약. 설사 안전함이 보장되더라도 기본적으로 이런 습관을 들이는 것은 좋지 않다.
val query = """SELECT * FROM some_table WHERE id=$id"""
```

## 엔티티 함수

특정 엔티티에 국한되는 어떤 함수를 만드는 경우 엔티티 바깥에 함수를 정의하기 보다 해당 엔티티 내부에 함수 정의를 추가하도록 한다.

```kotlin
// 적용 전
@Service
class ProductService(
    private val productRepository: ProductRepository,
) {
    fun calcPrice(id: Int): BigDecimal {
        ...
        return ProductEntity.price * ProductEntity.discountRate

    }
}

// 적용 후 
class ProductEntity {
    ...
} {
    fun calcPrice(): BigDecimal {
        return price * discountRate
    }
}
```
