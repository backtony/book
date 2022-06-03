## 목차
- [Intro](#1)
- [아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라](#2)
- [아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하자](#3)
- [아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라](#4)
- [아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라](#5)
- [아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라](#6)
- [아이템 6. 불필요한 객체 생성을 피하라](#7)
- [아이템 7. 다 쓴 객체 참조를 해제하라](#8)
- [아이템 8. finalizer와 cleaner 사용을 피하라](#9)
- [아이템 9. try-finally보다는 try-with-resources를 사용하라](#10)

---


## Intro <a name = "1"></a>
---
객체를 만들어야 할 때와 만들지 말아야 할 때를 구분하는 법, 올바른 객체 생성 방법과 불필요한 생성을 피하는 방법, 제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 방식에 대해 알아본다.  
<br>

## 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라 <a name = "2"></a>
---
클래스의 인스턴스를 얻는 수단은 2가지가 있다.  
```java
public class Book {
    private String name;
    private String author;
    private String publisher;

    // 생성자를 이용한 객체 초기화
    public Book(String name) {
        this.name = name;
    }

    // 펙토리 메서드를 이용한 객체 초기화
    public static createBookWithName(name) {
        Book book = new Book();
        book.name = name;
        return book;
    }
}
```
이 방식에는 5가지 장점과 2가지 단점이 존재한다.  
우선적으로 정리하면 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적 장단점을 이해하고 사용하는 것이 좋다.  
하지만 대부분 정적 팩터리를 사용하는 것이 유리하니 public 생성자만 사용하던 습관이 있다면 고치자.  
이제 장단점에 대해 하나씩 살펴보자.  

### 장점1: 이름을 가질 수 있다
생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다.  
반면에 정적 팰터리는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.  
또한, 하나의 시그니처로는 생성자를 하나만 만들 수 있지만 정적 팩터리 메서드는 이에 대한 제약이 없다.  
풀어서 말하자면, 똑같은 타입에 인수명만 다른 생성자 오버로딩이 불가능하다는 것이다.

__예시 1: 이름으로 인한 반환 객체 설명__  
```java
public static void main(String[] args) {
    //생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 특성을 설명하지 못하고 인자가 무엇인지도 모름
    Book book = new Book("backtony");

    // 반활될 객체의 특성을 설명할 수 있음 -> 단번에 인자가 책의 이름이라는 것을 알 수 있음
    Book book = Book.createBookWithName("backtony");
}
```
<br>

__예시 2: 제한이 없는 정적 팩터리 메서드__  
```java
public class Book {
    private String name;
    private String author;
    private String publisher;

    // 생성자 방식
    public Book(String name) {
        this.name = name;
    }

    // 생성자 방식
    // 메소드 시그니처 중복으로 오류 발생 -> 똑같은 타입을 파라미터로 받는 오버로딩 생성자 불가능
    public Book(String author) { 
        this.author = author;
    }
    
    // 펙토리 메서드 방식
    public static createBookWithName(String name) {
        Book book = new Book();
        book.name = name;
        return book;
    }

    // 펙토리 메서드 방식
    public static createBookWithAuthor(String author) {
        Book book = new Book();
        book.author = author;
        return book;
    }
}
```

### 장점 2: 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다
정적 팩터리 메서드를 통해 인스턴스 캐싱과 인스턴스 통제를 할 수 있다.  
+ 인스턴스 캐싱
    - 인스턴스를 미리 생성해두고 필요할 때마다 가져다쓰는 방식
    - 불필요한 객체 생성을 피하여 자주 요청되는 상황에서 성능 향상
+ 인스턴스 통제
    - 인스턴스의 생명주기를 통제하는 것을 의미
    - 인스턴스를 통제하는 이유
        - 싱글턴을 만들기 위해
        - 인스턴스화 불가로 만들기 위해
        - 불변 클래스에서 동치인 인스턴스가 하나뿐임을 보장

<br>

__예시: 인스턴스 캐싱__  
```java
public final class Boolean implements java.io.Serializable, Comparable<Boolean> {
    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);

    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
    ...
}
```

### 장점 3: 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다
반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 유연성이 생긴다.  
```java
static class CARD{
    public static final String SAMSUNG_CARD = "삼성";
    public static final String KAKAO_CARD="카카오";

    static Payment payment(String card){
        switch (card){
            case SAMSUNG_CARD:
                // 3. 반환 타입의 하위타입 객체를 반환할 수 있는 능력이 있다.
                return new SamSungPayment();
            case KAKAO_CARD:
                return new KakaoPayment();
        }
        throw new IllegalArgumentException();
    }

}
```
Payment라는 상위타입 Interface로 하위 타입인 SamSungPayment 클래스를 반환할 수 있다.  

<br>

__cf) 인터페이스 default, static__  
자바 8부터 인터페이스에 default와 static 메서드를 사용할 수 있게 되었다.  
+ default
    - 인터페이스에 메서드 자체를 구현한다.
    - 구현체에서 바로 사용할 수도, 재정의하여 사용할 수도 있다.
    - 여러 인터페이스의 디폴트 메서드가 충돌하는 경우, 디폴트 메서드를 재정의 해야 한다.
    - 인터페이스의 디폴트 메서드와 상위 클래스의 메서드가 충돌하는 경우, 디폴트 메서드를 재정의 해야 한다.
+ static
    - 인스턴스 생성과 관계없이 인터페이스 타입으로 호출하는 메서드
    - static 예약어를 사용하고 접근 제어자는 항상 public으로 생략 가능
    - 정적 메서드를 사용할 때는 인터페이스를 직접 참조하여 사용한다.

### 장점 4: 입력 매개변수에 따라 매번 다른 클래스 객체를 반환할 수 있다
장점3의 연장선에 있는 내용이다.  
```java
static class CARD{
    public static final String SAMSUNG_CARD = "삼성";
    public static final String KAKAO_CARD="카카오";
    static Payment payment(String card){ // 들어오는 card 의 값 에 따라 반환하는 객체가 달름
        switch (card){
            case SAMSUNG_CARD:
                return new SamSungPayment();
            case KAKAO_CARD:
                return new KakaoPayment();
        }
        throw new IllegalArgumentException();
    }
}
```
매개변수에 따라 다른 구현체가 반환된다.  


### 장점 5: 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다
책에서는 JDBC를 예로 들어 설명한다.  
정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다 라는 말의 의미는
반환할 객체를 펙토리 메서드 방식을 이용하여 만들기 때문에, JDBC 에서도 각 상황에 따라서 펙토리 메서드 내용만 바꿔서
연결에 필요한 객체를 얻을 수 있다는 말이다.  


### 단점 1: 하위 클래스를 만들 수 없다
상속을 하려면 public이나 protected 생성자가 필요하다.  
따라서 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

### 단점 2: 정적 팩터리 메서드는 프로그래머가 찾기 어렵다
생성자처럼 API 설명에 명확히 드러나지 않기 때문에 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 찾아야만 한다.  
이에 대한 해결책은 널리 알려진 규악을 따라 짓는 식으로 명명하는 방안이다.  
+ from
    - 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
    - ex) Date d = Date.from(instant);
+ of
    - 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
    - ex) Set\<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
+ valueOf
    - from과 of의 더 자세한 버전
    - BigInetger prime = BigInteger.valueOf(Integer.MAX_VALUE);
+ instance, getInstance
    - 매개변수를 받는다면 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.
    - Stackwalker luke = StackWalker.getInstance(options);
+ create, newInstance
    - instance, getInstance와 같지만 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
+ getType
    - getInstance와 같으나 생성할 클래스가 아닌 다른 클래스에서 팩터리 메서드를 정의할 때 사용한다.
    - Type은 반환할 객체의 타입이다.
    - ex) FileStore fs = Files.getFileStore(path);
+ newType
    - newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에서 팩터리 메서드를 정의할 때 사용한다.
    - Type은 반환할 객체의 타입이다.
    - ex) BufferedReader br = Files.newBufferedReader(path);
+ type
    - getType, newType의 간결 버전
    - List\<Complaint> litany = Collections.list(legacyLitany);

<br>

## 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하자 <a name = "3"></a>
---
정적 팩터리와 생성자는 선택적 매개변수가 많을 때 대응하기 어렵다.  
이 경우에는 점측적 생성자 패턴, 자바 빈즈 패턴, 빌더 패턴을 고려해 볼 수 있다.  

### 점측정 생성자 패턴
+ 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개 변수 1개를 받는 생성자, 필수 매개변수와 선택 매개 변수 2개를 받는 생성자 ... 선택 매개 변수를 전부 받는 생성자 까지 늘려가는 방식을 의미한다.  
+ 확장하기 어렵고 매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다는 단점이 있다. 

```java
public class NutritionFacts {
	private final int servingSize;  // 필수
	private final int servings;     // 필수
	private final int calories;     // 선택
	private final int fat;          // 선택
	private final int sodium;       // 선택
	private final int carbohydrate; // 선택

	public NutritionFacts(int servingSize, int servings) {
		this(servingSize, servings, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories) {
		this(servingSize, servings, calories, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat) {
		this(servingSize, servings, calories, fat, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
		this(servingSize, servings, calories, fat, 0);
	}

		public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
		this.servingSize = servingSize;
		this.servings = servings;
		this.calories = calories;
		this.fat = fat;
		this.sodium = sodium;
		this.carbohydrate = carbohydrate;
	}
}
```


### 자바 빈즈 패턴
+ 매개변수가 없는 생성자로 객체를 만든 후, Setter메서드를 호출해 원하는 매개변수의 값을 설정하는 방식이다.  
+ 인스턴스를 만들기 쉽고 가독성이 좋아진다.
+ 객체 하나를 만들기 위해 여러 Setter메서드를 호출해야하고 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓인다는 단점이 있다.
+ 불변으로 만들 수 없으며 스레드 안정성을 위한 추가 작업이 필요하다.

```java
public class NutritionFacts {
	private int servingSize = -1;  // 필수
	private int servings = -1;     // 필수
	private int calories = 0;
	private int fat = 0;
	private int sodium = 0;
	private int carbohydrate = 0;

	public NutritionFacts() {}

	public void setServingSize(int val) { servingSize = val; }
	public void setServings(int val) { servings = val; }
	public void setCalories(int val) { calories = val; }
	public void setFat(int val) { fat = val; }
	public void setSodium(int val) { sodium = val; }
	public void setCarbohydrate(int val) {carbohydrate = val; }
}
```

### 빌더 패턴
+ 필수 매개변수와 생성자(혹은 정적 팩터리)를 통해 객체 생성을 위한 빌더 객체를 얻고, 빌더 객체가 제공하는 일종의 Setter 메서드들로 원하는 선택 매개변수를 설정하고 마지막에 build 메서드를 호출해 객체를 만드는 방식이다.
+ 빌터의 Setter 메서드들은 Builder 자신을 반환하기 때문에 메서드 연쇄가 가능하다.
+ 점층적 생성자 패턴의 안정성(유효성 검사)과 자바 빈즈 패턴의 가독성이라는 장점을 갖는다.
+ 빌더 하나로 순회하며 만들 수 있고, 매개변수에 따른 다른 객체를 만드는 등 유연하게 사용할 수 있따.
+ 객체를 만들기 위해 빌더부터 만들어야 하므로 성능에 민감한 상황에서 문제가 될 수 있다.
+ 점층적 생성자 패턴보다 코드가 장황해서 매개변수가 4개 이상일 때 유용하나 Lombok 라이브러리의 @Builder를 사용하면 해결할 수 있는 문제다.

```java

public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        private final int servingSize;  // 필수
        private final int servings;     // 필수
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.fat;
        carbohydrate = builder.carbohydrate;
    }
}
```

### 빌더 패턴 -  계층적 빌더
+ 빌더 패턴은 계층적으로 설계된 클래스와 함께 사용하기 좋다.
+ 추상 클래스에는 추상 빌더를, 구체 클래스에서는 구체 빌더를 작성하고 하위 클래스의 build 메서드는 구체 클래스를 반환하도록 선언하는 방식이다.

```java
public abstract class Member {
    // 모든 서브 타입 객체에 공통적으로 필요한 타입
    public enum Authority { READ, WRITE, DELETE }
    final Set<Authority> authorities;

    // 재귀적 타입 파라미터를 가진 제너릭 타입
    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Authority> authorities = EnumSet.noneOf(Authority.class);
        // 서브타입에서 권한을 정의할 수 있음
        public T addAuthorities(Authority types) {
            authorities.add(Objects.requireNonNull(types));
            return self();
        }

        abstract Member build();

        // 하위 클래스는 반드시 자기 자신을 반환하는 메서드를 오버라이드 해야 한다.
        protected abstract T self();
    }

    Member(Builder<?> builder) {
        authorities = builder.authorities.clone();
    }
}
```
```java
public class Guest extends Member {
    private final String name;

    public static class Builder extends Member.Builder<Builder> {
        private final String name;

        // 1. Guest 객체가 가져야할 인스턴스 변수
        public Builder(String name) {
            this.name = Objects.requireNonNull(name);
        }

        // 2. 마지막으로 호출되는 빌드 완성 메서드
        // - 공변 반환 타이핑(covariant return typing)을 사용하면, 빌더를 사용하는 클라이언트는 캐스팅이 필요없다.
        @Override
        Guest build() {
            return new Guest(this);
        }

        // 3. 부모 타입에서 필요한 서브 타입의 참조
        // 부모타입의 addAuthorities를 호출하고 self를 반환하게 만들어 하위 클래스에서 다시 형변환을 하지 않고도 연쇄 가능
        @Override
        protected Builder self() {
            return this;
        }
    }

    private Guest(Builder builder) {
        super(builder);
        name = builder.name;
    }
}
```
<br>

## 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라 <a name = "4"></a>
---
### 싱글턴
인스턴스를 오직 하나만 생성할 수 있는 클래스를 의미한다.  
+ 사용처
    - 무상태 객체
    - 설계상 유일해야 하는 시스템 컴포넌트
+ 주의사항
    - 상태를 가진 객체는 싱글턴으로 만들면 안 된다.
    - 멀티 스레드 환경을 가정해보면 이를 전역에서 접근하고 각기 다른 스레드에서 상태를 마구잡이로 변경할 여지가 있기 때문이다.
+ 장점
    - 한번의 객체 생성으로 재사용이 가능해 메모리 낭비를 방지
    - 전역성을 갖기 때문에 다른 객체와 공유가 용이
+ 단점
    - 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가짜 구현(mock)으로 대체할 수 없기 때문에 클래스를 싱글턴으로 만들면 테스트하기 어렵다.

```java
// 클래스를 싱글턴으로 만든 경우
public class Printer {
    private static final Printer INSTANCE = new Printer();

    public static final Printer getInstance() {
        return INSTANCE;
    }

    private int maxPrintPage;
    private Printer() {
        this.maxPrintPage = 50;
    }

    public void print(int printPage) {
        if (prnitPage > maxPrintPage) {
            throw new Exception();
        }
    }
}

// 테스트하기 어렵다. 
class Service {
    private final Printer printer = Printer.getInstance();

    // printer의 print 테스트
    // maxPrintPage 값 바꿔 테스트하고 싶을 경우엔....
    public void print(int printPage){
        printer.print(printPage);
    }
}
```
```java
// 인터페이스의 구현체를 싱글턴으로 만든 경우
public interface Printer {
    void print(int printPage);
}

class RealPrinter implements Printer {
    private static final Printer INSTANCE = new RealPrinter();

    public static final Printer getInstance() {
        return INSTANCE;
    }

    private int maxPrintPage = 50;
    private Printer() { }

    @Override
    public void print(int printPage) {
        if (prnitPage > maxPrintPage) {
            throw new Exception();
        }
    }
}

// 테스트하기 위한 목 객체를 만들어 사용할 수 있다.
class MockPrinter implements Printer {
    private static final Printer INSTANCE = new MockPrinter();
    
    public static final Printer getInstance() {
        return INSTANCE;
    }
    private int maxPrintPage;
    private MockPrinter() { }

    // 테스트를 위한 코드
    public void changeMaxPrintPage(int mockPrintPage){
        this.maxPrintPage = mockPrintPage;
    }

    @Override
    public void print(int printPage) {
        if (prnitPage > maxPrintPage) {
            throw new Exception();
        }
    }
}
```

### 만드는 방식
#### public static final 필드 방식
```java
public class Printer {
    public static final Printer INSTANCE = new Printer();
    private Printer() { ... }

    public void print() { ... }
}
```
해당 클래스가 싱글턴임이 API에 명백히 드러나고 간결하다.  
예외적으로 권한이 있는 클라이언트는 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다.  
이러한 공격을 방지하고자 한다면, 생성자에서 두 번 객체를 생성하려고 할 때 예외를 던지게 하면 된다.  

#### 정적 팩터리 방식
```java
public class Printer {
    private static final Printer INSTANCE = new Printer();
    private Printer() { ... }
    public static Printer getInstance() { return INSTANCE; }

    public void print() { ... }
}
```
+ API를 바꾸지 않고도 싱글턴이 아니게 변경할 수도 있다.
    - 호출하는 스레드 별로 다른 인스턴스를 넘기게 하는 등
+ 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
+ 마찬가지로 리플랙션 API에 의한 예외적인 상황이 발생할 수 있다.

<br><br>

앞선 두가지 방식으로 만들어진 싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현하는 것만으로는 부족하다.  
모든 인스턴스 필드에 transient를 선언하고, readResolve 메서드를 제공해야만 역직렬화시에 새로운 인스턴스가 만들어지는 것을 방지할 수 있다.  
만약 이렇게 하지 않으면 초기화해둔 인스턴스가 아닌 다른 인스턴스가 반환된다.  
```java
private Object readResolve() {
    return INSTANCE;
}
```

#### 열거 타입 방식의 싱글턴
```java
public enum Printer {
    INSTANCE;

    public void print() { ... }

    public static void main(String[] args){
        Printer printer = Printer.INSTANCE;
        printer.print();
    }
}
```
+ 앞의 예제들에 비해 더 간결하고, 추가 노력 없이 직렬화할 수 있다.
+ 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 막아준다.  
+ 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.  
+ 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.
    - enum 클래스는 내부적으로 Enum\<T>를 상속받고 있기 때문에 다른 클래스를 상속할 수 없다.
    - 단, 다른 인터페이스를 구현하도록 선언할 수는 있다.



### 싱글턴이 안티패턴이라고 불리는 이유
+ SOLID 원칙을 지키기 위해서는, 인터페이스를 사용하여 실제 구현체의 코드가 변경되더라도 이를 사용한 클라이언트 쪽에서는 코드에 영향을 받지 않아야 하는데 싱글톤 방식은 대부분 인터페이스를 사용하지 않는다.
+ 싱글턴을 사용하는 곳과 싱글턴 클래스 사이에 의존성이 생기게 되고 클래스 사이의 강한 의존성, 높은 결합이 생기면 수정, 단위 테스트의 어려움 등의 문제가 발생한다.
+ private 생성자를 갖기 때문에 상속할 수 없다.
+ private 생성자를 두었어도 reflection을 통해 하나 이상의 오브젝트가 만들어질 수 있다.

<br>

## 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라 <a name = "5"></a>
---
+ 정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 사용하라고 설계한 것이 아니다. 
    - 예를 들어 java.lang.Math 같은 경우 private 생성자로 인스턴스화를 막아두고 정적 메서드를 제공해 어디서든 사용될 수 있으며 그대로 재사용이 가능하다.
+ 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어 주므로 인스턴스화를 막기 위해서는 private을 명시한 생성자가 필요하다.

### 주의 사항
+ 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다. 하위 클래스를 만들어 인스턴스화하면 되기 때문이다.
+ 다른 개발자들이 코드를 보았을 때 상속해서 쓰라는 의미로 오해할 수 있다.

```java
/* 추상 클래스 */
public class AbstractPrivateConstructorTest {
    
}

/* 하위 클래스 */
public class PrivateConstructorTest extends AbstractPrivateConstructorTest {
    public PrivateConstructorTest() { }
}

/* 테스트 */
@Test
void 추상클래스_Private_생성자_테스트() {
    PrivateConstructorTest privateConstructorTest = new PrivateConstructorTest();

    assertThat(privateConstructorTest).isInstanceOf(AbstractPrivateConstructorTest.class);
}
```

### 해결책
+ 모든 생성자는 명시적이든 묵시적이든 상위 클래스의 생성자를 호출한다.
+ 따라서 private으로 생성자를 만들어 자동으로 public 기본 생성자가 생성되는 것을 막고 하위 클래스에서 생성자에 대한 접근을 막는다.

```java
// 내부에서 혹시 모를 생성자 호출을 대비해 에러 코드를 담아 놓는다.
public class CustomStringUtils {
    
    private CustomStringUtils() {
        throw new IllegalStateException("유틸리티 클래스를 인스턴스화할 수 없습니다!");
    }

    public static boolean isBlank(String input) {
        return input == null || input.trim().isEmpty();
    }    
    ...
}
```
```java
// 주석으로 생성자를 호출하지 말라고 달아놓는다.
public final class Math {

    /**
     * Don't let anyone instantiate this class.
     */
    private Math() {}

    public static final double E = 2.7182818284590452354;
    public static final double PI = 3.14159265358979323846;
    
    // ...

    public static int max(int a, int b) {
        return (a >= b) ? a : b;
    }
}
```
<Br>

## 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라 <a name = "6"></a>
---
### 문제점
맞춤법 검사기를 구현한다 했을때, 아래와 같이 정적 유틸리티 클래스나 싱글톤으로 구현하는 경우가 많다.  
__정적 유틸리티 클래스 방식__
```java
public class SpellChecker {

    private static final Lexicon DICTIONARY = ...; //직접생성

    private SpellChecker() { //객체 생성 방지
    }

    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String typo) {...}
}
```
<br>

__싱글턴 방식__
```java
public class SpellChecker {

    private final Lexicon dictionary = ...; //직접생성

    private SpellChecker(...) {...}
    public static SpellChecker INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) {...}
    public List<String> suggestions(String typo) {...}
}
```
두 방식의 문제점은 다음과 같다.
+ dctionary(자원)에 의존하고 있다.
+ 언어별 사전, 특수 어휘용 사전이 필요하게 되면 클래스 내부 메서드 동작이 달라져야 한다.
+ 유연하지 않고 테스트하기 어렵다.

따라서 정적 유틸리티 클래스나 싱글턴 방식은 적합하지 않다.

### 해결책
클래스가 여러 자원을 지원하고, 클라이언트가 원하는 자원을 사용하게 하는 방법은 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 것이다.  
이것을 __의존 객체 주입__ 방식이라고 한다.  

+ 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.
+ 의존 객체가 불변이라면, 여러 클라이언트가 의존 객체들을 공유할 수 있다.
+ 의존 객체 주입 패턴의 변형으로는 생성자에 자원 팩터리를 넘겨주는 방식이 있다.
    - 팩터리 : 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체 
    - 보통 Suppliert\<T> 를 이용한다.

__의존 객체 주입 방식__  
```java
// 의존 객체 주입은 유연성과 테스트 용이성을 높여준다 
public class SpellChecker{
	private final Lexicon dictionary;
	public SpellChecker(Lexicon dictionary){
		this.dictionay = Objects.requiredNonNull(dictionay);
	}
	public boolean isValid(String word){...}
	public List<String> suggestions(String typo){...}
}
```
<br>

이 패턴의 쓸만한 변형으로, 생성자에 자원 팩터리를 넘겨주는 방식이 있다. 팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다.  
즉, 팩터리 메서드 패턴(Factory Method Pattern)을 구현한 것이다.  
자바 8에서 소개한 Supplier 인터페이스가 팩터리를 표현한 완벽한 예이다.  
Supplier를 입력으로 받는 메서드는 일반적으로 한정적 와일드카드 타입 (bounded wildcard type)을 사용해 팩터리의 타입 매개변수를 제한해야 한다.  
이 방식을 사용해 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다.  
예컨대 다음 코드는 클라이언트가 제공한 팩터리가 생성한 타일(Tile)들로 구성된 모자이크(Mosaic)를 만드는 메서드다.  
```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

<br>

__팩터리 메서드 패턴 예시__  
```java
abstract class Product {
    public abstract void use();
}
```
```java
class IDCard extends Product {
    private String owner;

    public IDCard(String owner) {
        System.out.println(owner + "의 카드를 만듭니다.");
        this.owner = owner;
    }

    @Override
    public void use() {
        System.out.println(owner + "의 카드를 사용합니다.");
    }

    public String getOwner() {
        return owner;
    }
}
```
```java
abstract class Factory {
    public final Product create(String owner) {
        Product p = createProduct(owner);
        registerProduct(p);
        return p;
    }
    protected abstract Product createProduct(String owner);
    protected abstract void registerProduct(Product p);
}
```
```java
class IDCardFactory extends Factory {
    private List<String> owners = new ArrayList<>();

    @Override
    protected Product createProduct(String owner) {
        return new IDCard(owner);
    }

    @Override
    protected void registerProduct(Product p) {
        owners.add(((IDCard) p).getOwner());
    }

    public List<String> getOwners() {
        return owners;
    }
}
```
```java
Factory factory = new IDCardFactory();
Product card1 = factory.create("홍길동");
Product card2 = factory.create("이순신");
Product card3 = factory.create("강감찬");
card1.use();
card2.use();
card3.use();
```
```java
홍길동의 카드를 만듭니다.
이순신의 카드를 만듭니다.
강감찬의 카드를 만듭니다.
홍길동의 카드를 사용합니다.
이순신의 카드를 사용합니다.
강감찬의 카드를 사용합니다.
```


의존 객체 주입은 유연성과 테스트 용이성을 개선해주지만, 의존성이 많은 경우 코드를 어지럽게 할 수 있다.  
의존 객체 주입 프로임워크(스프링, 주스 등)을 사용하면 해결할 수 있는 문제다.  

### 정리
클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.  
이 자원들을 클래스가 직접 만들게 해서도 안 된다.  
대신 필요한 자원을 생성자에 넘겨주자.  
의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.  
<br>

## 아이템 6. 불필요한 객체 생성을 피하라 <a name = "7"></a>
---
같은 기능을 하는 객체를 사용할 때마다 새로 만드는 것보다 객체 하나를 재사용하는 편이 낫다.

### String pool
```java
String s1 = new String("java");
String s2 = new String("java");

System.out.println(s1 == s2); // false

String s3 = "java";
String s4 = "java";

System.out.println(s3 == s4); // true
```
s1, s2의 생성 방법은 java라는 문자열을 매번 새로 생성하지만,  
s3, s4는 String Constant Pool을 사용하여 java라는 문자열을 캐싱하여 사용한다.  

### 정적 팩터리 메서드 사용
Wrapper Class에서는 캐싱을 지원해주는 valueOf()라는 메소드가 존재한다.  
대표적으로 자주 사용하는 Boolean의 경우에는 true/false를, Short, Integer와 Long의 경우에는 -128 ~ 127까지의 수의 캐싱을 지원해준다.  
즉, 매번 새로 생성되는게 아니라는 의미이다.  

### 생성 비용이 비싸다면
생성 비용이 비싼 객체들이 있다.  
비싼 객체가 반복해서 필요하다면 캐싱해서 재사용하길 권장한다.  
비싼 객체는 다음과 같다.  
+ 데이터 크기가 크거나 객체 내부에 여러 객체를 포함하는 경우
+ 연관관계가 복잡한 경우
+ 시스템 자원을 많이 먹는 경우

```java
// 반복 사용에 부적합하다.
static boolean isRomanNumeralSlow(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}

// 미리 컴파일 해놓은 객체를 불러 사용하는 방식으로 개선했다 
private static final Pattern ROMAN = Pattern.compile(
    "^(?=.)M*(C[MD]|D?C{0,3})"
        + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

static boolean isRomanNumeralFast(String s) {
    return ROMAN.matcher(s).matches();
}
```

### 오토 박싱
오토 박싱이란 기본 타입과 박싱 타입을 섞어서 사용할 때 자동으로 상호 변환해주는 기능이다.  
```java
Integer i = 1;
```
위와 같이 i는 Integer로 박싱타입을 가지지만 대입은 기본 int 타입인 1을 하고 있다.  
이경우 Java는 자동으로 1이라는 int를 Integer로 변환한다.  
겉으로보면 큰 문제는 없어보이지만 이렇게 타입을 변환하는데에도 비용이 든다.
따라서 불필요한 박싱 객체 사용보다는 기본 타입을 권장하며 의도치 않은 오토박싱이 일어나지 않도록 주의해야한다.  

### 불필요한 객체의 생성을 피하라는 것이지 객체 생성을 피하라는 말이 아니다
VM에서 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 것은 크게 부담이 되지 않는 일이다.  
또한 프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것은 일반적으로 좋다.  
특히 값 객체 VO, value object로써 객체 자체를 값으로 사용하는 방식은 Thread Safe하므로 멀티스레드 환경에서 안전한 프로그래밍을 할 수 있다.  
단순히 객체 생성을 피하고자 객체 풀을 만드는 것 또한 지양해야한다.  
객체 풀은 코드를 헷갈리게 만들 뿐만 아니라 메모리 사용량을 늘리고 성능을 떨어뜨릴 수 있다.  
<br>

## 아이템 7. 다 쓴 객체 참조를 해제하라 <a name = "8"></a> 
---
JVM은 GC가 메모리를 관리해주지만 GC가 있다고 해서 메모리 관리를 개발자가 신경쓰지 않아도 되는 것은 아니다.

### 문제점
```java
public class Stack {
	private static final int DEFAULT_INITAL_CAPACITY = 16;

	private Obejct[] elements;
	private int size = 0;
	
	public Stack() {
		elements = new Object[DEFAULT_INITAL_CAPACITY];
	}

	public void push(Object e) {
		ensureCapacity();
		elements[size++];
	}

	public Object pop() {
		if (size == 0) {
			throw new EmptyStackException();
		}
		return elements[--size];
	}

	private void ensureCapacity() {
		if (elements.length == size) {
			elements = Arrays.copyOf(elements, 2 * size + 1);
		}
}
```
봤을 때 별 문제가 없다고 생각할 수도 있지만, pop()의 기능은 Stack의 특징은 LIFO 처럼 가장 최근에 들어온 요소가 반환되고 삭제되는 메서드이다.  
elements[--size] 이와 같이 반환값을 해놓는다면 실제 값은 삭제되지 않고 인덱스만 한칸씩 이동하는 것으로 메모리 누수가 발생하게 된다.  
바로 다 쓴 참조(obsolete reference)를 여전히 가지고 있다는 것이다. 
즉, elements 배열의 활성 영역 밖의 참조들이 모두 여기에 해당한다는 것이다.  
GC는 의도치 않게 객체를 살려두는 메모리 누수를 찾는 것이 아주 까다롭다.  
객체 참조 하나를 살려두면 GC는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체를 회수해가지 못하고 성능에 영향을 줄 수 있다.  

### 해결책
해당 참조를 다 쓰면 null(참조 해제)로 초기화 해주면 된다.  
```java
public Object pop() {
	if (size == 0) {
		return new EmptyStackException();
	}
	Object result = elements[--size];
	elements[size] = null;
	return result;
```
null을 사용함으로써 또 다른 이점은 해당 요소가 없는(null 처리한 참조) 메모리 공간을 사용하려고 하면 프로그램은 즉시 NPE를 발생시키며 프로그램을 종료할 것이다.  

### Stack 클래스가 메모리 누수에 취약한 이유
+ 스택이 자기 자신의 메모리를 직접 관리 하기 때문이다.
+ Stack 클래스는 배열로 저장소 풀을 만들어 원소들을 직접 관리한다.
+ 배열의 활성 영역부분에 속한 원소들은 사용되고, 비활성 영역은 쓰이지 않는데 문제점은 비활성 영역을 가비지 컬렉터가 알 방법이 없다는 것이다.
+ 보통 자신의 메모리를 직접 관리하는 클래스는 프로그래머가 항상 메모리 누수에 주의해야 한다.

### 모든 것을 null 처리해야 하는가?
모든 객체를 null로 만들면 프로그램을 필요 이상으로 지저분하게 만들 뿐이다.  
객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.  
즉, 참조 해제의 가장 좋은 방법은 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이다.  
이 내용은 아이템 57. 지역변수의 범위를 최소화하라를 참고하면 이해할 수 있다.  

### WeakHashMap & LinkedHashMap
캐시, 리스너, 콜백의 메모리 누수를 해결하기 위한 컬렉션인 약한 참조 해시맵에 대해 알아보자.

#### 강한 잠조 vs 부드러운 참조 vs 약한 참조
+ 강한 참조란 Integer prime = 1 와 같이 일반적인 유형의 참조이다. 이때 GC의 대상이 되지 않는다.
+ 부드러운 참조란 SoftReference\<Integer> soft = new SoftReference<>(prime); 와 같이 더 이상 원본은 없고 대상을 참조하는 객체만 존재할 경우 GC대상으로 들어가도록 JVM은 동작한다.
+ 약한 참조란 WeakReference weak = new WeakReference<>(prime);와 같이 prim가 null이 되면 GC의 대상이 된다.

#### WeakHashMap
```java
public static void main(String[] args) {
    Map<Integer, String> map = new WeakHashMap<>();

    Integer key1 = 1000;
    Integer key2 = 2000;

    map.put(key1, "test a");
    map.put(key2, "test b");

    key1 = null;

    System.gc();  //강제 Garbage Collection

    map.entrySet().stream().forEach(el -> System.out.println(el)); // 2000=test b
}
```
key1이 gc 처리 되었다.

#### LinkedHashMap
캐시에 새로운 항목이 추가될 때 removeEldestEntry 메소드를 실행하는데 이게 가장 오래된 캐시를 제거하는 것이다.
```java
public static void main(String[] args) {
    LinkedHashMap<Integer, Integer> map = new LinkedHashMap<>(1000, 0.75f, true) {

        private final static int MAX = 10;

        protected boolean removeEldestEntry(java.util.Map.Entry<Integer, Integer> eldest) {
            return size() >= MAX;
        }
    };

    for (int i = 0; i < 20; i++) {
        map.put(i, i);
    }

    for (Map.Entry<Integer, Integer> string : map.entrySet()) {
        System.out.println(string);
    }
}

// 출력
11=11
12=12
13=13
14=14
15=15
16=16
17=17
18=18
19=19
```

### 메모리 누수의 주범
+ Stack 처럼 자기 메모리를 직접 관리하는 경우
+ 캐시
    - 객체 참조를 캐시에 넣고 객체를 다 쓴 이후에도 그냥 두는 경우
    - 해결 방법
        - 캐시 외부에서 키를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 경우 WeakHashMap을 사용한다.
        - 쓰지 않는 엔트리를 청소한다.
            - ScheduledThreadPoolExecutor와 같은 백그라운드 스레드를 활용하거나, 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하는 방법을 이용하면 된다.
            - LinkedHashMap은 removeEldestEntry 메서드를 사용해 후자의 방식으로 처리한다.
+ 리스너 혹은 콜백
    - 콜백이란 이벤트가 발생하면 특정 메소드를 호출해 알려주는 것이다.(1개)
    - 리스너는 이벤트가 발생하면 연결된 리스너(핸들러)들에게 이벤트를 전달한다.(n개)
    - 클라이언트가 콜백을 등록만 하고 해지하지 않는다면 콜백은 쌓이게 될 것이다.
    - 이럴 때 콜백을 약한 참조(weak reference)로 저장하면 GC가 즉시 수거해간다.
    - 예를 들어 WeakHashMap에 키로 저장해두면 된다.


<br>

## 아이템 8. finalizer와 cleaner 사용을 피하라 <a name = "9"></a>
---
+ Finalizer
    - Object에 존재하는 finalize()를 의미한다.  
    - 클래스의 객체가 더 이상 사용되지 않으면 GC가 자동으로 호출한다.
+ Cleaner
    - Java 9에서는 fianlizer가 deprecated 됐고 cleaner가 새로 생겼다.
    - cleaner는 별도의 쓰레드를 사용해서 finalizer보다는 덜 위험하지만, 여전히 예측불가하고, 느리며 불필요하다.
+ 둘의 문제점
    - 성능 저하
    - 실행이 안될 가능성 존재
    - 예외 발생 무시
    - 인스턴스 반잡 지연
    - 보안 문제
+ 사용하는 경우
    - 자원 소유자가 close를 호출하지 않을 것을 대비해 늦게하도 회수하는 경우
    - C/C++이나 어셈블리 프로그램을 컴파일한 기계어 프로그램이자, GC가 존재를 알지 못하는 리소스 정리

### 대안책 - AutoCloseable
클라이언트에서 인스턴스를 다 쓰고 나면 close()를 호출하면 된다.  
예외가 발생해도 제대로 종료할 수 있게 처리해주는 try-with-resouces를 사용하는 것이 좋다.  
```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // 청소가 필요한 자원. 절대 Room을 참조하면 안됨.
    private static class State implements Runnable {
        int numJunkPiles; // 방(room) 안의 쓰레기 수

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메서드나 cleaner가 호출
        @Override public void run(){
            System.out.println("방 청소");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room (int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override public void close(){
        cleanable.clean();
    }
}
```
static으로 선언된 중첩 클래스인 State는 cleaner가 방을 청소할 때 수거할 자원들을 담고 있다.  
이 예에서는 단순한 방 안의 쓰레기 수를 뜻하는 numJunkPiles 필드가 수거할 자원에 해당한다.  
State는 Runnable을 구현하고, run 메서드는 cleanable에 의해 딱 한 번만 호출될 것이다.  
run 메서드가 호출되는 상황은 둘 중 하나다.  
보통은 room의 close 메서드를 호출할 때로, close 메서드에서 Cleanable의 clean을 호출하면 이 메서드 안에서 run을 호춣나다.  
혹은 가비지 컬렉터가 room을 회수할 때까지 클라이언트가 close를 호출하지 않는다면 cleaner가 State 메서드를 호출해줄 것이다.  
State 인스턴스는 절대로 Room 인스턴스를 참조해서는 안된다.  
Room 인스턴스를 참조할 경우 순환 참조가 생겨 가비지 컬렉터가 Room 인스턴스를 회수해갈 기회가 오지 않기 때문이다.  
State가 정적 중첩 클래스인 이유가 바로 정적이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 갖게 되기 때문이다.
```java
public class Test {
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
    }
}
```
위와 같이 try-with-resource를 사용하면 자원이 해제되면서 알아서 청소가 되지만 사용하지 않으면 아래처럼 명시를 해야 한다.
```java
public class Test {
    public static void main(String[] args) {
        Room room = new Room(99);
        System.out.println("아무렴");
        room.close();
    }
}
```
<Br>

## 아이템 9. try-finally보다는 try-with-resources를 사용하라 <a name = "10"></a>
---
close() 를 통해서 직접 닫아줘야 하는 자원은 많이 있다.(InputStream, OutputStream, Connection 등)  
과거에는 자원을 제대로 닫기 위해 try-finally 방식을 사용했다.
```java
//try-finally방식
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```
자바 7부터 try-with-resources 방식이 등장하고 이는 아래와 같이 개선되었다.
```java
//try-with-resources방식
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```
장점은 다음과 같다.
+ 가독성이 좋다.
    - 자원이 2개 이상인 경우 중첩해서 try문을 구성할 필요가 없어졌다.
+ 개발자가 직접 close를 호출하지 않아도 되기에 실수를 막을 수 있다.
+ 문제를 진단하기에 더 좋아졌다.

```java
//try-finally방식
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();//실패
    } finally {
        br.close();//실패
    }
}

//try-with-resources방식
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```
try-finally방식은 br.readLine()에 대한 스택 추적 내역은 남지 않고 br.close()의 스택 추적 내역만 남는다.  
try-with-resources방식은 br.readLine()은 기록되고, br.close()는 '숨겨졌다'(Suppressed)는 꼬리표를 달고 출력된다.  
__따라서 회수해야 하는 자원을 다룰 때는 예외없이 try-finally 보다는 try-with-resources를 쓰자.__



