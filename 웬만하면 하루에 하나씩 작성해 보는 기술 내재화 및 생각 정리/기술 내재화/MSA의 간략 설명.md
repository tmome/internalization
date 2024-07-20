# Microservice Architecture
> Martin Folwer.
> 
> "the microservice architectural style is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API. These services are built around business capabilities and independently deployable by fully automated deployment machinery."
> 
>> 마틴 파울러가 말한 마이크로서비스 아키텍처 스타일의 핵심은 하나의 애플리케이션을 소규모 서비스 제품군으로 개발하는 접근 방식으로 
>  독립적으로 배포할 수 있다는 것이라고 생각합니다.
> 
> - MSA의 장점
>   - 배포 관점
>     - 독립적 배포가 가능하다.
>   - 확장 관점
>     - 특정 서비스에 대한 확정이 상대적으로 용이함.
>   - 장애 관점
>     - 장애가 전체로 propagation 될 가능성이 줄어듬.
> - MSA 단점
>   - 설계 난이도가 올라가고 통신 latency 가 발생할 수 있음
>   - 서비스의 분리로 인해 트랜잭션 관리의 난이도가 올라가고, 많은 자원이 필요할 수 있음.
>   - 분산된 데이터로 인해 정합성 관리의 어려움이 있을 수 있음.
> - 해결 방안
>   - Rest 방식이 아닌 pub-sub 구조
>   - CQRS 패턴
> 
>
> 
>  **추후 자세히 정리해 볼 예정**
> 
> 
>![martinfowler_msa](https://github.com/user-attachments/assets/fa53fa23-4e32-4891-98a9-05058cafa95e)
> 