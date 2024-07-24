# 스프링 프레임워크를 중심으로 작성해보는 Design Pattern
> Proxy Pattern (구조패턴)
> - Proxy 는 대리자를 의미를 가지고 있는데 말그대로 누군가가 그역할을 수행하는 존재이며 Proxy에게 일을 대신 시키는 것이다
> 어떤 객체를 사용하고자 할때 직접적으로 객체를 참조하는것이 아니라 해당 객체를 대신할 수 있는 대상 객체에 접근하여 
> 해당 객체가 메모리 상에 존재하지 않더라도 참조할 수 있고 가장 중요한 것은 실제 객체가 필요한 시점까지 생성을 미룰 수 있다.
> 
> 
> 
> 
> 
> <img width="763" alt="스크린샷 2024-07-23 23 29 13" src="https://github.com/user-attachments/assets/65b1ccb0-e50a-4eb1-ba10-27f259bb85ea">

```kotlin
interface Image {
    fun display()
}

class RealImage(private val fileName: String) : Iamge {
    init {
        loadFromDisk(fileName)
    }

    private fun loadFromDisk(fileName: String) {
        println("Loading $fileName")
    }

    override fun display() {
        println("display $filename")
    }

    /**
     * Proxy 호출
     * 참고 
     * lateinit : 초기화 이후에 계속하여 값이 바뀔 수 있을 때
     * by lazy : 초기화 이후에 읽기 전용 값으로 사용할 때
     */
    class ProxyImage(private val fileName: String) : Image {
        private var realImage: RealImage by lazy { realImage(fileName)}

        override fun display() {
            realImage.display()
        }
    }

    /**
     * ===Console 예측 값===
     * Loading image1.jpg
     * Displaying image1.jpg

     * Displaying image1.jpg

     * Loading image2.jpg
     * Displaying image2.jpg

     * Displaying image2.jpg
     */
    fun main() {
        val image1: Image = ProxyImage("image1.jpg")
        val image2: Image = ProxyImage("image2.jpg")

        image1.display()
        println()

        image1.display()
        println()

        image2.display()
        println()

        image2.display()
    }    
}
```