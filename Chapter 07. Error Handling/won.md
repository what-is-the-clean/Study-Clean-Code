# 7장 오류 처리

### 오류 코드보다 예외를 사용하라
```
-- 1. 오류 코드를 반환하는 방법
class DeviceCotroller {
    fun sendShutDown() {
        var handle = getHandle(DEV1)
        if(handle != DeviceHandle.INVALID) {
            retrieveDeviceRecord(handle)
            
            if(record.getStatus() != DEVICE_SUSPENDED) {
                pauseDevice(handle)
                clearDeviceWorkQueue(handle)
                closeDevice(handle)
            } else {
                logger.log("")
            }
        } else {
            logger.log("")
        }
    }
}
-- 복잡한 호출자 코드
```
```
-- 2. 예외를 던지는 코드
class DeviceCotroller {
    fun sendShutDown() {
        try {
            tryToShutDown()
        } catch (e : DeviceShutDownError) {
            logger.log(e)
        }
    }
    fun tryToShutDown(exception: DeviceShutDownError) {
        var handle : DeviceHandle = getHandle(DEV1)
        var record : DeviceRecord = retrieveDeviceRecord(handle)
        pauseDevice(handle)
        clearDeviceWorkQueue(handle)
        closeDevice(handle)
    }
    private fun getHandle(id : DeviceID) : DeviceHandle {
        throw DeviceShutDownError()
    }
}
```

> 디바이스를 종료하는 알고리즘과 오류를 처리하는 알고리즘이 분리 되었다!

### Try, Catch, Finally문부터 작성하자!
+ 예외는 프로그램 안에다 범위를 정의한다. -> 트랜잭션과 비슷하게 catch블록은 일관성을 유지해야 한다.
+ 파일이 없으면 예외를 던지는 알아보는 단위테스트로 기대하는 상태를 정의하기 쉬워진다.
```
catch(e : Exception)
| 예외 유형을 좁혀 범위를 구체적으로 정의한다.
V
catch(FileNotFoundException)
```
> 강제로 예외를 일으키는 테스트 케이스를 작성한 후 테스트롤 통과하게 코드를 작성한다.
> -> 자연스럽게 try 블록의 트랜잭션 범위부터 구현하게 되어 범위 내에서 트랜잭션의 본질을 유지하기 쉬워진다!

### 미확인 에러를 사용하라
+ 확인된 오류의 문제점 : OCP(개방 폐쇄 원칙)을 위반함.
+ ex) 확인된 예외를 던졌는데 catch 블록이 세단계 위에 있다면 그 사이에 모든 메서드가 예외를 정의해야함
+ -> 만약 하위 단계에서 코드를 변경하면 상위 단계 메서드 선언부를 전부 고쳐야함.(throws 추가) -> 캡슐화가 깨짐

## 예외에 의미를 제공하라
+ 전후 상황을 충분히 덧붙이자
+ 호출 스택만으로 부족한 부분은 오류 메시지에 정보를 담아 예외와 함께 던진다.
+ 실패한 연산 이름과 실패 유형도 언급한다.(애플리케이션의 로깅 기능)

### 호출자를 고려한 예외 클래스 정의
```
fun open() {
  try {
    innerPort.open();
  } catch(e : DeviceResponseException) {
    throw PortDeviceFailure(e);
  } catch(e : ATM12Exception) {
    throw PortDeviceFailure(e);
  }
}
```
> 같은 예외를 던져주는 wrapper 클래스로 상위 메서드에서 PortDeviceFailure 예외만 처리하면 되게 해준다.
>  이와 같은 코드는 외부 API의 의존성을 크게 줄여준다!

### 정상 흐름을 정의하자!
```
try {
  var expenses : MealExpenses = expenseReportDAO.getMeals(employee.getID())
  m_total += expenses.getTotal() --  식비를 비용으로 청구
} catch(e : MealExpensesNotFound) {
  m_total += getMealPerDiem() -- 식비를 비용으로 청구하지 않는다
}
|
V
class PerDiemMealExpenses : MealExpenses {
  fun getTotal() : Int {}
}
```
> 이와 같이 클래스나 객체를 조작해 특수 사례를 처리하는 방식이 특수 사례 패턴이라 한다.

### null을 반환하지 말자
+ null 반환하는 코드는 누구 하나라도 확인을 빼먹는 순간 통제불능이 되어버린다.
+ null 대신 예외나 특수 사례 객체를 던져 해결 하자
```
var employees : List<Employee>? = getEmployees()
if(employees != null) { -- null 을 반환 시키지 않으면 필요가 없어진다!
  for(e : Employee in employees) {
    totalPay += e.getPay()
  }
}
```
+ 해결방법
  1. NullPointerException 대신 대체 예외클래스를 던져준다. 그러나 처리기에서 대체 예외 클래스를 어떻게 처리할 지 고민이 필요해진다.
  2. assert문을 사용한다. -> 코드는 읽기 편해지지만 문제는 해결되지 않는다. 여전히 실행 오류가 발생한다.
+ 따라서, null을 반환 하지 못하도록 금지하는 정책이 합리적이다!
> 결론 : 깨끗한 코드는 읽기와 안전성이 높아야 하는데, 이 둘은 양립 가능한 목표이며, 오류 처리는 읽기와 안전성 모두를 향상시켜 준다!
