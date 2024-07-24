# 스프링 프레임워크를 중심으로 작성해보는 Design Pattern
> Adapter Pattern (구조패턴)
> - 호출당하는 쪽의 메소드를 호출하는 쪽의 코드에 대응하도록 중간에 변환기를 통해 호출하는 패턴
> 
>   호환성이 없는 기존 클래스의 인터페이스를 변환해서 재사용하기 위해 필요.
> ex : JDBC, JRE
> - 서로 다른 두 인터페이스 사이에 통신이 가능하게 하는것
>

```kotlin
interface Animal {
    fun walk()
}

class Dog : Animal {

    override fun walk() {
        println("강아지가 걷는다")
    }
}

class Cat : Animal {

    override fun walk() {
        println("고양이가 걷는다")
    }
}

class Fish {
    fun swim() {
        println("물고기가 수영한다")
    }
}

class FishAdapter(private val fish: Fish) : Animal {
    override fun walk() {
        fish.swim()
    }
}

fun main() {
    val dog = Dog()
    dog.walk()

    val cat = Cat()
    cat.walk()

    val fish = Fish()
    fish.swim()
    
    val fishAdapter = FishAdapter(fish)
    fishAdapter.walk()
}
```

<img width="746" alt="스크린샷 2024-07-25 01 14 58" src="https://github.com/user-attachments/assets/c4b88665-a968-4b35-8f23-4fd33a8879da">
