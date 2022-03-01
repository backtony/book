## 목차
- [Intro](#1)
- [자료 추상화](#2)
- [자료/객체 비대칭](#3)
- [디미터 법칙](#4)
    - [기차 충돌](#5)
    - [잡종 구조](#6)
    - [구조체 감추기](#7)
- [자료 전달 객체](#8)
	- [활성 레코드](#9)
- [결론](#10)

---


## Intro <a name = "1"></a>
변수를 비공개(private)로 정의하는 이유가 있다.  
남들이 변수에 의존하지 않게 만들고 싶어서다.  
충동이든 변덕이든, 변수 타입이나 구현을 맘대로 바꾸고 싶어서다.  
그렇다면 어째서 수많은 프로그래머가 조회(get) 함수와 설정(set) 함수를 당연하게 공개(public)해 비공개 변수를 외부에 노출할까?  

<br>

## 자료 추상화 <a name = "2"></a>
```java
// 구체적인 Point 클래스
public class Point { 
  public double x; 
  public double y;
}
```
위 코드는 개별적으로 좌표값을 읽고 설정하게 강제한다.  
변수를 private으로 선언하더라도 각 값마다 get과 set함수를 제공한다면 구현을 외부로 노출하는 셈이다.  
변수 사이에 함수라는 계층을 넣는다고 구현이 저절로 감춰지지는 않는다.  
<br>

```java
// 추상적인 Point 클래스
public interface Point {
  double getX();
  double getY();
  void setCartesian(double x, double y); 
  double getR();
  double getTheta();
  void setPolar(double r, double theta); 
}
```
반면에 위 코드는 구현을 완전히 숨긴다.  
즉, 구현을 감추려면 __추상화__ 가 필요하다.  
추상 인터페이스를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스다.  
조금 풀어서 설명하자면, 구체적인 Point 클래스에서는 x와 y를 외부에서 참조할 수 있기 때문에 결과적으로 외부로 구현을 노출한 것이다.  
반면에 Point 인터페이스의 경우 getX와 getY로 x, y값을 가져오는데 실제로 Point클래스에서 x와 y가 어떻게 가져와지는지, x와 y값이 존재는 하는건지에 대한 구체적인 내용은 모른다.  
그저 getX하면 'X값을 어떻게 해서든지 가져오겠구나' 정도로만 알 수밖에 없는 것이다.  
__정리하자면, 자료를 세세하게 공개하기보다는 추상적인 개념으로 표현하는 편이 좋고, 인터페이스나 조회/설정 함수만으로 추상화가 이뤄지지는 않기에 객체가 포함하는 자료를 표현할 수 있는 가장 좋은 방법을 고민해야 한다.__  
<br>

## 자료/객체 비대칭 <a name = "3"></a>
+ __객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다.__
+ __자료 구조는 자료를 그대로 공개하면서 별다른 함수는 제공하지 않는다.__

__절차적인 도형__
```java
public class Square { 
  public Point topLeft; 
  public double side;
}
--------------------------------------------
public class Rectangle { 
  public Point topLeft; 
  public double height; 
  public double width;
}
--------------------------------------------
public class Circle { 
  public Point center; 
  public double radius;
}
--------------------------------------------
public class Geometry {
  public final double PI = 3.141592653589793;
  
  public double area(Object shape) throws NoSuchShapeException {
    if (shape instanceof Square) { 
      Square s = (Square)shape; 
      return s.side * s.side;
    } else if (shape instanceof Rectangle) { 
      Rectangle r = (Rectangle)shape; 
      return r.height * r.width;
    } else if (shape instanceof Circle) {
      Circle c = (Circle)shape;
      return PI * c.radius * c.radius; 
    }
    throw new NoSuchShapeException(); 
  }
}
```
각 도형 클래스는 간단한 자료구조다.  
아무런 메서드도 제공하지 않는다.  
도형이 동작하는 방식은 Geometry 클래스에서 구현된다.  
위의 절차적인 코드에서는 각 도형의 둘레를 구하는 perimeter 함수를 추가하고 싶다면 도형 클래스는 아무런 영향도 받지 않는다.  
하지만 새 도형을 추가하고 싶다면 Geometry 클래스를 수정해야 한다.  
<br>

__객체지향적 도형__
```java
// 객체는 자료를 숨기고(필드 자체에 대한 조회 함수 X), 자료를 사용한 함수를 공개한다.
// 아래 코드를 보면 각 필드를 조회하는 함수는 없고, 각 필드들을 사용해서 핵심적인 기능 메서드를 제공한다.
public class Square implements Shape { 
  private Point topLeft;
  private double side;
  
  public double area() { 
    return side * side;
  } 
}
--------------------------------------------
public class Rectangle implements Shape { 
  private Point topLeft;
  private double height;
  private double width;

  public double area() { 
    return height * width;
  } 
}
--------------------------------------------
public class Circle implements Shape { 
  private Point center;
  private double radius;
  public final double PI = 3.141592653589793;

  public double area() {
    return PI * radius * radius;
  } 
}
```
객체 지향적 도형에서는 Geometry클래스가 필요 없다.  
그러므로 새 도형을 추가해도 기존 함수에 아무런 영향을 미치지 않는다.  
반면에 새 함수를 추가하고 싶다면 도형 클래스 전부를 고쳐야 한다.  
<br>

__정리하면, 객체 지향 코드에서 어려운 변경은 절차적인 코드에서 쉬우며, 절차적인 코드에서 어려운 변경은 객체 지향 코드에서 쉽다.__  
따라서, 상황에 맞게 사용해야 한다.  

<br>

## 디미터 법칙 <a name = "4"></a>
디미터 법칙은 잘 알려진 휴리스틱(경험에 기반하여 문제를 해결하거나 학습하거나 발견해 내는 방법)으로, __모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙__ 이다.  
좀 더 정확히 표현하자면, 디미터 법칙은 "클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다"고 주장한다.
+ 클래스 C
+ f가 생성한 객체
+ f 인수로 넘어온 객체
+ C 인스턴스 변수에 저장된 객체

하지만 위 객체에서 허용된 메서드가 반환하는 객체의 메서드는 호출하면 안 된다.  
다시 말해, 낯선 사람은 경계하고 친구랑만 놀라는 의미이다.  

### 기차 충돌 <a name = "5"></a>
```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```
위와 같은 코드를 여러 객차가 한 줄로 이어진 기차처럼 보이기 때문에 기차 충돌 이라고 한다.  
일반적으로 조잡하다고 여겨지기에 다음과 같이 나누는 편이 좋다.  
```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```
위 코드가 디미터 법칙을 위반하는지 여부는 변수들이 객체인지 자료 구조인지에 달렸다.  
만약 객체라면 get 조회 메서드를 통해 내부구조를 노출하고 있기 때문에 디미터 법칙을 위반한다.  
반면, 자료 구조라면 당연히 내부 구조를 노출하므로 문제가 되지 않는다.  

### 잡종 구조 <a name = "6"></a>
때때로 절반은 객체, 절반은 자료 구조인 잡종 구조가 나온다.  
잡종 구조는 중요한 기능을 수행하는 함수도 있고, 공개 변수나 공개 get/set 함수도 있다.  
이런 구조는 새로운 함수는 물론이고 새로운 자료 구조도 추가하기 어렵다.  
양쪽 세상에서 단점만 모아놓은 구조다.  
그러므로 되도록 이런 구조는 피하도록 하자.  
프로그래머가 함수나 타입을 보호할지 공개할지 확신하지 못해 (더 나쁘게는 무지해) 어중간하게 내놓은 설계에 불과하다.  

### 구조체 감추기 <a name = "7"></a>
기차 충돌에서 제시한 예시에서 만약 객체라면 어떻게 개선해야 할까?  
일단 왜 절대 경로가 필요한지 모듈에서 찾아보면 다음과 같은 코드가 있다.
```java
String outFile = outputDir + "/" + className.replace('.', '/') + ".class"; 
FileOutputStream fout = new FileOutputStream(outFile); 
BufferedOutputStream bos = new BufferedOutputStream(fout);
```
추상화 수준을 뒤섞어 놓아 다소 불편하다.  
점, 슬래시, 파일 확장자, File 객체를 부주의하게 마구 뒤섞으면 안 된다.  
어찌 되었거나, 코드를 보니 임시 디렉터리의 절대 경로를 얻으려는 이유가 임시 파일을 생성하기 위한 목적이었다.  
그렇다면 ctxt 객체에 임시 파일을 생성하라고 시키면 어떨까?  
```java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```
객체에 맡기기에 적당한 임무로 보인다.  
cxtx는 내부 구조를 드러내지 않으며, 모듈에서 해당 함수는 자신이 몰라야 하는 여러 객체를 탐색할 필요가 없다.  
따라서 디미터 법칙을 위반하지 않는다.  

<br>

## 자료 전달 객체 <a name = "8"></a>
자료 구조체의 전형적인 형태는 __공개 변수만 있고 함수가 없는 클래스__ 다.  
이를 때로는 자료 전달 객체Data Transfer Object, DTO라 한다.  
```java
public class Address { 
  public String street; 
  public String streetExtra; 
  public String city; 
  public String state; 
  public String zip;
}
```
DTO는 팀마다 사용하는 방식이 다른데, 어떤 곳은 위 코드처럼 public으로 두고 접근해서 사용하는 반면에 어떤 곳은 private으로 두고 Getter/Setter을 두고 사용한다.  
개인적으로 프로젝트를 진행할 때는 private으로 두고 사용했다.  

### 활성 레코드 <a name = "9"></a>
DTO의 특수한 형태다.  
공개 변수가 있거나 비공개 변수에 getter/setter가 있는 자료 구조지만, 대게 save나 find와 같은 탐색 함수도 제공한다.  
활성 레코드는 데이터베이스 테이블이나 다른 소스에서 자료를 직접 변환한 결과다.  
불행히도 활성 레코드에 비즈니스 규칙 메서드를 추가해 이런 자료 구조를 객체로 취급하는 개발자가 흔하다.  
하지만 이렇게 하게 되면 잡종 구조가 나오게 된다.  
해결책은 당연하다.  
비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성한다.  
__즉, 활성 레코드는 자료 구조로 취급한다.__  

<br>

## 결론 <a name = "10"></a>
객체는 동작을 공개하고 자료를 숨긴다.  
그래서 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기 쉬운 반면, 기존 객체에 새 동작을 추가하기는 어렵다.  
자료구조는 별다른 동작 없이 자료를 노출한다.  
그래서 기존 자료 구조에 새 동작을 추가하기는 쉬우나, 기존 함수에 새 자료 구조를 추가하기는 어렵다.  
(어떤) 시스템을 구현할 때, 새로운 자료 타입을 추가하는 유연성이 필요하면 객체가 더 적합하다.  
반면에 새로운 동작을 추가하는 유연성이 필요하면 자료 구조와 절차적인 코드가 더 적합하다.  
우수한 소프트웨어 개발자는 편견 없이 이 사실을 이해해 직면한 문제에 최적인 해결책을 선택한다.