## 목차
- [작게 만들어라](#1)
- [블록과 들여쓰기](#2)
- [한 가지만 해라](#3)
- [함수 당 추상화 수준은 하나로](#4)
- [Switch문](#5)
- [서술적인 이름을 사용하라](#6)
- [함수 인수](#7)
- [부수 효과를 일으키지 마라](#8)
- [명령과 조회를 분리하라](#9)
- [오류 코드보다 예외를 사용하라](#10)
- [반복하지 마라](#11)
- [구조적 프로그래밍](#12)
- [함수를 어떻게 짜죠](#13)

---

## 작게 만들어라! <a name = "1"></a>
말 그대로 함수는 최대한 작게 만들어야 한다.  
이에 대한 증거나 자료를 제시하기 어렵지만, 저자는 자신의 경험을 통해 __함수의 길이를 3~5줄 정도로 권장한다.__  
함수를 줄이다 보면 각 함수는 목적과 의미는 명확해진다.  
<br>

## 블록과 들여쓰기 <a name = "2"></a>
if, else, while 문 등에 들어가는 블록은 한 줄이어야 한다.  
즉, 중첩 구조가 생길만큼 함수가 커져서는 안 된다는 의미이다.  
블록 안에서 함수를 호출하면 바깥을 감싸는 함수가 작아질 뿐 아니라, 블록 안에서 호출하는 함수 이름을 적절히 짓는다면 코드는 이해하기 쉬워진다.  
정리하면, __중첩 구조가 생길만큼 함수가 커지면 안되고, 들여쓰기 수준이 2단을 넘어서는 안된다.__  

<br>

## 한 가지만 해라! <a name = "3"></a>
__함수는 한 가지만 해야하며, 그 한 가지를 잘해야 한다.__  
지정한 함수 이름 아래 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한 가지 작업만 한다.  
쉽게 풀어보자면, __단순히 다른 표현이 아니라 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 추상화 수준이 하나가 아닌 여러 개의 추상화 수준이 존재하는 것이다.__  
또한, 한 함수에서 여러 섹션으로 자연스럽게 나눌 수 있다면 여러 작업을 한다는 증거라고 보면 된다.  
<br>


## 함수 당 추상화 수준은 하나로! <a name = "4"></a>
__함수가 확실히 한 가지 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 한다.__  
추상화 수준이 도대체 무엇을 뜻하는지 처음에는 와닿지 않을 수 있다.  
추상화 수준은 말 그대로 __얼마나 추상적인가?__ 를 의미한다.  
당연히 상대적이고 추상적인 것을 구현할수록 추상화 수준(레벨)은 낮아진다.  
```java
getHtml() // 추상화 수준이 높다.
String pagePathName = PathParser.render(pagepath); // 추상화 수준이 중간이다.
.append("\n"); // 추상화 수준이 낮다.
```
위의 코드를 책에서는 주석처럼 추상화 수준을 표현하고 있다.  
조금만 생각해보면 이해할 수 있다.  
append는 아주 구체적인 구현 사항을 담고 있으니 추상화 수준이 낮다고 볼 수 있다.  
String pagePathName = PathParser.render(pagepath)는 append처럼 아주 구체적인 구현 사항을 보여주진 않으니 append보다는 추상화 수준이 높다.  
getHtml를 PathParser.render(pagepath)과 비교하면 getHtml이 추상화 수준이 더 높다고 볼 수 있다.  
따라서 3개를 상대적으로 보면 위와 같은 추상화 수준이 나오게 되는 것이다.  
<br>

한 가지 예시를 더 보자.  
회원가입이라는 기능은 아래 3가지 작업으로 세분화고 해보자.  
+ ID 유효성 검사
+ DB에 저장
+ 페이지 전환

회원가입은 추상화 수준이 매우 높다.  
그에 반해 안에 있는 3가지 기능은 추상화 수준이 회원가입에 비해 낮다고 볼 수 있다.  
만약 각 3가지 작업에 대해서 안에서 더 추상화 수준을 나눌 수 있다면 3가지 작업은 추상화 수준이 중간이 되는 것이다.  
<br>

### 위에서 아래로 코드 읽기: 내려가기 규칙
추상화 수준을 이해했다면, __해당 함수 다음에는 추상화 수준이 한 단계 낮은 함수가 온다는 것을 확인할 수 있다.__  
이를 __내려가기 규칙__ 이라고 한다.  
다르게 말하면, 코드는 위에서 아래로 이야기처럼 읽여햐 한다는 의미이다.
```
TO 설정 페이지와 해제 페이지를 포함하려면, ...
    TO 설정 페이지를 포함하려면, ...
    To 해제 페이지를 포함하려면, ....
```
<br>

## Switch 문 <a name = "5"></a>
```java
public Money calculatePay(Employee e) throws InvalidEmployeeType {
	switch (e.type) { 
		case COMMISSIONED:
			return calculateCommissionedPay(e); 
		case HOURLY:
			return calculateHourlyPay(e); 
		case SALARIED:
			return calculateSalariedPay(e); 
		default:
			throw new InvalidEmployeeType(e.type); 
	}
}
```
위의 switch문은 문제가 있다.  
1. 직원 유형을 추가할 때마다 코드 수정이 일어난다.(OCP 위반)
2. 계산도 하고, Money도 생성한다. (SRP 위반)

이를 개선하면 다음과 같다.  
```java
public abstract class Employee {
	public abstract boolean isPayday();
	public abstract Money calculatePay();
	public abstract void deliverPay(Money pay);
}
---------------------------------------------------
public interface EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType; 
}
---------------------------------------------------
public class EmployeeFactoryImpl implements EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
		switch (r.type) {
			case COMMISSIONED:
				return new CommissionedEmployee(r) ;
			case HOURLY:
				return new HourlyEmployee(r);
			case SALARIED:
				return new SalariedEmploye(r);
			default:
				throw new InvalidEmployeeType(r.type);
		} 
	}
}
```
switch문을 추상 팩토리(인터페이스)에 숨기고 아무에게도 보여주지 않는다.  
팩토리는 switch문을 사용해 적절한 Employee 파생 클래스의 인스턴스를 생성한다.  
calculatePay등과 같은 함수는 다형성으로 인해 실제 파생 클래스의 함수되어 실행된다.  
객체 생성을 캡슐화화여 사용하는 측에서는 구체적으로 어떤 타입의 객체가 생성되었는지 알 필요도, 알 수도 없게 된다.  
보통 Switch문은 이렇게 객체 생성을 위임하는 팩토리에서 많이 사용된다.  
정리하면, __switch문은 다형적 객체를 생성하기 위한 경우에, 추상 팩토리 구현체 안에 숨기고 노출하지 않도록 사용한다.__  
이외에는 되도록이면 switch문을 사용하지 않는다.  
<br>

## 서술적인 이름을 사용하라! <a name = "6"></a>
서술적인 이름을 사용하면 개발자 머릿속에서도 설계가 뚜렷해지므로 코드를 개선하기 쉬워진다.  
이름이 길어도 괜찮다.  
길고 서줄적인 이름이 짧고 어려운 이름보다 좋다.  
길고 서술적인 이름은 길고 서술적인 주석보다 좋다.  

<br>

## 함수 인수 <a name = "7"></a>
함수에서 가장 이상적인 인수 개수는 0개(무항)다.  
차선으로는 1개이다.  
인수의 개수가 많아질수록 코드의 이해를 방해하는 요소가 될 뿐이다.  

### 많이 쓰는 단항 형식
+ 인수에 질문을 던지는 경우
    - boolean fileExists("MyFile")
+ 인수를 뭔가로 변환해 결과를 반환하는 경우
    - InputStream fileOpen(“MyFile”)
+ 이벤트 함수
    - 입력 인수로 시스템의 상태를 바꾼다.
    - 이벤트라는 사실이 코드에 명확히 드러나야 한다.

위의 경우가 아니라면 단항 함수는 가급적 피한다.  

### 플래그 인수
함수로 부울 값을 넘기는 것은 정말 끔찍하다.  
분명 코드를 보면 if문으로 true 일때는 이것을, false일 때는 저것을 하도록 코딩되어 있을 것이다.  
즉, 함수가 한꺼번에 여러 가지를 처리한다고 공표하는 셈이다.  

### 이항 함수
단항 함수보다 이해하기 어렵다.  
무조건 나쁜건 아니지만, 웬만하면 단항으로 바꾸도록 노력해야 한다.  
단, Point 클래스 같은 경우, 자연적인 순서가 있기 때문에 이항 함수가 적절하다.  

### 삼항 함수
이항 함수보다 이해하기 어렵다.  
순서, 주춤, 무시로 야기되는 문제가 두 배 이상 늘어나므로 신중히 고려해서 만들어야 한다.  

### 인수 객체
인수가 많아질 경우, 일부 인수를 독자적인 클래스 변수로 선언하자.  
예를 들면 x,y를 넘기는 것보다 Point로 넘기는 것이 더 낫다.  

### 인수 목록
인수가 가변적인 함수도 필요하다.  
String.format 메서드가 좋은 예시다.
```java
String.format("%s worked %2.f hours.",name,hours);

public String format(String format,Ojbect... args)
```
가변 인수 전부를 동등하게 취급하면 List형 인수 하나로 취급할 수 있다.  
즉, String.format은 사실상 이항 함수이다.  

### 동사 키워드
+ 단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야 한다.  
    - writeField(name)은 누구나 곧바로 이해한다.  
+ 함수 이름에 인수 이름을 넣는다.
    - assertEquals 보다 assertExpectedEqualsActual이 인수 순서를 기억할 필요가 없어 더 낫다.  

<br>

## 부수 효과를 일으키지 마라! <a name = "8"></a>
부수 효과는 거짓말이다.  
함수에서 한 가지를 하겠다고 약속하고선 남몰래 다른 짓을 하는 것이다.
```java
public class UserValidator {
	private Cryptographer cryptographer;
	public boolean checkPassword(String userName, String password) { 
		User user = UserGateway.findByName(userName);
		if (user != User.NULL) {
			String codedPhrase = user.getPhraseEncodedByPassword(); 
			String phrase = cryptographer.decrypt(codedPhrase, password); 
			if ("Valid Password".equals(phrase)) {
				Session.initialize();
				return true; 
			}
		}
		return false; 
	}
}
```
함수의 이름은 checkPassword로 패스워드가 맞는지만 확인하고 boolean 값을 리턴하는 작업만 한다면 한 가지 일만 하는 것이다.  
하지만 중간에 Session.initialize라는 다른 일을 한다.  
함수 이름만 봐서는 세션이 초기화되는 건지 아닌지 확인할 수 없다.  
만약 꼭 필요한 상황이라면 함수명을 checkPasswordAndInitializeSession이라고 지어야 한다.  
물론 한 가지만을 한다는 규칙을 위반한다.  

### 출력 인수
일반적으로 함수의 인수를 입력으로 해석한다.  
__출력 인수라고 하는 것은 입력 인수의 상태를 변경하는 것을 의미하고 이는 피해야 한다.__  
```java
appendFooter(s);
```
위의 코드를 봤을 때, s를 바닥글로 첨부하는 것인지, s에 바닥글을 첨부하는 것인지 모호하다.
```java
public void appendFooter(StringBuffer report)
```
함수의 선언부까지 확인하니 s가 출력인수라는 사실을 파악할 수 있다.  
함수 선언부를 찾아보는 행위는 코드를 보다가 주춤하는 행위와 동급이므로 인지적으로 거슬린다는 의미이다.  
이를 개선하면 다음과 같다.
```java
report.appendFooter()
```
일반적으로 출력 인수는 피해야 한다.  
__함수에서 상태를 변경해야 한다면 함수가 속해있는 객체의 상태를 변경하는 방식을 택한다.__  

<br>

## 명령과 조회를 분리하라! <a name = "9"></a>
함수는 객체 상태를 변경하거나 객체 정보를 반환하거나 둘 중 하나만 해야 한다.  
둘다 하는 예시를 보자.
```java
public boolean set(String attribute, String value)
```
위의 경우에는 속성 값 설정 성공 시, true를 반환하므로 괴상한 코드가 작성된다.  

<br>

## 오류 코드보다 예외를 사용하라! <a name = "10"></a>

### Try-Catch 블록 뽑아내기
```java
// bad
if (deletePage(page) == E_OK) {
	if (registry.deleteReference(page.name) == E_OK) {
		if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
			logger.log("page deleted");
		} else {
			logger.log("configKey not deleted");
		}
	} else {
		logger.log("deleteReference from registry failed"); 
	} 
} else {
	logger.log("delete failed"); 
	return E_ERROR;
}
```
deletePage는 명령이지만 == 을 통해 조회로도 사용되고 있다.  
제일 바깥의 if문을 보면 정상 동작과 오류 처리를 하고 있는 추한 구조이다.  
이를 개선해보자.
```java
public void delete(Page page) {
	try {
		deletePageAndAllReferences(page);
  	} catch (Exception e) {
  		logError(e);
  	}
}

private void deletePageAndAllReferences(Page page) throws Exception { 
	deletePage(page);
	registry.deleteReference(page.name); 
	configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) { 
	logger.log(e.getMessage());
}
```
if/else 블록은 별도 함수로 뽑아내고, try-catch 문으로 추한 구조를 해결했다.  

### 오류 처리도 한 가지 작업이다.
함수는 한 가지 작업만 해야 한다.  
오류 처리도 한 가지 작업에 속한다.  
따라서 오류를 처리하는 함수는 오류만 처리해야 한다.(위 예시에서 보았듯이)  
즉, __함수에 키워드 try가 있다면 함수는 try로 시작해서 catch/finally 문으로 끝나야 한다.__  

### Error.java 의존성 자석
```java
public enum Error { 
	OK,
	INVALID,
	NO_SUCH,
	LOCKED,
	OUT_OF_RESOURCES, 	
	WAITING_FOR_EVENT;
}
```
오류를 처리하는 곳곳에서 오류 코드를 사용한다면 분명 enum class를 사용하게 된다.  
하지만 이는 __의존성 자석(사용하는 곳마다 import) 일뿐만 아니라 새 오류 코드를 추가하거나 변경할 때마다 수정이 필요하다.__  

<br>

## 반복하지 마라! <a name = "11"></a>
중복은 모든 소프트웨어에서 악의 근원이므로 늘 중복을 없애도록 노력해야 한다.

<br>

## 구조적 프로그래밍 <a name = "12"></a>
모든 함수와 함수 내 모든 블록에 입구와 출구가 하나만 존재해야 한다.  
즉, __함수는 return문이 하나여야 한다.__  
루프 안에서 break, continue를 사용해선 안되며, goto(특정 줄번호나 레이블로 건너뛰는)는 __절대로__ 안 된다.  
__만약 함수를 작게 만든다면 간혹 return, break, continue를 여러 차례 사용해도 괜찮다.__  
때로는 단일 입/출구 규칙보다 의도를 표현하기 쉬워진다.  
반면에 goto문은 큰 함수에만 의미가 있으므로 작은 함수에서는 피해야만 한다.  
<br>

## 함수를 어떻게 짜죠? <a name = "13"></a>
처음에는 길고 복잡하고 인수도 길고 중복도 많다.  
하지만 이를 빠짐없이 테스트하는 단위 테스트 케이스를 만든다.  
그리고 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거한다.  
처음부터 탁 짜는게 가능한 사람은 없다.  














 
