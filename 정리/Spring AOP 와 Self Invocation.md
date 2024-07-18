# 웬만하면 하루에 하나씩 작성해 보는 기술 내재화 및 생각 정리

> Spring AOP 와 Self Invocation
> Self Invocation 문제
> - AOP Prxoy가 동일 클래스 내에서 호출되는 메서드를 Proxy 처리하지 않는 경우에 발생
> ```
> @Service
> class userService {
> 
>     @Autowired
>     private lateinit var userRepository: UserRepository
> 
>     @Autowired
>     private lateinit var em: EntityManager
> 
>     fun findByUserName() {
>         getUserName()
>     }
> 
>     @Transactional
>     fun getUserName() {
>         val user = User()
>         user.name = "hyunSeung"
> 
>         val saveUser = userRepository.save(user)
>         println(em.contains(saveUser))
>     }
> }
>
> @SpringBootTest
> class UserServiceTests {
> 
>     @Autowired
>     private lateinit var userService: UserService
> 
>     @Test
>     @DisplayName("self invocation 발생")
>     @Throws(Exception::class)
>     fun test() {
>         println("== start ==")
>         UserService.findByUserName()
>         println("== end ==")
>     }
> }
> ```
>   - 해당 로직에서 findByUserName()에서 getUserName()을 self Invocation 하였기 때문에
>     getUserName()이 트랜잭션이 적용된 상태이기에 영속성 컨텍스트에 존재하고 1차 캐시에 보관되어야 할것이라고 기대했지만
>     실제론 em.contains를 확인해보면 false가 나옵니다.
>   - 원인
>     - 스프링 AOP의 동작방식이 Spring AOP는 JDK Dynamic Proxy 또는 CGLIB Proxy를 사용하는데
>       실제로 호출되는 객체는 Proxy이기 떄문에 내부에서 다른 메서드를 직접 호출 시 Proxy를 거치지 않고
>       직접 메서드를 호출하는것과 다르지 않기 때문에 기대하는 결과가 나오지 않는다.
>       추가적으로 메서드가 public인 이유도 private 같은 경우 AOP가 가로채지 못하고 advice를 적용할 수 없기 떄문이다.
  
