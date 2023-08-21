# 객체와 자료구조

## 자료 추상화

### 예시 1
```kotlin
class BasicPoint(var x:Double, var y:Double)

class AbstractPoint(private var x:Double, private var y:Double){
    fun getX():Double = x
    fun getY():Double = y
    fun setCartesian(x:Double, y:Double){
        this.x = x
        this.y = y
    }
    fun setPolar(rho:Double, theta:Double){
        x = rho * Math.cos(theta)
        y = rho * Math.sin(theta)
    }
}
```

- **`BasicPoint`** 의 경우 x와 y가 각각 외부에서 접근, 설정이 가능하기때문에 외부로 노출되어있다.
  - 이는 변수에 의존하게될 수 있다.
  - 직교좌표계를 사용하는지, 극좌표계를 사용하는지 알 수 없다.
- **`AbstractPoint`** 의 경우 x와 y는 private 로 되어있어 곧바로 접근할 수 없고 getX, getY를 통해서 접근이 가능하다.
  - setter가 없기 때문에 직접 설정할 수 없고 추상화된 메서드를 통해서만 접근이 가능하다.
  - 메서드의 이름으로 직교좌표계를 사용하는지, 극좌표계를 사용하는지 알 수 있다.

### 예시 2

```kotlin
interface BasicVehicle{
    fun getFuelTankCapacityInGallons():Double
    fun getGallonsOfGasoline():Double
}

interface AbstractVehicle {
    fun getPercentFuelRemaining():Double
}
```

- **`BasicVehicle`** 의 경우 각각의 메서드를 통해서 가솔린 용량, 연료탱크의 용량 등을 접근 가능하다.
  - 메서드 이름이 너무 구체적이라서 용량 단위 같은 알지 않아도 되는 정보들까지 접하게 된다.
  - 남은 연료를 보려면 다시 계산이 필요하다. 
  - 추후 단위나 이런게 바뀔 경우 메서드 이름을 전부 변경하거나 해야하는 불상사가 일어날 수 있다.
- **`AbstractVehicle`** 의 경우 남은 연료를 구하는 메서드 하나만 있다.
  - 남은 연료를 구한다고 메서드 이름으로 명확히 알 수 있다.
  - 나머지 연료, 용량의 구체적인 정보는 알 필요가 없다.
  - 추후 단위나 이런게 바뀔 경우 해당 메서드는 변경될 것이 없다.

### 결론
- 불필요한 getter, setter 사용 금지
  - 무분별한 getter, setter는 외부에 노출되어있기 때문에 의도치않은 버그를 유도할 수 있다.
- 메서드의 경우 추상화하여 꼭 필요한 정보만 반환하도록 하자.

## 자료/객체 비대칭

### 예시 1
```kotlin
// 절차적인 도형 클래스
class Square (var side:Double)
class Rectangle (var height:Double, var width:Double)
class Circle (var radius:Double)

class Geometry{
  private val pi:Double = 3.141592653589793

  fun area(shape:Any): Double {
    if(shape is Square){
      return shape.side * shape.side
    }
    if(shape is Rectangle){
      return shape.height * shape.width
    }
    if(shape is Circle){
      return pi * shape.radius * shape.radius
    }
    throw IllegalArgumentException("Unknown shape")
  }
}
```

- WTF
- 내눈엔 별로 좋아보이지 않는다... 새로운 도형이 추가될 때마다 area 메서드를 수정해야한다.
  - 저자는 둘레 길이를 구하는 메서드를 추가하고 싶을때 도형은 아무런 영향이 없다고 설명하지만 내눈엔 그렇다고 하더라도 각각 구하는 로직이 다르기때문에 여러개의 로직이 하나의 메서드에 있는 느낌이다.

```kotlin
// 객체지향적인 도형 클래스
interface Shape{
  fun area():Double
}

class Square (var side:Double):Shape{
  override fun area(): Double = side * side
}
class Rectangle (var height:Double, var width:Double):Shape{
  override fun area(): Double = height * width
}
class Circle (var radius:Double):Shape{
  private val pi:Double = 3.141592653589793
  override fun area(): Double = pi * radius * radius
}
```
- Geometry 클래스가 필요없다.
- 새 도형을 추가해도 영향이 없다.
- 새 함수를 추가시킨다면 각각 도형 클래스 전부를 고쳐야한다.
  - 하지만 이건 관심사를 집중시킬 수 있기 때문에 나쁘지 않다고 생각된다.

### 결론
|절차적인 코드| 객체 지향적인 코드                        |
|--|-----------------------------------|
|기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉽다.| 기존 함수를 변경하지 않으면서 새 자료구조를 추가하기 쉽다. |
|새로운 자료 구조를 추가하기 어렵다.| 새로운 함수를 추가하기 어렵다.                 |

- 절차적인 코드가 더 좋을때도 있으니 너무 맹신하지 말자.
  - 아직까진 잘 모르겠다..

## 디미터 법칙
- 내부구조를 굳이 드러내지 마라.
```kotlin
// 기차 충돌
val outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath()

val outFile = outputDir + "/" + className.replace('.', '/') + ".class"
val fout = new FileOutputStream(outFile)
val out = new BufferedOutputStream(fout)
```
- 위 코드에서 결국에는 파일을 생성하기 위해 절대 경로를 찾는것임을 알 수 있다
- 그렇다면 ctxt에 파일 생성하는 method를 만드는 것이 좋다 (기존코드는 public method가 너무 많아지고 내부구조를 드러내게됨)

```kotlin
val out = ctxt.createScratchFileStream(classFileName)
```

## 자료 전달 객체(DTO)
- Data Transfer Object(DTO)
### DTO
- 공개 변수만 있고 함수가 없는 클래스

### Bean
- 비공개 변수와 getter, setter가 있는 클래스

### 활성레코드
- 비공개 변수와 getter, setter, save, find 같은 데이터베이스 관련 함수가 있는 클래스

### 결론
- 코틀린에는 data class가 있어 DTO를 만들때 최소 bean으로 만들어지는 것 같다.
  - getter, setter 이외에 유용한 함수들 제공

## 결과
- 객체는 동작을 공개하고 자료를 숨긴다.
  - 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉬움
  - 하지만 기존 객체에 새 동작을 추가하기는 어렵다
    - 각 객체마다 작업해주어야함.
- 자료구조는 별다른 동작없이 자료를 노출한다
  - 기존 자료구조에 새 동작을 추가하기는 쉬우나 새 자료구조를 추가하기는 어렵다.
    - 기존 동작을 변경해주어야함.
- 시스템을 구현할 때 새로운 자료 타입을 추가하는 유연성이 필요하다면
  - -> 객체
- 새로운 동작을 추가하는 유연성이 필요하다면
  - -> 자료구조, 절차적 코드
