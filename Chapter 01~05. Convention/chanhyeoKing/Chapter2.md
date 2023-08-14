# [클린코드 핥아먹기 시리즈] 2. 이름 잘 지어라

## [목차]

#### 0. [개요](#개요)

#### 1. [의도를 분명히 밝혀라](#의도를-분명히-밝혀라)

#### 2. [그릇된 정보를 피하라](#그릇된-정보를-피하라)
 
#### 3. [의미 있게 구분하라](#의미-있게-구분하라)

#### 4. [발음하기 쉬운 이름 사용해라](#발음하기-쉬운-이름-사용해라)
 
#### 5. [검색하기 쉬운 이름을 사용해라](#검색하기-쉬운-이름을-사용해라)
 
#### 6. [너 분명히 기억 못하니까 i, j, k 말고 되도 않는 변수명 같은 거 쓰지 마라](#너-분명히-기억-못하니까-i,-j,-k-말고-되도-않는-변수명-같은-거-쓰지-마라)

#### 7. [클래스 이름과 메서드 이름](#클래스-이름과-메서드-이름)

#### 8. [너만 아는 밈으로 이름 짓지 마라](#너만-아는-밈으로-이름-짓지-마라)
 
#### 9. [비슷한 개념은 최대한 통일하되 서로 다른 로직은 확실히 구분하라](#비슷한-개념은-최대한-통일하되-서로-다른-로직은-확실히-구분하라)
 
#### 10. [의미 있는 맥락을 추가하라](#의미-있는-맥락을-추가하라)
 
#### 11. [불필요한 맥락을 없애라](#불필요한-맥락을-없애라)


#### 12. [마치며](#마치며)

## <br><br>
 
---

0. ## 개요

> 지난 기록에선 로버트 C. 마틴의 "클린 코드"의 [1장. 깨끗한 코드]에 대해서 다뤘다.
> 
> 이번 기록에선 클린 코드의 [2장. 의미 있는 이름(너의 이름은)]에 대해서 다루려고 한다.

---

1. ## 의도를 분명히 밝혀라

저자는 아래 예시를 통해 이를 설명한다.

​

    int d; // 경과 시간(단위: 날짜)
  <br>
  
  > 개인적으로 위 코드 정도는 이해 못할 수준은 아니라고 생각한다. 다만 여러 개발자의 손을 거쳐 주석이 지워진다거나 하는 경우에는 디버깅에 꽤나 머리가 아플수도 있을 것 같다. 또 주석 자체가 전반적인 코드를 지저분해 보이게 할 수도 있을 것 같다. 그래서 저자는 변수 이름을 통해 의도를 명확하게 표현하기를 제안한다.

​

      [저자가 제안한 이름]

      int elapsedTimeInDays;
      int daysSinceCreation;
      int daysSinceModification;
      int fileAgeInDays;

> 세부적인 상황에 맞춰 위와 같이 직접적인 표현을 하는것이 클린하다는 의미로 이해했다.
> 저자는 또 다른 예시를 보여주는데

<br>

```
public List<int[]> getThem() {
    List<int[]> list1 = new ArrayList<int[]>();
    for (int[] x: theList)
       if (x[0] == 4)
           list1.add(x);
    return list1;
}
```

<br>

위 코드를 보고 아래 4가지 사항을 체크해보자.

<br>

````
Q1. theList에는 뭐가 들었을까?
Q2. theList에서 0 번째 값을 4와 비교하네? 저기 뭐가 들은거여?
Q3. 4는 무슨 타입을 의미하는거여?
Q4. 저 list1은 호출한 곳에서 어떻게 쓰이는거지?
````


> ﻿A1. 일단 theList가 뭔지 찾아야하고,

> A2. 0번째 값에 뭐가 들었는지 디버깅 해봐야한다.

> A3. 4값이 뭘 의미하는지 히스토리를 찾아봐야한다. (심지어 다른 업무단에서 관리하는 거라면? 또 물어봐야 한다)

> A4. 저 메서드가 쓰이는 클래스를 찾아가서 확인해봐야 한다. 

<br>

![image](https://github.com/choichanhyeok/---/assets/68278903/a935a52f-20f0-45eb-a624-aa00e089dcea)


<br>

> 시뮬레이션만 해봤는데 벌써 짜증난다. 저렇게 간단한 코드는 그래도 금방 이해할 수 있을 거 같은데 조금만 복잡해져도 하루 종일 보고 있어야할 게 뻔할 일이다. 저자는 "저렇게 짜면 안된다는 건 알겠는데 그럼 어떻게 짜라는거지?" 내 생각에 대한 답변을 아래 코드로 제시했다.

​
```
public List<int[]> getFlaggedCells() {
    List<int[]> flaggedCells = new ArrayList<int[]>();
    for (int[] cell : gameBoard)
       if (cell[STATUS_VALUE] == FLAGGED)
           flaggedCells.add(cell);
    return flaggedCells;
}
```

<br><br>

![image](https://github.com/choichanhyeok/---/assets/68278903/33cff2e1-ea49-46e6-bfab-64608a8b3d56)


<br>

> 일단 코드만 딱 봤을 때 내가 해석할 수 있는 부분만 다뤄보면

> infer 1. getflaggedCells()라는 메서드 이름을 보니 게임보드 상에 체크된 애들 정보를 받는건가?
>
> infer 2. int[] 리스트인 flaggedCells라는 객체를 생성했네? 응답으로 내려줄 응답 객체인가보다.
>
> infer 3. 실제 게임보드에서 각 요소를 cell로 추출해서 STATUS_VALUE라는 상태값이 FLAGGED인지 확인하고 응답 객체(flaggedCells)에 넣어주네

> 위 코드만 봐도 대략적인 이해가 가능해진다. 이정도만 되도 사실 땡큐 땡큐 떙떙큐인데, 저자는 여기서 한 가지 개선 사항을 더 제시한다.

​
``` 
public List<Cell> getFlaggedCells() {
    List<Cell> flaggedCells = new ArrayList<Cell>();

    for (Cell cell: gameBoard)
        if (cell.isFlagged())
           flaggedCells.add(cell);
    return flaggedCells;
}
```

<br><br>

![image](https://github.com/choichanhyeok/---/assets/68278903/69af8963-4e19-4272-b77c-f499bd74ac34)


<br>

> 사실 코드를 검토하는 입장에서는 int[]의 인자로 상태를 표시하는게 이해하기가 쉽지 않다. 객체로 빼고 상태값은 ENUM으로 처리하는게 조금 더 좋을 거 같다는 생각을 했는데, 저자도 비슷한 맥락으로 객체를 하나 생성토록 하고 해당 객체의 isFlagged()를 통해 클라이언트에서 세부적인 로직을 모르더라도 코드를 이해할 수 있도록 개선했다. 

​

> + 본 예제에서는 int[]에 대해 객체로 대체했지만 map도 마찬가지이다.


<br>

![image](https://github.com/choichanhyeok/---/assets/68278903/e527254e-93dd-4b3d-98d3-ee290ae16c88)


<br>

> 이제 처음 코드와 저자가 제시한 최종 코드를 비교해보면 확실히 코드의 가독성이나 복잡성이 줄어들었음을 확인할 수 있다. 사실 변수나 메서드명 명확하게 쓰는거나 배열, 맵 대신 객체 적용하는 건 당연한 거여서 이걸 설명하고 싶은게 맞나 싶긴 했지만 아무튼 재밌게 잘 배웠다.

<br><br><br><br>
​

---

2. ## 그릇된 정보를 피하라

<br>

![image](https://github.com/choichanhyeok/---/assets/68278903/fd444935-ffa0-4441-89c0-d2b90e3ee318)


​<br>

> 사실 이것도 당연한 얘긴데 이름을 지을 때 나중에 독자가 이 코드를 보고 헷갈릴 수 있을 이름을 쓰지 말라는거다. 예를 들어 ..

```
fun Product(): Unit{
    val productList: IntArray = intArrayOf()
    .
    .
}
```
<br>


> 위 코드를 보면 productList는 int형 배열인데 List라는 이름을 사용하고 있다. 개발자가 한테 List와 Array는 전혀 다른 개념이기 때문에 이런 부분도 잘못된 부분인 거 알면서 대강 넘어가지 말고 장인 정신 가지고 섬세하게 짜라는게 이 절의 주제인 거 같다.

​<br>


> 다음으로 두번째 예시를 들었는데

<br>

```
getProductAllByUserName()
getProductAnyByUserName()
```

<br>

> 위 코드에서 뭐가 다른지 파악이 빨리 되는가? 심지어 다른 모듈에 들어있다고 하면 일관성이 떨어져 각 모듈의 역할을 파악하기 어렵다. 그래서 위코드처럼 지나치게 유사한 조합의 요소만 보고 잘못된 메서드나 객체를 사용하게 될 수 있다. "이 또한 그릇된 정보"라고 저자는 주장한다.

​
<br>

> 마지막으로 l이랑 o 같은 코드 때문에 헷갈리게 하지 말라는 얘기를 하는데

```
#case 1.
int a = l;
int a = 1;

#case 2.
int b = o;
int b = 0;
```

<br>

> 위 두 코드를 보면 무엇이 문제인지 알겠는가? 다행히도 내가 작성하는 환경은 1과 l을 구분하기 좋은 글꼴을 지원하지만 때로는 저 글꼴들이 헷갈리게 떨어질 수 있다. 그래서 l이나 I, 혹은 1 같은 것들을 식별 구분자로 두지 않는게 좋다는 말을 하고 해당 절을 마무리 했다.

​
<br><br><br><br>

---

## 3. 의미 있게 구분하라

<br>

> 대충 의미 없는 불용어 가져다 붙여서 변수 구분하지 마라

<br>

```
fun capyChars(a1: charArray, a2: charArray){
    for (i in a1.indices)
        a2[i] = a1[i]
}
```

<br>

> 보면 그냥 a라는 변수 때려넣고 그 뒤에 하나 더 필요하니까 [a1, a2, a3, ... aN]까지 연속적인 숫자를 이용하는 경우들이 있는데 이건 앞서 설명한 "그릇된 정보"를 제공한다고 보기도 힘들고 그냥 아무런 정보를 제공하지 못하는 이름이 된다. 

​

> the, money 같은 것들도 불용어가 될 수 있다.

```
(1) Money와 MoneyAmount
(2) theKing과 King
(3) Product와 ProductInfo
```

> 그냥 봤을 때 각 클래스들의 차이를 이해할 수 있겠는가? 이것도 일종의 "그릇된 정보"가 될 수 있다. 

> 결론은 "읽는 사람이 차이를 알 수 있도록 이름을 지어라" 라는게 본 절의 주제인 것 같다.

​<br><br><br><br>

---

4. ## 발음하기 쉬운 이름 사용해라

<br><br>

![image](https://github.com/choichanhyeok/---/assets/68278903/c26aa2db-b17e-40ae-87a5-6c841fffeb4f)


<br><br>

> 이거 우리 회사에서도 자주 보던 내용인데 예를 들어 가상의 전문 명을 Raty04가 있다고 가정하면 저자가 말한대로 바보같이 읽는다. 

> "선배님 ! 여기 보면 알에이티와이공사 전문이 어쩌고 ~~." 이런 것들을 편하게 읽을 수 있도록 작성하라는 의미로 이해했다.

​
<br><br><br><br>

---

5. ## 검색하기 쉬운 이름을 사용해라

> 이건 사실 진짜 겪으면 귀찮은 상황인데, 만약에 내가 찾으려는 변수의 이름이 List<String> productNumbers3 이라고 가정하자. 운영 서버에 대해서 grep을 통해 로그를 확인한다고 하면 "grep productNumber3 server.log"를 쳐서 검색을 하려고 하는 경우에 문제가 생긴다. 

​
<br>

![image](https://github.com/choichanhyeok/---/assets/68278903/3b6d6b63-de67-4dcd-8ed6-1391af727512)


> 만약productNumbers33이라는 녀석이 있으면 이 녀석들까지 무자비로 찾아오게 되는 것이다. 위 그림은 이에 대한 예제는 아니지만 3이라는 숫자의 경우도 여기저기서 많이 쓰기 때문에 검색을 어렵게하는 녀석이라는 점을 기록하기 위해 가져왔다.

​<br><br>​<br><br>

---

6. ## 너 분명히 기억 못하니까 i, j, k 말고 되도 않는 변수명 같은 거 쓰지 마라

<br><br>

![image](https://github.com/choichanhyeok/---/assets/68278903/b7664a09-73cb-4c30-87f1-7ac5fcfdfc4c)


<br>

> 사람의 기억력엔 한계가 있으니 제발 이상한 변수명 대충 지어놓고 기억한다는 소리 하지 말라는 내용이다. 예를 들어서 "akab123이라는 변수는 신용대출 고객들 중에 AML 대상이 아니면서 1원 이체를 수행하지 않은 고객의 암호화된 성명을 저장한 변수"라는 거를 기억할 수 있는게 아니라면 그렇게 하지 말라는게 저자의 주장이었다.

​<br><br><br><br>

---

7. ## 클래스 이름과 메서드 이름

> 클래스 이름은 동사 쓰지 말고, 명사나 명사구 써라.

> Manager, Data 같은 모호한 이름도 피해라

<br><br>

![image](https://github.com/choichanhyeok/---/assets/68278903/23fc68cc-ff36-4205-b76e-f7a27319e59e)


<br>

> 세계적인 축구스타 손흥민의 아버님 손웅정씨께서도 월드라는 클래스명은 의미가 모호하기 때문에 안된다고 열심히 피력중이시다.

​

> (1) 메서드 이름은 동사나 동사구 써라.
>
> (2) 접근자, 변경자, 조건자는 javabean 표준에 따라
>
> (3) [get, set, is]를 붙여라


<br>


![image](https://github.com/choichanhyeok/---/assets/68278903/1ed72ba0-5d77-488f-89d9-5b34a94543f1)


<br><br><br><br>

---

8. ## 너만 아는 밈으로 이름 짓지 마라


<br><br> 


 


![image](https://github.com/choichanhyeok/---/assets/68278903/1f0c7817-2023-4b0c-8ef8-67771a2eae40)

![image](https://github.com/choichanhyeok/---/assets/68278903/ccda4f4e-6814-41cd-8bb3-460873d5d4b9)


<br>

> 저자는 특정 문화에서만 사용하는 농담 같은걸 이름으로 짓는 걸 경계하라고 했다. 그래서 kill 대신에 whack()같은 걸 쓰지 말라고 당부했다.

​
<br><br><br><br>
​
---

9. ## 비슷한 개념은 최대한 통일하되 서로 다른 로직은 확실히 구분하라

### (1) 비슷한 개념은 최대한 통일하라


> 조회 관련 클래스들의 조직도를 파악하는 일을 해야한다고 가정하자. 만약에 이 조회라는 개념에 대해 ["get", "fetch", "find", "retrieve"]등의 다양한 이름으로 변수명들이 작성되었다고 할 때 개발자는 위 네 상황뿐 아니라 다른 조회 관련 표현이 있는지도 찾아봐야 하고 현실적으로 찾지 못할수도 있다. 저자는 이를 방지하기 위해 비슷한 개념은 최대한 통일 하라는 당부를 했다.

​

### (2) 통일하되 로직이 다르면 확실히 구분하라


```
(1) fun addEachValue(baseValue: Int, increment: Int): Int{ return baseValue + increment}
(2) fun addProductList(newProduct: Product): Unit{ ... productList.add(newProduct) }
```
<br>

> 위 두 코드를 보면 (1)의 add는 두 변수를 더하는거고 (2)의 add는 리스트의 값을 append 하는 역할을 한다. 물론 List의 메서드가 add이기 때문에 저쪽까진 건드릴 수없지만 두 메서드는 전혀 다른 역할을 하고 있기 때문에 (2)의 경우 appendProductList가 더 적합하다고 저자는 주장했다.

​<br><br><br><br>

---

10. ## 의미 있는 맥락을 추가하라

> 맥락이 불분명한 코드


```
private void printGuessStatistics(char candidate, int count) {
    String number;
    String verb;
    String pluraModifier;

    if (count == 0) {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    } else if (count == 1) {
        number = "1";
        verb = "is";
        pluralModifier = " 
    } else {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s"; 
    }

    String guessMessage = String.format(
        "There %s %s %s%s", verb, number, candidate, pluralModifier
    );
    print(guessMessage);
}
```
<br>

> 위 코드는 왜 맥락이 불분명하다고 하는걸까? 

​<br>

> 주요하게 사용되는 변수는 number, verb, pluralModifier인데 이 녀석들의 이름만 딱 봤을 땐 추상적인 표현이어서 맥락을 파악할 수가 없다.
> 만약 이 녀석들에 대해 좀 더 깊게 이해하려면 알고리즘을 다 분석해봐야 한다. 예를 들어 아래와 같다.

<br>​

> (1) 자 ... 어디보자 candidate? 따로 연산하는 건 없고 마지막 출력문에 영향을 주는 놈이고
> 
> (2) count는 문자열에 들어갈 요소를 분류하는 값이네
> 
> (3) There %s %s %s%s 구조인데, verb는 is, are 같은게 담길거고 pluralModifiers에는 아 .. 그니까 count에 따라 문장 요소 분류해서 출력하는 함수구나 !

> 이제 저자가 제안하는 개선된 코드를 확인해보자.
​
```
public class GuessStatisticsMessage {
    private String number;
    private String verb;
    private String pluralModifier;

    public String make(char candidate, int count) {
        createPluralDependentMessageParts(count);
        return String.format(
            "There %s %s %s%s", 
             verb, number, candidate, pluralModifier);
    }

    private void createPluralDependentMessageParts(int count) {
        if (count == 0) {
            thereAreNoLetters();
        } else if (count == 1) {
            thereIsOneLetter();
        } else {
            thereAreManyLetters(count);
        }
    }

    private void thereAreManyLetters(int count) {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }

    private void thereIsOneLetter(int count) {
        number = "1;
        verb = "is";
        pluralModifier = "";
    }

    private void thereAreNoLetters(int count) {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    }
}
```

<br><br>

> (1) 자 .. 보면 createPluralDependentMessageParts()를 통해 메시지 생성하나보네?
>
> (2) 음 .. count 값에 따라 알맞는 문장 던져주는 구조인가보다!

> # 파악을 위한 과정이 훨씬 간단해진다.

<br><br><br><br>

---

11. ## 불필요한 맥락을 없애라

> 아니 방금까진 맥락 추가하라며?

<br>

![image](https://github.com/choichanhyeok/---/assets/68278903/11bdd730-be45-4e0b-bbc7-715ce66936ee)


<br>

> 저자가 말하길 "일반적으로는 짧은 이름이 긴 이름보다 좋다. 단, 의미가 분명한 경우에 한해서"라고 말했다.
> 따라서 의미가 분명한 경우에는 "불필요한 맥락을 덜어" 이름을 짧게 만들라는게 이번 절의 주제가 아닌가 싶다.

​<br><br><br><br>

---

12. ## 마치며

```
좋은 이름 선택을 위해선 동료와 문화적 배경이 같아야한다.
이름 변경에 대해 타인의 질책같은 두려움이 있을 수 있지만
그렇다고 코드를 개선하려는 노력을 중단해서는  안된다.
```
​

​<br><br><br><br>

#### 2023-08-14 개발자 최찬혁





