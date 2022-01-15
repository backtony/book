## 목차
- [Intro](#1)
- [클래스 체계](#2)
	- [캡슐화](#3)
- [클래스는 작아야 한다](#4)
	- [단일 책임 원칙](#5)
  	- [응집도](#6)
  	- [응집도를 유지하면 작은 클래스 여럿이 나온다](#7)
- [변경하기 쉬운 클래스](#8)
	- [변경으로부터 격리](#9)
---



## Intro <a name = "1"></a>
이전 챕터에서는 코드 행, 코드 블록, 함수를 올바로 작성하고 구현하는 방식과 함수가 서로 관련을 맺는 방식을 공부했다.  
하지만 더 차원 높은 단계까지 신경쓰지 않으면 깨끗한 코드를 얻기 어렵다.  
이 장에서는 깨끗한 클래스를 다룬다.  
<br>

## 클래스 체계 <a name = "2"></a>
표준 자바 관례에 따르면 클래스 안의 변수 목록 순서는 다음과 같다.  

> static public 상수 - static private 변수 - private 인스턴스 변수 - public 변수(필요한 경우는 거의 없다.)

변수 목록 이후에는 public 함수가 나오고 private 함수는 자신을 호출하는 public 함수 직후에 나온다.  
즉, 추상화 단계가 순차적으로 내려간다.  

### 캡슐화 <a name = "3"></a>
변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫지만 반드시 숨겨야 하는 것은 아니다.  
우리에게 테스트는 중요하므로 테스트를 위해 protected로 선언해서 접근을 허용하기도 한다.  
하지만 비공개 상태를 유지할 온갖 방법을 강구하고, 캡슐화를 풀어주는 결정은 언제나 최후의 수단이다.  

<br>

## 클래스는 작아야 한다! <a name = "4"></a>
클래스는 함수와 마찬가지로 작아야한다.  
하지만 함수와는 다르게 기준이 물리적인 행 수가 아니라 __클래스가 맡은 책임__ 이다.  
```java
// 어마어마하게 큰 슈퍼 만능 클래스

public class SuperDashboard extends JFrame implements MetaDataUser {
	public String getCustomizerLanguagePath()
	public void setSystemConfigPath(String systemConfigPath) 
	public String getSystemConfigDocument()
	public void setSystemConfigDocument(String systemConfigDocument) 
	public boolean getGuruState()
	public boolean getNoviceState()
	public boolean getOpenSourceState()
	public void showObject(MetaObject object) 
	public void showProgress(String s)
	public boolean isMetadataDirty()
	public void setIsMetadataDirty(boolean isMetadataDirty)
	public Component getLastFocusedComponent()
	public void setLastFocused(Component lastFocused)
	public void setMouseSelectState(boolean isMouseSelected) 
	
	// ... many non-public methods follow ...
}
```
```java
// 메소드를 5개로 줄인다고 하더라도 여전히 책임이 많다..

public class SuperDashboard extends JFrame implements MetaDataUser {
	public Component getLastFocusedComponent()
	public void setLastFocused(Component lastFocused)
	public int getMajorVersionNumber()
	public int getMinorVersionNumber()
	public int getBuildNumber() 
}
```
클래스 이름은 해당 클래스 책임을 기술해야 한다.  
실제로 __작명은 클래스 크기를 줄이는 첫 번째 관문__ 이다.  
간결한 이름이 떠오르지 않는다면 클래스 크기가 너무 커서이고, 클래스 이름이 모호하다면 클래스 책임이 너무 많기 때문이다.
또한, 클래스 설명은 if, and, or, but을 사용하지 않고서 25단어 내외로 가능해야 한다.  

### 단일 책임 원칙(SRP) <a name = "5"></a>
단일 책임 원칙(SRP)는 클래스나 모듈을 변경할 이유가 단 하나뿐이어야 한다는 원칙이다.  
책임 즉, 변경할 이유를 파악하려 애쓰다 보면 코드를 추상화하기도 쉬워진다.  
```java
// 이 코드는 작아보이지만, 변경할 이유가 2가지이다.

public class SuperDashboard extends JFrame implements MetaDataUser {
	public Component getLastFocusedComponent()
	public void setLastFocused(Component lastFocused)
	public int getMajorVersionNumber()
	public int getMinorVersionNumber()
	public int getBuildNumber() 
}
```
```java
// 위 코드에서 버전 정보를 다루는 메서드 3개를 따로 빼서
// Version이라는 독자적인 클래스를 만들어 다른 곳에서 재사용하기 쉬워졌다.

public class Version {
	public int getMajorVersionNumber() 
	public int getMinorVersionNumber() 
	public int getBuildNumber()
}
```
SRP는 객체지향설계에서 더욱 중요한 개념이고, 지키기 수월한 개념인데, 개발자가 가장 무시하는 규칙 중 하나이다.  
대부분의 프로그래머들이 돌아가는 소프트웨어에 초점을 맞춘다.  
전적으로 올바른 태도이기는 하지만, 돌아가는 소프트웨어가 작성되면 깨끗하고 체계적인 소프트웨어라는 __다음 관심사로 전환__ 을 해야한다.  
많은 개발자들은 자잘한 단일 책임 클래스가 많아지면 큰 그림을 이해하기 어려워진다고 우려하지만, 실제로는 작은 클래스가 많은 시스템이든, 큰 클래스가 몇 개뿐인 시스템이든 돌아가는 부품은 그 수가 비슷하다.  

> 도구 상자를 어떻게 관리하고 싶은가? 작은 서랍을 많이 두고 기능과 이름이 명확한 컴포넌트를 나눠 넣고 싶은가? 아니면 큰 서랍 몇 개를 두고 모두 던져넣고 싶은가?

큰 클래스 몇 개가 아니라 작은 클래스 여럿으로 이뤄진 시스템이 더 바람직하다.  
작은 클래스는 각자 맡은 책임이 하나이며, 변경할 이유가 하나이며, 다른 작은 클래스와 협력해 시스템에 필요한 동작을 수행한다.  

### 응집도 <a name = "6"></a>
클래스는 인스턴스 변수의 수가 작아야하며, 각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야한다.  
일반적으로 메서드가 변수를 더 많이 사용할수록 메서드와 클래스는 응집도가 높아진다.  
모든 인스턴스 변수를 메서드마다 사용하는 클래스는 응집도가 가장 높다고 볼 수 있지만, 일반적으로 이처럼 응집도가 가장 높은 클래스는 가능하지도 바람직하지도 않다.  
하지만 __가능한한 응집도가 높은 클래스를 지향__ 해야 한다.  
__응집도가 높다는 말은 클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 의미이기 때문이다.__  
```java
// Stack을 구현한 코드, 응집도가 높은 편이다.

public class Stack {
	private int topOfStack = 0;
	List<Integer> elements = new LinkedList<Integer>();

	public int size() { 
		return topOfStack;
	}

	public void push(int element) { 
		topOfStack++; 
		elements.add(element);
	}
	
	public int pop() throws PoppedWhenEmpty { 
		if (topOfStack == 0)
			throw new PoppedWhenEmpty();
		int element = elements.get(--topOfStack); 
		elements.remove(topOfStack);
		return element;
	}
}
```
'함수를 작게, 매개변수 목록을 짧게'라는 전략을 따르다 보면 때때로 몇몇 메서드만 사용하는 인스턴스 변수가 아주 많아진다.  
이는 __새로운 클래스를 쪼개야 한다는 신호__ 로, 응집도가 높아지도록 변수와 메서드를 적절히 분리해 새로운 클래스 두 세개로 쪼개줘야 한다.  

### 응집도를 유지하면 작은 클래스 여럿이 나온다 <a name = "7"></a>
큰 함수를 작은 함수 여럿으로 나누기만 해도 클래스 수가 많아진다.  
예를 들어, 변수가 아주 많은 큰 함수 하나가 있다.  
큰 함수 일부를 작은 함수 하나로 빼내고 싶은데, 빼내려는 코드가 큰 함수에 정의된 변수 넷을 사용한다.  
그렇다면 네 개를 새 함수의 인자로 넘겨야 옳을까?  
아니다. 네 변수를 인스턴스 변수로 승격하면 새 함수는 인자가 필요 없다.  
그만큼 함수를 쪼개기 쉬워진다.  
하지만 이는 클래스 내에서 몇몇 함수만 사용하는 인스턴스 변수가 늘어나는 행위로 클래스가 응집력을 잃게 된다.  
몇몇 함수만이 몇몇 변수를 사용한다면 독자적인 클래스로 분리하여 클래스 응집력을 높이면 된다.  
__즉, 큰 함수를 작은 함수 여럿으로 쪼개다 보면 종종 작은 클래스 여럿으로 쪼갤 기회가 생기는 것이다.__  
이때는 가장 먼저 테스트 코드를 작성하고 하나씩 변경할 때마다 테스트를 수행해 동작을 검증해야한다.  

<br>

## 변경하기 쉬운 클래스 <a name = "8"></a>
대다수의 시스템은 지속적인 변경이 가해지기에 뭔가 변경할 때마다 시스템이 의도대로 동작하지 않을 위험이 따른다.  
깨끗한 시스템은 클래스를 체계적으로 정리해 변경에 수반하는 위험을 낮춘다.
```java
public class Sql {
	public Sql(String table, Column[] columns)
	public String create()
	public String insert(Object[] fields)
	public String selectAll()
	public String findByKey(String keyColumn, String keyValue)
	public String select(Column column, String pattern)
	public String select(Criteria criteria)
	public String preparedInsert()
	private String columnList(Column[] columns)
	private String valuesList(Object[] fields, final Column[] columns) 
    private String selectWithCriteria(String criteria)
	private String placeholderList(Column[] columns)
}
```
위 코드는 새로운 SQL을 추가할 때 Sql클래스를 손봐야하고, 기존 메서드를 수정할 때도 Sql클래스를 손봐야 한다.  
즉, 클래스를 변경해야 하는 이유가 2개가 되기 때문에 SRP 위반임과 더불어 새로운 기능 추가에 기존 Sql 클래스를 수정해야 하기에 OCP도 위반이다.  
또한, private 메서드가 일부의 public 메서드에서만 사용되고 있다면 코드 개선의 잠재적인 여지를 시사한다.  
위 코드를 개선해보자.  
```java
abstract public class Sql {
    public Sql(String table, Column[] columns) 
    abstract public String generate();
}
public class CreateSql extends Sql {
    public CreateSql(String table, Column[] columns) 
    @Override public String generate()
}

public class SelectSql extends Sql {
    public SelectSql(String table, Column[] columns) 
    @Override public String generate()
}

public class InsertSql extends Sql {
    public InsertSql(String table, Column[] columns, Object[] fields) 
    @Override public String generate()
    private String valuesList(Object[] fields, final Column[] columns)
}

public class SelectWithCriteriaSql extends Sql { 
    public SelectWithCriteriaSql(
    String table, Column[] columns, Criteria criteria) 
    @Override public String generate()
}

public class SelectWithMatchSql extends Sql { 
    public SelectWithMatchSql(String table, Column[] columns, Column column, String pattern) 
    @Override public String generate()
}

public class FindByKeySql extends Sql public FindByKeySql(
    String table, Column[] columns, String keyColumn, String keyValue) 
    @Override public String generate()
}

public class PreparedInsertSql extends Sql {
    public PreparedInsertSql(String table, Column[] columns) 
    @Override public String generate() {
    private String placeholderList(Column[] columns)
}

public class Where {
    public Where(String criteria) public String generate()
}

public class ColumnList {
    public ColumnList(Column[] columns) public String generate()
}
```
Sql 추상클래스를 만들고 기존의 기능들은 추상클래스를 파생하도록 클래스를 만들었다.  
일부에서만 사용하던 private 메서드는 해당 파생 클래스로 옮겼고, 모든 파생 클래스가 공통적으로 사용하는 private 메서드의 경우 where과 ColumnList라는 새로운 유틸 클래스에 넣었다.  
클래스가 극도로 단순해졌기에 이해하기 쉬워졌으며 SRP, OCP 원칙도 지킬 수 있게 되었다.  
__결론은 잘 짜여진 시스템은 추가와 수정에 있어서 건드릴 코드가 최소이다.__  

### 변경으로부터 격리 <a name = "9"></a>
자바에는 구체적인(concrete) 클래스와 추상(abstract) 클래스가 있다.  
구체적인(concrete) 클래스에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험에 빠진다.  
따라서 __인터페이스와 추상 클래스를 사용해 구현이 미치는 영향을 격리__ 시켜야 한다.  
더불어, 상세한 구현에 의존한 코드는 테스트가 어려운 반면에 추상화에 의존할 경우 테스트가 쉽다.  
예시를 하나 보자.  
Portfolio 클래스를 만드는데 이는 외부 TokyoStockExchange API를 사용해 포트폴리오 값을 계산한다. 
외부 API에서 제공하는 값이 5분마다 달라진다면 테스트 코드를 짜기 쉽지 않다.  
하지만 이는 API를 직접 호출하는 대신 인터페이스를 사용한다면 쉽게 테스트할 수 있다.  
```java
public interface StockExchange { 
	Money currentPrice(String symbol);
}
------------------------------------------
public Portfolio {
	private StockExchange exchange;
	public Portfolio(StockExchange exchange) {
		this.exchange = exchange; 
	}
	// ... 
}
------------------------------------------
public class PortfolioTest {
	private FixedStockExchangeStub exchange;
	private Portfolio portfolio;
	
	@Before
	protected void setUp() throws Exception {
		exchange = new FixedStockExchangeStub(); 
		exchange.fix("MSFT", 100);
		portfolio = new Portfolio(exchange);
	}

	@Test
	public void GivenFiveMSFTTotalShouldBe500() throws Exception {
		portfolio.add(5, "MSFT");
		Assert.assertEquals(500, portfolio.value()); 
	}
}
```
실제 코드에서는 StockExchange의 구현체로 TokyoStockExchange API를 호출하겠지만, 테스트 코드에서는 StockExchange의 구현체로 고정된 주가를 반환하도록 만들 수 있다.  
위와 같이 테스트가 가능할 정도로 시스템의 결합도를 낮추면 유연성과 재사용성도 더욱 높아진다.  
또한, 실제 주가를 얻어오는 출처나 얻어오는 방식 등과 같은 구체적인 사실을 모두 숨길 수 있다.  
<br>

__결합도가 낮다는 소리는 각 시스템 요소가 다른 요소로부터, 변경으로부터 잘 격리되어 있다는 의미다.__  
시스템 요소가 서로 잘 격리되어 있으면 각 요소를 이해하기도 더 쉬워진다.  
__결합도를 최소로 줄이면 자연스럽게 DIP(상세한 구현보다 추상화에 의존하라)를 따르는 클래스가 나오게 된다.__












