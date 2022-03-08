

## 테스트 코드와 유지보수
빠른 서비스 출시를 위해 CI/CD를 도입하는 곳이 증가하고 있다.  
지속적으로 코드를 통합하고 출시 가능한 상태로 만들고 배포하려면 새로 추가한 코드가 기존 기능을 망가뜨리지 않는지 확인할 수 있어야 하며 이런 이유로 자동화 테스트는 CI/CD의 필수 요건 중 하나이다.  
TDD를 하는 과정에서 작성한 테스트 코드는 CI/CD에서 자동화 테스트로 사용되어 버그가 배포되는 것을 막아주고 이는 소프트웨어 품질이 저하되는 것을 방지한다.  
테스트 코드는 그 자체로 코드이기 때문에 제품 코드와 동일하게 유지보수 대상이 된다.  
테스트 코드 자체의 유지 보수성이 좋아야 지속적으로 테스트룰 작성하게 되고 결과적으로 소프트웨어의 품질이 떨어지는 것도 막을 수 있다.  

## 변수나 필드를 사용해서 기댓값 표현하지 않기
```java
public class Junit5Test {
    @Test
    void dateFormat() {
        LocalDate date = LocalDate.of(1945, 8, 15);
        String dateStr = formatDate(date);
        assertEquals(date.getYear() + "년 " + 
                date.getMonthValue() + "월 " + 
                date.getDayOfMonth()+ "일 ", dateStr);
    }
}
```
기댓값이 너무 난잡하다. date의 API를 실수로 잘못 사용할 수도 있다.  
```java
public class Junit5Test {
    @Test
    void dateFormat() {
        LocalDate date = LocalDate.of(1945, 8, 15);
        String dateStr = formatDate(date);
        assertEquals("1945년 8월 15일", dateStr);
    }
}
```
문자열 값을 사용하면 가독성도 좋고 API를 잘못 사용할 일도 없다.
<br>

이번엔 변수로 사용한 코드를 보자.
```java
public class Junit5Test {

    // 변수 선언
    ...

    @Test
    void dateFormat() {
        
        // 위에서 선언한 변수를 사용하여 환경 만들기
        ...

        assertAll(
                () -> assertEquals(respondentId, savedAnswer.getRespondentId()),
                () -> assertEquals(answer.size(), savedAnswer.getAnswers().size()),
                () -> assertEquals(answer.get(0), savedAnswer.getAnswers().get(0)),
                ...
        );
    }
}
```
테스트가 첫 번째에서 실패했다면 기대한 값이 왜 틀렸는지 필드를 다시 위로 올라가서 확인해야 한다.  
테스트에 통과하더라도 테스트 코드를 처음 보는 사람은 변수와 필드를 오가며 테스트 코드를 이해해야 한다.
```java
public class Junit5Test {
    @Test
    void dateFormat() {
        
        // 변수 선언을 밖에서 하지 않고 여기서 하고 상황 만들기
        ...

        assertAll(
                () -> assertEquals(100L, savedAnswer.getRespondentId()),
                () -> assertEquals(4, savedAnswer.getAnswers().size()),
                () -> assertEquals(2, savedAnswer.getAnswers().get(0)),
                ...
        );
    }
}
```
전과 비교하면 가독성이 좋아져서 테스트 코드를 더욱 쉽게 파악할 수 있다.  
그리고 전에는 변수를 바깥에 두고 그걸 사용해서 환경을 만드느라 위, 아래 왔다갔다 하면서 확인해야 했지만 안쪽에서 선언하고 사용함으로 인해 왔다갔다 할 필요도 없어졌다.

## 두 개 이상을 검증하지 않기
한 테스트에 가능한 많은 단언을 하려고 시도하면 그 과정에서 서로 다른 검증을 섞는 경우가 있다.
```java
public class Junit5Test {
    
    @DisplayName("같은 ID가 없으면 가입에 성공하고 메일을 전송")
    @Test
    void registerAndSendMail() {
        // 검증1 : 회원 데이터가 올바르게 저장되었는지 검증
        ...

        // 검증2 : 이메일 발송을 요청했는지 검증
        ...
    }
}
```
회원 가입 이후 데이터가 올바르게 저장되는지 검증하고 두 번째는 이메일 발송을 올바르게 요청하는지 검증한다.  
테스트가 잘못된 것은 아니지만 집중도가 떨어진다.  
```java
public class Junit5Test {
    
    @DisplayName("같은 ID가 없으면 가입에 성공")
    @Test
    void noDupId_register_success() {
        // 회원 데이터가 올바르게 저장되었는지 검증
        ...
    }

    @DisplayName("가입하면 메일 전송")
    @Test
    void noDupId_register_success() {
        // 회원 가입 코드가 있지만 메일을 전송하는 것만 검증
        ...
    }
}
```
테스트가 반드시 한 가지만 검증해야 하는 것은 아니지만, 검증 대상이 명확하게 구분된다면 테스트 메서드도 구분하는 것이 유지보수에 유리하다.

## 정확하게 일치하는 값으로 모의 객체 설정하지 않기
```java
@DisplayName("약한 암호면 가입 실패")
@Test
void weakPassword() {
    BDDMockito.given(mockPasswordChecker.checkPasswordWeak("pw")).willReturn(true);

    assertThrows(WeakPasswordException.class, () -> {
        userRegister.register("id", "pw", "email");
    });
}
```
모의 객체를 사용하여 pw 문자열은 약한 암호로 처리하도록 지정했다.  
이 테스트는 pw 대신 pwa를 넣는 등의 작은 변화에도 실패한다.  
이 코드는 약한 암호인 경우 원하는대로 동작하는지 확인하기 위한 테스트이지 pw나 pwa가 약한 암호인지 확인하는 테스트가 아니다.
```java
@DisplayName("약한 암호면 가입 실패")
@Test
void weakPassword() {
    BDDMockito.given(mockPasswordChecker.checkPasswordWeak(Mockito.anyString())).willReturn(true);

    assertThrows(WeakPasswordException.class, () -> {
        userRegister.register("id", "pw", "email");
    });
}
```
임의의 String 값에 일치한다는 것으로 코드를 수정해주도록 한다. 이는 모의 객체를 호출했는지 여부를 확인하는 경우에도 동일하다.  
__즉, 모의 객체는 가능한 범용적인 값을 사용해서 기술해야 한다.__  

## 과도하게 구현 검증하지 않기
테스트 코드를 작성할 때 주의할 점은 테스트 대생의 내부 구현을 검증하는 것이다.  
모의 객체를 처음 사용할 때 특히 이런 유혹에 빠지기 쉽다.
```java
@DisplayName("회원 가입시 암호 검사 수행함")
@Test
void checkPassword() {
    userRegister.register("id", "pw", "email");

    // checkPasswordWeak 메서드 호출 여부 확인
    BDDMockito.then(mockPasswordChecker)
            .should()
            .checkPasswordWeak(Mockito.anyString());

    // findById 호출 여부 검사
    BDDMockito.then(mockRepository)
            .should(Mockito.never())
            .findById(Mockito.anyString());    
}
```
위 코드는 register 메서드에서 사용하는 메서드들의 호출 여부를 검증한다.  
즉, register 메서드가 내부적으로 어떤 메서드를 호출하고 어떤 메서드를 호출하지 않는지 내부 구현을 검증하는 것이다.  
내부 구현을 검증하는 것이 나쁜 것은 아니나 단점이 있다.  
__구현을 조금만 변경해도 테스트가 깨질 가능성이 커진다는 것이다.__  
__내부 구현은 언제든지 바뀔 수 있기 때문에 테스트 코드는 내부 구현보다 실행 결과를 검증해야 한다.__  
위의 코드에서는 내부 구현을 검증하는 것보다 약한 암호일 때 register 결과가 올바른지 검증해야 한다.

## 셋업을 이용해서 중복된 상황을 설정하지 않기
테스트 코드를 작성하다 보면 각 테스트 코드에서 동일한 상황이 필요할 때가 있다.  
이 경우 중복된 코드를 제거하기 위해 @BeforeEach 메서드를 이용해 상황을 구성할 수 있다.
```java
public class UserRegisterMockTest {
   ...

    @BeforeEach
    void setUp() {
        memberRepository.save(new User(...));
    }

    ...
}
```
중복을 제거하고 코드 길이도 짧아져서 코드 품질이 좋아졌다고 생각할 수 있지만, 테스트에서는 상황이 달라진다.  
예를 들어 몇 달만에 테스트 코드를 다시 볼 일이 생겼고 테스트가 실패한 이유를 알려면 어떤 상황인지 확인해야 한다.  
이때 setUp 메서드를 확인해야 하기 때문에 위아래로 이동하면서 실패 원인을 분석해야 한다.  
setUp 메서드를 이용한 상황 설정으로 인해 발생할 수 있는 또 다른 문제는 테스트가 깨지기 쉬운 구조가 된다는 점이다.  
모든 테스트 메서드가 동일한 상황 코드를 공유하기 때문에 조금만 내용을 변경해도 테스트가 깨질 수 있다.  
그렇다고 각 상황별로 BeforeEach코드 안에 작성하는 것도 좋지 못하다.  
테스트 메서드는 검증을 목표로 하는 하나의 완전한 프로그램이어야 한다.  
즉, 각 테스트 메서드는 별도의 프로그램으로서 검증 내용을 스스로 잘 설명할 수 있어야 한다.  
그러기 위해서는 상황 구성 코드가 테스트 메서드 안에 위치해야 한다.  
그래야 테스트 메서드 스스로 완전하게 테스트 내용을 설명할 수 있다.  
```java
public class UserRegisterMockTest {

    @Test
    void test1(){
        memberRepository.save(new User(...));
    }

    @Test
    void test1(){
        memberRepository.save(new User(...));        
    }
    ...
}
```
코드는 분명 BeforeEach를 사용한 것보다 길어진다. 하지만 테스트 메서드 자체는 스스로를 더 잘 설명하고 있다.  
테스트에 실패해도 코드를 이리저리 왔다 갔다 하면서 보지 않아도 된다.  
각 테스트에 맞게 상황을 설정하는 것도 쉽다. 한 테스트 메서드의 상황을 변경해도 다른 테스트에 영향을 주지 않기 때문이다.  
__셋업 메서드를 이용해서 여러 메서드에 동일한 상황을 적용하는 것이 처음에는 편리하지만, 시간이 지나면 테스트 코드를 이해하고 유지 보수하는데 오히려 방해된다.__  
__테스트 메서드는 자체적으로 검증하는 내용을 완전히 기술하고 있어야 테스트 코드를 유지보수하는 노력을 줄일 수 있다.__  
중복 코드의 문제는 뒤에서 해결한다.

## 통합 테스트에서 데이터 공유 주의하기
통합 테스트 코드를 만들 때는 다음의 두 가지로 초기화 데이터를 나눠서 생각해야 한다.
+ 모든 테스트가 같은 값을 사용하는 데이터
    - 코드값 데이터
+ 테스트 메서드에서만 필요한 데이터
    - 중복 ID 검사를 위한 회원 데이터

코드값 데이터는 거의 바뀌지 않기에 모든 테스트가 동일한 코드값 데이터를 사용해도 문제없다.  
반면에 특정 테스트에서만 의미 있는 데이터는 모든 테스트가 공유할 필요가 없다.  
이런 데이터는 특정 테스트에서만 생성해서 테스트 코드가 완전한 하나가 되도록 해야 한다.
```java
@Test
void 동일ID가_이미_존재하면_익셉션() {
    // 상황
    jdbcTemplate.update(
            "insert into user values (?,?,?) " +
            "on duplicate key update password = ?, email = ?",
            "cbk", "pw", "cbk@cbk.com", "pw", "cbk@cbk.com");

    // 실행, 결과 확인
    assertThrows(DupIdException.class,
            () -> register.register("cbk", "strongpw", "email@email.com")
    );
}
```

## 통합 테스트의 상황 설정을 위한 보조 클래스 사용하기
각 테스트 메서드에서 상황을 직접 구성함으로써 테스트 메서드를 분석하기는 좋아졌는데 반대로 상황을 만들기 위한 코드가 여러 테스트 코드에 중복되는 문제가 있다.  
중복되는 코드는 유지보수를 어렵게 하므로 어떻게든 해결해야 한다.  
테스트 메서드에서 직접 상황을 구성하면서 중복을 없애는 방법은 상황 설정을 위한 보조 클래스를 사용하는 것이다.  
예를 들어 중복 ID를 가진 회원이 존재해야 하는 상황이 존재해야 한다고 가정하고 보조 클래스르 만들어보자.
```java
public class UserGivenHelper {
    private JdbcTemplate jdbcTemplate;

    public UserGivenHelper(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    
    public void givenUser(String id, String pw, String email){
        jdbcTemplate.update(
                "insert into user values(?,?,?) " +
                        "on duplicate key update password =?, email =?",
                id,pw,email,pw,email);
    }
}
```
```java
public class Junit5Test {
    @Autowired
    JdbcTemplate jdbcTemplate;
    private UserGivenHelper given;
    
    @BeforeEach
    void setUp(){
        given = new UserGivenHelper(jdbcTemplate);
    }
    
    @Test
    void test() {
        given.givenUser(...);
        
        // 테스트 로직
        
    }
}
```
독자는 givenUser라는 메서드 이름을 통해 어떤 상황을 구성하는지 이해할 수 있고 개발자는 중복 코드를 없앨 수 있다.  
상황을 구성하는 것뿐만 아니라 검증을 위해 데이터를 조회하는 코드도 여러 테스트 메서드에 중복되어 있으면 보조 클래스를 만들어 사용할 수 있다.  

## 실행 환경이 다르다고 실패하지 않기
테스트를 실행하는 환경에 따라 테스트가 다르게 동작하면 안 된다.  
실행 환경에 따라 문제가 되는 전형적인 예가 파일 경로다.  
```java
public class Junit5Test {
    
    private String bulkFilePath = "D:\\backtony\\tmp\\bulk.txt";

    @Test
    void test() {
        BulkLoader loader = new BulkLoader();
        loader.load(bulkFilePath);
        ...
    }
}
```
위 코드는 특정 개발자 환경에서만 동작할 가능성이 크다.  
이렇게 테스트에서 사용하는 파일은 프로젝트 폴더를 기준으로 상대 경로를 사용해야 한다.  
예를 들어 src/test/resources 와 같은 폴더에 bulk.txt 파일을 생성하고 테스트 코드는 아래와 같이 작성해야 한다.
```java
public class Junit5Test {
    
    private String bulkFilePath = "src/test/resources/bulk.txt";

    @Test
    void test() {
        BulkLoader loader = new BulkLoader();
        loader.load(bulkFilePath);
        ...
    }
}
```
<Br>

테스트 코드에서 파일을 생성하는 경우에도 특정 OS나 특정 개발자 환경에서만 동작하도록 하는 경우는 없어야 한다.  
이때는 OS가 제공하는 임시 폴더에 파일을 생성하면 실행 환경에 따라 테스트가 다르게 동작하는 것을 방지할 수 있다.
```java
public class Junit5Test {
    
    @Test
    void test() {
        String folder = System.getProperty("java.io.tmpdir");
        Exporter exporter = new Exporter(folder);
        exporter.export("file.txt");

        Path path = Paths.get(folder, "file.txt");
        assertTrue(Files.exists(path));
    }
}
```
이 코드는 실행 환경에 알맞은 임시 폴더 경로를 구해서 동작하기 때문에 환경이 달라 테스트가 실패하는 상황은 벌어지지 않는다.

## 실행 시점이 다르다고 실패하지 않기
```java
public class Junit5Test {

    @Test
    void test() {
        LocalDateTime expiry = LocalDateTime.of(2019, 12, 31, 0, 0, 0);
        Member m = Member.builder().expiry(expiry).build();
        assertFalse(m.isExpired());
    }
}
```
위 코드는 회원 만료일을 2019년 12월 31일로 설정했다. 해당 시간 이전의 테스트는 성공하겠지만 그 이후에는 실패한다.  
이보다는 테스트 코드에서 시간을 명시적으로 제어할 수 있는 방법은 선택하는 방식이 더 낫다.
```java
public class Member {
    private LocalDateTime expiryDate;

    public boolean passedExpiryDate(LocalDateTime time) {
        return expiryDate.isBefore(time);
    }
}
```
```java
public class Junit5Test {
    @Test
    void test() {
        LocalDateTime expiry = LocalDateTime.of(2019, 12, 31, 0, 0, 0);
        Member m = Member.builder().expiry(expiry).build();
        assertFalse(m.passedExpiryDate(LocalDateTime.of(2019,12,30,0,0,0)));
    }
}
```
이 테스트는 실행 시점에 상관없이 항상 통과한다.  
시간을 전달하면 경계 조건도 섬세하게 테스트할 수 있다.  

## 랜덤하게 실패하지 않기
실행 시점에 따라 테스트가 실패하는 또 다른 이유는 랜덤 값에 따라 달라지는 결과를 검증할 때 주로 발생한다.  
이때는 랜덤 값을 생성하는 기능을 하는 클래스를 따로 만들어서 테스트에서는 대역을 사용해 원하는 값을 주입하도록 한다.

## 필요하지 않은 값을 설정하지 않기
```java
public class Junit5Test {

    @Test
    void test() {
       User user = User.builder()
               .id("dupId")
               .name("이름")
               .password("password")
               .build();       
        
        ...       
    }
}
```
위 코드가 동일한 ID가 존재하는 경우 가입을 할 수 있는지 없는지를 테스트하기 위한 코드라고 해보자.  
동일한 ID가 존재하는 상황을 만들 때 이름, 패스워드 등의 다른 정보는 필요하지 않다.  
객체를 생성할 때도 검증에 필요한 값만 지정하면 된다.  

## 단위 테스트를 위한 객체 생성 보조 클래스
단위 테스트 코드를 작성하다 보면 상황 구성을 위해 필요한 데이터가 복잡할 때가 있다.  
예를 들어 설문에 답변하는 기능을 구현하기 위해 다음에 해당하는 설문이 존재하는 상황이 필요하다고 가정하자.
+ 설문이 공개 상태
+ 설문 조사 기간이 끝나지 않음
+ 설문 객관식 문항이 두 개
+ 각 객관식 문항의 보기가 두 개

설문에 답변하는 기능을 테스트하기 위해서는 위의 상황을 구성해야 하는 코드가 필요하다.  
만약 구성하는 과정에서 null이면 안되는 필수 속성이 많다면 상황 구성 코드가 굉장히 길어진다.  
이런 경우에는 테스트를 위한 객체 생성 클래스를 따로 만들어 복잡함을 줄이도록 한다.  
대표적으로 팩토리 패턴으로 클래스를 만들어 사용하면 된다.
```java
public class TestSurveyFactory {
    public static Survey createAnswerableSurvey(Long id) {
        return Survey.builder()
                ...
                .build();
    }
}
```
<br>


Builder 클래스를 제공하면 더 유연하게 처리할 수 있다.  
기본값들은 미리 세팅해두고 필요한 부분만 Builder로 추가 세팅해서 받도록 하는 방식이다.
```java
public class TestSurveyBuilder {
    
    // 필수값 선언
    private Long id = 1L;
    ...

    // 유연하게 바꿀 수 있도록
    public TestSurveyBuilder id (Long id) {
        this.id = id;
        return this;
    }

    ...

    // 유연하게 바꾸지 않았다면 기본 값들로 생성된다.
    public Survey Build() {
        return Survey.builder()
            .id(id)
            ...
            .build();
    }
}
```

## 조건부로 검증하지 않기
테스트 코드 안에서 조건문을 만들고 그 안에 단언부를 넣으면 어떨 때는 수행되고 어떨 때는 수행되지 않는다.  
이런 코드는 만들지 마라.  

## 통합 테스트는 필요하지 않은 범위까지 연동하지 않기
대표적으로 Spring과 DB 연동 테스트를 하고 싶은데 @SpringBootTest를 사용하는 것이다.  
사용하는데 문제는 없지만 @SpringBootTest 애노테이션은 서비스, 컨트롤러 등 모든 스프링 빈을 초기화하기 때문에 DB 이외에 나머지 설정도 처리하므로 스프링을 초기화하는 시간이 길어진다.  
스프링 부트가 제공하는 @JdbcTest를 사용하면 DataSource, JdbcTemplate 등 DB 연동과 관련된 설정만 초기화한다.  

## 쓸모없는 테스트 코드
단지 테스트 커버리지를 높이기 위한 목적으로 작성한 테스트 코드는 유지할 필요도 작성할 필요도 없다.  
대표적으로 엔티티의 get메서드 같이 단순한 메서드를 테스트하는 것은 그저 테스트 커버리지를 높이기 위한 테스트일 뿐이다.




