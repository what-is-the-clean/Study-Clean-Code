# [클린코드 핥아먹기 시리즈] 3. 함수 잘 만드는 법 알려줄까?

## [목차]

#### [0. 개요](#개요)

#### [1. 개떡같은 코드](#개떡같은-코드)

#### [2. 작게 쪼개라](#작게-쪼개라)

#### [3. 함수의 들여쓰기 수준은 1단이나 2단을 넘어서면 안된다](#함수의-들여쓰기-수준은-1단이나-2단을-넘어서면-안된다)

#### [4. 한 가지만 해라 (단일 책임 원칙)](#한-가지만-해라)

#### [5. 이상적인 함수의 인수는 0개(무항)이다](#이상적인-함수의-인수는-0개)

#### [6. 플래그 인수는 추하다](#플래그-인수는-추하다)
  
#### [7. 인수 객체](#인수-객체)
 
#### [8. 임마! 그럼 가변 파라미터는!](#그럼-가변-파라미터는)
 
#### ﻿[9. 부수 효과는 거짓말을 하는것과 마찬가지다](#부수-효과는-거짓말을-하는것과-마찬가지다)

#### [10. 일반적으로 출력 인수는 피하는게 좋다](#일반적으로-출력-인수는-피하는게-좋다)
 
#### [11. 함수에서 값의 변경과 조회 작업을 꼭 분리해라](#함수에서-값의-변경과-조회-작업을-꼭-분리해라)
 
#### [12. 반복하지 마라](#반복하지-마라)
 
#### [13. return은 하나만 (구조적 프로그래밍)](#return은-하나만)
 
#### [14. 마치며](#마치며)


 <br><br>

---

0. ## 개요

>지난 기록에서는 "의미 있는 이름"을 짓는 법에 대해서 배웠다.
>
>이번 기록에서는 클린 코드 3장 ["함수"]에 대해서 정리하려 한다.

 <br><br> <br>

---

1. ## 개떡같은 코드
 <br>
 
```
public static String testableHtml(PageData pageData, boolean includeSuiteSetup) throws Exception {
    WikiPage wikiPage = pageData.getWikiPage();
    StringBuffer buffer = new StringBuffer();
    
    if (pageData.hasAttribute("Test")) {
        if (includeSuiteSetup) {
            WikiPage suiteSetup = PageCrawlerImpl.getInheritedPage(SuiteResponder.SUITE_SETUP_NAME, wikiPage);
            if (suiteSetup != null) {
                WikiPagePath pagePath = suiteSetup.getPageCrawler().getFullPath(suiteSetup);
                String pagePathName = PathParser.render(pagePath);
                buffer.append("!include -setup .").append(pagePathName).append("\n");
            }
        }
        
        WikiPage setup = PageCrawlerImpl.getInheritedPage("SetUp", wikiPage);
        if (setup != null) {
            WikiPagePath setupPath = wikiPage.getPageCrawler().getFullPath(setup);
            String setupPathName = PathParser.render(setupPath);
            buffer.append("!include -setup .").append(setupPathName).append("\n");
        }

        buffer.append(pageData.getContent());

        if (pageData.hasAttribute("Test")) {
            WikiPage teardown = PageCrawlerImpl.getInheritedPage("TearDown", wikiPage);
            if (teardown != null) {
                WikiPagePath tearDownPath = wikiPage.getPageCrawler().getFullPath(teardown);
                String tearDownPathName = PathParser.render(tearDownPath);
                buffer.append("\n").append("!include -teardown .").append(tearDownPathName).append("\n");

                if (includeSuiteSetup) {
                    WikiPage suiteTeardown = PageCrawlerImpl.getInheritedPage(SuiteResponder.SUITE_TEARDOWN_NAME, wikiPage);
                    if (suiteTeardown != null) {
                        WikiPagePath pagePath = suiteTeardown.getPageCrawler().getFullPath(suiteTeardown);
                        String pagePathName = PathParser.render(pagePath);
                        buffer.append("!include -teardown .").append(pagePathName).append("\n");
                    }
                }
            }
        }
    }

    pageData.setContent(buffer.toString());
    return pageData.getHtml();
}
```

<br>


> 위 코드 한번 이해해봐라

> 저자는 대뜸 위 코드를 주며 3분 줄테니 이해해 보라고 독자를 무시한다. "우리 회사 코드도 개떡같은거라면 어디가서  꿀리지 않는데 내가 그래도 그 개떡같은 코드들 속에서 신용대출 프로세스에 기능 반영도 하는데 나를 무시하다니, 보험사 레거시 코드로 단련된 내 솜씨를 보여주마" 하고 도전했다가 1분 30초만에 상황 파악하고 개선안을 확인했다.



<br>
​

```
public static String testableHtml(PageData pageData, boolean includeSuiteSetup) throws Exception {
    boolean isTestPage = pageData.hasAttribute("Test");
    if (isTestPage) {
        WikiPage testPage = pageData.getWikiPage();
        StringBuffer newPageContent = new StringBuffer();
        includeSetupPage(testPage, newPagecontent, isSuite);
        newPageContent.append(pageData.getContent());
        includeTeardownPages(testPage, newPageContent, isSuite);
        pageData.setContent(newPageContent.toString());
    }
    return pageData.getHtml();
```

<br>

> 위 처럼 함수들을 짧은 단위로 분리해서 보면 그래도 어느정도 눈에 들어온다. 사실 저자는 저 코드도 FitNewsse에 익숙하지 않으면 이해하기 어려울거라고 말하지만, 그래도 대략적인 내용은 파악할 수 있다. 이 처럼 보다 직관적으로 프로그램의 흐름을 파악할 수 있도록 하기 위해선 뭐가 필요할까?




<br><br><br><br>

--- 

2. ## 작게 쪼개라

<br>

![image](https://github.com/choichanhyeok/---/assets/68278903/a3936782-6754-4216-bb48-0392c18fe8e0)


<br>

> 저자는 무조건 작게 쪼개라고 한다. 캔트백의 코드를 봤을 때 상상 이상으로 작아 놀랐다는 일화를 얘기하며 기존에 개선된 코드보다 더 줄일것을 제안했는데 먼저 코드는 아래와 같다.

​
```
public static String testableHtml(PageData pageData, boolean isSuite) throws Exception {
    if (isTestPage(pageData))
        includeSetupAndTeardownPages(pageData, iusSuite);

    return pageData.getHtml();  
}
```


> 하나의 메서드는 위 코드 수준으로 굉장히 짧아야 한다고 주장한다. 캔트백의 코드는 전부 다 2줄 ~ 3줄 짜리였다나 ..

<br><br><br><br>

---

3. ## 함수의 들여쓰기 수준은 1단이나 2단을 넘어서면 안된다.



```
fun gotoMarry(choichanhyeok: Choichanhyeok): Unit{
   if (choichanhyeok.get재산() > 5억){
        if (choichanhyeok.get나이() < 30) {
             if(choichanhyeok.isGirlfriendApprove()){
                 choichanhyeok.changeStatus("기혼")
            }
        }
    }
}
```


> 위 코드는 내가 결혼을 할 수 있는지에 대해 판별하는 간단한 함수이다.
> 
> 보면 if문이 3개가 중첩되면서 들여쓰기가 3개가 됐다. 이러면 안된다는 얘기이다. 이걸 저저가 말한대로 들여쓰기 하나로 리팩토링하면 ..

​<br>

```
fun gotoMarry(choichanhyeok: Choichanhyeok): Unit{
    if (choichanhyeok.canMarry()){ choichanhyeok.changeStatus("기혼") }
}
```

<br>

> 위 처럼 한 줄로 리팩토링하는 것이 좋다는게 저자가 이번 절에서 말하고자 하는 바로 이해했다.

<br><br><br><br>

---
​

4. ## 한 가지만 해라 (단일 책임 원칙)

> "의미 있는 다른 이름으로 함수를 추출할 수 있다면 그 함수는 아직도 여러 작업을 하는 셈이다"
>
> 저자는 메서드는 하나의 책임만 지는게 좋다고 주장한다. 위 gotoMarry 함수 역시 별도로 함수를 추출할 수 없기 때문에 한가지 일만 한다고 볼 수 있다. 이런 부분에 대해 저자는 "추상화 수준을 하나로 !"라고 표현했다.

​<br>

> 단, 내려가기 규칙을 준수해야 보다 읽기 좋은 코드가 된다는 점을 강조했는데 내려가기 규칙이란 아래와 같다.

​

>(1) 한 함수 다음에는 추상화 수준이 한 단계 낮은 함수가 온다.
>
>(2) 즉, 위에서 아래로 프로그램을 읽으면 함수 추상화 수준이 한 번에 한 단계씩 낮아져야 한다.

<br><br><br><br>

---

5. ## 이상적인 함수의 인수는 0개(무항)이다.

> 저자는 인수는 적을수록 좋다고 가정한다. 또 3항은 피하는 편이 좋고 4항은 특별한 이유가 필요하다고 주장하는데 옛날에 김동현 튜터님이 말씀해주신 부분이 이 부분이었던 거 같다. 내가 파이썬으로 GSMTP 모듈을 구현했을 때 이 얘기를 해주셨는데 그 때는 그냥 "테스트 코드 짤 때 어려움이 생겨서 그런갑다 ~"하고 넘어갔는데 지금 보니 클린 코드 관점에서도 좋지 않은 상태인 것 같다.

​<br><br>

> 최선은 입력 인수가 없는 경우이며, 차선은 입력 인수가 1개뿐인 경우다.
> (로버트 C. 마틴)

​<br><br><br><br>

---

6. ## 플래그 인수는 추하다

> 사실 저자도 별 도리가 없을 땐 사용하지만, 플래그 인수를 사용한다는건 SRP(단일 책임 원칙)을 위배하고 있다거나 추상화 레벨이 1이 아니라는 반증이 된다고 한다. 이 절에서 저자는 "플래그 인수는 추하다"라고 강하게 표현했지만 그것보다는 "플래그 인수를 사용한다는 건 SRP를 위배하고 있다는 반증이 될 수 있다."라고 표현하는게 조금 더 우아하다고 생각한다.

​
<br><br><br><br>

---
​

7. ## 인수 객체

<br>

> 사실 파라미터는 왠만하면 1항 넘기지 말라고 할 때부터 생각했던 건데 저자가 딱 그 부분을 긁어줬다.

<br>​

> "객체를 생성해 인수를 줄이는 방법이 눈속임이라 여겨질지 모르지만 그렇지 않다. 파라미터를 묶어 넘기려면 이름을 붙여야 하므로 결국은 개념을 표현하게 된다."

<br>
​

```
Circle makeCircle(double x, double y, double radius);
circle makeCircle(Point center, double radius);
```


<br><br><br><br>

---

8. ## 임마! 그럼 가변 파라미터는!

<br>

> 마! 그럼 가변 파라미터는?


![image](https://github.com/choichanhyeok/---/assets/68278903/9114074d-ce12-42f8-8966-2a4213f1c5ba)


<br>


> 저자는 "가변 인수 전부를 동등하게 취급하면 List형 인수 하나로 취급할 수 있다"라고 한다. 이것도 삼항 사항 파라미터가 되지 않도록 주의를 주며 해당 절을 마무리 했다.

<br><br><br><br>



---

9. ## 부수 효과는 거짓말을 하는것과 마찬가지다

> "부수 효과는 거짓말이다. 함수에서 한 가지를 하겠다고 약속하고선 남몰래 다른짓도 하니까"

​<br>

> 문장에서 느껴지는 분노가 너무 날이 서 있어 당황했다. 마치 바람피다 걸린 남자친구한테 할 법한 어투라고나 할까? 뭐 그렇게 까지 서운할 일인가 싶긴 한데 어쨌든 부수 효과를 주의해야 한다고 한다.


<br><br><br><br>

---
​

10. ## 일반적으로 출력 인수는 피하는게 좋다.

```
appendFooter(s);
```

> 위 코드를 보자. 바닥글에 s면 아마 string 일까? 어쨋든 이게 s라는걸 바닥글로 첨부한다는건지 s라는 객체에 바닥글을 첨부한다는 건지 헷갈린다. 사실 일반적으로는 전자가 더 자연스럽지만 s라는걸 바닥글로 첨부한다고 하더라도 출력 자체를 저렇게 인자로 넘기는건 좋지 않다고 한다.

​<br>

> 바로 this 때문인데, this가 나온 이유가 출력 인수로 사용하기 위해서라고 한다. 만약 s라는걸 출력 하려면 해당 객체의 메서드를 이용하는게 좋다.

```
report.appendFooter()
```

<br><br><br><br>

---

11. ## 함수에서 값의 변경과 조회 작업을 꼭 분리해라

<br>

> "객체 상태를 변경하거나 아니면 객체 정보를 반환하거나 둘 중 하나만 해라. 둘 다 하면 안된다."

​<br>

> 사실 이건 클린 코드에서 제시한 예시를 들지 않고도 직관적으로 봤을 때 당연한 소리긴 하다. 지난 학습 기록까지 자주 등장한 SRP를 준수하기 위해서 뿐 아니라 메서드의 명을 명확히 쓰기 위해서라도 조회와 수정이 동시에 일어나는 건 좋지 않긴 한데 예외는 있을 것 같다.

​


```
fun updateProfile(updateUserRequest: UpdateUserRequest): Int{
    .
    .
    val updatedUserId = userRepository.update(updateUserRequest)
    return updatedUserId
}
```



> 위 코드처럼 유저 정보 업데이트를 했을 때 정책에 의해서 유저 ID를 응답해줘야 한다던지, 특정 정책의 updateUserResponse 객체를 내려줘야 하는 경우가 있을 수 있는데 이 경우에는 수정과 조회 작업을 어쩔 수 없이 병행해야 할 것 같다.

> #### 마찬가지로 오류 코드 같은것들도 직접 내려서 비교 하는 건 좋지 않고 예외처리를 통해 코드단에서 신경쓰지 않고 응답토록 하는걸 권고했다.

​
<br><br><br><br>

---
​

12. ## 반복하지 마라

> 알고리즘 문제 같은것들도 마찬가지지만 어느샌가 보면 유사하게 반복적으로 작성되는 코드들이 있다. 이런 것들을 최소화 하고 단순히 함수로 추출하는 방법 외에도 ["구조적 프로그래밍", "AOP", "COP"]등의 중복 제거 전략을 제안해주었다.

​<br>


﻿> "하위 루틴을 발명한 이래로 소프트웨어 갭라에서 지금까지 일어난 혁신은 소스 코드에서 중복을 제거하려는 지속적인 노력으로 보인다."
> > (어쩌고 C. 마틴)
​
<br><br><br><br>

---

13. ## return은 하나만 (구조적 프로그래밍)

<br>

> "break나 continue를 사용해선 안되며 goto는 절대!!! 로 안된다!"

> 라는게 에츠허르 데이크스트라의 구조적 프로그래밍의 원칙이라고 하는데 사실 저자는 이 원칙의 목표와 규율은 공감하지만 함수가 작다면 사실 위 규칙은 별 이익을 제공하지 못한다고 한다. 저자는 지금까지 SRP를 준수하기 위해 작은 함수를 강조해왔기에 이 원칙에 대해 "함수를 작게 만든다면 간혹 return, break, continue를 여러 차례 사용해도 괜찮다"라고 주장했다. 다만 goto 문에 대해서는 똑같이 피하라고 권고했는데 이유는 "goto문은 큰 함수에서 의미가 있으므로 작은 함수를 작성해야 하는 우리는 피해야한다"였다. 

​
<br><br>
​


 

![image](https://github.com/choichanhyeok/---/assets/68278903/2c71823e-ffbd-4129-8440-4215de4848cb)


​
<br><br><br><br>

--- 

14. ## 마치며

> 함수는 동사며 클래스는 명사이다.

> 대가 프로그래머는 시스템을 "구현해야 하는 프로그램"이 아니라 "풀어갈 이야기"로 여긴다.

> 이 장에서는 함수를 잘 만드는 기교를 소개했다. 이를 믿고 따른다면 체계 잡힌 좋은 함수가 나오리라 믿는다.

<br><br>

> 하지만 진짜 목표는 시스템이라는 이야기를 풀어가는 데 있다는 사실을 명심하라.

> 여러분이 작성하는 함수가 분명하고 정확한 언어로 깔끔하게 같이 맞아 떨어져야 이야기를 풀어가기 쉬울 것이다.

​

​

[+] 전반적으로 이 마틴 아저씨는 프로그래머는 작가라는 메타포로 글을 전개해 나가는 것 같다.

​

#### 2023-08-14 개발자 최찬혁
