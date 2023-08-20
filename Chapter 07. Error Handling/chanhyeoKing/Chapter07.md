# [클린코드 핥아먹기 시리즈] 5. 오류 처리

## [목차]

#### 0. 개요

#### 1. 오류 코드대신 예외를 이용하라

#### 2. Try-Catch-Finally문 부터 작성하라

#### 3. 미확인 예외(UnChecked Exception)를 사용하라

#### 4. 오류를 분류하는 방법

#### 5. 예외 때문에 논리를 따라가기 어려워진 경우

#### 6. null을 반환하거나 함수 인자로 전달하지 마라

#### 7. 결론


<br><br>

---
## 0. 개요

> 지난 기록에서는 "클린 코드"의 6장에 대해 다뤘는데 정리하자면 클래스라고 다 같은 클래스가 아니라
> > (1) 자료 구조형
> > 
> > (2) 우리가 생각하는 객체
> > 
> 로 나눌 수 있다는 내용이었다. 자료 구조라는건 말 그대로 자료를 다루기 위한 목적의 클래스로 DTO가 그 예이고, 보다 세부적인 예는
> Getter와 Setter로 자료 구조를 다루라는 Java Beans가 있었다.

<br>

> 우리는 흔히 "현실 세계의 모든 것은 것들은 객체와 매핑 된다"같은 20년도 더 전에 나온 격언을 믿곤 하는데 사실 그건 잘못된 얘기고
> 절차지향적 성질의 "자료구조" 같은 것들도 잘 활용해 좋은 이야기(저자는 프로그램을 이렇게 표현했다)만들 수 있는 저자(프로그래머)
> 가 되라는 내용이었다.

<br> 

오늘은 클린 코드의 7장인 "오류처리"에 대한 내용을 정리하려 한다.

<br><br><br><br>


--- 

## 1. 오류 코드대신 예외를 이용하라

> 저자는 "예외를 이용하지 않는 경우"의 예시를 들며 예외처리가 필요한 이유에 대해 설명한다.


```
public class DeviceController {
    ...
    public void sendShutDown() {
        DeviceHandle handle = getHandle(DEV1);
        
        // 디바이스 상태 점검
        if (handle != DeviceHandle.INVALID) {
            // 레코드 필드에 디바이스 상태 저장
            retrieveDeviceRecord(handle);
            
            // 디바이스가 일시정지 상태가 아니라면 종료
            if (record.getStatus() != DEVICE_SUSPENDED) {
                pauseDevice(handle);
                clearDeviceWorkQueue(handle);
                closeDevice(handle);
            } else {
                logger.log("Device suspended. Unable to shut down");
            }
        } else {
            logger.log("Invalid handle for: " + DEV1);
        }
    }
    ...
}

# Q1. 이런 코드의 단점은 무엇일까
# A1. 함수를 호출할 때마다 오류를 확인해야 한다
```

<br>



> 위 방법처럼 예외를 사용치 않고 직접 오류를 확인하는 경우에는 메서드등을 호출해 해당 "메서드의 작업에 문제가 생기는 경우를 전부 파악해 분기처리 해준 뒤 로그로 찍어야한다." 딱 봐도 까먹고 안하거나 의도적으로 깜빡한척 하기 다분한 환경이다. 그렇다면 어떻게 해야할까?

​
<br>


```
public class DeviceController {
    ...
    public void sendShutDown() {
        try {
            tryToShutDown();
        } catch (DeviceShutDownError e) {
            logger.log(e);
        }
    }

    private void tryToShutDown() throws DeviceShutDownError {
        DeviceHandle handle = getHandle(DEV1);
        DeviceRecord record = retrieveDeviceRecord(handle);

        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
    }

    private DeviceHandle getHandle(DeviceID id) {
        // ... 다른 코드

        throw new DeviceShutDownError("Invalid handle for: " + id.toString());
    }
    
    // ... 다른 메서드 및 변수들
}
```


> getHandle()에서 예외를 던져주도록 구현해서 tryToShutDown에서 해당 에러를 받아볼 수 있게 된다. 그래서 sendShutDown()의 try 구문에서 에러를 검토하다 에러 발생시 로그를 찍도록 했는데 각 구조의 역할은 아래와 같다.
>
> > ﻿(1) 프로세스의 전체적인 흐름은 sendShutDown으로 확인하고
> > 
> > (2) ShutDown 동작을 수행하기 위한 메서드의 호출은 tryToShutDown에서 담당하는데, 이 때 각 메서드의 에러를 채집할 수 있다.
> > 
> > (3) getHandle과 같은 추상화 수준 1 짜리의 각각의 메서드들은 에러를 던져주는 책임을 가지고 있다.
​
<br>

> 결국 저자가 제안한 구조는 전체적인 흐름을 관리하는 메서드와 특정 작업을 수행하는데 필요한 메서드를 관리하는 메서드, 그리고 자신의 에러를 예외처리 해 던져주는 각각의 메서드들의 집합인데 이 처럼 구현하면 논리와 오류 처리 코드가 뒤섞이지 않아 코드가 깔끔해졌고 각 개념을 독립적으로 살펴보고 이해할 수 있게 됐다고 저자는 주장한다.

​
<br>

## 2. Try-Catch-Finally문 부터 작성하라

> 이 절의 제목은 마치 "TDD"를 연상케 했는데 이런 주장을 한 이유는 "try 블록에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기 쉬워진다."였다. 정말 그런지 저자가 제시한 아래 예시를 통해 확인해보자.


​
```
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowInvalidFileName() {
    sectionStore.retrieveSection("invalid-file");
}

// 예외 반환을 하지 않는 예시 (위 @Test(expected = StorageException.class에 매핑 못함)
public List<RecordedGrip> retrieveSection(String sectionName) {
    // 실제로 구현할 때까지 비어 있는 더미를 반환
    return new ArrayList<RecordedGrip>();
}
```


> 일단 retrieveSectionShouldThrowInvalidFileName() 라는 테스트 메서드를 정의해 StorageException을 발생시키려고 하는 거 같은데 현재 retrieveSection()의 경우 예외를 던지는 구간이 없다. 왜 굳이 예외처리를 안한 코드를 먼저 보여줬는지는 모르겠지만 테스트 코드  쓸 때 excepted 인자를 통해 쉽게 구현할 때 안된다는 걸 보여주기 위해서 그랬다고 치고 .. 마저 진행한다.

<br>
​

```
public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName);
    } catch (Exception e) {
        throw new StorageException("retrieval error", e);
    }
    return new ArrayList<RecordedGrip>();
}
```


<br>

> 이제 위 코드를 보면 StorageException라는 에러에 대해 예외처리를 해주고 있다. 이 경우 retrieveSectionShouldThrowInvalidFileName()에서 해당 예외를 보고 테스트를 성공시킬 수 있게 된다.
>
> 그런데 .. 한가지 더 눈에 밟히는 부분이 있지 않은가? 그렇다. catch문을 보면 잡아낼 예외에 대해 Exception으로 범용적이다. 이를 FileNotFoundException으로 변경해 세밀하게 FileInputStream 생성자가 던지는 FileNotFoundException을 잡아낼 수 있다.

<br>
​

```
결론은.

(1) 먼저 강제로 예외를 일으키는 테스트 케이스를 작성 후
(2) 테스트를 통과하면 코드를 작성하는 방법을 권장
(3) 그러면 자연스레 try 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하기 쉬워진다.
```


<br><br><br><br>

---

## 3. 미확인 예외(UnChecked Exception)를 사용하라



```
"논쟁은 끝났다. 여러 해 동안 자바 프로그래머들은 확인된 예외의 장단점을 놓고  논쟁을 벌여왔다."

"당시에는 확인된 에러(checked Exception)을 멋진 아이디어로 여겼다. 메서드 선언시에 메서드가 반환할 예외를 모두 열거했단 말이다."

"하지만 지금은 미확인 예외가 안정적인 소프트웨어를 제작하기 위해 반드시 필요하지 않다는게 분명해졌다. 파이썬, C++, C#을 보라"

​

"그러므로 우리는 진실되게 고민해봐야 한다. 과연 확인된 오류를 사용할 때 치르게 될 비용, 그에 상응하는 이익을 얻을 수 있는지를"
```
​

<img width="874" alt="image" src="https://github.com/choichanhyeok/---/assets/68278903/8aa4734a-3a05-4a25-9d9e-76e23b1801ac">


<br>


> 뜬금없이 등장한 비용이란 용어에 대해 소프트웨어적 관점에서 "연산량"등이 생각난다. 그럼 checkedException을 이용하면 연산량이 많아진다는 얘기일까? 얼추 맞는 거 같기도 하다. 하지만 저자가 말하는 비용은 "객체지향 관점에서의 손해"를 의미한다. 저런 확인된 에러를 이용하려면 우리는 OCP(개방 폐쇄 원칙)이라는 객체지향의 원칙 중 하나를 위반해야한다. 이는 독립성을 헤쳐 발생하는 비용의 이름은 "의존성"이다.
>
> 무슨 말인지 이해가 가지 않는가? 아래 예시를 확인해보자.

​
```
// (1) 최하위 함수
public class LowLevelService {
    public void processData() throws CustomCheckedException {
        // ... 로직 처리 ...
        throw new CustomCheckedException("Data is not valid");
    }
}
```

```
// (2) 중간 단계 함수
public class MidLevelService {
    private LowLevelService service = new LowLevelService();
    
    public void handleData() throws CustomCheckedException {
        service.processData();
    }
}
```

```
// (3) 최상위 함수
public class TopLevelService {
    private MidLevelService service = new MidLevelService();
    
    public void execute() {
        try {
            service.handleData();
        } catch (CustomCheckedException e) {
            // 처리 로직...
        }
    }
}
```

```
public class CustomCheckedException extends Exception {
    public CustomCheckedException(String message) {
        ...
    }
}
```

<br>

> 여기보면 의존성 주입도 안되있고 뭐가 문제가 많은 코드지만 일단 집중해서 볼 것은 (1) 최하위 함수의 processData()를 (2)중간 함수와 (3) 최상위 함수에서 가져다 쓴다는 것이다. 중간 함수의 경우 processData()에서 발생하는 에러를 throws를 통해 넘겨줬고 최상위 함수의 경우 try-catch를 통해 CustomCheckedException을 처리해줬다. 보이는가? 최하위 함수인 processData()에서 던지는 에러에 대한 내용이 해당 메서드를 사용하는 모든 상위 클래스에 영향을 준다. 이로인해 OCP를 위반하게 되는데 아래 질문에 대한 답을 생각해보자.


<br>


​
```
만약 최하위 함수의 processData()가 뱉는 에러가

CustomCheckedException이 아니라 LoveException으로 바뀐다면 뭘 해줘야 하나?

위 코드 기준으로 두 가지를 해줘야한다. (1) 중간 단계 함수의 throws에 보면 CustomCheckedException을 받게 되어 있기에 이 부분을 바꿔줘야하고, (2) 최상위 함수의 경우에 try-catch에 CustomCheckedException이 박혀있어 이것도 바꿔줘야한다.
```


​
<br>


> 이처럼 하나의 클래스를 수정했을 때 상관도 없는 다른 클래스의 내용까지 수정하는 경우에 우리는 "개방폐쇄원칙을 위배했다"라고 한다.
> 
> 그래서 결론은 확인된 예외에 대한 처리는 "OCP를 위반한다"는 것이다. 그렇다면 확인된 예외는 무조건 쓰지 말아야할까? 이 물음에 대한 저자의 답은 아래와 같다.

​<br>

> 아주 중요한 라이브러리를 작성중이라면 모든 예외를 잡아야한다. 이 때는 확인된 예외도 중요하다.
>
> 하지만 그렇지 않은 일반적인 애플리케이션에 있어서 확인된 예외가 발생시키는 의존성이라는 비용은 대부분 이익을 상회한다.
>
> (+ 하지만 전역적 예외처리를 한다면?)


<br><br><br><br>

---

## 4. 오류를 분류하는 방법

> 저자는 오류를 다양한 방법으로 분류할 수 있다고 말한다. 이런 다양한 분류법을 소개하기 전에 가장 거지같은 코드를 보여줬는데 이는 아래와 같다.


```
ACMEPort port = new ACMEPort(12);
try {
    port.open();
} catch (DeviceResponseException e) {
    reportPortError(e);
    logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
    reportPortError(e);
    logger.log("Unlock exception", e);
} catch (GMXError e) {
    reportPortError(e);
    logger.log("GMX error");
} finally {
    ..
}
```


> 위 코드는 어떤 예외분류 방법을 사용한걸까? 일명 "그딴거 모르겠고 예외 나열"방법을 사용한거다. 보면 중복 코드가 딱 보이지 않은가? 이렇게 바보같은 코드는 개선하기도 쉽다. 아래 코드를 보자


​
```
LocalPort port = new LocalPort(12);
try {
    port.open();
} catch (PortDeviceFailure e) {
    reportError(e);
    Logger.log(e.getMessage(), e);
} finally {
    ..
}
```


<br>


```
public class LocalPort {
    private ACMEPort innerPort;

    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }

    public void open() throws PortDeviceFailure {
        try {
            innerPort.open();  
        } catch (DeviceResponseException e) {
            throw new PortDeviceFailure(e);
        } catch (ATM1212UnlockedException e) {  
            throw new PortDeviceFailure(e);
        } finally {
            ...
        }
    }
    ...
}
```


> 단순히 ACMEPort를 간접적으로 다루는 LocalPort 클래스를 통해 에러 발생시 각 에러를 대표하는 PortDeviceFailure(e)을 생성해 보내도록 했는데 이런 감싸기 기법을 통해 의존성을 줄일 수 있다. 만약에 새로운 에러가 추가된다고 하더라도 개발자는 저 LocalPort만 수정하면 되기에 OCP를 위배하지 않을 수 있게 된다.

​

​<br><br><br><br>

---

## 5. 예외 때문에 논리를 따라가기 어려워진 경우
<br>


```
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch (MealExpensesNotFound e) {
    m_total += getMealPerDiem();
}
```

> 위 코드를 보자. try-catch문으로 감싸져 있고 사전에 expenseReportDAO의 getMeals에서 MealExpensesNotFound를 던지도록 구현되었을 때 이를 처리하는 코드이다. 보면 구현 논리와는 상관 없는 예외 처리 구문 때문에 코드를 읽기가 힘들다. (사실 그정도 까지 불편하진 않다.) 그래서 저자는 이에 대한 개선 코드를 제안한다.

​
<br>


```
MealExpenses expenses = expenseReportDAO.getMeals (employee. getID());
m_total += expenses.getTotal();
```


> 위 코드처럼 핵심적인 비즈니스 로직만 반영토록 하길 제안했는데 이를 구현하려면 어떻게 해야할까? expenseReportDAO의 getMeals에서 기존 MealExpensesNotFound가 떨어지는 경우에 대해 처리할 MEalExpenses 클래스를 상속받는 별도 클래스를 생성 후 getTotal()의 값을 재정의 해야한다.


<br>


```
public class PerDiemMealExpenses implements MealExpenses {
public int getTotal() {
    //기본값으로 일일기본 식비를 반환 ..
}}
```


> 위 클래스를 정의해 아래와 같이 활용하면 된다. 아래는 대략적인 전-후 과정의 코드이다.


---

​
```
# (1-1) 특수 사례 패턴 적용 전
public class ExpenseReportDAO {
    public MealExpenses getMeals(int employeeId) throws MealExpensesNotFound {
        // DB나 다른 데이터 소스에서 직원의 식사비 정보를 조회
        if (데이터가_존재한다면) {
            return new MealExpenses(조회된_데이터);
        } else {
            throw new MealExpensesNotFound("No meal expenses found for employeeId: " + employeeId); // 예외 던지기
        }
    }
}
```



```
# (1-2) 특수 사례 패턴 적용이 안된 경우 클라이언트 코드
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch (MealExpensesNotFound e) {
    m_total += getMealPerDiem();
}

> 예외 때문에 코드가 복잡한 모습
```


```
# (2-1) 특수 사례 패턴 적용 후
public class ExpenseReportDAO {
    public MealExpenses getMeals(int employeeId) {
        // DB나 다른 데이터 소스에서 직원의 식사비 정보를 조회
        if (데이터가_존재한다면) {
            return new MealExpenses(조회된_데이터);
        } else {
            return new PerDiemMealExpenses(); // 예외에 대한 처리가 적용된 클래스 던지기
        }
    }
}
```


```
# (2-2) 특수 사례 패턴 적용된 경우 클라이언트 코드
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();

> 코드가 직관적인 모습
```


> 확인된 에러를 이용해 예외를 던져주는 경우보다 MealExpenses를 상속받는 클래스를 통해 getTotal을 재정의 한 경우가 클라이언트 관점에서 코드가 보다 직관적이 된다는 점을 저자는 강조하고 싶었던 것 같다.

​
<br><br><br><br>


---


## 6. null을 반환하거나 함수 인자로 전달하지 마라

> 그냥 하지 말라면 하지 마라.

​
<br><br><br><br>

---


## 7. 결론

> 그래서 결론은 말이다 ..

​
```
1. 깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다(예외 처리).
2. 깨끗한 코드와 안정성 높은 코드는 상충하는 목표가 아니다. (확인된 예외를 사용하면 전역적 예외처리를 하지 않는경우 OCP를 위반하긴 하지만)
3. 단순히 throws로 선택적 예외를 던지는 방법 말고도 DIP를 이용해 예외를 처리하는 방법도 있다는 얘기다.
4. 따라서 그냥 thorws로 예외 던지는 방법 말고 DIP 이용해 "오류 처리를 프로그램 논리와 분리 하면" 깨끗하면서 안정성 높은 코드를 쓸 수 있다.
```
