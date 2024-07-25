# 스프링 프레임워크를 중심으로 작성해보는 Design Pattern
> Decorator Pattern(구조패턴)
> 
> 메소드 호출의 반환값에 변화를 주기 위해 중간에 장식자를 두는 패턴
> 
> **기존 클래스는 유지하되, 이후 필요한 형태를 꾸밀때 사용**
> - implements, extends 를 통해 확장하는 형태로 사용
> 
> 
>
> <img width="566" alt="스크린샷 2024-07-26 00 17 58" src="https://github.com/user-attachments/assets/24405791-b29e-40f8-a038-19b624f1796b">

```kotlin
interface Coffee {
    fun cost(): Double
    fun description(): String
}

class BasicCoffee: Coffee {
    override fun cost() {
        return 5.0
    }
    
    override fun description(): String {
        return "BasicCoffee"
    }
}

abstract class CoffeeDecorator(private val coffee: Coffee) : Coffee {
    override fun cost(): Double = coffee.cost()
    override fun description(): String = coffee.description()
}

class MilkDecorator(coffee: Coffee) : CoffeeDecorator(coffee) {
    override fun cost(): Double = super.cost() + 1.5
    override fun description(): String = super.description() + ", Milk"
}

class SugarDecorator(coffee: Coffee) : CoffeeDecorator(coffee) {
    override fun cost(): Double = super.cost() + 0.5
    override fun description(): String = super.description() + ", Sugar"
}

fun main() {
    var myCoffee: Coffee = SimpleCoffee()
    println("${myCoffee.description()} -> $${myCoffee.cost()}")

    myCoffee = MilkDecorator(myCoffee)
    println("${myCoffee.description()} -> $${myCoffee.cost()}")

    myCoffee = SugarDecorator(myCoffee)
    println("${myCoffee.description()} -> $${myCoffee.cost()}")
}
/*
 * Simple coffee -> $5.0
 * Simple coffee, Milk -> $6.5
 * Simple coffee, Milk, Sugar -> $7.0
 */
```

