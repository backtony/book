## 목차
- [Intro](#1)
- [아이템 10. equals는 일반 규약을 지켜 재정의하라](#2)
- [아이템 11. equals를 재정의하려거든 hashCode도 재정의하라](#3)
- [아이템 12. toString을 항상 재정의하라](#4)
- [아이템 13. clone 재정의는 주의해서 진행하라](#5)
- [아이템 14. Comparable을 구현할지 고려하라](#6)

---


## Intro <a name = "1"></a>
---
Object는 객체를 만들 수 있는 구체 클래스지만 기본적으로는 상속해서 사용하도록 설계되었다.  
Object에서 final이 아닌 메서드(equals, hashCode, toString, clone, finalize)는 모두 재정의를 염두에 두고 설계된 것이라 재정의 시 지켜야 하는 일반 규약이 명확히 정의되어 있다.  
메서드를 잘못 구현하면 대상 클래스가 이 규약을 준수한다고 가정하는 클래스(HashMap과 HashSet 등)를 오동작하게 만들 수 있다.  

<br>

## 아이템 10. equals는 일반 규약을 지켜 재정의하라 <a name = "2"></a>
---
equals 메서드는 재정의하기 쉬워 보이지만 곳곳에 함정이 도사리고 있어 자칫하면 끔찍한 결과를 초래한다.  
문제를 회피하는 가장 쉬운 길을 아애 재정의하지 않는 것이다.  
이 경우 클래스의 인스턴스는 오직 자기 자신과만 같게 된다.  
따라서 다음과 같은 상황이라면 재정의하지 않는 것이 최선이다.  
+ __각 인스턴스가 본질적으로 고유하다.__
    - 값을 표현하는게 아니라 동작하는 개체를 표현하는 클래스가 해당된다.
    - Thread가 좋은 예로, Object의 equals 메서드는 이러한 클래스에 딱 맞게 구현되어 있다.
+ __인스턴스의 논리적 동치성을 검사할 일이 없다.__
    - 논리적 동치성을 검사할 일이 없다면 Object의 기본 equals만으로 해결된다.
+ __상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.__
    - 예를 들면, 대부분의 Set의 구현체는 AbstractSet, List의 구현체는 AbstractList, Map의 구현체는 AbstractMap으로부터 equals를 상속받아 그대로 쓴다.
+ __클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.__
    - 사용할 일이 없으므로 재정의할 필요가 없다.


### equals를 재정의해야하는 시점
객체 식별성이 아니라 __논리적 동치성__ 을 확인해야하는 시점에서 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 경우 equals를 재정의해야 한다.  
논리적 동치성을 확인하도록 재정의해두면, 해당 값을 Map과 Set의 키값으로도 사용할 수 있다.  

### equals 메서드 일반 규약
equals 메서드를 재정의할 때는 반드시 일반 규약을 따라야 한다.  
#### 반사성
객체는 자기 자신과 같아야 한다.
```java
x.equlas(x) == true
```

#### 대칭성
null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)가 true면 y.equals(x)도 true여야 한다.
```java
public class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = s;
    }

    @Override
    public boolean equals(Object o) {
        // ...생략
        if (o instanceof String) {  //String과의 비교 연산 시도
            return s.equalsIgnoreCase((String) o);
        }
        // ...생략
        return false;
    }
}
----------------------------------------------------
CaseInsensitiveString cis = new CaseInsensitiveString("AAA");
String s = "aaa";

cis.equals(s); //true
s.euqals(cis); //false
```
CaseInsensitiveString에서 재정의한 equals는 일반 문자열과 비교를 시도한다.  
하지만 String의 equals는 CaseInsensitiveString의 존재자체를 모르기 때문에 비교할 수 없다.  
즉, CaseInsensitiveString의 equals를 String과 연동하겠다는 자체가 잘못된 생각이다.  
위의 equals를 적절하게 고치면 다음과 같다.
```java
@Override
public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

#### 추이성
첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다.
```java
x.equals(y) == true
y.equals(z) == true
x.equals(z) == true
```
추이성 문제는 주로 상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가하는 상황에서 발생한다.  
위배되는 구체적인 예시를 보자.
```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
    ...
}
---------------------------------------------------------------------------
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    @Override public boolean equals(Object o) {
        // o가 일반 Point면 색상을 무시하고 비교한다.
        if (!(o instanceof ColorPoint))
            return o.equals(this);
        
        // o가 ColorPoint면 색상까지 비교
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}
---------------------------------------------------------------------------
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

p1.equals(p2) // true
p2.equals(p3) // true
p1.equals(p3) // false
```
ColorPoint에서 재정의한 equals는 Object가 Point면 좌표만 비교하고, ColorPoint면 색상까지 비교한다.  
따라서 추이성이 맞지 않는 상황이 발생한다.  
이런 상황에서의 해법은 무엇일까?  
__구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.__  
따라서 상속 대신 컴포지션을 사용하면 된다.  
즉, Point를 ColorPoint의 private 필드로 두는 방식이다. 
```java
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    @Override
    public boolean equals(Object o) {
        if(!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
```
instanceof로 우선 클래스를 제한해주고 이후 내부값에 대해 비교하는 형식이다.  

#### 일관성
두 객체가 같다면 앞으로도 영원히 같아야 한다.
가변 객체는 비교 시점에 따라 서로 다를 수 있지만 불변 객체는 한 번 다르면 끝까지 달라야 한다.  

#### null-아님
모든 객체는 null과 같지 않아야 한다.
```java
if(o == null) 
    return false;
```
위와 같은 로직을 equals에 넣을 수도 있겠지만 instanceof가 처리해주는 작업이므로 불필요한 작업이다.  
instanceof는 두번째 피연산자와 무관하게 첫번째 피연산자가 null이면 무조건 false를 반환한다.
```java
if(!(o instanceof MyType)) 
    return false;
```


### equals 메서드 구현 방법 정리
+ == 연산자를 사용해 입력이 자기 자신의 참조인지 확인하고 자기 자신이면 true로 반환한다.
    - 이는 성능 최적화용으로 비교 작업이 복잡한 상황일 때 값어치를 한다.
+ instanceof 연산자로 입력이 올바른 타입인지 확인한다.
+ 입력을 올바른 타입으로 형변환한다.
    - instanceof 검사를 했다면 이 단계는 100% 성공한다.
+ 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.
+ 참고사항
    - __float와 double을 제외한 기본 타입 필드는 == 연산자로 비교__ 한다.
    - __참조타입 필드는 equals 메서드로 비교__ 한다.
    - float와 double 필드는 각각 정적 메서드인 __Float.compare(float, float), Double.compare(double, double)로 비교__ 한다.
    - 특별 취급하는 이유는 특수한 부동 소수값들을 다뤄야하기 때문이다. 이를 대신해서 Float.equals, Double.equals를 사용할 수 있으나 불필요한 오토박싱을 수반할 수 있어 성능상 좋지 않다.

<Br>

## 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라 <a name = "3"></a>
---
자바의 최상위 클래스 Object 클래스는 hashCode 메서드를 제공하기 때문에 모든 객체는 hashCode 메서드를 갖는다.  
__equals를 재정의한 클래스는 반드시 hashCode를 재정의해야 한다.__  
그렇지 않으면 hasCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap, HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킨다.  
다음은 __hashCode의 3가지 규약__ 이다.  
+ equals 비교에 사용되는 정보가 변경되지 않는다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
+ equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
+ equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 하지만 달라야 해시테이블의 성능이 좋다.

hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째다.  
즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.  

```java
public class Phone {
    
    private int number;
    
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Phone phone = (Phone) o;
        return number == phone.number && Objects.equals(name, phone.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(number, name);
    }
}
```
Objects 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 hash를 제공한다.  
이는 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱도 거쳐야하기 때문에 성능에 좋지는 않다.  
객체의 해시키가 로직에 많이 사용되는 경우라면 사용하지 않는 것이 좋으나, 많이 사용하지 않는다면 위 방식을 채택해도 된다.  


### 규악에 맞는 hashCode 메서드 만들기
좋은 해시 함수라면 서로 다른 인스턴스끼리 다른 해시코드를 반환해야 하고, 가장 이상적인 해시 함수는 주어진 인스턴스들이 32비트 정수 범위에 균일하게 분배되어야 한다.  
이에 따라 hashCode를 짜는 방법을 알아보자.
1. int 타입의 변수 result를 선언한 후 값을 c로 초기화한다. 이때 c는 해당 객체의 첫번째 핵심 필드를 2.a 방식으로 계산한 해시코드이다. 여기서 말하는 핵심 필드는 equals 비교에 사용되는 필드이다. __equals에서 사용되지 않는 필드는 반드시 hashCode에서 제외애햐 한다.__ 그렇지 않다면 두 번째 규약을 어기게 된다.
2. 해당 객체의 나머지 핵심 필드 f에 대해 각각 다음 작업을 수행한다.
    + a. 필드의 해시 코드 c를 계산한다.
        - 기본 필드라면 Type.hashCode(f)를 수행한다. 이때 Type은 기본 타입에 매핑되는 래퍼 클래스다.
        - 참조 필드이면서 클래스의 equals가 이 필드의 equals를 재귀적으로 호출해 비교한다면 이 필드의 hashCode가 재귀적으로 호출한다. 만약 필드의 값이 null이면 0을 반환한다.
        - 배열이라면 핵심 원소 각각을 별도의 필드로 나눈다. 별도로 나눈 필드를 위 규칙을 적용하여 해시코드를 계산한 후 다음 소개되는 2.b방식으로 갱신한다. 모든 원소가 핵심이라면 Arrays.hashCode를 이용한다.
    + b. 2.a로 계산한 해시코드 c로 result를 갱신한다.

3. result를 반환한다.

```java
public class Phone {

    private int number;

    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Phone phone = (Phone) o;
        return number == phone.number && Objects.equals(name, phone.name);
    }

    @Override
    public int hashCode() {
        int result = Integer.hashCode(number);
        result = 31 * result + name.hashCode();
        return result;
    }
}
```
<br>


만약 핵심 필드가 많아지고 hashCode 함수를 자주 사용하는 상황이라면 캐시해서 사용하는 방식이 있다.
```java
public class Phone {

    private int number;

    private String name;

    private int hashCode;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Phone phone = (Phone) o;
        return number == phone.number && Objects.equals(name, phone.name);
    }

    @Override
    public int hashCode() {
        // 해시코드 연산은 hashCode 함수를 처음 호출하는 시점에만 계산하게 된다.
        int result = hashCode;
        if (result == 0){
            result = Integer.hashCode(number);
            result = 31 * result + name.hashCode();
            hashCode = result;
        }
        return result;
    }
}
```
<br>

## 아이템 12. toString을 항상 재정의하라 <a name = "4"></a>
---
자바의 최상위 클래스 Object 클래스는 toString 메서드를 제공하기 때문에 모든 객체는 toString 메서드를 갖는다.  
toString은 기본적으로 아래와 같은 방식으로 객체를 표현하는 문자열을 리턴한다.
```java
public String toString() {
    // 클래스이름@16진수해시코드
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```
toString은 기본적으로 객체를 표현하는 문자열을 리턴해주기 때문에 개발할 때 사용된다.
+ 콘솔로 객체를 확인할 때
+ 디버깅 할 때
+ 로깅할 때
+ 유틸리티 클래스는 toString을 사용할 이유가 없고, Enum은 이미 toString이 재정의되어 있으므로 따로 재정의 하지 않아도 된다.  

위와 같이 기본적으로 16진수 해시코드로 toString을 제공하면 객체 안에 뭐가 있는지 확인할 수 없다.  
따라서 toString을 재정의하여 개발자에게 유익한 정보를 제공해야 한다.  
재정의할 때 가장 중요한 것은 객체 스스로를 완벽히 설명하는 문자열이어야 한다는 것이다.  
보통 객체가 가진 주요 정보를 모두 반환하는 것이 좋다.  
이때 toString의 format을 정할 수도 있고 정하지 않을 수도 있는데 만약 정한다면 앞으로의 유지 보수에도 항상 해당 포멧을 사용해야 하기 때문에 신중하게 결정할 필요가 있다.
```java
// 포맷을 정했을 때
/** 
 * 전화번호의 문자열 표현을 반환합니다.
 * 이 문자열은 XXX-YYYY-ZZZZ 형태의 11글자로 구성됩니다.
 * XXX는 지역코드, YYYY는 접두사, ZZZZ는 가입자 번호입니다.
 * 블라블라~
*/
@Override
public String toString() {
    return String.format("%03d-%04d-%04d", areaCode, prefix, lineNum);
}

// 포맷을 정하지 않았을 때
public class Phone {

    private int number;

    private String name;

    @Override
    public String toString() {
        return "Phone{" +
                "number=" + number +
                ", name='" + name + '\'' +
                '}';
    }
}
```

### 주의사항
IDE에서 제공하는 toString, lombok에서 제공하는 @ToString을 사용하는 것도 방법이긴 하나 무분별하게 사용할 경우 StackOverflowError가 발생할 수 있다.  
```java
public class Member {
    
    private String name;
    
    private Phone phone;

    @Override
    public String toString() {
        return "Member{" +
                "name='" + name + '\'' +
                ", phone=" + phone +
                '}';
    }
}
------------------------------------------------------------------
public class Phone {

    private int number;

    private Member member;

    @Override
    public String toString() {
        return "Phone{" +
                "number=" + number +
                ", member=" + member +
                '}';
    }
}
```
IDE에서 제공하는 toString을 사용해서 만들었다.  
Member의 toString 코드를 보면 phone을 참조하고 있다.  
Phone의 toString을 호출하게 되는데 Phone의 toString은 다시 member을 참조한다.  
즉, 무한 호출이 발생하는 것이다.  
따라서 무분별하게 사용하기 보다는 기능을 이용하면서 적절하게 수정해주는 작업이 필요하다.  
<br>

## 아이템 13. clone 재정의는 주의해서 진행하라 <a name = "5"></a>
---
Cloneable을 구현하는 클래스는 clone 메서드가 정확히 동작해야 한다.  
일반적인 clone 메서드 규약은 제약은 다음과 같다.
+ x.clone() != x 는 참이여야 한다.
+ x.clone().getClass() == x.getClass() 는 참이여야 한다.
+ x.clone().equals(x) 는 참이여야 한다.

위 규약은 클래스가 기본 타입 필드와 불변 객체 참조만 갖는 클래스라면 쉽게 지킬 수 있다.
```java
public class Phone implements Cloneable {

    private int number;

    private String name;

    // 사용하는 곳에서 편하도록 throws를 넘기지 말고 try-catch로 처리한다.
    @Override
    protected Phone clone() {
        try {
            return (Phone) super.clone(); // clone은 기본적으로 Object를 반환하므로 타입 캐스팅 필요       
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
<br>

참조형 타입을 필드로 갖는 경우에는 고려해야할 사항이 있다.  
스택을 예시로 구현해보자.
```java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    ..... 생략

    @Override
    protected Stack clone() {
        try {
            return (Stack) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
위와 같이 코딩하면 될 것 같지만 안된다.  
size같은 기본 필드는 복제되지만, elements 필드는 복제 후에도 여전히 원본 Stack 인스턴스와 똑같은 배열을 참조하게 된다.  
즉, 복제본이나 원본에서 값을 수정하면 다른쪽에서도 수정된 값을 읽게 된다는 의미다.  
이럴 경우 배열 자체도 복사해야한다.  
```java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    ..... 생략

    @Override
    protected Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            // 배열의 clone은 런타임 타입과 컴파일타임 타입 모두 원본 배열과 일치하는 타입을 반환하므로 타입 캐스팅 불필요
            result.elements = elements.clone(); 
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
<br>

더 나아가 해시 테이블을 살펴보자.  
```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
        // contstructor...
    }

    @Override public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = buckets.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결 리스트를 참조한다.  
즉, 배열 안에 있는 연결 리스트도 따로 복제해야한다는 의미다. 
```java
public class HashTable implements Cloneable {
    private Entry[] buckets;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        Entry deepCopy() {
            Entry result = new Entry(key, value, next);
            for (Entry p = result; p.next != null; p = p.next) {
                p.next = new Entry(p.next.key, p.next.value, p.next.next);
            }
            return result;
        }
    }

    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            // 연결 리스트도 하나씩 복제
            for (int i = 0; i < buckets.length; i++)
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
지금까지 내용을 정리하자면 Cloneable을 구현하는 모든 클래스는 clone을 재정의 해야한다.  
접근 제한자는 public으로, 반환 타입은 클래스 자기 자신으로 변경한다.  
메서드는 가장 먼저 super.clonoe을 호출한 후 필요한 필드를 앞서 했던 방식으로 수정한다.  
기본 타입과 불변 객체로만 이뤄져있다면 수정할 필요없다.  

### 복사 생성자와 복사 팩토리
위와 같은 clone 작업은 복잡하고 정확하게 동작하길 기대하기 어렵다(clone 메서드는 thread-safe하지 않다.)  
이보다 더 나은 방식은 복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공하는 것이다.  
이미 Collections나 Map 인터페이스에서도 이렇게 구현되어 있다.
```java
// copy라는 정적 메서드로 제공중
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    int srcSize = src.size();
    if (srcSize > dest.size())
        throw new IndexOutOfBoundsException("Source does not fit in dest");

    if (srcSize < COPY_THRESHOLD ||
        (src instanceof RandomAccess && dest instanceof RandomAccess)) {
        for (int i=0; i<srcSize; i++)
            dest.set(i, src.get(i));
    } else {
        ListIterator<? super T> di=dest.listIterator();
        ListIterator<? extends T> si=src.listIterator();
        for (int i=0; i<srcSize; i++) {
            di.next();
            di.set(si.next());
        }
    }
}
```
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    // 복사 생성자
    public Stack(Object[] elements, int size) {
        this.elements = elements.clone();
        this.size = size;
    }

    // 복사 정적 팩터리
    public static Stack newInstance(Stack s) {
        return new Stack(s.elements, s.size);
    }
}
```
<br>

## 아이템 14. Comparable을 구현할지 고려하라 <a name = "6"></a>
---
```java
package java.lang;
import java.util.*;

public interface Comparable<T> {

    public int compareTo(T o);
}
```
+ 단순 동치성 비교 + 순서까지 비교 + 제네릭
+ 이 3가지의 특징을 가지고 있는 메서드가 바로 compareTo이다.
+ 따라서 Comparable을 구현한 객체의 배열은 손쉽게 정렬이 가능하다.
+ 아울러 자바 플랫폼 라이브러리의 모든 값 클래스, 열거 타입은 모두 Compareable을 구현하고 있다.

### 규약
+ 객체가 주어진 객체보다 작으면 음의 정수(-1), 같으면 0, 크면 양수(1)을 반환하도록 한다.
+ 객체와 매개변수로 들어온 객체의 타입이 다르다면 ClassCaseException을 던진다.

#### x.compareTo(y) == -y.compareTo(x)
+ 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다는 의미다.
+ 즉, x.compareTo(y)가 1이라면 y.compareTo(x)는 -1이다.

#### 추이성 (x.compareTo(y) > 0 && y.compareTo(z) > 0) 이면 x.compareTo(z) > 0
+ x가 y보다 크고 y가 z보다 크다면 x는 z보다 크다

#### x.compareTo(y) == 0 이면 (x.compareTo(z) == y.compareTo(z))
+ x와 y가 같다면 x와 y의 각각 z와 비교값은 같다.

#### (x.compareTo(y) == 0) == x.eqauls(y)
+ 이 권고는 필수가 아니지만, 꼭 지키는것이 좋다. (혹시 지키지 못하면 명시해줘야 한다.)
+ 지키지 않고 컬렉션에 넣으면 해당 컬렉션이 구현한 인터페이스에 정의한 동작과 엇박자를 낼 것이다.
+ __정렬된 컬렉션(TreeSet 등)은 equals가 아닌 compareTo를 사용해 동치성을 비교하기 떄문이다.__

```java
@Test
void test() {
    BigDecimal number1 = new BigDecimal("1.0");
    BigDecimal number2 = new BigDecimal("1.00");

    Set<BigDecimal> set1 = new HashSet<>();
    set1.add(number1);
    set1.add(number2);

    Set<BigDecimal> set2 = new TreeSet<>();
    set2.add(number1);
    set2.add(number2);

    assertThat(set1).hasSize(2);
    assertThat(set2).hasSize(1);
}
```
new BigDecimal("1.0")과 new BigDecimal("1.00")을 빈 HashSet 인스턴스에 추가하면 __HashSet은 equals로 검사하기__ 때문에 서로 달라서 HashSet은 2개의 원소를 갖게 된다.  
TreeSet을 이용하게 되면 __TreeSet은 equals가 아닌 CompareTo를 사용하기 때문에__ 원소를 1개만 갖게 된다.  


### 주의사항
equals()와 같이 상속을 이용해 기존 클래스를 확장한 구체 클래스에서 새로운 값을 추가한다면 CompareTo 규약을 지킬 수 없다.  
따라서 Comparable을 구현한 클래스를 확장해 값 컴포넌트를 추가하도 싶다면, 상속 대신 컴포지션을 사용한다.  
이렇게 하면 바깥 클래스에 우리가 원하는 CompareTo 메서드를 구현해넣을 수 있다.  

### 작성 요령
1. 타입을 인수로 받는 제네릭 인터페이스이므로 컴파일 시 인수타입은 정해진다.
2. 동치인지를 비교하는 게 아니라 순서를 비교한다.
3. compareTo()에서 관계연산자 <, >를 사용하는 이전 방식은 거추장스럽게 오류를 유발하기 객체의 compare메서드를 사용하자.
4. 핵심 필드가 여러 개이면 어느 것을 먼저 비교하느냐에 따라 중요해진다. 따라서 핵심적인 필드를 먼저 비교하자

```java
public class Student implements Comparable<Student> {
    private int grade;
    private String name;
    private int age;

    @Override
    public int compareTo(Student o) {
        int result = Integer.compare(grade, o.grade);

        if (result == 0) {
            result = CharSequence.compare(name, o.name);
            if (result == 0) {
                result = Integer.compare(age, o.age);
            }
        }

        return result;
    }
}
```
인터넷에 CompareTo를 구현한 자료들을 보면 == 연산자를 사용한 것이 많다.  
자바 7부터는 박싱된 기본 타입 클래스에 정적 메서드 compare이 추가되었기 때문에 compare을 사용하길 권장한다.  
이유는 연산 과정에서 정수 오버플로우가 발생하거나 부동소수점 계산 방식에 따른 오류가 발생할 수 있기 때문이다.  

### Comparator
자바 8에서는 Comparator 인터페이스가 일련의 비교 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.  
Comparator는 Comparable 인터페이스의 CompareTo 메서드를 구현하는데 멋지게 활용할 수 있다.  
이 방식은 코드가 매우 짧아지는 이점이 있지만 약간의 성능 저하가 뒤따른다.  
```java
public class Student implements Comparable<Student> {
    private static final Comparator<Student> COMPARATOR =
            Comparator.comparingInt((Student st) -> st.grade)
                    .thenComparing(st -> st.name).reversed() // 내림차순
                    .thenComparingInt(st -> st.age);

    private int grade;
    private String name;
    private int age;


    @Override
    public int compareTo(Student o) {
        return COMPARATOR.compare(this, o);
    }
}
```
첫 번째 비교는 comparingXX로 진행하고 이후 연쇄적인 비교는 then으로 체이닝해서 정의한다.  
첫 번째 비교 시에는 반드시 타입을 명시해줘야 하고 이후 체이닝에서는 타입명은 제외하고 변수명만 명시해줘도 된다.  
reversed()를 붙이면 내림차순으로 가능하다.  
<br>  

Comparator를 객체 내부에 구현해서 사용할 수도 있지만 일회성인 경우 다음과 같이 사용한다.
```java
Arrays.sort(student, (s1, s2) -> Double.compare(s1.score, s2.score));

Arrays.sort(students, Comparator.comparingInt((Student st) -> st.getGrade())
                .thenComparing(st -> st.getName()));
```
Arrays.sort의 두번째 인자로 Comparator를 넣을 수 있는데 람다식으로 해결할 수 있다.  

### 결론
순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현한다.  
CompareTo 메서드에서 필드의 값을 비교할 때 연산자를 사용하지 않고 박싱 타입의 compare메서드나 Comparator 인터페이스가 제공하는 비교 생성 메서드를 사용한다.
