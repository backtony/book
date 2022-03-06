

## 대역의 필요성
테스트를 작성하다 보면 외부 요인이 필요한 시점이 있다.
+ 테스트 대상에서 파일 시스템을 사용
+ 테스트 대상에서 DB로부터 데이터를 조회하거나 데이터를 추가
+ 테스트 대상에서 외부의 HTTP 서버와 통신

테스트 대상이 이런 외부 요인에 의존하면 테스트를 작성하고 실행하기 어려워진다.  
테스트 대상 코드에서 사용하는 외부 API 서버가 일시적으로 장애가 나면 테스트를 원활하게 수행할 수 없다.  
예를 들어 보자.

> 자동 이체 정보 등록 기능 -> 카드 번호 검사기 -> 카드 번호 검사 대행업체(카드 정보 검사 API)

자동 이체 정보 등록 기능을 테스트하는 코드는 외부 요인인 카드 정보 검사 대행업체의 상태에 따라 테스트를 할 수 없는 상황이 발생하게 된다.  
코드로 보자. 자세하게 볼 필요는 없고 아래 설명만 읽어도 된다.
```java
public class AutoDebitRegister {
    private CardNumberValidator validator;
    private AutoDebitInfoRepository repository;

    public AutoDebitRegister(CardNumberValidator validator, AutoDebitInfoRepository repository) {
        this.validator = validator;
        this.repository = repository;
    }

    public RegisterResult register(AutoDebitReq req) {
        CardValidity validity = validator.validate(req.getCardNumber());
        if (validity != CardValidity.VALID) {
            return RegisterResult.error(validity);
        }
        AutoDebitInfo info = repository.findOne(req.getUserId());
        if (info != null) {
            info.changeCardNumber(req.getCardNumber());
        } else {
            AutoDebitInfo newInfo = new AutoDebitInfo(req.getUserId(), req.getCardNumber(), LocalDateTime.now());
            repository.save(newInfo);
        }
        return RegisterResult.success();
    }
}
```
```java
public class CardNumberValidator {
    public CardValidity validate(String cardNumber) {
        HttpClient httpClient = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://some-external-pg.com/card"))
                .header("Content-Type", "text/plain")
                .POST(BodyPublishers.ofString(cardNumber))
                .build();
        try {
            HttpResponse<String> response = httpClient.send(request, BodyHandlers.ofString());
            switch (response.body()) {
                case "ok": return CardValidity.VALID;
                case "bad": return CardValidity.INVALID;
                case "expired": return CardValidity.EXPIRED;
                case "theft": return CardValidity.THEFT;
                default: return CardValidity.UNKNOWN;
            }
        } catch (IOException | InterruptedException e) {
            return CardValidity.ERROR;
        }
    }
}
```
AutoDebitRegister 클래스에서 사용하는 CardNumberValidator 클래스는 외부 서비스에서 제공하는 HTTP URL을 이용해서 카드번호가 유효한지 검사하고 그 결과를 리턴한다.  
```java
public class AutoDebitRegisterTest {
    private AutoDebitRegister register;

    @BeforeEach
    void setUp() {
        CardNumberValidator validator = new CardNumberValidator();
        AutoDebitInfoRepository repository = new JpaAutoDebitInfoRepository();
        register = new AutoDebitRegister(validator, repository);
    }

    @Test
    void validCard() {
        // 업체에서 받은 테스트용 유효한 카드 번호 사용
        AutoDebitReq req = new AutoDebitReq("user1", "1234123412341234");
        RegisterResult result = this.register.register(req);
        assertEquals(VALID, result.getValidity());
    }

    @Test
    void theftCard() {
        // 업체에서 받은 도난 테스트용 카드 번호 사용
        AutoDebitReq req = new AutoDebitReq("user1", "1234567890123456");
        RegisterResult result = this.register.register(req);
        assertEquals(THEFT, result.getValidity());
    }
}
```
만약 외부 업체에서 테스트 목적으로 유효한 카드 번호를 제공한다면 위와 같이 테스트를 작성할 수 있다.  
하지만 업체에서 해당 카드 번호를 제거한다면 테스트는 실패하게 된다.  
이렇게 테스트 대상에서 의존하는 요인 때문에 테스트가 어려울 때는 대역을 써서 테스트를 진행한다.

## 대역을 이용한 테스트
```java
public class StubCardNumberValidator extends CardNumberValidator {
    private String invalidNo;
    private String theftNo;

    public void setInvalidNo(String invalidNo) {
        this.invalidNo = invalidNo;
    }

    public void setTheftNo(String theftNo) {
        this.theftNo = theftNo;
    }

    @Override
    public CardValidity validate(String cardNumber) {
        if (invalidNo != null && invalidNo.equals(cardNumber)) {
            return CardValidity.INVALID;
        }
        if (theftNo != null && theftNo.equals(cardNumber)) {
            return CardValidity.THEFT;
        }
        return CardValidity.VALID;
    }
}
```
StubCardNumberValidator는 실제 카드번호 검증 기능을 구현하지 않고 단순하게 실제 기능을 대체한다.  
StubCardNumberValidator는 setInvalidNo 메서드로 지정한 카드 번호에 대해서는 Invalid를 리턴하고 나머지는 Valid를 리턴한다.
이를 이용해 테스트 코드를 작성하면 다음과 같다.
```java
public class AutoDebitRegister_Stub_Test {
    private AutoDebitRegister register;
    private StubCardNumberValidator stubValidator;
    private StubAutoDebitInfoRepository stubRepository;

    @BeforeEach
    void setUp() {
        stubValidator = new StubCardNumberValidator();
        stubRepository = new StubAutoDebitInfoRepository();
        register = new AutoDebitRegister(stubValidator, stubRepository);
    }

    @Test
    void invalidCard() {
        stubValidator.setInvalidNo("111122223333");

        AutoDebitReq req = new AutoDebitReq("user1", "111122223333");
        RegisterResult result = this.register.register(req);

        assertEquals(INVALID, result.getValidity());
    }

    @Test
    void theftCard() {
        stubValidator.setTheftNo("1234567890123456");

        AutoDebitReq req = new AutoDebitReq("user1", "1234567890123456");
        RegisterResult result = this.register.register(req);

        assertEquals(CardValidity.THEFT, result.getValidity());
    }

    @Test
    void validCard() {
        AutoDebitReq req = new AutoDebitReq("user1", "1234123412341234");
        RegisterResult result = this.register.register(req);
        assertEquals(VALID, result.getValidity());
    }
}
```
정리하자면, 실제 코드는 외부와 연동해서 사용하지만 테스트할 때는 해당 기능을 대신하는 것을 만들어 테스트에 사용한다는 것이다.  

## 대역의 종류

종류|설명
---|---
스텁(Stub)|구현을 단순한 것으로 대체한다. 테스트에 맞게 단순히 원하는 동작을 수행한다. 앞선 예시가 이에 해당한다.
가짜(Fake)|제품에는 적합하지 않지만, 실제 동작하는 구현을 제공한다. DB 대신에 메모리를 이용해서 구현한 DB가 이에 해당한다.
스파이(Spy)|호출된 내역을 기록한다. 기록된 내용은 테스트 결과를 검증할 때 사용한다. 스텁이기도 하다.
모의(Mock)|기대한 대로 상호작용하는지 행위를 검증한다. 기대한 대로 동작하지 않으면 Exception을 발생시킬 수 있다. 모의 객체는 스텁이자 스파이도 된다.

<br>

예를 이용해서 대역을 알아보자. 사용할 예는 회원 가입 기능이다.
+ UserRegister : 회원 가입에 대한 핵심 로직
+ WeakPAsswordChecker : 암호가 약한지 검사
+ UserRepository : 회원 정보를 저장하고 조회하는 기능 제공
+ EmailNotifier : 이메일 발송 기능을 제공

### 약한 암호 확인 기능에 스텁 사용하기
암호가 약한 경우 회원 가입에 실패하는 테스트부터 시작하자.  
테스트 대상이 UserRegister이므로 WeakPasswordChecker는 대역을 사용할 것이다.  
실제 동작하는 구현을 필요하지 않고 그저 약한 암호 인지 여부를 알려주기만 하면 되므로 스텁으로 충분하다.


```java
public class UserRegisterTest {
    private UserRegister userRegister;
    private StubWeakPasswordChecker stubPasswordChecker = new StubWeakPasswordChecker();

    @BeforeEach
    void setUp() {
        userRegister = new UserRegister(stubPasswordChecker);
    }

    @DisplayName("약한 암호면 가입 실패")
    @Test
    void weakPassword() {
        stubPasswordChecker.setWeak(true); // 암호가 약하다고 응답하도록 설정

        assertThrows(WeakPasswordException.class, () -> {
            userRegister.register("id", "pw", "email");
        });
    }

}
```
아직 구현하기 전이니까 그냥 setWeak를 true로 두면 약하다고 응답하겠다고 만들어두었다.  
이제 컴파일 에러를 해결하자.
```java
public class WeakPasswordException extends RuntimeException {
}
```
```java
public interface WeakPasswordChecker {
}
```
```java
public class StubWeakPasswordChecker implements WeakPasswordChecker {
    private boolean weak;

    public void setWeak(boolean weak) {
        this.weak = weak;
    }
}
```
```java
public class UserRegister {
    private WeakPasswordChecker passwordChecker;

    public UserRegister(WeakPasswordChecker passwordChecker) {
        this.passwordChecker = passwordChecker;
    }

    public void register(String id, String pw, String email) {
        throw new WeakPasswordException();       
    }
}
```
그냥 Exception을 터트리도록 했다. 이제 구현을 좀 더 일반화시킬 차례다. 약한 암호인 경우 Exception을 터트리도록 하자.
```java
public class UserRegister {
    private WeakPasswordChecker passwordChecker;

    public UserRegister(WeakPasswordChecker passwordChecker) {
        this.passwordChecker = passwordChecker;
    }

    public void register(String id, String pw, String email) {
         if (passwordChecker.checkPasswordWeak(pw)) {
            throw new WeakPasswordException();
        }       
    }
}
```
```java
public interface WeakPasswordChecker {
    boolean checkPasswordWeak(String pw);
}
```
```java
public class StubWeakPasswordChecker implements WeakPasswordChecker {
    private boolean weak;

    public void setWeak(boolean weak) {
        this.weak = weak;
    }

    @Override
    public boolean checkPasswordWeak(String pw) {
        return weak;
    }
}
```
이 정도만 구현해도 약한 암호인 경우와 그렇지 않은 경우에 대해 올바르게 동작하는지 확인할 수 있다.

### 리포지토리를 가짜 구현으로 사용
이번에는 동일 ID를 가진 회원이 존재할 경우 Exception을 발생하는 테스트를 작성해보자.
```java
@Test
void dupIdExists() {
    // 동일 아이디를 가진 회원 존재 코드 생략
    
    assertThrows(DupIdException.class, () -> {
        userRegister.register("id", "pw", "email");
    });
}
```
UserRepository도 실제와 동일하게 동작하는 가짜 대역을 이용해서 이미 같은 ID를 가진 사용자가 존재하는 상황을 만들어야 한다.  
이를 염두해두고 테스트 코드를 추가하자.
```java
public class UserRegisterTest {
    private UserRegister userRegister;
    private StubWeakPasswordChecker stubPasswordChecker = new StubWeakPasswordChecker();
    private MemoryUserRepository fakeRepository = new MemoryUserRepository();

    @BeforeEach
    void setUp() {
        userRegister = new UserRegister(stubPasswordChecker,
                fakeRepository);
    }

    @DisplayName("이미 같은 ID가 존재하면 가입 실패")
    @Test
    void dupIdExists() {
        // 이미 같은 ID 존재하는 상황 만들기
        fakeRepository.save(new User("id", "pw1", "email@email.com"));

        assertThrows(DupIdException.class, () -> {
            userRegister.register("id", "pw2", "email");
        });
    }
}
```
이제 컴파일 에러를 해결해보자.
```java
public interface UserRepository {
    void save(User user);
}
```
```java
public class MemoryUserRepository implements UserRepository {
    private Map<String, User> users = new HashMap<>();

    @Override
    public void save(User user) {
        users.put(user.getId(), user);
    }
}
```
```java
public class UserRegister {
    private WeakPasswordChecker passwordChecker;
    private UserRepository userRepository;

    public UserRegister(WeakPasswordChecker passwordChecker,
                        UserRepository userRepository) {
        this.passwordChecker = passwordChecker;
        this.userRepository = userRepository;
    }

    public void register(String id, String pw, String email) {
        if (passwordChecker.checkPasswordWeak(pw)) {
            throw new WeakPasswordException();
        }      
    }
}
```
```java
public class User {
    private String id;
    private String password;
    private String email;

    public User(String id, String password, String email) {
        this.id = id;
        this.password = password;
        this.email = email;
    }

    public String getId() {
        return id;
    }

    public String getEmail() {
        return email;
    }
}
```
```java
public class DupIdException extends RuntimeException {
}
```
이제 컴파일 에러는 전부 해결했으니 테스트를 돌려보자. 테스트는 실패한다. UserRegister 코드를 수정해야 할 것 같다.
```java
public class UserRegister {
    private WeakPasswordChecker passwordChecker;
    private UserRepository userRepository;

    public UserRegister(WeakPasswordChecker passwordChecker,
                        UserRepository userRepository) {
        this.passwordChecker = passwordChecker;
        this.userRepository = userRepository;
    }

    public void register(String id, String pw, String email) {
        if (passwordChecker.checkPasswordWeak(pw)) {
            throw new WeakPasswordException();
        }      
        return new DupIdException();
    }
}
```
이렇게 간단히 구현하면 성공은 한다. 이제 일반화시켜보자.
```java
public class UserRegister {
    private WeakPasswordChecker passwordChecker;
    private UserRepository userRepository;

    public UserRegister(WeakPasswordChecker passwordChecker,
                        UserRepository userRepository) {
        this.passwordChecker = passwordChecker;
        this.userRepository = userRepository;
    }

    public void register(String id, String pw, String email) {
        if (passwordChecker.checkPasswordWeak(pw)) {
            throw new WeakPasswordException();
        }      
         User user = userRepository.findById(id);
        if (user != null) {
            throw new DupIdException();
        }
    }
}
```
```java
public interface UserRepository {
    void save(User user);

    User findById(String id);
}
```
```java
public class MemoryUserRepository implements UserRepository {
    private Map<String, User> users = new HashMap<>();

    @Override
    public void save(User user) {
        users.put(user.getId(), user);
    }

    @Override
    public User findById(String id) {
        return users.get(id);
    }
}
```
이제 테스트를 다시 돌려보면 성공한다.  
이번에는 중복 아이디가 존재하지 않을 경우 회원 가입에 성공하는 경우도 테스트하자. 이것도 가짜 대역을 사용한다.
```java
@DisplayName("같은 ID가 없으면 가입 성공함")
@Test
void noDupId_RegisterSuccess() {
    userRegister.register("id", "pw", "email");

    User savedUser = fakeRepository.findById("id");
    assertEquals("id", savedUser.getId());
    assertEquals("email", savedUser.getEmail());
}
```
테스트를 돌리면 NullPointException이 발생한다. 이는 savedUser에서 구한 User가 null임을 의미한다.  
UserRegister에서 User를 저장하는 코드를 추가해야 할 것 같다.
```java
public class UserRegister {
    private WeakPasswordChecker passwordChecker;
    private UserRepository userRepository;

    public UserRegister(WeakPasswordChecker passwordChecker,
                        UserRepository userRepository) {
        this.passwordChecker = passwordChecker;
        this.userRepository = userRepository;
    }

    public void register(String id, String pw, String email) {
        if (passwordChecker.checkPasswordWeak(pw)) {
            throw new WeakPasswordException();
        }      
         User user = userRepository.findById(id);
        if (user != null) {
            throw new DupIdException();
        }
        userRepository.save(new User(id, pw, email));
    }
}
```
이제 테스트가 통과한다.

### 이메일 발송 여부를 확인하기 위해 스파이를 사용
회원가입이 성공하면 이메일로 회원 가입 안내 메일을 발송한다고 해보자.
```java
// 실행
userRegister.register("id", "pw", "email@gmail.com");

// 결과
email@gmail.com 으로 이메일 발송을 요쳥했는지 확인
```
이메일 발송 여부는 어떻게 확인해야 할까? 방법 중 하나는 Usergister가 EmailNotifier의 메일 발송 기능을 실행할 때 이메일 주소로 email@gmail.com을 사용했는지 확인하는 것이다. 이런 용도로 사용할 수 있는 것이 스파이 대역이다.
```java
public class UserRegisterTest {
    private UserRegister userRegister;
    private StubWeakPasswordChecker stubPasswordChecker = new StubWeakPasswordChecker();
    private MemoryUserRepository fakeRepository = new MemoryUserRepository();
    private SpyEmailNotifier spyEmailNotifier = new SpyEmailNotifier();

    @BeforeEach
    void setUp() {
        userRegister = new UserRegister(stubPasswordChecker,
                fakeRepository,
                spyEmailNotifier);
    }

    @DisplayName("가입하면 메일을 전송함")
    @Test
    void whenRegisterThenSendMail() {
        userRegister.register("id", "pw", "email@email.com");

        assertTrue(spyEmailNotifier.isCalled());
        assertEquals(
                "email@email.com",
                spyEmailNotifier.getEmail());
    }
}
```
테스트 코드를 작성했으니 이제 컴파일 에러를 해결해보자.
```java
public interface EmailNotifier {
}
```
```java
public class SpyEmailNotifier implements EmailNotifier {
    private boolean called;
    private String email;

    public boolean isCalled() {
        return called;
    }

    public String getEmail() {
        return email;
    }
}
```
여기서 테스트를 돌리면 isCalled에서 실패한다. 코드를 수정하자.
```java
public class UserRegister {
    private WeakPasswordChecker passwordChecker;
    private UserRepository userRepository;
    private EmailNotifier emailNotifier;

    public UserRegister(WeakPasswordChecker passwordChecker,
                        UserRepository userRepository,
                        EmailNotifier emailNotifier) {
        this.passwordChecker = passwordChecker;
        this.userRepository = userRepository;
        this.emailNotifier = emailNotifier;
    }

    public void register(String id, String pw, String email) {
        if (passwordChecker.checkPasswordWeak(pw)) {
            throw new WeakPasswordException();
        }
        User user = userRepository.findById(id);
        if (user != null) {
            throw new DupIdException();
        }
        userRepository.save(new User(id, pw, email));

        emailNotifier.sendRegisterEmail(email);
    }
}
```
```java
public interface EmailNotifier {
    void sendRegisterEmail(String email);
}
```
```java
public class SpyEmailNotifier implements EmailNotifier {
    private boolean called;
    private String email;

    public boolean isCalled() {
        return called;
    }

    public String getEmail() {
        return email;
    }

    @Override
    public void sendRegisterEmail(String email) {
        this.called = true;
    }
}
```
이제 isCalled는 성공하나 email이 맞지 않아서 실패한다. 살짝 코드를 추가해보자.
```java
public class SpyEmailNotifier implements EmailNotifier {
    ...

    @Override
    public void sendRegisterEmail(String email) {
        this.called = true;
        this.email = email;
    }
}
```
이제 테스트가 모두 통과한다.

### 모의 객체로 스텁과 스파이 대체
스텁과 스파이를 사용하면 사실 따로 클래스를 만들어야 하기 때문에 불편함이 있다. 이를 해결하는 것이 모의 객체다.  
모의 객체로 전환해보자.
```java
public class UserRegisterMockTest {
    private UserRegister userRegister;
    private WeakPasswordChecker mockPasswordChecker = Mockito.mock(WeakPasswordChecker.class);
    private MemoryUserRepository fakeRepository = new MemoryUserRepository();
    private EmailNotifier mockEmailNotifier = Mockito.mock(EmailNotifier.class);

    @BeforeEach
    void setUp() {
        userRegister = new UserRegister(mockPasswordChecker,
                fakeRepository,
                mockEmailNotifier);
    }

    @DisplayName("약한 암호면 가입 실패")
    @Test
    void weakPassword() {
        BDDMockito
            // "pw" 인자를 사용해서 모의 객체의 checkPasswordWeak 메서드를 호출하면 
            .given(mockPasswordChecker.checkPasswordWeak("pw"))
            // 결과를 true를 리턴하라
            .willReturn(true);

        assertThrows(WeakPasswordException.class, () -> {
            userRegister.register("id", "pw", "email");
        });
    }
    ...
}
```
모의 객체를 만들 때는 Mockito.mock()를 이용한다. Mockito.mock() 메서드는 인자로 전달받은 타입의 모의 객체를 생성한다.  
```java
@DisplayName("회원 가입시 암호 검사 수행함")
@Test
void checkPassword() {
    userRegister.register("id", "pw", "email");

    // 인자로 전달한 mockPasswordChecker
    BDDMockito.then(mockPasswordChecker)
            // 특정 메서드가 호출됐는지 검증하는데
            .should()
            // 임의의 String 타입 인자를 이용해서 checkPasswordWeak 메서드 호출 여부를 확인한다.
            .checkPasswordWeak(Mockito.matches("pw"));
}
```
모의 객체가 기대한 대로 불렸는지도 검증할 수 있다. register 메서드가 호출되었을 때 모의 객체가 어떤 식으로 동작했는지를 검증할 수 있다.

