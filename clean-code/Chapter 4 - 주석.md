## 목차
- [주석은 나쁜 코드를 보완하지 못한다](#1)
- [코드로 의도를 표현하라](#2)
- [좋은 주석](#3)
    - [법적인 주석](#4)
    - [정보를 제공하는 주석](#5)
    - [의도를 설명하는 주석](#6)
    - [의미를 명료하게 밝히는 주석](#7)
    - [결과를 경고하는 주석](#8)
    - [TODO 주석](#9)
    - [중요성을 강조하는 주석](#10)
- [나쁜 주석](#11)
    - [주절거리는 주석](#12)
    - [같은 이야기를 중복하는 주석](#13)
    - [오해할 여지가 있는 주석](#14)
    - [의무적으로 다는 주석](#15)
    - [이력을 기록하는 주석](#16)
    - [있으나 마나 한 주석](#17)
    - [함수나 변수로 표현할 수 있다면 주석을 달지 마라](#18)
    - [위치를 표시하는 주석](#19)
    - [닫는 괄호에 다는 주석](#20)
    - [공로를 돌리거나 저자를 표시하는 주석](#21)
    - [주석으로 처리한 코드](#22)
    - [HTML 주석](#23)
    - [전역 정보](#24)
    - [너무 많은 정보](#25)
    - [모호한 관계](#26)
    - [비공개 코드에서 javadocs](#27)

---


> 나쁜 코드에 주석을 달지 마라. 새로 짜라.
> - 브라이언 W. 커니핸, P.J. 플라우거

주석은 필요악이다.  
프로그래밍 언어를 사용해 의도를 표현할 능력이 있다면, 주석을 거의 필요하지 않다.  
우리는 코드로 의도를 표현하지 못해 즉, 실패를 만회하기 위해 주석을 사용할 뿐이다.  
<Br>

## 주석은 나쁜 코드를 보완하지 못한다. <a name = "1"></a>
코드에 주석을 추가하는 일반적인 이유는 코드 품질이 나쁘기 때문이다.  
표현력이 풍부하고 깔끔하며 주석이 거의 없는 코드가, 복잡하고 어수선하며 주석이 많이 달린 코드보다 훨씬 좋다.  
__자신이 저지른 난장판을 주석으로 설명하려 애쓰는 대신 그 난장판을 깨끗이 치우는데 시간을 보내라!__  
<br>

## 코드로 의도를 표현하라! <a name = "2"></a>
```java
// bad
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다. 
if ((emplotee.flags & HOURLY_FLAG) && (employee.age > 65)

// good
if (employee.isEligibleForFullBenefits())
```
조금만 생각하면 주석 없이도 충분히 깔끔하게 표현할 수 있다.  
<br>

## 좋은 주석 <a name = "3"></a>
### 법적인 주석 <a name = "4"></a>
각 소스 파일 첫머리에 주석으로 들어가는 저작권 정보와 소유권 정보는 필요하고 타당하다.
```
// Copyright (C) 2003, 2004, 2005 by Object Montor, Inc. All right reserved.
// GNU General Public License
```

### 정보를 제공하는 주석 <a name = "5"></a>
```java
// kk:mm:ss EEE, MMM dd, yyyy 형식이다.
Pattern timeMatcher = Pattern.compile("\\d*:\\d*\\d* \\w*, \\w*, \\d*, \\d*");
```
위 주석은 정규표현식이 시각과 날짜를 뜻한다고 설명한다.  

### 의도를 설명하는 주석 <a name = "6"></a>
```java
// 스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다. 
for (int i = 0; i > 2500; i++) {
    WidgetBuilderThread widgetBuilderThread = 
        new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
    Thread thread = new Thread(widgetBuilderThread);
    thread.start();
}
```

### 의미를 명료하게 밝히는 주석 <a name = "7"></a>
모호한 인수나 반환값을 읽기 좋게 표현하면 이해하기 쉽지만, 표준 라이브러리나 변경하지 못하는 코드에 속한다면 의미를 명료하게 밝히는 주석이 유용하다.
```java
assertTrue(a.compareTo(a) == 0) // a == a
assertTrue(a.compareTo(b) != 0) // a!= b
```

### 결과를 경고하는 주석 <a name = "8"></a>
```java
// 여유 시간이 충분하지 않다면 실행하지 마십시오.
public void _testWithReallyBigFile() {
}
```
요즘에는 @Ignore 속성을 이용해 테스트 케이스를 꺼버리고 속성에 이유를 문자열로 이유를 작성한다.  

### TODO 주석 <a name = "9"></a>
```java
// TODO-MdM 현재 필요하지 않다.
// 체크아웃 모델을 도입하면 함수가 필요 없다.
protected VersionInfo makeVersion() throws Exception {
    return null;
}
```
TODO 주석은 앞으로 할 일을 남겨두면 편리하다.  
해당 위치에서 필요하긴 하지만 당장 구현하기 어려운 업무를 기술하거나 필요 없는 기능을 삭제하라는 알림, 봐달라는 요청 등 유용하게 사용된다.  
요증 IDE는 TODO 주석을 전부 찾아서 보여주는 기능을 제공하므로 잊어버릴 염려는 없다.  

### 중요성을 강조하는 주석 <a name = "10"></a>
```java
String listItemContent = match.group(3).trim();
// 여기서 trim은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
// 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 때문이다. 
new ListItemWidget(this, listItemContent, this.level + 1);
return buildList(text.substring(match.end()));
```
<br>

## 나쁜 주석 <a name = "11"></a>
### 주절거리는 주석 <a name = "12"></a>
```java
public void loadProperties() {
    try {
        String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE;
        FileInputStream propertiesStream = new FileInputStream(propertiesPath);
        loadedProperties.load(propertiesStream);
    } catch (IOException e) {
        // 속성 파일이 없다면 기본값을 모두 메모리로 읽어 들였다는 의미다. 
    }
}
```
catch 블록의 주석은 저자에게는 의미가 있겠지만, 독자에게는 의미가 없다.  
답을 알아내려면 다른 코드를 뒤져야할 수 밖에 없기 때문이다.  

### 같은 이야기를 중복하는 주석 <a name = "13"></a>
```java
// this.closed가 true일 때 반환되는 유틸리티 메서드다.
// 타임아웃에 도달하면 예외를 던진다. 
public synchronized void waitForClose(final long timeoutMillis) throws Exception {
    if (!closed) {
        wait(timeoutMillis);
        if (!closed) {
            throw new Exception("MockResponseSender could not be closed");
        }
    }
}
```
코드 내용을 그대로 해석해서 주석으로 달았다.  
자칫하면 코드보다 주석을 읽는 시간이 더 오래 걸린다.  

### 오해할 여지가 있는 주석 <a name = "14"></a>
위 코드의 주석을 다시 보자.  
실제로 코드상으로는 closed가 true로 변하는 순간 반환되지 않는다.  
타임아웃을 기다리는 코드가 있기 때문이다.  
즉, 엄밀한 주석이 아니고 오해의 여지가 있는 주석이다.  
주석에 담긴 '살짝 잘못된 정보'로 인해 어느 프로그래머가 경솔하게 함수를 호출해 자기 코드가 아주 느려진 이유를 못찾게 되는 것이다.  

### 의무적으로 다는 주석 <a name = "15"></a>
모든 함수에 Javadocs를 달거나 모든 변수에 주석을 달아야 한다는 규칙은 어리석기 그지없다.  
이런 주석은 코드를 복잡하게 만들며, 거짓말을 퍼뜨리고, 혼동과 무질서를 초래한다. 아래와 같은 주석은 아무 가치도 없다.
```java
/**
 *
 * @param title CD 제목
 * @param author CD 저자
 * @param tracks CD 트랙 숫자
 * @param durationInMinutes CD 길이(단위: 분)
 */
public void addCD(String title, String author, int tracks, int durationInMinutes) {
    CD cd = new CD();
    cd.title = title;
    cd.author = author;
    cd.tracks = tracks;
    cd.duration = durationInMinutes;
    cdList.add(cd);
}
```

### 이력을 기록하는 주석 <a name = "16"></a>
현재는 소스 코드 관리 시스템이 있으니 전혀 필요 없다.
```java
* 변경 이력 (11-Oct-2001부터)
* ------------------------------------------------
* 11-Oct-2001 : 클래스를 다시 정리하고 새로운 패키징
* 05-Nov-2001: getDescription() 메소드 추가
* 이하 생략
```

### 있으나 마나 한 주석 <a name = "17"></a>
```java
/*
 * 기본 생성자
 */
protected AnnualDateRule() {}
```

### 함수나 변수로 표현할 수 있다면 주석을 달지 마라 <a name = "18"></a>
```java
// bad
// 전역 목록 <smodule>에 속하는 모듈이 우리가 속한 하위 시스템에 의존하는가?
if (module.getDependSubsystems().contains(subSysMod.getSubSystem()))
```
```java
// good
ArrayList moduleDependencies = smodule.getDependSubSystems();
String ourSubSystem = subSysMod.getSubSystem();
if (moduleDependees.contains(ourSubSystem))
```
충분히 함수나 변수로 의미를 전달할 수 있다면 주석 대신 코드로 표현해야 한다.  

### 위치를 표시하는 주석 <a name = "19"></a>
```java
// Actions /////////////////////////////////////////////
```
특정 위치를 표시하려는 주석은 가독성만 낮추므로 제거해야 마땅하다.  

### 닫는 괄호에 다는 주석 <a name = "20"></a>
```java
tyr{
	while{

	} // while
} // try
```
중첩이 심하고 장황한 함수라면 의미가 있을지도 모르겠지만, 우리가 선호하는 작고 캡슐화된 함수에는 잡음일 뿐이다.  
닫는 괄호에 주석을 달아야겠다는 생각이 든다면 대신에 함수를 줄이도록 노력하자.  

### 공로를 돌리거나 저자를 표시하는 주석 <a name = "21"></a>
```java
/* 릭이 추가함 */
```
소스 코드 관리 시스템이 다 기억한다.  
따라서 위와 같은 주석은 소스 코드 관리 시스템에 저장하는 편이 좋다.  

### 주석으로 처리한 코드 <a name = "22"></a>
```java
this.bytePos = writeBytes(pngIdBytes, 0);
//hdrPos = bytePos;
writeHeader();
writeResolution();
//dataPos = bytePos;
if (writeImageData()) {
    wirteEnd();
    this.pngBytes = resizeByteArray(this.pngBytes, this.maxPos);
} else {
    this.pngBytes = null;
}
return this.pngBytes;
```
사람들은 주석으로 처리된 코드를 지우기 주저한다.  
하지만 소스 코드 관리 시스템이 우리를 대신해 코드를 기억해준다.  
이제는 의미가 없어졌으므로 삭제하라.  

### HTML 주석 <a name = "23"></a>
HTML 주석은 혐오 그 자체다.  
HTML 주석은 주석을 읽기 쉬워야 하는 편집기/IDE에서 조차 읽기 어렵다.  

### 전역 정보 <a name = "24"></a>
주석을 달아야 한다면 근처에 있는 코드만 기술하라.  
코드 일부에 주석을 달면서 시스템의 전반적인 정보를 기술하지 마라.  
```java
/**
 * 적합성 테스트가 동작하는 포트: 기본값은 <b>8082</b>.
 *
 * @param fitnessePort
 */
public void setFitnessePort(int fitnessePort) {
    this.fitnewssePort = fitnessePort;
}
```
위의 함수 자체는 포트 기본값을 전혀 통제하지 못한다.  
그러니까 주석은 아래 함수가 아니라 시스템 어딘가에 있는 다른 함수를 설명한다는 뜻이다.  

### 너무 많은 정보 <a name = "25"></a>
주석에다 흥미로운 역사나 관련 없는 정보를 장황하게 늘어놓지 마라.  

### 모호한 관계 <a name = "26"></a>
주석과 주석이 설명하는 코드는 둘 사이 관계가 명백해야 한다.  
이왕 공들여 주석을 달았다면 적어도 독자가 주석과 코드를 읽어보고 무슨 소린지 알아야 한다.  
주석 자체가 다시 설명을 요구하면 그것은 잘못된 주석이다.  

### 비공캐 코드에서 javadocs <a name = "27"></a>
공개 API는 Javadocs가 유용하지만 공개하지 않을 코드라면 javadocs는 쓸모가 없다.  
코드만 보기 싫고 산만해질 뿐이다.




