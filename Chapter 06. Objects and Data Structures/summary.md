# 6장 객체와 자료 구조

## 자료 추상화
### 추상화를 사용하여 구현을 숨기자!
```
class Point(val x : Double, val y : Double)
      V
interface Point {
  fun getX()
  fun getY()
  fun setCartasian(x: Double, y: Double)
}
```

### 메소드명도 추상적으로!
```
interface Vehicle {
  fun getFuelTankCapacityInGallons()
  fun getGallonOfGasoline()
}
    V
interface Vehicle() {
  fun getPercentFuelRemaining()
}
```

#### 의문점? 그렇다면 어디까지 추상적인 것이 좋을까 
```
fun getPercentFuelRemaining() -> getRemaining() 으로 무엇의 잔여량인지 숨기면서 행동만 정의하는 추상화는 좋은 추상화일까?
```

## 자료/객체 비대칭
### 절차적 프로그래밍
```
class Square(val topLeft : Point, val side : Double)
class Rectangle(val topLeft : Point, val height : Double, val width : Double)
class Circle(val center : Point, val radius : Double)

class Geometry(val PI : Double = 3.141592) {
    fun area(shape : Any) : Double {
        if(shape is Square) {
            val s : Square = shape
            return s.side * s.side
        }
        if(shape is Rectangle) {
            val r : Rectangle = shape
            return r.height * r.width;
        }
        if(shape is Circle) {
            val c : Circle = shape
            return c.radius * c.radius * PI
        }
    }
}
```

### 객체형 프로그래밍
```
class Square(val topLeft : Point, val side : Double) : Shape {
    fun area() : Double {
        return side * side
    }
}
class Rectangle(val topLeft : Point, val height : Double, val width : Double) : Shape {
    fun area() : Double {
        return height * width
    }
}
class Circle(val center : Point, val radius : Double, val PI : Double = 3.141592) : Shape {
    fun area() : Double {
        return radius * radius * PI
    }
}
```

#### 정답은 없다. 때에 따라 적절하게 활용하자
+ 절차적 코드는 새로운 자료구조를 추가하기 어렵다. -> 모든 함수를 고쳐야 한다.
+ 객체 지향 코드는 새로운 함수를 추가하기 어렵다. -> 모든 클래스를 고쳐야 한다.

## 디미터 법칙
### 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다.
```
var outputDir : String = ctxt.getOptions().getSearchDir().getAbsolutePath()
자료구조형
_________
var opts : Options = ctxt.getOptions()
var scratchDir : File = opts.getScratchDir()
val ouputDir : String = scratchDir.getAbsolutePath()
객체형
```
=> 자료구조라면 드러내고, 객체라면 감춘다.

### if) 만약 절반은 객체, 절반은 자료구조라면? 잡종 구조
+ 이러한 잡종 구조는 되도록 피하자!
+ 해결 방법? 구조체 감추기!
```
var bos : BufferedOutputStream = ctxt.createScratchFileSystem(classFileName)  
```
## 자료 전달 객체 DTO
````
class Address(var street : String, var streetExtra : String, var city : String, var state : String, var zip : String) {
    fun getStreet() : String {
        return street
    }
    fun getStreetExtra() : String {
        return streetExtra
    }
}
...
```
+ 활성 레코드란? 대개 save나 find와 같은 탐색 함수도 제공한다. db테이블이나 다른 소스에서 자료를 직접 변환한다.
  + 활성 레코드는 자료구조? 객체? => 자료구조

## 결론
+ 객체는 동작을 공개하고 자료를 숨긴다.
+ 동작 변경과 추가는 어렵지만 새 객체를 생성하거나 변경은 쉽다.
+ 자료구조는 반대
+ 새로운 자료 타입을 추가하는 유연성 => 객체
+ 새로운 동작을 추가하는 우연성 => 자료구조
