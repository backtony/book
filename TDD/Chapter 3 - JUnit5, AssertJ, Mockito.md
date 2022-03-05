
## JUnit 5
### 모듈 구성
JUnit 5는 크게 세 개의 요소로 구성된다.
+ JUnit 플랫폼
    - 테스팅 프레임워크를 구동하기 위한 런처와 테스트 엔진을 위한 API를 제공
+ JUnit 주피터
    - JUnit 5를 위한 테스트 API와 실행 엔진을 제공    
+ JUnit 빈티지
    - JUnit 3, 4로 작성된 테스트를 JUnit 5 플랫폼에서 실행하기 위한 모듈을 제공


### @Test
테스트로 사용할 클래스를 만들고 @Test 애노테이션을 메서드에 붙이면 JUnit을 이용한 테스트 코드를 작성하고 실행할 수 있다.
```java
public class CalculatorTest {

    @Test
    void plus() throws Exception{
        int result = 2+3;
        Assertions.assertEquals(5,result);
    }
}
```
테스트 클래스의 이름을 작성하는 특별한 규칙은 없지만 보통 다른 클래스와 구분을 쉽게하기 위해 Test를 접미사로 붙인다.

### 주요 단언 메서드
JUnit의 Assertions 클래스는 assertEquals() 메서드와 같이 값을 검증하기 위한 목적의 다양한 정적 메서드를 제공한다.  

메서드|설명
---|---
assertEquals(expected, actual)|실제 값이 기대하는 값과 같은지 검사
assertNotEquals(unexpected, actual)|실제 값이 특정 값과 같지 않은지 검사
assertSame(Object expected, Object actual)|두 객체가 동일한 객체인지 검사
assertNotSame(Object unexpected, Object actual)|두 객체가 동일하지 않은 객체인지 검사
assertTrue(boolean condition)|값이 true인지 검사
assertFalse(boolean condition)|값이 false인지 검사
assertNull(Object actual)|값이 null인지 검사
assertNotNull(Object actual)|값이 null이 아닌지 검사
assertThrows(Class\<T> expectedType, Executable executable)|executable을 실행한 결과로 지정한 타입의 익센셥이 발생하는지 검사
assertDoesNotThrow(Executable executable)|executable을 실행한 결과로 익셉션이 발생하지 않는지 검사
fail()|테스트를 실패 처리


```java
@Test
void test() throws Exception{
    Assertions.assertThrows(IllegalAccessException.class,
            () -> {
                AuthService authService = new AuthService();
                authService.authenticate(null, null);
            });
}
```
위처럼 알맞은 Exception이 터지는지 확인할 수 있다.
```java
 @Test
void test() throws Exception{
    IllegalAccessException thrown = Assertions.assertThrows(IllegalAccessException.class,
            () -> {
                AuthService authService = new AuthService();
                authService.authenticate(null, null);
            });
    Assertions.assertTrue(thrown.getMessage().contains("id"));
}
```
assertThrows는 해당 Exception을 반환하기 때문에 Exception을 가지고 추가적인 검증을 할 수 있다.  
assert 메서드는 실패하면 다음 코드는 실행하지 않고 바로 exception을 날린다.  
그런데 경우에 따라 일단 모든 검증을 수행하고 그중에 실패한 것이 있는지 확인하고 싶을 때가 있다.  
이때는 assertAll() 메서드를 사용한다.
```java
@Test
void test() throws Exception{
    Assertions.assertAll(
            () -> Assertions.assertEquals(),
            () -> Assertions.assertEquals(),
            ...
    );
}
```

### 테스트 라이프사이클
#### @BeforeEach, @AfterEach
JUnit은 각 테스트 메서드마다 다음 순서대로 코드를 실행한다.
1. 테스트 메서드를 포함한 객체 생성
2. @BeforeEach 애노테이션이 붙은 메서드 실행
3. @Test 애노테이션이 붙은 메서드 실행
4. @AfterEach 애노테이션이 붙은 메서드 실행

```java
public class LifeCycleTest {

    public LifeCycleTest() {
        System.out.println("new LiftCycleTest");
    }

    @BeforeEach
    void setUp(){
        System.out.println("setUp");
    }

    @Test
    void a(){
        System.out.println("A");
    }

    @Test
    void b(){
        System.out.println("B");
    }

    @AfterEach
    void tearDown(){
        System.out.println("tearDown");
    }
}

// 출력
new LiftCycleTest
setUp
A
tearDown
new LiftCycleTest
setUp
B
tearDown
```
출력을 보면 @Test 메서드를 실행할 때마다 객체를 새로 생성하고 테스트 메서드를 실행하기 전과 후에 @BeforeEach, @AfterEach 메서드를 실행한다는 것을 확인할 수 있다.  
@BeforeEach는 테스트를 실행하는데 필요한 준비 작업을 할 때 사용한다.  
@AfterEach는 테스트를 실행한 후에 정리할 것이 있을 때 사용한다.

#### @BeforeAll, @AfterAll
+ @BeforeAll
    - 한 클래스의 모든 테스트 메서드가 실행되기 전에 수행
    - 정적 메서드여야 한다.
+ @AfterAll
    - 한 클래스의 모든 테스트 메서드가 실행된 후
    - 정적 메서드여야 한다.

### 테스트 메서드 간 실행 순서 의존과 필드 공유하지 않기
```java
public class BadTest {

   private FileOperator op = new FileOperator();
   private static File file;
   
    @Test
    void fileCreationTest(){
       File createdFile = op.createFile();
       assertTrue(createdFile.length() > 0);
       this.file = createdFile;
   }

    @Test
    void readFileTest(){
        long data = op.readData(file);
        assertTrue(data > 0);
    }
}
```
이 코드는 첫 번째 Test에서 생성한 File을 보관하고 두 번째 테스트에서 사용하고 있다.  
테스트 메서드가 특정 순서대로 실행된다는 가정하에 테스트를 작성하면 안 된다.  
JUnit이 테스트 순서를 결정하긴 하지만 그 순서는 버전에 따라 달라질 수 있기 때문에 순서가 달라지면 실패하는 테스트는 작성하면 안 된다.  
각 테스트 메서드는 서로 독립적으로 동작해야 한다. 즉, 한 테스트 메서드의 결과에 따라 다른 테스트 메서드의 실행 결과가 달라지면 안 된다.  
그런 의미에서 테스트 메서드가 서로 필드를 공유한다거나 실행 순서를 가정하고 테스트를 작성하지 말아야 한다.

### @DisplayName, @Disbabled
+ @DisplayName
    - 애노테이션을 사용해 테스트에 표시 이름을 붙일 수 있다.  
    - 자바는 메서드 이름에 공백이나 특수 문자를 사용할 수 없기 때문에 메서드 이름만으로 테스트 내용을 설명하기 부족할 때 사용한다.
+ @Disabled
    - 특정 테스트를 실행하지 않고 싶을 때 사용한다.

```java
public class BadTest {

   private FileOperator op = new FileOperator();
   private static File file;

    @Test
    @DisplayName("파일 생성 테스트")
    void fileCreationTest(){
       File createdFile = op.createFile();
       assertTrue(createdFile.length() > 0);
       this.file = createdFile;
   }
    
   @Disabled
    @Test
    void readFileTest(){
        long data = op.readData(file);
        assertTrue(data > 0);
    }
}
```

### @EnabledOnOs, @DisabledOnOs
+ @EnabledOnOs : 테스트 메서드가 특정 운영체제에서만 동작해야 할 때 사용
+ @DisabledOnOs : 테스트 메서드가 특정 운영체제에서만 동작하지 말아야 할 때 사용

```java
public class Junit5Test {

    @Test
    @EnabledOnOs(OS.WINDOWS)
    void fileCreationTest(){
      ...
   }

    @Test
    @DisabledOnOs(OS.LINUX)
    void readFileTest(){
        ...
    }
}
```

### @EnableOnJre, @DisabledOnJre
자바 버전에 따라 테스트를 진행하고 싶을 때 사용한다.
```java
public class Junit5Test {

    @Test
    @EnabledOnJre(JRE.JAVA_8)
    void fileCreationTest(){
      ...
   }

    @Test
    @DisabledOnJre(JRE.JAVA_9)
    void readFileTest(){
        ...
    }
}
```

### @EnabledIfSystemProperty, @DisabledIfSystemProperty
시스템 프로퍼티 값을 비교하여 테스트 실행 여부를 결정한다.
```java
public class Junit5Test {

    @Test
    @EnabledIfSystemProperty(named = "java.vm.name", matches = ".*OpenJDK.*")
    void fileCreationTest(){
      ...
   }
}
```
named 속성은 시스템 프로퍼티의 이름을 지정하고 matches 속성에는 값의 일치 여부를 검사할 때 사용할 정규 표현식을 지정한다.

### @EnabledIfEnvironmentVariable
@EnabledIfSystemProperty과 사용 방식은 같고 차이점은 named 속성에 환경변수 이름을 사용한다는 것만 다르다.

### 태깅과 필터링
+ @Tag
    - 테스트에 태그를 달 때 사용하며 클래스와 테스트 메서드에 적용할 수 있다.
    - 메이븐이나 그레이들에서 실행할 테스트를 선택할 때 사용한다.
    - 규칙
        - null이나 공백이면 안 된다.
        - 좌우 공백을 제거한 뒤에 공백을 포함하면 안 된다.
        - ISO 제어 문자를 포함하면 안 된다.
        - , () & | ! 를 포함하면 안 된다.


```java
@Tag("integration")
public class Junit5Test {

    @Tag("very-slow")
    @Test
    void test(){
        int result = 4;
        Assertions.assertEquals(4,result);
   }
}
```
```groovy
// gradle
test {
    useJUnitPlatform {
        includeTags 'integration'
        excludeTags 'slow | very-slow'
    }
}
```
포함하거나 제외할 때 태그 식을 사용하는데 다음 연산을 사용해 조합한다.
+ ! : not 연산
+ & : and 연산
+ | : OR 연산


### 중첩 구성
@Nested 애노테이션을 사용하면 중첩 클래스에 테스트 메서드를 추가할 수 있다.

```java
public class Outer {
    
    @BeforeEach
    void outerBefore(){}
    
    @Test
    void outer() {}
    
    @AfterEach
    void outerAfter() {}
    
    @Nested
    class NestedA{
        @BeforeEach
        void nestedBefore(){}

        @Test
        void nested1() {}

        @AfterEach
        void nestedAfter() {}
    }
}
```
중첩 클래스의 nested1 메서드를 실행하면 실행 순서는 다음과 같다.
1. Outer 객체 생성
2. NestedA 객체 생성
3. outerBefore 메서드 실행
4. nestedBefore 메서드 실행
5. nested1 테스트 실행
6. nestedAfter 메서드 실행
7. outerAfter 메서드 실행

중첩된 클래스는 내부 클래스이므로 외부 클래스의 멤버에 접근할 수 있다.  
이 특징을 활용하면 상황별로 중첩 테스트 클래스를 분리해서 테스트 코드를 구성할 수 있다.

### @TempDir
파일과 관련된 테스트 코드를 만들다 보면 임시로 사용할 폴더가 필요할 때가 있다.  
테스트 시작 전에 임시로 만들고 끝나면 폴더를 삭제할 수 있지만 성가신 작업이다.  
@TempDir은 필드 또는 라이프사이클 관련 메서드나 테스트 메서드의 파라미터로 사용하면 JUnit은 임시 폴더를 생성하고 @TempDir을 붙인 필드나 파라미터에 임시 폴더 경로를 전달한다.  
File 타입이나 Path 타입에 적용할 수 있다.

```java
public class Junit5Test {
    
    @TempDir
    File tempFolder;
    
    @Test
    void test(){
        ...
   }
}
```
테스트 메서드를 실행하기 전에 @TempDir로 인해 임시 폴더가 생성되고 해당 정보가 tempFolder필드에 할당된다.  
필드에 적용하면 각 테스트 메서드를 실행할 때마다 임시 폴더를 생성한다.  
만약 특정 메서드에서만 임시 폴더를 생성해서 사용하고 싶다면 테스트 메서드의 파라미터로 사용하면 된다.
```java
public class Junit5Test {

    @Test
    void test(@TempDir Path tempFolder){
        ...
   }
}
```
당연히 테스트가 끝나면 임시 폴더는 삭제된다.  
<br>

특정 테스트 클래스 단위로 임시 폴더를 생성하고 싶다면 정적 필드에 @TempDir 애노테이션을 붙인다.  
테스트 클래스의 모든 테스트 메서드를 실행하기 전에 임시 폴더를 한 번만 생성하고 모든 테스트 메서드의 실행이 끝나고 난 뒤에 임시 폴더를 삭제한다.
```java
public class Junit5Test {

    @TempDir 
    static File tempFolder;
            
    @Test
    void test(){
        ...
   }
}
```
```java
public class Junit5Test {
    @BeforeAll
    static void setup(@TempDir File tempFolder){
        ...
    }
}
```
BeforeAll 메서드의 파라미터에 사용해도 된다.

### @Timeout 
JUnit 5.5부터 지원하는 애노테이션으로 테스트가 일정 시간 내에 실행되는지 검증할 수 있다.  
일정 시간 내에 완료되지 않으면 TimeoutException이 발생한다.

```java
public class Junit5Test {
    @Test
    @Timeout(value = 500, unit = TimeUnit.MILLISECONDS)
    void test(){
        ...
   }
}
```

## AssertJ
JUnit은 테스트 실행을 위한 프레임워크를 제공하지만 단언에 대한 표현력이 부족하다는 단점이 있다.
```java
assertTrue(id.contains("a"));
```
assertTrue는 인자가 true인지 검사한다. 이 코드는 id가 a를 포함하고 있는지 여부를 검사한다.  
하지만 단언 코드는 값이 true인지 여부를 확인하는 assertTrue를 사용하고 있어 실제 검사하는 내용을 표현하고 있지 않다.  
실패 시에도 true와 false를 보여줄 뿐 id가 a를 포함하고 있지 않아서 테스트에 실패했다는 사실을 알려주지 않는다.
```java
// assertJ
assertThat(id).contains("a");
```
assertJ를 사용하면 직관적으로 바로 이해할 수 있는 코드로 검사할 수 있고 예외도 더 자세히 알려준다.

### 기본 사용법
```java
assertThat(실제값).검증메서드(기댓값)
```
assertThat 메서드는 org.assertj.core.api.Assertions 클래스에 정적 메서드로 정의되어 있다.  
주요 타입별로 assertThat 메서드가 존재하며 타입에 따라 다른 검증 메서드를 제공한다.

### 검증 메서드
#### 기본 검증 메서드
+ isEqualTo(기댓값) : 값과 같은지 검증
+ isNotEqualTo(기댓값) : 값과 같지 않은지 검증
+ isNull() : null인지 검증
+ isNotNull() : null이 아닌지 검증
+ isIn(값 목록) : 값 목록에 포함되어 있는지 검증
+ isNotIn(값 목록) : 값 목록에 포함되어 있지 않은지 검증

isIn과 isNotIn의 값 목록은 가변 인자로 주거나 List와 같이 iterable을 구현하는 타입을 이용해서 전달한다.  
<br>

Comparable 인터페이스를 구현한 타입이거나 int, double 같은 숫자 타입의 경우 다음 메서드를 이용해 값을 검증할 수 있다.
+ isLessThan(값) : 값보다 작은지 검증
+ isLessThanOrEqualTo(값) : 값보다 작거나 같은지 검증
+ isGreaterThan(값) : 값보다 큰지 검증
+ isGreaterThanOrEqualTo(값) : 값보다 크거나 같은지 검증
+ isBetween(값1, 값2) : 값1과 값2 사이에 포함되는지 검증

<br>

Boolean과 boolean 타입을 위한 검증 메서드는 다음과 같다.
+ isTrue() : 값이 true인지 검증
+ isFalse() : 값이 false인지 검증

#### String 검증 메서드
+ contains(CharSequence ...values) : 인자로 지정한 문자열들을 모두 포함하고 있는지 검증
+ containsOnlyOnce(CharSequence sequence) : 해당 문자열을 딱 한 번만 포함하는지 검증
+ containsOnlyDigits() : 숫자만 포함하는지 검증
+ containsWhitespaces() : 공백 문자를 포함하는지 검증
+ containsOnlyWhitespaces() : 공백 문자만 포함하는지 검증
+ containsPattern(CharSequence regex) : 지정한 정규 표현식에 일치하는 문자를 포함하는지 검증
+ containsPattern(Pattern pattern) : 지정한 정규 표현식에 일치하는 문자를 포함하는지 검증
+ doesNotContain(CharSequence... values) : 인자로 지정한 문자열들을 모두 포함하고 있지 않은지 검증
+ doesNotContainAnyWhitespaces() : 공백 문자를 포함하고 있지 않은지 검증
+ doesNotContainOnlyWhitespaces() : 공백 문자만 포함하고 있지 않은지 검증
+ doesNotContainPattern(Pattern pattern) : 정규 표현식에 일치하는 문자를 포함하고 있지 않은지를 검증
+ doesNotContainPattern(CharSequence pattern) : 정규 표현식에 일치하는 문자를 포함하고 있지 않은지 검증
+ startsWith(CharSequence pattern) : 지정한 문자열로 시작하는지 검증
+ doesNotStartWith(CharSequence prefix) : 지정한 문자열로 시작하지 않는지 검증
+ endsWith(CharSequence suffix) : 지정한 문자열로 끝나는지 검증
+ doesNotEndWith(CharSequence suffix) : 지정한 문자열로 끝나지 않는지 검증

#### 숫자 검증 메서드
+ isZero(), isNotZero() : 0인지 아닌지 검증
+ isOne() : 1인지 검증
+ isPositive(), isNotPositive() : 양수인지 양수가 아닌지 검증
+ isNegative(), isNotNegative() : 음수인지 음수가 아닌지 검증

#### 날짜/시간 검증 메서드
+ isBefore(비교할 값) : 비교할 값보다 이전인지 검증
+ isBeforeOrEqualTo(비교할 값) : 비교할 값보다 이전이거나 같은지 검증
+ isAfter(비교할 값) : 비교할 값보다 이후인지 검증
+ isAfterOrEqualTo(비교할 값) : 비교할 값보다 이후이거나 같은지 검증

<br>

LocalDateTime, OffsetDateTime, ZonedDateTime 타입은 추가로 다음의 검증 메서드를 제공한다.
+ isEqualToIgnoringNanos(비교할 값) : 나노 시간을 제외한 나머지 값이 같은지 검증한다. 즉, 초 단위까지 값이 같은지 검증
+ isEqualToIgnoringSeconds(비교할 값) : 초 이하 시간을 제외한 나머지 값이 같은지 검증한다. 즉, 분 단위까지 값이 같은지 검증
+ isEqualToIgnoringMinutes(비교할 값) : 분 이하 시간을 제외한 나머지 값이 같은지 검증한다. 즉, 시 단위까지 값이 같은지 검증
+ isEqualToIgnoringHours(비교할 값) : 시 이하 시간을 제외한 나머지 값이 같은지 검증한다. 즉, 일 단위까지 값이 같은지 검증

#### 콜렉션 검증 메서드
+ hasSize(int expected) : 콜렉션 크기가 지정한 값과 같은지 검증
+ contains(E... values) : 콜렉션이 지정한 값을 포함하는지 검증
+ containsOnly(E... values) : 콜렉션이 지정한 값만 포함하는지 검증
+ containsAnyOf(E... values) : 콜렉션이 지정한 값 중 일부를 포함하는지 검증
+ containsOnlyOnce(E... values) : 콜렉션이 지정한 값을 한 번만 포함하는지 검증

<br>

Map을 위한 추가 검증 메서드는 다음과 같다.
+ containsKey(K key) : 지정한 키를 포함하는지 검증
+ containsKeys(K... keys) : 지정한 키들을 포함하는지 검증
+ containsOnlyKeys(K... keys) : 지정한 키만 포함하는지 검증
+ doesNotContainKeys(K... keys) : 지정한 키들을 포함하지 않는지 검증
+ containsValues(VALUE... values) : 지정한 값들을 포함하는지 검증
+ contains(Entry\<K,V>... values) : 지정한 엔트리를 포함하는지 검증

#### Exception 검증 메서드
```java
// exception이 터지는지 검증
assertThatThrownBy(() -> readFile(new File("test.txt")));

// 지정한 exception이 터지는지 검증
assertThatThrownBy(() -> readFile(new File("test.txt"))).isInstanceOf(IOException.class);

// 지정한 exception이 터지는지 검증
assertThatExceptionOfType(IOException.class)
    .isThrownBy(() -> {
        readFile(new File("test.txt"));
    })

// 3번째 예시 축약 버전
assertThatIOException(
        .isThrownBy(() -> {
            readFile(new File("test.txt"));
        })
)
```
마지막 예시에서 타입이 합쳐진 것 볼 수 있는데 assertThatNullPointerException, assertThatIllegalArgumentException, assertThatIllegalStateException 메서드가 존재한다.

```java
// 예외를 발생시키지 않는지 검증
assertThatCode(() -> {
    readFile(new File("text.txt"));
}).doesNotThrowAnyException();
```


### 모아서 검증하기
org.assertj.core.api.SoftAssertions는 JUnit 5의 assertAll과 비슷하게 여러 검증을 한 번에 수행하고 싶을 때 사용한다.
```java
// 객체 생성
SoftAssertions soft = new SoftAssertions();

// 검증 메서드 저장
soft.assertThat(1).isBetween(0,2);
soft.assertThat(1).isGreaterThen(2);

// 검증 실행
soft.assertAll();
```
SoftAssertions 객체가 제공하는 assertThat 메서드는 해당 시점에 바로 검증을 수행하지 않는다.  
가독성이 조금 떨어지는 느낌이다. 다음과 같이 축약해서 사용할 수 있다.
```java
SoftAssertions.assertSoftly(soft -> {
    soft.assertThat(1).isBetween(0,2);
    soft.assertThat(1).isGreaterThen(2);
});
```

### 설명 달기
as 메서드는 테스트에 설명을 붙인다.
```java
assertThat(id).as("ID 검사").isEqualTo("ABC");
```
실패시 실패 메시지에 as 메서드로 지정한 설명 문구가 표시된다.  
문자열 포맷을 사용할 수 있기 때문에 한 테스트 메서드에서 다수의 검증 메서드를 실행할 때 유용하게 사용할 수 있다.
```java
List<Integer> result = getResults();
List<Integer> expected = getExpected();
SoftAssertions soft = new SoftAssertions();
for(int i=0; i< 100; i++){
    soft.assertThat(result.get(i)).as("result[%d]",i).isEqualTo(expected.get(i));
}
soft.assertAll();
```
이렇게 실행하면 몇 번째 검증에서 실패했는데 간단하게 찾을 수 있다.  
as 대신 describedAs를 사용해도 된다. 똑같다.

## Mockito
Mockito는 모의 객체 생성, 검증, 스텁을 지원하는 프레임워크이다.  
```java
public class MockitoTest {
    @Test
    void mockTest(){
        GameNumGen genMock = Mockito.mock(GameNumGen.class); 
    }
}
```
기본적으로 Mockito.mock() 메서드를 사용하면 모의 객체를 생성한다.  
아래와 같이 더 간단하게 사용할 수도 있다.
```java
@ExtendWith(MockitoExtension.class)
public class MockitoTest {

    @Mock
    GameNumGen genMock;
    
    @Test
    void mockTest(){
        ..
    }
}
```

### 스텁 설정
BDDMockito 클래스를 이용해 모의 객체에 스텁을 구성할 수 있다.
```java
@ExtendWith(MockitoExtension.class)
public class MockitoTest {
    @Mock
    GameNumGen genMock;
    
    @Test
    void mockInterface() throws Exception {
        // call 메서드 호출 시 10을 반환하라
        BDDMockito.given(genMock.call()).willReturn(10);        

        // 반환값이 존재하는 경우 Exception을 날리도록 할 경우 사용
        BDDMockito.given(genMock.call()).willThrow(new IllegalArgumentException());        

        // 반환값이 void인 경우 Exception을 날리도록 할 경우
         BDDMockito.willThrow(IllegalAccessException.class)
                .given(genMock)
                .call();

        ...
    }
}
```

### 인자 매칭
모의 객체를 사용해서 인자를 구체적으로 매칭할 경우 테스트가 조금만 수정되어도 실패하기 때문에 보통 임의의 값에 일치하도록 설정한다.

```java
BDDMockito.given(genMock.call(any())).willReturn(10);
```

+ any() : 임의 타입에 대한 일치
+ anyString() : 문자열에 대한 임의 값 일치
+ anyInt(), anyBoolean(), anyLong() .. : 기본 데이터 타입에 대한 임의 값 일치
+ anyList(), anySet(), anyMap(), anyCollection() : 임의 콜렉션에 대한 일치
+ matches(String), matches(Pattern) : 정규표현식을 이용한 String 값 일치 여부
+ eq(값) : 특정 값과 일치 여부

스텁을 설정할 메서드의 인자가 두 개 이상인 경우 주의할 점이 있다.  
```java
// 오류 발생
BDDMockito.given(genMock.call(any(),"123")).willReturn(10);

// 해결
BDDMockito.given(genMock.call(any(),ArgumentMatchers.eq("123")).willReturn(10);
```
언뜻 보기에는 문제가 없어보이지만 실제로는 문제가 있다.  
ArgumentMatchers의 any() 등의 메서드는 내부적으로 인자의 일치 여부를 판단하기 위해 ArugumentMatcher를 등록한다.  
__Mockito는 한 인자라도 ArgumentMatcher를 사용해서 설정한 경우 모든 인자를 ArgumentMatcher를 이용해서 설정하도록 하고 있다.__  
임의의 값과 일치하는 인자와 정확하게 일치하는 인자를 함께 사용하고 싶다면 __ArugmentMatchers.eq()__ 를 사용해야 한다.

### 행위 검증
모의 객체의 역할 중 하나는 실제로 모의 객체가 호출되었는지 검증하는 것이다.
```java
// genMock의 generate 메서드가 GameLevel.EASY 인자로 한 번만 호출되었는지 확인
then(genMock).should(only()).generate(GameLevel.EASY);
```

+ only() : 한 번만 호출
+ times(int) : 지정한 횟수만큼 호출
+ never() : 호출하지 않음
+ atLeast(int) : 적어도 지정한 횟수만큼 호출
+ atLeastOnce() : atLeast(1)과 동일
+ atMost(int) : 최대 지정한 횟수만큼 호출

### 인자 캡쳐
단위 테스트를 실행하다보면 모의 객체를 호출할 때 사용한 인자를 검증해야 할 때가 있다.  
String이나 int 같은 타입은 쉽게 검증할 수 있지만 많은 속성을 가진 객체는 쉽게 검증하기 어렵다.  
Mockito의 ArgumentCaptor를 사용하면 메서드 호출 여부를 검증하는 과정에서 실제 호출할 때 전달한 인자를 보관할 수 있다.  
```java
@Test
void noDupId_RegisterSuccess() {
    userRegister.register("id", "pw", "email");

    ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);
    // register 메서드에서 mockRepository.save 메서드 호출 시 사용한 User 인스턴스 캡쳐
    BDDMockito.then(mockRepository).should().save(captor.capture());

    // 캡쳐한 인스턴스 가져오기
    User savedUser = captor.getValue();
    assertEquals("id", savedUser.getId());
    assertEquals("email", savedUser.getEmail());
}
```