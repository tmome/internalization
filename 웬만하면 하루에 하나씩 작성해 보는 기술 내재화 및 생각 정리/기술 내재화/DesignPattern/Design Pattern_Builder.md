# 스프링 프레임워크를 중심으로 작성해보는 Design Pattern
> Builder Pattern(생성패턴)
> 
> 빌더 패턴(Builder Pattern)은 객체 생성 패턴 중 하나로, 복잡한 객체를 단계별로 생성할 수 있도록 도와준다.
> <br> 빌더 패턴을 사용하면 객체 생성 과정이 보다 명확하고 유연해집니다. 
> <br> 특히, 객체의 생성자가 많거나 객체 생성 시 다양한 설정이 필요한 경우에 유용합니다.

```kotlin
class Person private constructor(
    val firstName: String?,
    val lastName: String?,
    val age: Int?
) {
    data class Builder(
        var firstName: String? = null,
        var lastName: String? = null,
        var age: Int? = null
    ) {
        fun firstName(firstName: String) = apply { this.firstName = firstName }
        fun lastName(lastName: String) = apply { this.lastName = lastName }
        fun age(age: Int) = apply { this.age = age }
        fun build() = Person(firstName, lastName, age)
    }
}

fun main() {
    val person = Person.Builder()
        .firstName("Cheon")
        .lastName("hyun seung")
        .age(30)
        .build()

    println("Person: ${person.firstName} ${person.lastName}, Age: ${person.age}")
}
```

> 1. Person 클래스: 이 클래스는 생성자가 private으로 되어 있어 직접 객체를 생성할 수 없다.
> 2. Builder 클래스: 내부 클래스이며, Person 객체를 생성하는 데 사용됩니다. 초기값은 모두 null로 설정되어 있다.
> 3. 빌더 메서드: 각 속성에 대해 설정 메서드가 존재하며, apply를 사용해 빌더 객체 자신을 반환하고 이를 통해 메서드 체이닝이 가능
> 4. build() 메서드: 최종적으로 build() 메서드를 호출하면 설정된 값들을 이용해 Person 객체를 생성
> 
![builder-diagram](https://github.com/user-attachments/assets/483d8838-33aa-48dd-910c-b91ab3316846)