## 목차
- [Intro](#1)
- [TDD 법칙 세 가지](#2)
- [깨끗한 테스트 코드 유지하기](#3)
    - [테스트는 유연성, 유지보수성, 재사용성을 제공한다](#4)
- [깨끗한 테스트 코드](#5)
  - [도메인에 특화된 언어](#6)
  - [이중 표준](#7)
- [테스트 당 assert 하나?](#8)
  - [테스트 당 개념 하나](#9)
- [F.I.R.S.T](#10)
- [결론](#11)
---



## Intro <a name = "1"></a>
1997년도만 해도 TDD(Test Driven Development) 개념은 아무도 몰랐다.  
대다수에게 단위 테스트란 자기 프로그램이 돌아간다는 사실만 확인하는 일회성 코드에 불과했다.  
지금은 애자일과 TDD 덕택에 단위 테스트를 자동화하는 프로그래머들이 이미 많아졌으며 점점 늘어나는 추세이다.  
하지만 우리 분야에 테스트를 추가하려고 급하게 서두르는 와중에 많은 프로그래머들이 제대로 된 테스트 케이스를 작성해야 한다는 좀 더 미묘한 (그리고 더 중요한)사실을 놓쳐버렸다.  
<br>

## TDD 법칙 세 가지 <a name = "2"></a>
현재 TDD가 실제 코드를 짜기 전에 단위 테스트부터 짜라고 요구하는 사실은 모르는 사람은 없다.  
하지만 이는 빙산의 일각에 불가하다.  
3가지 법칙을 보자.
1. 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
2. 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
3. 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

위 세 가지 규칙을 따르면 개발과 테스트가 대략 30초 주기로 묶인다.  
이렇게 일하면 결국 엄청난 양의 테스트 케이스가 나오고, 실제 코드를 사실상 전부 테스트하는 테스트 케이스가 나온다.  
하지만 실제 코드와 맞먹을 정도로 방대한 테스트 코드는 심각한 관리 문제를 유발한다.  
<br>

## 깨끗한 테스트 코드 유지하기 <a name = "3"></a>
책에서는 저자의 예시가 나오지만 요점은 다음과 같다.  
테스트 코드는 사고와 설계와 주의가 필요하며, 실제 코드 못지않게 중요하고 깨끗해야 한다.  

### 테스트는 유연성, 유지보수성, 재사용성을 제공한다 <a name = "4"></a>
코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목이 바로 __단위 테스트__ 다.  
이유는 단순하다.  
테스트 케이스가 있으면 변경이 두렵지 않기 때문에 변경이 쉬워진다.  
그러므로 실제 코드를 점검하는 자동화된 단위 테스트는 설계와 아키텍처를 최대한 깨끗하게 보존하는 열쇠다.  
<br>

## 깨끗한 테스트 코드 <a name = "5"></a>
깨끗한 테스트 코드를 만들려면 __가독성__ 이 가장 중요하다.  
여느 코드와 마찬가지로 가독성을 높이려면 명료성, 단순성, 풍부한 표현력이 필요하다.  
```java
public void testGetPageHieratchyAsXml() throws Exception {
  crawler.addPage(root, PathParser.parse("PageOne"));
  crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
  crawler.addPage(root, PathParser.parse("PageTwo"));

  request.setResource("root");
  request.addInput("type", "pages");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response =
    (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("<name>PageOne</name>", xml);
  assertSubString("<name>PageTwo</name>", xml);
  assertSubString("<name>ChildOne</name>", xml);
}

public void testGetPageHieratchyAsXmlDoesntContainSymbolicLinks() throws Exception {
  WikiPage pageOne = crawler.addPage(root, PathParser.parse("PageOne"));
  crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
  crawler.addPage(root, PathParser.parse("PageTwo"));

  PageData data = pageOne.getData();
  WikiPageProperties properties = data.getProperties();
  WikiPageProperty symLinks = properties.set(SymbolicPage.PROPERTY_NAME);
  symLinks.set("SymPage", "PageTwo");
  pageOne.commit(data);

  request.setResource("root");
  request.addInput("type", "pages");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response =
    (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("<name>PageOne</name>", xml);
  assertSubString("<name>PageTwo</name>", xml);
  assertSubString("<name>ChildOne</name>", xml);
  assertNotSubString("SymPage", xml);
}

public void testGetDataAsHtml() throws Exception {
  crawler.addPage(root, PathParser.parse("TestPageOne"), "test page");

  request.setResource("TestPageOne"); request.addInput("type", "data");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response =
    (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("test page", xml);
  assertSubString("<Test", xml);
}
```
PathParser의 호출을 살펴보자.  
PathParser는 문자열을 pagePath 인스턴스로 변환한다.  
이 코드는 테스트와 무관하며 테스트 코드의 의도만 흐린다.  
responder 객체를 생성하는 코드와 response를 수집해 변환하는 코드 역시 잡음에 불과하다.  
게다가 resource와 인수에서 요청 URL을 만드는 어설픈 코드도 보인다.  
마지막으로 위 코드는 독자를 고려하지 않는다.  
독자들은 온갖 잡다하고 무관한 코드를 이해한 후에야 간신히 테스트 케이스를 이해한다.  
이를 개선하면 다음과 같다.  
```java
public void testGetPageHierarchyAsXml() throws Exception {
  makePages("PageOne", "PageOne.ChildOne", "PageTwo");

  submitRequest("root", "type:pages");

  assertResponseIsXML();
  assertResponseContains(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
}

public void testSymbolicLinksAreNotInXmlPageHierarchy() throws Exception {
  WikiPage page = makePage("PageOne");
  makePages("PageOne.ChildOne", "PageTwo");

  addLinkTo(page, "PageTwo", "SymPage");

  submitRequest("root", "type:pages");

  assertResponseIsXML();
  assertResponseContains(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
  assertResponseDoesNotContain("SymPage");
}

public void testGetDataAsXml() throws Exception {
  makePageWithContent("TestPageOne", "test page");

  submitRequest("TestPageOne", "type:data");

  assertResponseIsXML();
  assertResponseContains("test page", "<Test");
}
```
잡다하고 세세한 코드는 거의 다 없애고 테스트 코드는 본론에 돌입해 진짜 필요한 자료 유형과 함수만을 사용한다.  
결과적으로 독자는 부가적인 부분을 이해할 필요 없이 메서드 이름만으로 어떤 동작을 수행하고 있는지 알 수 있다.  

### 도메인에 특화된 언어 <a name = "6"></a>
바로 위에서 리팩토링한 방식은 도메인에 특화된 언어(DSL)로 테스트 코드를 구현하는 기법을 보여준다.  
흔히 쓰는 시스템 조작 API를 사용하는 대신 API 위에 함수와 유틸리티를 구현한 후 그 함수와 유틸리티를 사용하므로 테스트 코드를 짜기도 읽기도 쉬워진다.  
이렇게 구현한 유틸리티는 테스트 코드에서 사용하는 특수 API가 된다.  
__즉, 테스트를 구현하는 당사자와 나중에 테스트를 읽어볼 독자를 도와주는 테스트 언어이다.__  

### 이중 표준 <a name = "7"></a>
테스트 API코드에 적용하는 표준은 실제 코드에 적용하는 표준과 확실히 다르다.  
단순하고 간결하고 표현력이 풍부해야 하지만, 실제 코드만큼 효율적일 필요는 없다.  
실제 환경이 아니라 테스트 환경에서 돌아가는 코드이기 때문인데, 실제 환경과 테스트 환경은 요구사항이 판이하게 다르다.  
```java
@Test
public void turnOnLoTempAlarmAtThreashold() throws Exception {
  hw.setTemp(WAY_TOO_COLD); 
  controller.tic(); 
  assertTrue(hw.heaterState());   
  assertTrue(hw.blowerState()); 
  assertFalse(hw.coolerState()); 
  assertFalse(hw.hiTempAlarm());       
  assertTrue(hw.loTempAlarm());
}
```
위 코드는 세세한 사항이 아주 많다.  
일단 tic 함수는 신경쓰지 말고 온도가 급강하 했다는 것만 신경써서 보자.  
위 코드를 읽으면 점검하는 상태 이름과 상태값을 확인하느라 눈길이 이리저리 흩어진다.  
heaterState라는 상태 이름을 확인하고 왼쪽으로 눈길을 돌려 aaserTrue를 읽는다.  
이런식으로 모두 확인하는 것은 불편하고 읽기 어렵다.  
이를 개선해보자.  
```java
@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
  wayTooCold();
  assertEquals("HBchL", hw.getState()); 
}
```
tic 함수는 wayTooCold라는 함수를 만들어 숨겼다.  
그런데 assertEquals에 들어있는 이상한 문자열에 주목한다.  
대문자는 '켜짐'이고 소문자는 '꺼짐'을 뜻한다.  
문자는 항상 '{heater, blower, cooler, hi-temp-alarm, lo-temp-alarm}' 순서다.  
비롯 위 방식이 그릇된 정보를 피하라는 규칙의 위반에 가깝지만 여기서는 적절해 보인다.  
일단 의미만 안다면 눈길이 문자열을 따라 움직이며 결과를 재빨리 판단한다.  
테스트 코드를 읽기가 사뭇 즐거워진다.  
<Br>

## 테스트 당 assert 하나? <a name = "8"></a>
JUnit으로 테스트 코드를 짤 때 함수마다 assert를 단 하나만 사용해야 한다고 주장하는 학파가 있다.  
가혹하다 여길지 모르지만 확실히 장점이 있다.  
assert가 하나라면 결론이 하나기 때문에 코드를 이해하기 빠르고 쉽다.  
기존에 xml인지 확인하고, 특정 문자열이 포함되어 있는가를 테스트하는 코드가 있다고 했을 때 이를 두 개로 쪼개 각자가 assert를 수행하도록 하면 테스트 당 하나의 assert를 만들 수 있다.  
```java
public void testGetPageHierarchyAsXml() throws Exception { 
  givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
  
  whenRequestIsIssued("root", "type:pages");
  
  thenResponseShouldBeXML(); 
}

public void testGetPageHierarchyHasRightTags() throws Exception { 
  givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
  
  whenRequestIsIssued("root", "type:pages");
  
  thenResponseShouldContain(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
  ); 
}
```
하지만 이렇게 되면 중복되는 코드가 많아진다.  
해당 테스트만 별도의 클래스로 분리해 @Before 함수에 given/when 부분을 담아주거나 또는 부모 클래스를 만들어 given/when을 넣어두고 자식에서 then 부분만 실행하면 중복 코드를 줄일 수는 있다.  
하지만 이는 배보다 배꼽이 더 크다.  
따라서 __저자는 때로 주저 없이 함수 하나에 여러 assert문을 넣는 것도 좋으나 되도록이면 assert문의 개수는 최대한 줄여야 한다고 말한다.__  

### 테스트 당 개념 하나 <a name = "9"></a>
어쩌면 __테스트 함수마다 한 개념만 테스트하라__ 라는 규칙이 더 낫겠다.  
```java
/**
 * addMonth() 메서드를 테스트하는 장황한 코드
 */
public void testAddMonths() {
  SerialDate d1 = SerialDate.createInstance(31, 5, 2004);

  SerialDate d2 = SerialDate.addMonths(1, d1); 
  assertEquals(30, d2.getDayOfMonth()); 
  assertEquals(6, d2.getMonth()); 
  assertEquals(2004, d2.getYYYY());
  
  SerialDate d3 = SerialDate.addMonths(2, d1); 
  assertEquals(31, d3.getDayOfMonth()); 
  assertEquals(7, d3.getMonth()); 
  assertEquals(2004, d3.getYYYY());
  
  SerialDate d4 = SerialDate.addMonths(1, SerialDate.addMonths(1, d1)); 
  assertEquals(30, d4.getDayOfMonth());
  assertEquals(7, d4.getMonth());
  assertEquals(2004, d4.getYYYY());
}
```
위 코드는 독자적인 개념 3개를 테스트하므로 독자적인 테스트 세 개로 쪼개야 마땅하다.  
+ (5월처럼) 31일로 끝나는 달의 마지막 날짜가 주어지는 경우
    - (6월처럼) 30일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되어서는 안 된다.
    - 두 달을 더하면 그리고 두 번째 달이 31일로 끝나면 날짜는 31일이 되어야 한다.
+ (6월처럼) 30일로 끝나는 달의 마지막 날짜가 주어지는 경우
    - 31일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되면 안 된다.

개념들을 이렇게 정리해 표현하면 장황한 코드 속에 여러 개념을 테스트하고 있음을 알 수 있다.  
이 경우 assert 문이 여럿이라는 사실이 문제가 아니라, 한 테스트 함수에서 여러 개념을 테스트한다는 사실이 문제다.  
__그러므로 가장 좋은 규칙은 "개념 당 assert 문 수를 최소로 줄여라"와 "테스트 함수 하나는 개념 하나만 테스트하라"이다.__  

<br>

## F.I.R.S.T <a name = "10"></a>
깨끗한 테스트는 다음 다섯 가지 규칙을 따른다.
+ F : Fast
    - 테스트가 느리면 자주 돌릴 엄두를 못 낸다. 따라서 테스트는 빨라야 한다.
+ I : Independent
    - 각 테스트는 서로 의존하면 안 된다. 
    - 각 테스트는 독립적으로 그리고 어떤 순서로 실행해도 괜찮아야 한다.
    - 서로 의존하게 되면 하나가 실패할 때 나머지도 잇달아 실패하므로 원인을 찾기 어렵다.
+ R : Repeatable
    - 테스트는 어떤 환경에서도 반복 가능해야 한다.
    -  테스트가 돌아가지 않는 환경이 하나라도 있다면 테스트가 실패한 이유를 둘러댈 변명이 생기고, 환경이 지원되지 않기에 테스트를 수행하지 못하는 상황에 직면한다.
+ S : Self-Validating
    - 테스트는 부울(bool) 값으로 결과를 내야 한다. (성공 혹은 실패)
    - 통과 여부를 알려고 추가적인 작업을 하면 안 된다.
+ T : Timely
    - 테스트는 적시에 작성해야 한다.
    - 단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다.
    - 실제 코드를 구현한 다음에 테스트 코드를 만들면 실제 코드가 테스트하기 어렵거나 테스트 자체가 불가능하도록 설계될 수도 있다.



<br>

## 결론 <a name = "11"></a>
테스트 코드는 실제 코드만큼이나 프로젝트 건강에 중요하다.  
테스트 코드는 실제 코드의 유연성, 유지보수성, 재사용성을 보존하고 강화하기 때문이다.  
그러므로 테스트 코드는 지속적으로 깨끗하게 관리하자.  
표현력을 높이고 간결하게 정리하자.  
테스트 API를 구현해 도메인 특화 언어(Domain Specific Language, DSL)를 만들자.  
그러면 그만큼 테스트 코드를 짜기가 쉬워진다.  
테스트 코드가 방치되어 망가지면 실제 코드도 망가진다.  
테스트 코드를 깨끗하게 유지하자.