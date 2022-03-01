## 목차
- [의도를 분명히 밝혀라](#1)
- [그릇된 정보는 피하라](#2)
- [의미 있게 구분하라](#3)
- [발음하기 쉬운 이름을 사용하라](#4)
- [검색하기 쉬운 이름을 사용하라](#5)
- [인코딩을 피하라](#6)
- [자신의 기억력을 자랑하지 마라](#7)
- [클래스 이름](#8)
- [메서드 이름](#9)
- [기발한 이름은 피하라](#10)
- [한 개념에 한 단아를 사용하라](#11)
- [말장난을 하지 마라](#12)
- [해법 영역에서 가져온 이름을 사용하라](#13)
- [문제 영역에서 가져온 이름을 사용하라](#14)
- [의미 있는 맥락을 추가하라](#15)
- [불필요한 맥락을 없애라](#16)
- [마치면서](#17)

---

## 의도를 분명히 밝혀라 <a name = "1"></a>
__좋은 이름을 지으려면 시간이 걸리지만 좋은 이름으로 절약하는 시간이 훨씬 더 많다.__  
변수, 함수, 클래스 이름은 다음과 같은 질문에 모두 답해야 한다.
+ 존재의 이유는?
+ 수행 기능은?
+ 사용 방법은?

이름으로 위의 의도를 파악할 수 없어 주석이 필요하다면 이름이 의도를 분명히 드러내지 못했다는 말이다.  
<br>

__예시 1__  
```java
// Bad
int d; // elapsed time in days

// Good
int elapsedTimeInDays;
int daysSinceCreation;
int daysSinceModification;
int fileAgeInDays;
```
이름 d는 아무 의미도 드러나지 않으나, 의도가 드러나는 이름을 사용하면 변수가 어떤 의도로 만들어졌는지 한번에 알 수 있다.  
<br>

__예시 2__  
```java
// Bad
public List<int[]> getThem() {
    List<int[]> list1 = new ArrayList<int[]>();
    for (int[] x : theList) {
        if (x[0] == 4) {
            list1.add(x);
        }
    }
    return list1;
}
```
theList는 무엇이고, 0번째 인덱스는 무엇이고 4는 무슨 의미인가?  

```java
// Good
public List<int[]> getFlaggedCells() {
    List<int[]> flaggedCells = new ArrayList<int[]>();
    for (int[] cell : gameBoard) {
        if (cell[STATUS_VALUE] == FLAGGED) {
            flaggedCells.add(cell);
        }
    }
    return flaggedCells;
}
```
단순히 이름만 고쳤는데도 함수가 하는 일을 이해하기 쉬워졌다.  
이것이 좋은 이름이 주는 위력이다.  
<br>

## 그릇된 정보는 피하라 <a name = "2"></a>
중의적으로 해석될 수 있는 이름과 비슷해 보이는 명명은 지양해야 한다.  
또한, 개발자에게 특수한 의미를 갖는 단어(List등)은 실제 List가 아니라면 accountList와 같이 변수명에 붙이지 않아야 한다.  
차라리 accountGroup, bunchOfAccounts, Accounts 등으로 명명하는 것이 낫다.  
<Br>

__예시__  
```java
// bad
int a = l;
if (O == 1)
a = 01;
```
소문자 l이나 대문자 O를 변수로 사용하는 것은 끔찍하다.    
l는 숫자 1처럼 보이고 대문자 O는 숫자 0처럼 보인다.  

<br>

## 의미 있게 구분하라 <a name = "3"></a>
연속적인 숫자를 덧붙인 이름(a1,a2,..)과 같이 숫자를 덧붙인 이름은 아무런 의도도 드러나지 않으므로 사용하지 말자.  
클래스 이름에는 Info, Data와 같은 불용어(noise word)를 사용하지 말자.  
<br>

__예시__  
+ NameString vs Name
+ ProductInfo vs ProductData
+ getActiveAccount vs getActiveAccounts vs getActiveAccountInfo
+ money vs moneyAmount

프로젝트에 위와 같은 변수명과 함수명이 함께 존재한다면 도대체 어떤 것을 사용해야 하는가?  
__읽는 사람이 차이를 알 수 있도록 이름을 지어라.__

<br>

## 발음하기 쉬운 이름을 사용하라 <a name = "4"></a>
```java
// Bad
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102";
    /* ... */
};
```
```java
// Good
class Customer {
    private Date generationTimestamp;
    private Date modificationTimestamp;
    private final String recordId = "102";
    /* ... */
};
```

<br>

## 검색하기 쉬운 이름을 사용하라 <a name = "5"></a>
변수 이름의 길이는 변수의 범위에 비례해서 길어진다.  
넓은 범위에서 사용한다면 그만큼 찾기 쉽도록 변수명은 길어진다는 의미이다.  
<br>

__예시__  
```java
// bad
static final int day = 4;

// good
static final WORK_DAYS_PER_WEEK = 4;
```
넓은 범위에서 day라고 검색하면 얼마나 많은 검색어가 나올까?  
이에 반해 WORK_DAYS_PER_WEEK는 빠르고 명확하게 찾을 수 있다.  
<br>

## 인코딩을 피하라(변수에 부가 정보를 더하지 말라) <a name = "6"></a>
+ 변수명에 해당 타입(String 등)을 적지 말자.
+ 멤버 변수 접두어(m_dsc 같이)를 붙이지 말자.
    - 사람들은 접두어를 무시하고 해독하는 방식을 익히기 때문에 접두어는 관심 밖으로 밀려난다.
+ 인터페이스와 구현 클래스 중 인코딩해야 한다면 구현 클래스에 인코딩하자.
    - ShapeFactory의 구현체 -> ShapeFactoryImpl (구현체에 Impl을 붙이는 인코딩)

<br>

## 자신의 기억력을 자랑하지 마라 <a name = "7"></a>
+ 독자가 코드를 읽으면서 변수 이름을 자신이 아는 이름으로 한번 더 생각해서 이해해야 한다면 그 변수 이름은 바람직하지 못하다.  
+ 루프 범위가 아주 작고 다른 이름과 충돌하지 않는다면 루프에서 반복 횟수를 세는 변수 i,j,k 정도는 괜찮다.

<br>

## 클래스 이름 <a name = "8"></a>
+ 클래스 이름과 객체 이름은 __명사 혹은 명사구__ 가 적합하다.
    - Customer, WikiPage, Account, AddressParser
+ __Manager, Processor, Data, Info와 같은 단어는 피한다.__
+ 동사는 사용하지 않는다.

<br>

## 메서드 이름 <a name = "9"></a>
+ 메서드 이름은 __동사나 동사구__ 가 적합하다.
    - postPayment, deletePage, save
+ 접근자, 변경자, 조건자는 값 앞에 get, set, is를 붙인다.
    - setName, getName, isPosted
+ 생성자를 오버로딩할 때는 정적 팩토리 메서드를 사용한다.
    - 메서드는 인수를 설명하는 이름을 사용한다.

```java
Complex fulcrumPoint = Complex.FromRealNumber(23.0);  
```

<Br>

## 기발한 이름은 피하라 <a name = "10"></a>
특정 문화에서만 사용하는 농담같은 이름은 피하고 의도를 분명하고 솔직하게 표현하라.
```java
HolyHandGrenade -> deleteItems
whack -> kill
eatMyShort -> abort
```

<Br>

## 한 개념에 한 단어를 사용하라 <a name = "11"></a>
추상적인 개념 하나에는 단어 하나를 선택하고 이를 고수하라.  
__일관성 있는 어휘를 사용하란 뜻이다.__  
예를 들면, 같은 의미의 메서드를 각 클래스마다 fetch, retrieve, get으로 제각기 부르면 혼란스럽다.

<br>

## 말장난을 하지 마라 <a name = "12"></a>
__한 단어를 두가지 목적으로 사용하지 마라.__  
앞서 '한 개념에 한 단어를 사용하자'라는 규칙을 따랐더니 여러 클래스에 add 메서드가 생겼다고 해보자.  
지금까지 구현한 모든 add 메서드는 기존 값 두 개를 더하거나 이어서 새로운 값을 만들 때 사용했다고 해보자.  
그렇다면 집합에 값 하나를 추가하는 의미의 새로운 메서드를 add라고 정해도 될까?  
기존 add 메서드와 맥락이 다르다.  
그러므로 insert나 append라는 이름이 적당하다.  
<br>

## 해법 영역에서 가져온 이름을 사용하라 <a name = "13"></a>
개발자라면 당연히 알고 있을 JobQueue, AccountVisitor(Visitor Pattern)등은 사용하지 않을 이유가 없다.  
전사용어, 알고리즘 이름, 패턴 이름, 수학 용어 등은 사용해도 좋다.  
<br>

## 문제 영역에서 가져온 이름을 사용하라 <a name = "14"></a>
적절한 프로그래머 용어가 없다면 문제 영역에서 이름을 가져온다.  
문제 영역 개념과 관련이 깊은 코드라면 문제 영역에서 이름을 가져와야 한다.  
후에 코드를 보수하는 프로그래머는 분야 전문가에게 의미를 물어 파악할 수 있다.  
<br>

## 의미 있는 맥락을 추가하라 <a name = "15"></a>
대부분의 이름은 스스로 의미가 분명하지 않다.  
따라서 클래스, 함수, 이름 공간에 넣어 맥락을 부여하도록 한다.  
모든 방법이 실패하면 마지막 수단으로 접두어를 붙인다.  
<br>

__예시__  
```java
// Bad
private void printGuessStatistics(char candidate, int count) {
    String number;
    String verb;
    String pluralModifier;
    if (count == 0) {  
        number = "no";  
        verb = "are";  
        pluralModifier = "s";  
    }  else if (count == 1) {
        number = "1";  
        verb = "is";  
        pluralModifier = "";  
    }  else {
        number = Integer.toString(count);  
        verb = "are";  
        pluralModifier = "s";  
    }
    String guessMessage = String.format("There %s %s %s%s", verb, number, candidate, pluralModifier );

    print(guessMessage);
}
```
함수를 끝까지 읽어보고 나서야 number, verb, pluralModifier라는 변수 세 개가 통계 추측(guess statics)메시지에 사용된다는 사실이 들어난다.
```java
// Good
public class GuessStatisticsMessage {
    private String number;
    private String verb;
    private String pluralModifier;

    public String make(char candidate, int count) {
        createPluralDependentMessageParts(count);
        return String.format("There %s %s %s%s", verb, number, candidate, pluralModifier );
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

    private void thereIsOneLetter() {
        number = "1";
        verb = "is";
        pluralModifier = "";
    }

    private void thereAreNoLetters() {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    }
}
```
함수를 조각으로 쪼개고 GuessStaticsMessage라는 클래스를 만든 후 3개의 변수를 뽑아서 클래스 안에 넣었다.  
클래스명 덕분에 변수명의 맥락이 분명해지고, 이에 따라 알고리즘도 명확해진다.  

<br>

## 불필요한 맥락을 없애라 <a name = "16"></a>
Gas Station Deluxe 애플리케이션을 만든다고 해서 클래스 이름 앞에 GSD를 붙이지는 말자.  
g를 입력하고 자동완성 시 모든 클래스가 표시되는 등 효율적이지 못하다.  

<br>

## 마치면서 <a name = "17"></a>
두려워하지 말고 서로의 명명을 지적하고 고치자.  
결과적으로 이름을 암기하는데 시간을 빼앗기지 않고 '자연스럽게 읽히는 코드'를 짜는데 집중할 수 있다.  

