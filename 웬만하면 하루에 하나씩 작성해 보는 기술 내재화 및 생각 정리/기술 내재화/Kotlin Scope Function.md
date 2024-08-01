# Kotlin Scope Function
> 개요
> - 코틀린은 기본적으로 ‘표준 스코프 함수 (Scope Function)’ 라는 것을 제공한다.
> <br> 스코프 함수는 특정 객체의 컨텍스트 내에서 특정 동작 (프로퍼티 초기화, 활용 등) 을
> 실행하기 위한 목적만을 가진 함수다. <br>
> 스코프 함수를 람다 함수로 사용하게 되면 임시로 스코프를 형성하는데,
> 이 스코프 내에서는 객체의 이름을 통해 일일히
> 참조할 필요 없이 객체를 접근하고 핸들링할 수 있다는 편리한 장점이 있다.
> 
> 
> - Scope function 에는 let, run, with, apply, also 가 있다.
>
# Kotlin Scope Functions
| Function | Object Reference | Return Value  | Is Extension Function | Usage                                         |
|----------|------------------|---------------|-----------------------|-----------------------------------------------|
| `let`    | `it`             | Lambda result | Yes                   | Nullable 객체의 작업을 수행하고 결과를 반환     |
| `run`    | `this`           | Lambda result | Yes                   | 객체의 작업을 수행하고 결과를 반환             |
| `with`   | `this`           | Lambda result | No                    | 객체의 작업을 수행하고 결과를 반환             |
| `apply`  | `this`           | Context object| Yes                   | 객체의 설정을 하고 객체 자체를 반환            |
| `also`   | `it`             | Context object| Yes                   | 객체의 작업을 수행하고 객체 자체를 반환        |






