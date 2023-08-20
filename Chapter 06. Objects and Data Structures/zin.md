[객체지향적 자료와 구조를 설계하자](https://zins.tistory.com/7)

## **Summary**

#### **객체지향적 자료와 구조를 설계하자.**
**Design Object-Oriented Data and Structure.**

추상화된 인터페이스는 가능성을 열어두고, 이를 통해 구현된 class는 핵심 내용을 구체적으로 표현할 수 있어야 한다.  
An abstracted interface defines possibilities, allowing classes that implement it to represent core content concretely.


또한 핵심 로직에서 책임과 역할을 공개할 필요가 없다. 동작만 공개하고 특정 객체의 책임과 역할이라면 method를 통해 구현하고, 핵심 로직은 위임만 하면 된다.  
Additionally, there's no need to expose responsibilities and roles within the core logic. If pertaining to the responsibilities and roles of the object, these can be implemented through methods, while the core logic can be delegated.

---

## **Information Abstraction**

추상 인터페이스를 사용해 class의 구체적인 구현 내용과 관계 없이, 핵심 내용을 정의 및 사용할 수 있어야 한다.  
_Using an abstract interface, the core content should be defined and accessible within the class, regardless of the specific implementation details._

#### **Abstract & Concrete Class**

```
class Rectangle(
    val width: Int, 
    val height: Int,
) {
	fun area(): Int {
    	return width * height
    }
}
```

-   가로x세로가 넓이를 의미하는 사각형이라는 class임을 구체적으로 알 수 있다.  
    _It can be specifically understood that a rectangle represents a class in which the product of the width and height represents the area._

#### **Abstract & Concrete Class**

```kotlin
abstract class Shape(
	val width: Int, 
    val height: Int,
) {
    abstract fun area(): Int
}

class Rectangle(
	width: Int, 
    height: Int,
) : Shape(width, height) {
    override fun area(): Int {
    	return width * height
    }
}
```

```kotlin
interface Shape {
    val width: Int,
    val height: Int,
}

data class Rectangle (
	override val width: Int,
    override val height: Int,
) : Shape {
	val area get() = width * height
}
```

-   반면 Shape를 abstract 혹은 interface로 area의 구성 가능성을 열어둔다.  
    _On the other hand, defining Shape as an abstract class or interface opens up the possibility of configuring the calculation of area._
-   area 계산 세부 로직을 알 수 없다. 이 Class를 통해 구현된 domain이 이를 정의할 수 있을 것이다.  
    _The specific details of how the area calculation is performed are not revealed._

---

## **Hiding Structures**

객체는 동작을 공개하고 책임과 역할을 갖고 있는 class 내부 method에게 구현을 위임하여 구조체를 감춘다.  
Objects expose behaviors and delegate responsibilities and roles to classes with responsibilities, effectively hiding the underlying structure.

#### **Open Structure**

```kotlin
ctxt.getAbsoultePathOfScratchDirectory().getAbsolutePath()
```

-   내부 method를 공개하는 라인임에도 임시 파일의 절대경로를 얻으려는 목적이 분명하지 않다.  
    Even though it's a line exposing an internal method, the intention to obtain the absolute path of a temporary file is not evident.

#### **Hide and Detail Structure**

```kotlin
val bos: BufferedOutputStream = ctxt.createScratchFileStream(classFileName)
```

-   결론적으로 임시 파일을 생성하기 위함이었다면, 말그대로 임시 파일을 생성하라고 지시하면 된다.  
    Ultimately, if the purpose was to create a temporary file, you could simply instruct to create the temporary file explicitly.
-   모듈은 자신이 조작하는 객체의 내부 사정을 몰라야 한다는 디미터 법칙을 지키면서, 적절한 책임과 역할을 위임할 수 있게 된다.  
    By adhering to the Law of Demeter, a module should not have knowledge of the inner workings of the objects it manipulates. This allows for appropriate responsibilities and roles to be delegated effectively.




> _source:_ <br>
> _Chapter 6. Objects and Data Structures_ <br>
> _Clean Code, Robert C. Martin_
