## 목차
- [테스트 설계](#1)
- [기능 테스트와 E2E 테스트](#2)
- [통합 테스트](#3)
- [단위 테스트](#4)
- [테스트 범위 간 차이](#5)
- [테스트 범위에 따른 테스트 코드 개수와 시간](#6)
- [외부 연동이 필요한 테스트 예](#7)

---

## 테스트 설계 <a name = "1"></a>
### 테스트 하기 어려운 코드
+ 하드 코딩된 경로
+ 의존 객체를 직접 생성
+ 정적 메서드 사용
+ 실행 시점에 따라 달라지는 결과
+ 역할이 섞여 있는 코드
+ 메서드 중간에 소켓 통신 코드가 포함되어 있는 경우
+ 콘솔에서 입력받거나 결과를 콘솔에 출력하는 경우
+ 테스트 대상이 사용하는 의존 대상 클래스나 메서드가 final인 경우 대역으로 대체하기 어렵다.
+ 테스트 대상의 소스를 소유하고 있지 않은 경우

위의 경우에서 테스트가 어려운 주된 이유는 의존하는 코드를 교체할 수 있는 수단이 없기 때문이다.  

### 테스트 가능한 설계
상황에 따라 알맞은 방법을 적용하면 의존 코드를 교체할 수 있게 만들 수 있다.  
+ 하드 코딩된 상수를 생성자나 메서드 파라미터로 받기
    - 상수를 교체할 수 있는 기능을 추가하는 것이다.
+ 의존 대상을 주입받기
    - 생성자나 세터를 주입 수단으로 이용한다.
+ 테스트하고 싶은 코드를 분리하기
    - 메서드 내에서 테스트하고 싶은 코드와 그 이외의 코드가 섞여있다면 테스트하고 싶은 코드를 별도 기능으로 분리하면 된다.
+ 시간이나 임의값 생성 기능 분리하기
    - 시간이나 임의값을 사용하면 테스트 시점에 따라 결과가 달라진다면 테스트 대상이 사용하는 시간이나임의 값을 제공하는 기능을 별도로 분리해서 테스트한다.
    - 예를 들어 LocalDate.now()가 메서드 코드 내에 있으면 테스트를 실행하는 일자마다 값이 달라지므로 해당 기능을 제공하는 클래스를 따로 만들어 분리하고 분리한 대상을 주입받아 사용하도록 변경한다. 테스트에서는 모의 객체를 사용해서 쉽게 제어할 수 있게 된다.
+ 외부 라이브러리는 직접 사용하지 말고 감싸서 사용하기
    - 외부 라이브러리를 직접 사용하지 말고 연동하기 위한 타입을 따로 만들어 사용한다. 그리고 테스트 대상은 따로 만든 타입을 주입받아 사용하도록 만든다. 이를 통해 테스트 코드에서는 외부 연동이 필요한 기능을 쉽게 대역으로 대체할 수 있다.



## 기능 테스트와 E2E 테스트 <a name = "2"></a>
기능 테스트는 사용자 입장에서 시스템이 제공하는 기능이 올바르게 동작하는지 확인한다.  
따라서 사용자가 직접 사용하는 웹 브라우저나 모바일 앱부터 시작해서 데이터베이스나 외부 서비스에 이르기까지 모든 구성 요소를 하나로 엮어서 진행한다.  
끝에서 끝까지 올바른지 검사하기 때문에 E2E(End to end) 테스트로도 볼 수 있다.  
QA조직에서 수행하는 테스트가 주로 기능 테스트이다.  
이때 테스트는 시스템이 필요로 하는 데이터를 입력하고 결과가 올바른지 확인한다.  
예를 들면, 회원 가입 폼을 통해서 가입했을 때 해당 정보로 로그인이 잘 되는지, 회원 정보를 조회할 때 가입할 때 입력한 데이터가 올바르게 표시되는지 등을 확인한다.

## 통합 테스트 <a name = "3"></a>
통합 테스트는 시스템의 각 구성 요소가 올바르게 연동되는지 확인한다.  
기능 테스트가 사용자 입장에서 테스트하는 데 반해 통합 테스트는 소프트웨어의 코드를 직접 테스트한다.  
예를 들면, 기능 테스트는 앱을 통해 가입 기능을 테스트한다면 통합 테스트는 서버의 회원 가입 코드를 직접 테스트하는 식이다.  
통합 테스트를 수행한다면 스프링 프레임워크의 설정이 올바른지, SQL 쿼리가 맞는지, DB 트랜잭션이 잘 동작하는지 검증할 수 있다.  

## 단위 테스트 <a name = "4"></a>
단위 테스트는 개별 코드나 컴포넌트가 기대한 대로 동작하는지 확인한다.  
앞서 살펴본 테스트가 주로 단위 테스트 코드에 해당한다.  
단위 테스트는 한 클래스나 한 메서드와 같은 작은 범위를 테스트한다. 일부 의존 대상은 스텁이나 모의 객체 등을 이용해서 대역으로 대체한다.

## 테스트 범위 간 차이 <a name = "5"></a>
+ 통합 테스트를 수행하려면 DB나 캐시 서버와 같은 연동 대상을 구성해야 한다. 기능 테스트를 실행하려면 웹 서버를 구동하거나 모바일 앱을 핸드폰에 설치해야 할 수도 있다. 반면에 단위 테스트는 테스트 코드를 빼면 따로 준비할 것이 없다.
+ 통합 테스트는 여럿 초기화가 필요해 테스트 실행 속도를 느리게 만드는 요인이 많다. 기능 테스트도 알맞은 상호 작용이 필요하다. 반면에 단위 테스트는 서버를 구동하거나 DB를 준비할 필요가 없기 때문에 속도가 빠르다.
+ 통합 테스트나 기능 테스트로는 상황을 준비하거나 결과 확인이 어렵거나 불가능할 때가 있다. 외부 시스템 연동이 특히 그렇다. 이런 경우에는 단위 테스트와 대역을 조합해서 상황을 만들고 테스트해야 한다.

위와 같은 이유로 테스트를 작성하는 개발자라면 통합 테스트보다 단위 테스트를 더 많이 작성하게 된다.  
통합 테스트를 실행하라면 준비할 것이 많고 단위 테스트에 비해 실행 시간도 길지만, 그래도 반드시 필요하다.  
아무리 단위 테스트를 많이 만든다고 해도 결국 각 구성 요소가 올바르게 연동되는 것을 확인해야 하는데 이를 자동화하기 좋은 수단이 통합 테스트 코드이기 때문이다.

## 테스트 범위에 따른 테스트 코드 개수와 시간 <a name = "6"></a>
기능 테스트, 통합 테스트, 단위 테스트 등 전 범위에 대해 테스트를 자동화하는 시도가 증가하고 있다.  
테스트를 자동화한다는 것은 결국 코드로 작성한 테스트를 실행한다는 것을 의미한다.  
기능 테스트를 수행하려면 모든 환경이 갖춰져야 하기에 자동화하거나 다양한 상황별로 테스트하기 가장 어렵다.  
이런 이유로 정기적으로 수행하는 기능 테스트는 정상적인 경우와 몇 가지 특수한 상황만 테스트 범위로 잡는다.  
통합 테스트는 기능 테스트에 비해 상대적으로 실행 시간이 짧고 상황을 보다 유연하게 구성할 수 있기 때문에 보통 기능 테스트보다 통합 테스트를 더 많이 작성한다.  
단위 테스트는 통합 테스트로도 만들기 힘든 상황을 쉽게 구현할 수 있고 다양한 상황을 다루기 때문에 통합 테스트보다 더 많이 작성하게 된다.  
기능 테스트나 통합 테스트에서 모든 예외 상황을 테스트하면 단위 테스트는 줄어든다.  
왜냐하면 각 테스트가 다루는 내용이 이미 수행되었기 때문에 단위 테스트를 한다는 것은 중복이기 때문이다.  
__하지만 테스트 속도는 통합 테스트보다 단위 테스트가 빠르기 때문에 가능하면 단위 테스트에서 다양한 상황을 다루고, 통합 테스트나 기능 테스트는 주요 상황에 초점을 맞춰야 한다.__

## 외부 연동이 필요한 테스트 예 <a name = "7"></a>
외부 연동 대상은 쉽게 제어할 수 없기 때문에 연동할 대상이 늘어날수록 통합 테스트는 힘들어진다.  
모든 외부 연동 대상을 통합 테스트에서 다룰 수 없지만, 일부 외부 대상은 어느 정도 수준에서 제어가 가능하다.  

### 스프링 부트와 DB 통합 테스트
통합 테스트는 실제로 DB를 사용한다. 여러 번 실행도 결과가 같게 나와야 하므로 DB에 데이터 추가/삭제를 수행하면서 어느 정도 DB를 제어할 수 있다.

### wireMock을 이용한 REST 클라이언트 테스트
```java
public class CardNumberValidator {

    private String server;

    public CardNumberValidator(String server) {
        this.server = server;
    }

    public CardValidity validate(String cardNumber) {
        HttpClient httpClient = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(server + "/card"))
                .header("Content-Type", "text/plain")
                .POST(BodyPublishers.ofString(cardNumber))
                .timeout(Duration.ofSeconds(3))
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
        } catch (HttpTimeoutException e) {
            return CardValidity.TIMEOUT;
        } catch (IOException | InterruptedException e) {
            return CardValidity.ERROR;
        }
    }
}
```
외부 카드사 API를 사용해 카드번호가 유효한지 확인하는 코드다.  
외부 서버에 요청하여 응답을 받아야 하기 때문에 외부 서버에 의존하여 테스트를 작성하면 외부 서버가 다운되면 테스트는 실패한다.  
테스트에서 wireMock를 사용하면 테스트마다 모의 서버를 띄우고 서버가 요청을 받았을 때 수행할 동작을 정의할 수 있다.  
wireMock을 사용해 개선해보자.
```java
public class CardNumberValidatorTest {

    private WireMockServer wireMockServer;

    // 모의 서버 띄우기
    @BeforeEach
    void setUp() {
        wireMockServer = new WireMockServer(options().port(8089));
        wireMockServer.start();
    }

    // 모의 서버 내리기
    @AfterEach
    void tearDown() {
        wireMockServer.stop();
    }

    @Test
    void valid() {
        // 모의 서버 동작 정의하기
        wireMockServer.stubFor(post(urlEqualTo("/card")) // post 요청 url
                .withRequestBody(equalTo("1234567890")) // 요청 몸체가 이와 같다면
                .willReturn(aResponse() // 이와 같이 반환해라
                        .withHeader("Content-Type", "text/plain")
                        .withBody("ok"))
        );

        CardNumberValidator validator =
                new CardNumberValidator("http://localhost:8089");
        CardValidity validity = validator.validate("1234567890");
        assertEquals(CardValidity.VALID, validity);
    }

    @Test
    void timeout() {
        // 모의 서버 동작 정의하기
        wireMockServer.stubFor(post(urlEqualTo("/card"))
                .willReturn(aResponse()
                        .withFixedDelay(5000)) // 5초 뒤에 응답하도록 하라
        );

        CardNumberValidator validator =
                new CardNumberValidator("http://localhost:8089");
        CardValidity validity = validator.validate("1234567890");
        assertEquals(CardValidity.TIMEOUT, validity);
    }
}
```

### 스프링 부트 내장 서버를 이용한 API 기능 테스트
모바일 앱에서 회원 가입을 위해 사용하는 회원 가입 API가 올바르게 JSON을 응답하는지 검증해야 한다고 하자.  
회원 가입은 매우 중요하기 때문에 회원 가입 API를 검증하는 테스트 코드를 작성해서 검증 과정을 자동화하면 담당자가 수동으로 테스트하는 시간을 줄일 수 있다.  
스프링 부트를 사용한다면 내장 톰캣을 이용해 API에 대한 테스트를 JUnit 코드로 다음과 같이 작성할 수 있다.
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class UserApiE2ETest {
    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void weakPwResponse() {
        String reqBody = "{\"id\": \"id\", \"pw\": \"123\", \"email\": \"a@a.com\" }";
        RequestEntity<String> request =
                RequestEntity.post(URI.create("/users"))
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .body(reqBody);

        ResponseEntity<String> response = restTemplate.exchange(
                request,
                String.class);

        assertEquals(HttpStatus.BAD_REQUEST, response.getStatusCode());
        assertTrue(response.getBody().contains("WeakPasswordException"));
    }
}
```
스프링 부트는 테스트에서 웹 환경을 구동할 수 있는 기능을 제공한다.  
TestRestTemplate은 스프링 부트가 테스트 목적으로 제공하는 것으로 내장 서버에 연결하는 RestTemplate이다.  
@SpringBootTest의 속성으로 임의의 포트로 내장 서버를 구동하도록 설정했는데 만약 12091 포트로 동작했다면 testRestTemplate은 해당 포트로 요청을 보내게 된다.


