# 스프링 프레임워크를 중심으로 작성해보는 Design Pattern
> Factory Method(생성패턴)
> 팩토리 메서드 패턴은 객체 생성의 책임을 서브클래스로 위임하여 객체 생성의 변화를 통해 코드를 유연하게 만드는 생성 패턴 중 하나이다.
> <br>이 패턴은 인스턴스 생성 로직을 캡슐화하여 클라이언트 코드가 특정 클래스에 의존하지 않도록 한다.
>
>
>

```kotlin
// 생성될 객체의 인터페이스 또는 추상 클래스.
interface Product {
    fun use(): String
}

/**
 * ConcreteA/B 
 * Product 인터페이스를 구현하는 구체적인 클래스.
 */
class ConcreteProductA: Product {
    override fun use(): String {
        return "ConcreteA 반환"
    }
}

class ConcreteProductB: Product {
    override fun use(): String {
        return "ConcreteB 반환"
    }
}

// 팩토리 메서드를 선언하는 클래스. 이 클래스는 팩토리 메서드를 호출하여 객체를 생성
abstract class Creator {
    abstract fun factoryMethod(): Product
    
    fun operation(): String {
        val product = factoryMethod()
        return "Creator"
    }
}

/**
 * ConcreteCreatorA/B
 * Creator 클래스의 팩토리 메서드를 구현하는 서브클래스. 여기서 실제 객체가 생성
 */
class ConcreteCreatorA: Creator {
    override fun factoryMethod(): Product {
        return ConcreteCreatorA
    }
}

class ConcreteCreatorB: Creator {
    override fun factoryMethod(): Product {
        return ConcreteCreatorB
    }
}

fun main() {
    val creatorA: Creator = ConcreteCreatorA
    println(creatorA.operation())
    
    val creatorB: Creator = ConcreteCreatorB
    println(creatorB.operation())
}
```
<img width="616" alt="스크린샷 2024-07-30 22 24 04" src="https://github.com/user-attachments/assets/5d27a95b-730f-42f2-ae1f-31e0a52167d0">
