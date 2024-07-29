# 스프링 프레임워크를 중심으로 작성해보는 Design Pattern
> Facade Pattern(구조패턴)
> 
> 퍼사드 패턴은 구조패턴의 한 종류로써 복잡한 서브 클래스의 공통 기능을 정의하는 상위 인터페이를 제공하는 패턴이다.
> 
> - 퍼사드 객체(Facade Object)는 서브 클래스의 코드에 의존하는 일을 감소시켜 주고, 복잡한 소프트웨어를 간단히 사용 할 수 있게 간단한 인터페이스를 제공해준다.
> - 퍼사드 패턴을 통해 서브 시스템(SubSystem)들 간의 종속성을 줄여줄 수 있으며, 퍼사드 객체를 사용하는 곳(Client)에서는 여러 서브 클래스들을 호출할 필요 없이 편리하게 사용할 수 있다.

```kotlin
class Engine {
    fun start() {
        println("시동을 킨다")
    }

    fun stop() {
        println("시동을 끈다")
    }
}

class Lights {
    fun on() {
        println("라이트를 킨다")
    }

    fun off() {
        println("라이트를 끈다")
    }
}

class AudioSystem {
    fun turnOn() {
        println("오디오를 킨다")
    }

    fun turnOff() {
        println("오디오를 끈다")
    }

    fun setVolume(volumeLevel: Int) {
        println("볼륨을 킨다 $volumeLevel")
    }
}

class CarFacade(
    private val engine: Engine,
    private val lights: Lights,
    private val audioSystem: AudioSystem
) {
    fun startCar() {
        engine.start()
        lights.on()
        audioSystem.turnOn()
        audioSystem.setVolume(5)
        println("==startCar==")
    }

    fun stopCar() {
        engine.stop()
        lights.off()
        audioSystem.turnOff()
        println("==stopCar==")
    }
}

fun main() {
    val engine = Engine()
    val lights = Lights()
    val audioSystem = AudioSystem()

    val car = CarFacade(engine, lights, audioSystem)
    car.startCar()
    car.stopCar()
}
```

<img width="873" alt="facade" src="https://github.com/user-attachments/assets/1bec1198-d610-4844-81bf-8f36ae0e223f">
