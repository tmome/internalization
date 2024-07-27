# Spring ConditionalOnProperty 및 ConditionalOnBean
> * @ConditionalOnProperty 
>   * 특정한 프로퍼티의 값에 따라 Bean 을 등록할지 말지 결정
>     * 사용 예시 
>       * @ConditionalOnProperty는 application.properties 나 application.yml에 설정 프로퍼티를 기반으로 Bean의 생성 여부를 결정할때 사용
>     * 속성
>       * prefix : 프로터티 키의 접두사
>       * name : 프로퍼티 키의 이름
>       * havingValue : 프로퍼티가 특정 값을 가질 때 빈을 등록
>       * matchMissing : 프로퍼티가 없거나 값이 없을 때 Bean 을 등록할지 말지 결정 (Default : false)
> ```kotlin
> import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty
> import org.springframework.context.annotation.Bean
> import org.springframework.context.annotation.Configuration
> import javax.sql.DataSource
> 
> @Configuration
> class DataSourceConfig {
> 
>     @Bean
>     @ConditionalOnProperty(prefix = "spring.datasource", name = ["enabled"], havingValue = "true", matchIfMissing = false)
>     fun dataSource(): DataSource {
>         return HikariDataSource()
>     }
> }
> ```
> @ConditionalOnBean 
> - 기능 
>   - 특정 빈이 이미 생성되어 있는 경우에만 해당 빈을 생성합니다. 
> - 옵션
>   - value 또는 type: (선택 사항) 존재해야 할 빈의 타입. 
>   - name: (선택 사항) 존재해야 할 빈의 이름들.
> ```kotlin
> @Configuration
> class ProxyDataSourceConfiguration {
> 
>     @Bean
>     @ConditionalOnBean(name = ["routingDataSource"])
>     fun lazyConnectionDataSourceProxy(routingDataSource: DataSource): LazyConnectionDataSourceProxy {
>         return LazyConnectionDataSourceProxy(routingDataSource)
>     }
> }
> ```
