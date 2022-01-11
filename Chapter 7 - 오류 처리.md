## 목차
- [Intro](#1)
- [오류 코드보다 예외를 사용하라](#2)
- [Try-Catch-Finally 문부터 작성하라](#3)
- [Unchecked Exception을 사용하라](#4)
- [예외에 의미를 제공하라](#5)
- [호출자를 고려해 예외 클래스를 정의하라](#6)
- [정상 흐름을 정의하라(Default 값을 설정하라)](#7)
- [null을 반환하지 마라](#8)
- [null을 전달하지 마라](#9)
- [결론](#10)

---



## Intro <a name = "1"></a>
상당수 코드 기반은 전적으로 오류 처리 코드에 좌우되기 때문에 오류 처리는 중요하고 깨끗한 코드와 연관성이 있다.  
하지만 오류 처리 코드로 인해 프로그램 논리를 이해하기 어려워진다면 깨끗한 코드라 부르기 어렵다.  
<br>

## 오류 코드보다 예외를 사용하라 <a name = "2"></a>
예전 프로그래밍 언어들은 예외를 지원하지 않았다.  
그 당시에는 플래그를 설정하거나 호출자에게 오류 코드를 반환하는 방법이 전부였다.  
하지만 이러한 방식은 잊어버리기 쉽고 로직을 헷갈리게 한다.  
그러므로 오류가 발생하면 예외를 던지는 편이 더 깔끔해지고 논리가 오류와 뒤섞이지 않아서 좋다.  
```java
// Bad
public class DeviceController {
  ...
  public void sendShutDown() {
    DeviceHandle handle = getHandle(DEV1);
    // Check the state of the device
    if (handle != DeviceHandle.INVALID) {
      // Save the device status to the record field
      retrieveDeviceRecord(handle);
      // If not suspended, shut down
      if (record.getStatus() != DEVICE_SUSPENDED) {
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
      } else {
        logger.log("Device suspended. Unable to shut down");
      }
    } else {
      logger.log("Invalid handle for: " + DEV1.toString());
    }
  }
  ...
}
```
```java
// Good
public class DeviceController {
  ...
  public void sendShutDown() {
    try {
      tryToShutDown();
    } catch (DeviceShutDownError e) {
      logger.log(e);
    }
  }
    
  private void tryToShutDown() throws DeviceShutDownError {
    DeviceHandle handle = getHandle(DEV1);
    DeviceRecord record = retrieveDeviceRecord(handle);
    pauseDevice(handle); 
    clearDeviceWorkQueue(handle); 
    closeDevice(handle);
  }
  
  private DeviceHandle getHandle(DeviceID id) {
    ...
    throw new DeviceShutDownError("Invalid handle for: " + id.toString());
    ...
  }
  ...
}
```
<Br>

## Try-Catch-Finally 문부터 작성하라 <a name = "3"></a>
try문은 transaction처럼 동작하는 실행코드로, catch 블록은 try 블록에 관계없이 프로그램을 일관적인 상태로 유지한다.  
이렇게 함으로써 try 블록에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기 쉬워진다.  
예시로 파일이 없으면 예외를 던지는지 알아보는 단위 테스트를 보자.  
```java
// Step 1: StorageException을 던지지 않으므로 이 테스트는 실패한다.

@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
  sectionStore.retrieveSection("invalid - file");
}

public List<RecordedGrip> retrieveSection(String sectionName) {
  // dummy return until we have a real implementation
  return new ArrayList<RecordedGrip>();
}
```
```java
// Step 2: 이제 테스트는 통과한다.
public List<RecordedGrip> retrieveSection(String sectionName) {
  try {
    FileInputStream stream = new FileInputStream(sectionName)
  } catch (Exception e) {
    throw new StorageException("retrieval error", e);
  }
return new ArrayList<RecordedGrip>();
}
```
```java
// Step 3: Exception의 범위를 FileNotFoundException으로 줄여 정확히 어떤 Exception이 발생한지 체크하자.
public List<RecordedGrip> retrieveSection(String sectionName) {
  try {
    FileInputStream stream = new FileInputStream(sectionName);
    stream.close();
  } catch (FileNotFoundException e) {
    throw new StorageException("retrieval error", e);
  }
  return new ArrayList<RecordedGrip>();
}
```
try-catch 구조로 범위를 정의했으므로 TDD를 사용해 필요한 나머지 논리를 FileInputStream과 close 사이에 추가하며 오류나 예외가 전혀 발생하지 않는다고 가정한다.  

<br>

## UnChecked Exception을 사용하라 <a name = "4"></a>
Checked Exception를 치르는 비용 대비 이득을 생각해보자.  
예를 들어, 최상위 함수에서 호출하는 그 아래 함수 ... 그 아래 함수로 내려간다고 해보자.  
제일 마지막에 도달한 아래 함수에서 Checked Exception을 던진다면 선언부에 throws 절을 추가해야 한다.  
그러면 이 함수를 호출하는 함수에서는 catch 블록에서 새로운 예외를 처리하거나 선언부에 throw 절을 추가해야 한다.(OCP 위반)  
결과적으로 최상위부터 최하위까지 모두 수정이 필요하게 되고 모든 함수가 최하위 함수에서 던지는 에러를 알아야 하므로 캡슐화도 깨진다.  
즉, 필요한 경우 Chekced Exception을 사용해야 되지만, 일반적으로 득보다 실이 많다.  
<br>

## 예외에 의미를 제공하라 <a name = "5"></a>
예외를 던진다면 오류 메시지에 정보를 담아 예외와 함께 던진다.  
가능하다면 실패한 연산 이름과 실패 유형도 언급한다.  
즉, 실패한 코드의 의도를 바로 파악할 수 있도록 정보를 제공하라는 의미이다.  
<br>

## 호출자를 고려해 예외 클래스를 정의하라 <a name = "6"></a>
오류를 정의할 때 프로그래머에게 가장 중요한 것은 __오류를 잡아내는 방법__ 이다.  
다음은 오류를 형편없이 분류한 예시이다.  
외부 라이브러리를 호출에 try-catch-finally 문을 사용하여 외부 라이브러리가 던지는 예외를 모두 잡아낸다.
```java
  // Bad
  // catch문의 내용이 거의 같다.
  
  ACMEPort port = new ACMEPort(12);
  try {
    port.open();
  } catch (DeviceResponseException e) {
    reportPortError(e);
    logger.log("Device response exception", e);
  } catch (ATM1212UnlockedException e) {
    reportPortError(e);
    logger.log("Unlock exception", e);
  } catch (GMXError e) {
    reportPortError(e);
    logger.log("Device response exception");
  } finally {
    ...
  }
```
<Br>

위의 경우에는 catch문에서 예외를 대응하는 방식의 거의 동일하다.  
이는 호출하는 라이브러리 API를 감싸면서 예외 유형 하나를 만들어 반환하면 깔끔하게 만들 수 있다. 
```java
// Good
// ACME 클래스를 LocalPort 클래스로 래핑해 new ACMEPort().open() 메소드에서 던질 수 있는 exception들을 간략화

public class LocalPort {
  private ACMEPort innerPort;

  public LocalPort(int portNumber) {
    innerPort = new ACMEPort(portNumber);
  }
  
  public void open() {
    try {
      innerPort.open();
    } catch (DeviceResponseException e) {
      throw new PortDeviceFailure(e);
    } catch (ATM1212UnlockedException e) {
      throw new PortDeviceFailure(e);
    } catch (GMXError e) {
      throw new PortDeviceFailure(e);
    }
  }
  ...
}
```
위 코드에서는 모든 exception을 PortDeviceFailure로 처리하고 있다.  
보통 예외 클래스 하나만 있어도 충분히 깔끔하게 해결할 수 있다.  
추가적으로 외부 API를 사용할 때는 위 코드처럼 하나의 클래스를 만들어 감싸서 외부 API를 사용하는 것이 좋다.  
이렇게 하면 외부 라이브러리와 프로그램 사이의 의존성이 크게 줄어들어 나중에 변경 시에도 비용이 적다.  
또한, 외부 API를 호출하는 대신 테스트 코드를 넣어주는 방법으로 프로그램을 테스트하기 쉬워질 뿐만 아니라 외부 업체에서 설계한 API 설계에 따르지 않고 프로그램이 사용하기 편리한  API를 정의해서 만들어 사용할 수 있다.  
<br>

## 정상 흐름을 정의하라(Default 값을 설정하라) <a name = "7"></a>
일반적으로 위에서 했던 방식들이 유용하지만 Catch 문에서 특이한 상황을 처리해야 하는 경우 코드가 더러워질 수 있다.  
```java
// Bad

try {
  MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
  m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
  m_total += getMealPerDiem();
}
```
식비를 비용으로 청구했다면 직원이 청구한 식비를 총계에 더한다.  
식비를 청구하지 않았다면 일일 기본 식비를 총계에 더하는데 이 과정이 catch에서 이뤄지고 있다.  
특수 상황을 처리할 필요가 없다면 코드가 더욱 간결해질 것이다.  
```java
// Good

// caller logic.
...
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
...
--------------------------------------------------------------------------------------------
public class ExpenseReportDAO {
  ...
  public MealExpenses getMeals(int employeeId) {
    MealExpenses expenses;
    try {
      expenses = expenseReportDAO.getMeals(employee.getID());
    } catch(MealExpensesNotFound e) {
      expenses = new PerDiemMealExpenses();
    }
    
    return expenses;
  }
  ...
}
--------------------------------------------------------------------------------------------
public class PerDiemMealExpenses implements MealExpenses {
  public int getTotal() {
    // return the per diem default
  }
}
```
expenseReportDAO를 고쳐 언제나 MealExpense 객체를 반환하게 하고 청구한 식비가 없다면 기본 식비를 반환하는 MealExpense 객체를 반환한다.  
따라서 코드를 부르는 입장에서는 예외적인 상황을 신경쓰지 않아도 되고, 예외 상황은 캡슐화된다.  
<br>

## null을 반환하지마라 <a name = "8"></a>
null을 반환하고 싶다면 그 대신 예외를 던지거나 특수 사례 객체(위에서 처리한 방식)를 반환한다.  
만약 외부 API가 null을 반환한다면 감싸기 메서드를 구현해 예외를 던지거나 특수 사례 객체를 반환하는 방식을 고려한다.  
대부분의 경우 특수 사례 객체가 효율적인 방안이다.  
```java
// Bad
List<Employee> employees = getEmployees();
if (employees != null) {
  for(Employee e : employees) {
    totalPay += e.getPay();
  }
}
```
```java
// Good
List<Employee> employees = getEmployees();
for(Employee e : employees) {
  totalPay += e.getPay();
}

public List<Employee> getEmployees() {
  if( .. there are no employees .. )
    return Collections.emptyList();
  }
}
```
비어있다면 굳이 null을 반환할 필요가 없다.  
빈 리스트를 반환한다면 더욱 깔끔한 코드를 유지할 수 있다.  
<br>

## null을 전달하지 마라 <a name = "9"></a>
정상적인 인수로 null을 기대하는 API가 아니라면 메서드로 null을 전달하는 코드는 최대한 피한다.  
```java
// Bad
// calculator.xProjection(null, new Point(12, 13));
// 위와 같이 부를 경우 NullPointerException 발생
public class MetricsCalculator {
  public double xProjection(Point p1, Point p2) {
    return (p2.x – p1.x) * 1.5;
  }
  ...
}

// Bad
// NullPointerException은 안나지만 윗단계에서 InvalidArgumentException이 발생할 경우 처리해줘야 함.
public class MetricsCalculator {
  public double xProjection(Point p1, Point p2) {
    if(p1 == null || p2 == null){
      throw InvalidArgumentException("Invalid argument for MetricsCalculator.xProjection");
    }
    return (p2.x – p1.x) * 1.5;
  }
}

// Bad
// 좋은 명세이지만 첫번째 예시와 같이 NullPointerException 문제를 해결하지 못한다.
public class MetricsCalculator {
  public double xProjection(Point p1, Point p2) {
    assert p1 != null : "p1 should not be null";
    assert p2 != null : "p2 should not be null";
    
    return (p2.x – p1.x) * 1.5;
  }
}
```
결과적으로 코드가 전부 더럽다.  
대다수 프로그래밍 언어는 호출자가 실수로 넘기는 null을 적절히 처리하는 방법이 없다.  
따라서 애초에 null을 넘기지 못 하도록 금지하는 정책이 합리적이다.  

<br>

## 결론 <a name = "10"></a>
깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다.  
오류 처리를 프로그램 논리와 분리해 독자적인 사안으로 고려하면 튼튼하고 깨끗한 코드를 작성할 수 있다.  








