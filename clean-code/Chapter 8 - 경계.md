## 목차
- [Intro](#1)
- [외부 코드 사용하기](#2)
- [경계 살피고 익히기](#3)
    - [log4j 학습 테스트](#4)
    - [학습 테스트는 공짜 이상이다](#5)
- [아직 존재하지 않는 코드 사용하기](#6)
- [깨끗한 경계](#7)

---


## Intro <a name = "1"></a>
시스템에 들어가는 모든 소프트웨어를 직접 개발하는 경우는 드물다.  
어떤 식으로든 외부 코드를 사용하게 되고 우리는 코드에 깔끔하게 통합해야만 한다.  
<br>

## 외부 코드 사용하기 <a name = "2"></a>
패키지 제공자나 프레임워크 제공자는 적용성을 최대한 넓히려 애쓰는 반면에 사용자는 자신이 요구에 집중하는 인터페이스만을 바란다.  
Map으로 예를 들어 보자.  
Map은 다양한 인터페이스로 수많은 기능을 제공한다.  
Map이 제공하는 기능과 유연성은 확실히 유용하지만 위험도 크다.  
프로그램에서 Map을 만들어 여기저기 넘긴다고 하면 어디서든지, 누구나 clear을 통해 삭제가 가능하다.  
```java
// bad
Map sensors = new HashMap();
Sensor s = (Sensor)sensors.get(sensorId);

Map<String, Sensor> sensors = new HashMap<Sensor>();
Sensor s = sensors.get(sensorId);
```
Map은 기본적으로 Object를 반환하기 때문에 제네릭을 사용하지 않으면 지속적인 형변환이 필요하다.  
하지만 제네릭을 사용한다고 하더라도 사용자에게 필요하지않는 기능까지 제공한다는 사실은 변함없다.  
이를 개선해보자.  

```java
public class Sensors {

    private Map sensors = new HashMap();
    
    public Sensor getById(String id) {
        return (Sensor)sensors.get(id);
    }
}
```
Map을 Sensors 안으로 숨겼다.  
Sensors 클래스는 프로그램에 필요한 인터페이스만 제공한다.  
클래스 안에서 Map의 객체 유형을 관리하고 변환하기 때문에 프로그램에 필요한 인터페이스만 제공한다.  
이는 Map 클래스를 사용할 때마다 위와 같이 캡슐화하라는 소리가 아니다.  
Map(혹은 유사한 경계 인터페이스를) 여기저기 넘기지 말라는 말이다.  
Map과 같은 경계 인터페이스를 사용할 때는 이를 이용하는 클래스나 클래스 계열 밖으로 노출하지 않도록 주의하고 Map을 인수로 넘기거나 반환하지 않도록 한다.  
<br>

## 경계 살피고 익히기 <a name = "3"></a>
서드파티 코드를 사용할 때, 적어도 우리가 사용할 코드에 대해서는 테스트하는 것이 바람직하다.  
외부 코드를 익히고 통합하기는 어렵다.  
따라서 곧바로 우리쪽 코드를 작성해 외부 코드를 호출하는 대신 먼저 간단한 테스트 케이스를 작성해 외부 코드를 익히는 것이 좋다.  
이를 __학습 테스트__ 라고 한다.  

### log4j 학습 테스트 <a name = "4"></a>
로그 기능을 직접 구현하는 대신 아파치의 log4j 패키지를 사용한다고 가정하자.  
```java
// 1.
// 우선 log4j 라이브러리를 다운받자.
// 고민 많이 하지 말고 본능에 따라 "hello"가 출력되길 바라면서 아래의 테스트 코드를 작성해보자.
@Test
public void testLogCreate() {
    Logger logger = Logger.getLogger("MyLogger");
    logger.info("hello");
}

// 2.
// 위 테스트는 "Appender라는게 필요하다"는 에러를 뱉는다.
// 조금 더 읽어보니 ConsoleAppender라는게 있는걸 알아냈다.
// 그래서 ConsoleAppender라는 객체를 만들어 넣어줘봤다.
@Test
public void testLogAddAppender() {
    Logger logger = Logger.getLogger("MyLogger");
    ConsoleAppender appender = new ConsoleAppender();
    logger.addAppender(appender);
    logger.info("hello");
}

// 3.
// 위와 같이 하면 "Appender에 출력 스트림이 없다"고 한다.
// 이상하다. 가지고 있는게 이성적일것 같은데...
// 구글의 도움을 빌려, 다음과 같이 해보았다.
@Test
public void testLogAddAppender() {
    Logger logger = Logger.getLogger("MyLogger");
    logger.removeAllAppenders();
    logger.addAppender(new ConsoleAppender(
        new PatternLayout("%p %t %m%n"),
        ConsoleAppender.SYSTEM_OUT));
    logger.info("hello");
}

// 성공했다. 하지만 ConsoleAppender를 만들어놓고 ConsoleAppender.SYSTEM_OUT을 받는건 이상하다.
// 그래서 빼봤더니 잘 돌아간다.
// 하지만 PatternLayout을 제거하니 돌아가지 않는다.
// 그래서 문서를 살펴봤더니 "ConsoleAppender의 기본 생성자는 설정되지 않은 상태"란다.
```
```java
// 조금 더 구글링, 문서 읽기, 테스트를 거쳐 log4j의 동작법을 알아냈고 그것을 간단한 유닛테스트로 기록했다.
// 이제 이 지식을 기반으로 log4j를 래핑하는 클래스를 만들수 있다.
// 나머지 코드에서는 log4j의 동작원리에 대해 알 필요가 없게 됐다.

public class LogTest {
    private Logger logger;
    
    @Before
    public void initialize() {
        logger = Logger.getLogger("logger");
        logger.removeAllAppenders();
        Logger.getRootLogger().removeAllAppenders();
    }
    
    @Test
    public void basicLogger() {
        BasicConfigurator.configure();
        logger.info("basicLogger");
    }
    
    @Test
    public void addAppenderWithStream() {
        logger.addAppender(new ConsoleAppender(
            new PatternLayout("%p %t %m%n"),
            ConsoleAppender.SYSTEM_OUT));
        logger.info("addAppenderWithStream");
    }
    
    @Test
    public void addAppenderWithoutStream() {
        logger.addAppender(new ConsoleAppender(
            new PatternLayout("%p %t %m%n")));
        logger.info("addAppenderWithoutStream");
    }
}
```
<br>

### 학습 테스트는 공짜 이상이다 <a name = "5"></a>
+ 비용이 들지 않는다.
+ 메인 로직에 영향을 주지 않으며 서드파티 코드를 이해할 수 있다.
+ 서드파티 코드가 바뀔 경우 Learning test를 돌려 아직 우리가 필요한 기능이 잘 동작하는지 확인할 수 있다.

<Br>

## 아직 존재하지 않는 코드 사용하기 <a name = "6"></a>
아직 개발되지 않은 모듈이 필요한데 기능은 커녕 인터페이스조차 구현되지 않은 경우가 있을 수 있다.  
여기서 저자의 경험적 예시가 등작한다.  
저자는 무선통신 시스템을 구축하는 프로젝트를 하고 있었고 그 팀 안의 하부팀으로 송신기를 담당하는 팀이 있었는데 나머지 팀원들은 송신기에 대한 지식이 거의 없었다.  
송신기 팀은 아직 인터페이스도 정의하지 못한 상태였다.  
하지만 우리는 이러한 제약 때문에 구현이 늦춰지는걸 원치 않았기에 송신기 팀이 제공하는 걸 기다리는 대신 원하는 기능을 정의하고 인터페이스를 만들었다.  
이후 송신기 팀에서 기능을 제공했을 때는 어뎁터 클래스를 구현해 간극을 메웠다.  
<br>

## 깨끗한 경계 <a name = "7"></a>
소프트웨어 설계가 우수하다면 변경하는데 많은 투자와 재작업이 필요하지 않다.  
이를 위해서는 경계에 위치한 코드는 깔끔히 분리해야 한다.  
통제가 불가능한 외부 패키지에 의존하는 대신 통제가 가능한 우리 코드에 의존하는 편이 훨씬 좋다.  
이를 위해서는 외부 패키지를 호출하는 코드를 가능한 줄여 경계를 관리해야 한다.  
Map에서 보았듯이, 새로운 클래스로 경계를 감싸거나 어댑터 패턴을 사용해 우리가 원하는 인터페이스를 패키지가 제공하는 인터페이스로 변환하면 된다.  



