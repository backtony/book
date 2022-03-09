
## 목차
- [테스트 코드 작성 순서](#1)
- [초반에 복잡한 테스트부터 시작하면 안 되는 이유](#2)
- [예외 상황을 먼저 테스트해야 하는 이유](#3)
- [완급 조절](#4)
- [지속적인 리팩토링](#5)
- [테스트 작성 순서 연습](#6)
- [테스트할 목록 정리하기](#7)

---


## 테스트 코드 작성 순서 <a name = "1"></a>
+ 쉬운 경우에서 어려운 경우로 진행
+ 예외적인 경우에서 정상적인 경우에서 진행

앞선 포스팅에서 암호 강도 측정 기능을 구현했을 때 쉬운 경우에서 어려운 경우로 진행하는 방식을 선택했다.  
반대로 어려운 경우를 먼저 시작하거나 정상 상황을 먼저 시작하면 구현 과정이 원활하게 진행되지 않기도 한다.  

## 초반에 복잡한 테스트부터 시작하면 안 되는 이유 <a name = "2"></a>
초반부터 다양한 조합을 검사하는 복잡한 상황을 테스트로 추가하면 해당 테스트를 통과시키기 위해 한 번에 구현해야 할 코드가 많아진다.  
예를 들어 다음과 같은 조건이 있다고 해보자.
+ 모든 규칙을 충족하지 않는 경우
+ 한 규칙만 충족하는 경우
+ 두 규칙을 충족하는 경우

모든 규칙을 충족하지 않는 경우를 테스트하기 위해서는 결국 모든 규칙을 검사하는 코드를 구현해야 할 것 같다.  
한 번에 구현해야 할 코드가 너무 많아진다.  
한 규칙만 충족하는 경우에는 한 규칙을 충족하는지 여부를 검사해서 WEAK를 리턴하면 될 것 같다.  
가장 쉬운 것은 한 규칙만 충족하는 조건이라는 것을 알았으니 어떤 규칙을 가장 먼저 할지 고민해야 한다.
+ 길이가 8글자 이상
+ 대문자 포함
+ 숫자를 포함

8글자를 검사하는 것이 가장 쉬워보인다.
+ 길이가 8글자 미만이고 나머지 규칙은 충족
+ 길이가 8글자 이상이고 나머지 규칙은 충족하지 않으면 강도는 약함

둘 다 길이가 8글자 이상인지 여부를 판단하는 로직만 구현하면 테스트를 통과시킬 수 있다.  
이렇게 하나의 테스트를 통과했으면 그다음으로 구현하기 쉬운 테스트를 선택한다.  
이를 통해 점진적으로 구현을 완성해 나갈 수 있다.  
한 번에 구현하는 시간이 짧아지면 디버깅에 유리하다.  
작성한 코드가 많지 않고 작성 시간도 짧으면 머릿속에 코드에 대한 내용이 생생하게 남아 있기 때문에 디버깅할 때 문제 원인을 빠르게 찾을 수 있다.

## 예외 상황을 먼저 테스트해야 하는 이유 <a name = "3"></a>
다양한 예외 상황은 복잡한 if-else 블록을 동반할 때가 많다.  
예외 상황을 전혀 고려하지 않은 코드에 예외 상황을 반영하려면 코드를 복잡하게 만들어 버그 발생 가능성을 높인다.  
초반에 예외 상황을 테스트하면 이런 가능성이 줄어든다.  
if-else 구조가 미리 만들어지기 때문에 많은 코드를 완성한 뒤에 예외 상황을 반영할 때보다 코드 구조가 덜 바뀐다. 
TDD를 하는 동안 예외 상황을 찾고 테스트에 반영하면 예외 상황을 처리하지 않아 발생하는 버그도 줄어든다.

## 완급 조절 <a name = "4"></a>
처음 TDD로 구현할 때 어려운 것 중 하나는 한 번에 얼마만큼의 코드를 작성할 것인가이다.  
TDD를 처음 접할 때는 다음 단계에 따라 TDD를 익혀보자.  
1. 정해진 값을 리턴
2. 값 비교를 이용해서 정해진 값을 리턴
3. 다양한 테스트를 추가하면서 구현을 일반화

뻔한 구현이라도 위 단계를 거쳐서 연습하는 것과 바로 구현하는 것에는 차이가 있다.  
테스트를 만들고 통과시키는 과정에서 구현이 막힐 때가 있다.  
이럴 때 위 단계를 이용해서 TDD를 연습한 개발자는 조금씩 기능을 구현해 나갈 수 있다.

## 지속적인 리팩토링 <a name = "5"></a>
테스트를 통과한 뒤에는 리팩토링을 진행한다. 매번 진행하는 것은 아니지만 적당한 후보가 보이면 진행한다.  
코드의 중복은 대표적인 리팩토링 대상이고 코드가 길어지면 메서드 추출과 같은 기법으로 처리한다.  
TDD를 진행하는 과정에서 지속적인 리팩토링을 하면 코드 가독성이 높아지고 향후 유지보수에 도움이 된다.  

### 적절한 시점
테스트 대상 코드에서 상수를 변수로 바꾸거나 변수 이름을 변경하는 것과 같은 작은 리팩토링은 발견하는 순간 바로 해결한다.  
반면에 메서드 추출과 같이 메서드의 구조에 영향을 주는 리팩토링은 큰 틀에서 구현 흐름이 눈에 들어오기 시작한 뒤에 진행한다.  
구현 초기에 아직 구현의 전반적인 흐름을 모르기 때문에 메서드 추출과 같은 리팩토링을 진행하면 코드 구조를 잘못 잡을 가능성이 있기 때문에 주의해야 한다.

## 테스트 작성 순서 연습 <a name = "6"></a>
매달 비용을 지불해야 사용할 수 있는 유료 서비스가 있다고 해보자.  
이 서비스는 다음 규칙에 따라 서비스 만료일을 결정한다.
+ 서비스를 사용하려면 매달 1만원을 선불로 납부해야 한다. 납부일 기준으로 한 달 뒤가 서비스 만료일이다.
+ 2개월 이상 요금을 납부할 수 있다.
+ 10만원 납부하면 서비스를 1년 제공한다.

먼저 테스트 클래스 이름을 정하자. 좋은 이름 찾기는 어렵고 시간도 오래 걸리지만 그만큼 중요한 작업이다.  
번역기, 파파고, 인터넷 사전을 참고해서 잘 지어야 한다.
```java
public class ExpiryDateCalculatorTest {
}
```

### 첫 번째 테스트
+ 구현하기 쉬운 것부터 먼저 테스트
+ 예외 상황을 먼저 테스트

1만원을 납부하면 한 달 뒤 같은 날에 만료일을 계산하는 것이 가장 쉬워보인다.  
계산에 필요한 값은 납부일과 납부액이고 결과는 계산된 만료일이다. 테스트로 표현해보자.
```java
public class ExpiryDateCalculatorTest {

    @Test
    void 만원_납부하면_한달_뒤가_만료일() throws Exception{
        LocalDate billingDate = LocalDate.of(2019, 3, 1);
        int payAmount = 10_000;

        ExpiryDateCalculator cal = new ExpiryDateCalculator();
        LocalDate expiryDate = cal.calculateExpiryDate(billingDate, payAmount);

        assertEquals(LocalDate.of(2019,4,1),expiryDate);

        LocalDate billingDate2 = LocalDate.of(2019, 5, 5);
        int payAmount2 = 10_000;

        ExpiryDateCalculator cal2 = new ExpiryDateCalculator();
        LocalDate expiryDate2 = cal2.calculateExpiryDate(billingDate2, payAmount2);

        assertEquals(LocalDate.of(2019,6,5),expiryDate2);
    }
}
```
앞선 포스팅과 달리 테스트 메서드 이름을 한글로 작성했다.  
한글 사용에 대해 호불호가 존재하지만 공동 작업자에 외국인이 없다면 테스트 메서드 이름을 한글로 사용하는 것도 괜찮다.  
테스트는 당연히 실패하고 로직을 만들자.
```java
public class ExpiryDateCalculator {
    public LocalDate calculateExpiryDate(LocalDate billingDate, int payAmount) {
        return billingDate.plusMonths(1);        
    }
}
```

### 코드 정리
ExpiryDateCalculator 클래스부터 보자. calculateExpiryDate메서드는 파라미터가 두 개이다.  
파라미터가 더 많으면 객체 형태로 바꿔서 한 개로 만들겠지만 아직까지는 더 추가될지 알 수 없다.  
그렇다면 테스트 코드를 보자. 중복 코드가 보이므로 메서드로 뽑아내자.
```java
public class ExpiryDateCalculatorTest {

    @Test
    void 만원_납부하면_한달_뒤가_만료일() throws Exception{
        assertExpiryDate(LocalDate.of(2019,3,1),
                10_000,
                LocalDate.of(2019,4,1));

        assertExpiryDate(LocalDate.of(2019,3,1),
                10_000,
                LocalDate.of(2019,4,1));
    }

    private void assertExpiryDate(LocalDate billingDate, int payAmount, LocalDate expectedExpiryDate) {
        ExpiryDateCalculator cal = new ExpiryDateCalculator();
        LocalDate expiryDate = cal.calculateExpiryDate(billingDate, payAmount);
        assertEquals(expectedExpiryDate,expiryDate);
    }
}
```

### 예외 상황 처리
쉬운 구현을 하나 했으니 이제 예외 상황을 찾아보자.  
단순히 한 달 추가로 끝나지 않는 상황이 발생한다.
+ 납부일이 2019-1-31이고 납부액이 1만원이면 만료일은 2019-2-28
+ 납부일이 2019-5-31이고 납부액이 1만원이면 만료일은 2019-6-30
+ 납부일이 2020-1-31이고 납부액이 1만원이면 만료일은 2020-2-29

위 예시에서는 납부일 기준으로 다음 달의 같은 날이 만료일이 아니다. 테스트 코드를 추가해보자.

```java
public class ExpiryDateCalculatorTest {

    ...

    @Test
    void 납부일과_한달_뒤_일자가_같지_않음() throws Exception{
        assertExpiryDate(LocalDate.of(2019,1,31),
                10_000,
                LocalDate.of(2019,2,28));
        assertExpiryDate(LocalDate.of(2019,5,31),
                10_000,
                LocalDate.of(2019,6,30));
        assertExpiryDate(LocalDate.of(2020,1,31),
                10_000,
                LocalDate.of(2020,2,29));
    }
    ...

}
```
테스트는 통과하는데 이는 LocalDate의 plusMonths 메서드가 알아서 한 달을 처리해주기 때문이다.

### 다음 테스트 선택
이제 다음으로 쉽거나 예외적인 것을 선택하자.
+ 쉬운 것
    + 2만원 지불하면 만료일은 2달 뒤
    + 3만원 지불하면 만료일은 3달 뒤
+ 예외
    + 첫 납부일이 2019-01-31이고 만료되는 2019-02-28에 1만원을 납부하면 다음 만료일은 2019-03-31이다.
    + 첫 납부일이 2019-01-30이고 만료되는 2019-02-28에 1만원을 납부하면 다음 만료일은 2019-03-30이다.
    + 첫 납부일이 2019-05-31이고 만료되는 2019-06-30에 1만원을 납부하면 다음 만료일은 2019-07-31이다.

쉬운 것은 2개월 이상 요금 지불이고 예외 상황은 1개월 요금을 지불할 때 발생할 수 있는 예외 상황이다.  
이전 테스트가 1개월 요금 지불 기준으로 진행했으므로 먼저 예외를 처리하는 것이 나아 보인다.  


### 다음 테스트 전 리팩토링
예외 상황을 테스트하려면 첫 납부일이 필요하다. 앞서 작성한 테스트는 납부일과 납부액만 사용했기에 파라미터를 추가해야 한다.  
파라미터가 3개로 늘었기에 파라미터를 추가 할 지, 객체로 전달할지 고민해야 할 때가 왔다.  
파라미터 개수는 적을수록 가독성과 유지보수에 유리하므로 3개 이상이라면 객체로 바꾸는 것을 고려해야 한다.  
파라미터를 3개로 추가하기 전에 객체로 전달하는 것으로 리팩토링해보자.
```java
@Getter
@Builder
@NoArgsConstructor(access = AccessLevel.PRIVATE)
@AllArgsConstructor
public class PayData {
    LocalDate billingDate;
    int payAmount;
}
```
```java
public class ExpiryDateCalculator {
    public LocalDate calculateExpiryDate(PayData payData) {
        return payData.getBillingDate().plusMonths(1);
    }
}
```
```java
public class ExpiryDateCalculatorTest {

    @Test
    void 만원_납부하면_한달_뒤가_만료일() throws Exception{
        assertExpiryDate(
                PayData.builder()
                        .billingDate(LocalDate.of(2019,3,1))
                        .payAmount(10_000).build(),
                LocalDate.of(2019,4,1));

        assertExpiryDate(
                PayData.builder()
                        .billingDate(LocalDate.of(2019, 5, 5))
                        .payAmount(10_000).build(),
                LocalDate.of(2019,6,5));
    }

    @Test
    void 납부일과_한달_뒤_일자가_같지_않음() throws Exception{
        assertExpiryDate(
                PayData.builder()
                        .billingDate(LocalDate.of(2019,1,31))
                        .payAmount(10_000).build(),
                LocalDate.of(2019,2,28));
        assertExpiryDate(
                PayData.builder()
                        .billingDate(LocalDate.of(2019,5,31))
                        .payAmount(10_000).build(),
                LocalDate.of(2019,6,30));
        assertExpiryDate(
                PayData.builder()
                        .billingDate(LocalDate.of(2020,1,31))
                        .payAmount(10_000).build(),
                LocalDate.of(2020,2,29));
    }

    private void assertExpiryDate(PayData payData, LocalDate expectedExpiryDate) {

        ExpiryDateCalculator cal = new ExpiryDateCalculator();
        LocalDate expiryDate = cal.calculateExpiryDate(payData);
        assertEquals(expectedExpiryDate,expiryDate);
    }
}
```
테스트를 돌려보면 통과한다.

### 예외 상황 테스트 진행

```java
public class ExpiryDateCalculatorTest {
    @Test
    void 첫_납부일과_만료일_일자가_다를때_만원_납부() throws Exception{
        PayData payData= PayData.builder()
                .firstBillingDate(LocalDate.of(2019,1,31))
                .billingDate(LocalDate.of(2019,2,28))
                .payAmount(10_000)
                .build();
        
        assertExpiryDate(payData, LocalDate.of(2019,3,31));
    }
    ...
}
```
컴파일 에러가 나므로 수정해주자.
```java
@Getter
@Builder
@NoArgsConstructor(access = AccessLevel.PRIVATE)
@AllArgsConstructor
public class PayData {
    
    LocalDate firstBillingDate;
    
    LocalDate billingDate;
    
    int payAmount;
}
```
테스트를 돌려보면 실패한다. 일단 상수를 이용해서 테스트를 우선적으로 통과시켜보자.
```java
public class ExpiryDateCalculator {
    public LocalDate calculateExpiryDate(PayData payData) {
        if(payData.getFirstBillingDate().equals(LocalDate.of(2019,1,31)))
            return LocalDate.of(2019,3,31);
        return payData.getBillingDate().plusMonths(1);
    }
}
```
테스트를 돌려보면 현재 진행하는 테스트는 성공했으나 앞선 테스트들이 NullPointException으로 실패한다.  
따라서 Null 검사하는 로직을 추가해주자.
```java
public class ExpiryDateCalculator {
    public LocalDate calculateExpiryDate(PayData payData) {
        if(payData.getFirstBillingDate() != null){
            if(payData.getFirstBillingDate().equals(LocalDate.of(2019,1,31)))
                return LocalDate.of(2019,3,31);
        }
        return payData.getBillingDate().plusMonths(1);
    }
}
```
이제 테스트가 모두 통과한다. 새로운 테스트 케이스를 추가해서 일반화할 차례다.
```java
public class ExpiryDateCalculatorTest {

    @Test
    void 첫_납부일과_만료일_일자가_다를때_만원_납부() throws Exception{
        PayData payData= PayData.builder()
                .firstBillingDate(LocalDate.of(2019,1,31))
                .billingDate(LocalDate.of(2019,2,28))
                .payAmount(10_000)
                .build();

        assertExpiryDate(payData, LocalDate.of(2019,3,31));

        PayData payData2= PayData.builder()
                .firstBillingDate(LocalDate.of(2019,1,30))
                .billingDate(LocalDate.of(2019,2,28))
                .payAmount(10_000)
                .build();

        assertExpiryDate(payData2, LocalDate.of(2019,3,30));
    }

    ...
}
```
테스트가 깨진다. 테스트를 통과할 만큼만 구현을 일반화해보자.
```java
public class ExpiryDateCalculator {
    public LocalDate calculateExpiryDate(PayData payData) {
        if(payData.getFirstBillingDate() != null){
            // 후보 만료일
            LocalDate candidateExp = payData.getBillingDate().plusMonths(1);

            // 첫 납부일의 일자와 후보 만료일의 일자가 다르면 첫 납부일의 일자를 후보 만료일의 일자로 사용
            if(payData.getFirstBillingDate().getDayOfMonth() != candidateExp.getDayOfMonth()){
                return candidateExp.withDayOfMonth(payData.getFirstBillingDate().getDayOfMonth());
            }
        }
        return payData.getBillingDate().plusMonths(1);
    }
}
```
테스트가 통과하니 추가적인 테스트 코드를 작성한다.
```java
public class ExpiryDateCalculatorTest {

    @Test
    void 첫_납부일과_만료일_일자가_다를때_만원_납부() throws Exception{
        PayData payData= PayData.builder()
                .firstBillingDate(LocalDate.of(2019,1,31))
                .billingDate(LocalDate.of(2019,2,28))
                .payAmount(10_000)
                .build();

        assertExpiryDate(payData, LocalDate.of(2019,3,31));

        PayData payData2= PayData.builder()
                .firstBillingDate(LocalDate.of(2019,1,30))
                .billingDate(LocalDate.of(2019,2,28))
                .payAmount(10_000)
                .build();

        assertExpiryDate(payData2, LocalDate.of(2019,3,30));

        PayData payData3= PayData.builder()
                .firstBillingDate(LocalDate.of(2019,5,31))
                .billingDate(LocalDate.of(2019,6,30))
                .payAmount(10_000)
                .build();

        assertExpiryDate(payData3, LocalDate.of(2019,7,31));
    }

    ...
}
```
테스트가 통과한다.

### 코드 정리
테스트를 통과했으니 코드를 정리할 차례다. 아직까지 크게 바꿀 부분은 없어 보이는데 상수 1만 변수로 빼도록 하자.
```java
public class ExpiryDateCalculator {
    public LocalDate calculateExpiryDate(PayData payData) {
        int addedMonths = 1;
        if(payData.getFirstBillingDate() != null){
            LocalDate candidateExp = payData.getBillingDate().plusMonths(addedMonths);
            if(payData.getFirstBillingDate().getDayOfMonth() != candidateExp.getDayOfMonth()){
                return candidateExp.withDayOfMonth(payData.getFirstBillingDate().getDayOfMonth());
            }
        }
        return payData.getBillingDate().plusMonths(addedMonths);
    }
}
```

### 다음 테스트 선택
+ 2만원 지불하면 만료일이 두 달 뒤
+ 3만원 지불하면 만료일은 세 달 뒤

```java
public class ExpiryDateCalculatorTest {

    @Test
    void 이만원_이상_납부하면_비례해서_만료일_계산() throws Exception{
        assertExpiryDate(
                PayData.builder()
                .billingDate(LocalDate.of(2019,3,1))
                .payAmount(20_000)
                .build(),
                LocalDate.of(2019,5,1)
        );
    }
    ...
}
```
테스트는 실패하니 로직을 수정한다.
```java
public class ExpiryDateCalculator {
    public LocalDate calculateExpiryDate(PayData payData) {
        int addedMonths = payData.getPayAmount() / 10_000;

        if(payData.getFirstBillingDate() != null){
            LocalDate candidateExp = payData.getBillingDate().plusMonths(addedMonths);
            if(payData.getFirstBillingDate().getDayOfMonth() != candidateExp.getDayOfMonth()){
                return candidateExp.withDayOfMonth(payData.getFirstBillingDate().getDayOfMonth());
            }
        }
        return payData.getBillingDate().plusMonths(addedMonths);
    }
}
```
테스트가 통과하니 다른 케이스를 추가하자
```java
public class ExpiryDateCalculatorTest {

    @Test
    void 이만원_이상_납부하면_비례해서_만료일_계산() throws Exception{
        assertExpiryDate(
                PayData.builder()
                .billingDate(LocalDate.of(2019,3,1))
                .payAmount(20_000)
                .build(),
                LocalDate.of(2019,5,1)
        );

         assertExpiryDate(
                PayData.builder()
                        .billingDate(LocalDate.of(2019,3,1))
                        .payAmount(30_000)
                        .build(),
                LocalDate.of(2019,6,1)
        );
    }
    ...
}
```
테스트는 모두 통과한다.

### 예외 상황 테스트 추가
첫 납부일과 납부일자가 다를 때 2만원 이상 납부한 경우를 처리하자.
+ 첫 납부일이 2019-01-31이고 만료되는 2019-02-28에 2만원을 납부하면 다음 만료일은 2019-04-30이다.

```java
public class ExpiryDateCalculatorTest {

    @Test
    void 첫_납부일과_만료일_일자가_다를때_이만원_이상_납부() throws Exception{
        assertExpiryDate(
                PayData.builder()
                    .firstBillingDate(LocalDate.of(2019,1,31))
                    .billingDate(LocalDate.of(2019,2,28))
                    .payAmount(20_000)
                    .build(),
                LocalDate.of(2019,4,30)
        );
    }
    ...
}
```
테스트는 4월은 31이 없다고 실패한다. 로직을 수정하자.  
'후보 만료일이 포함된 달의 마지막 날 < 첫 납부일의 일자' 라면 만료일을 그달의 마지막 날로 조정해야한다.
```java
public class ExpiryDateCalculator {
    public LocalDate calculateExpiryDate(PayData payData) {
        int addedMonths = payData.getPayAmount() / 10_000;

        if(payData.getFirstBillingDate() != null){
            LocalDate candidateExp = payData.getBillingDate().plusMonths(addedMonths);
            if(payData.getFirstBillingDate().getDayOfMonth() != candidateExp.getDayOfMonth()){
                if (YearMonth.from(candidateExp).lengthOfMonth() < payData.getFirstBillingDate().getDayOfMonth()){
                    return candidateExp.withDayOfMonth(YearMonth.from(candidateExp).lengthOfMonth());
                }
                return candidateExp.withDayOfMonth(payData.getFirstBillingDate().getDayOfMonth());
            }
        }
        return payData.getBillingDate().plusMonths(addedMonths);
    }
}
```
이제 테스트는 통과한다.

### 코드 정리
방금 작성한 calculateExpiryDate 메서드 코드가 너무 복잡하다.  
코드가 복잡한 이유 중 하나는 날짜 관련 계산 코드가 중복해서 존재하기 때문이다.  
먼저 후보 만료일이 속한 월의 마지막 일자를 구하는 코드의 중복을 없애자.
```java
public class ExpiryDateCalculator {
    public LocalDate calculateExpiryDate(PayData payData) {
        int addedMonths = payData.getPayAmount() / 10_000;

        if(payData.getFirstBillingDate() != null){
            LocalDate candidateExp = payData.getBillingDate().plusMonths(addedMonths);
            if(payData.getFirstBillingDate().getDayOfMonth() != candidateExp.getDayOfMonth()){
                // 변수 추출로 중복 제거
                final int dayLenOfCandiMon = YearMonth.from(candidateExp).lengthOfMonth();
                if (dayLenOfCandiMon < payData.getFirstBillingDate().getDayOfMonth()){
                    return candidateExp.withDayOfMonth(dayLenOfCandiMon);
                }
                return candidateExp.withDayOfMonth(payData.getFirstBillingDate().getDayOfMonth());
            }
        }
        return payData.getBillingDate().plusMonths(addedMonths);
    }
}
```
테스트가 문제없는지 테스트를 돌려본다.  
다음은 첫 납부일의 일자를 구하는 중복 코드를 제거한다.
```java
public class ExpiryDateCalculator {
    public LocalDate calculateExpiryDate(PayData payData) {
        int addedMonths = payData.getPayAmount() / 10_000;

        if(payData.getFirstBillingDate() != null){
            LocalDate candidateExp = payData.getBillingDate().plusMonths(addedMonths);
            // 변수 추출로 중복 제거
            final int dayOfFirstBilling = payData.getFirstBillingDate().getDayOfMonth();
            if(dayOfFirstBilling != candidateExp.getDayOfMonth()){
                // 변수 추출로 중복 제거
                final int dayLenOfCandiMon = YearMonth.from(candidateExp).lengthOfMonth();
                if (dayLenOfCandiMon < dayOfFirstBilling){
                    return candidateExp.withDayOfMonth(dayLenOfCandiMon);
                }
                return candidateExp.withDayOfMonth(dayOfFirstBilling);
            }
        }
        return payData.getBillingDate().plusMonths(addedMonths);
    }
}
```
아직도 함수가 너무 크기에 가독성이 떨어진다. 의미있는 이름의 함수로 추출하도록 하자.
```java
public class ExpiryDateCalculator {
    public LocalDate calculateExpiryDate(PayData payData) {
        int addedMonths = payData.getPayAmount() / 10_000;

        if (payData.getFirstBillingDate() != null) {
            return expiryDateUsingFirstBillingDate(payData, addedMonths);
        } else {
            return payData.getBillingDate().plusMonths(addedMonths);
        }
    }

    private LocalDate expiryDateUsingFirstBillingDate(PayData payData, int addedMonths) {
        LocalDate candidateExp = payData.getBillingDate().plusMonths(addedMonths);
        final int dayOfFirstBilling = payData.getFirstBillingDate().getDayOfMonth();
        if (dayOfFirstBilling != candidateExp.getDayOfMonth()) {
            final int dayLenOfCandiMon = YearMonth.from(candidateExp).lengthOfMonth();
            if (dayLenOfCandiMon < dayOfFirstBilling) {
                return candidateExp.withDayOfMonth(dayLenOfCandiMon);
            }
            return candidateExp.withDayOfMonth(dayOfFirstBilling);
        } else {
            return candidateExp;
        }
    }
}
```
함수의 추상화 수준과 조건 경계문을 조금 더 깔끔하게 가져가보자.
```java
public class ExpiryDateCalculator {
    public LocalDate calculateExpiryDate(PayData payData) {
        int addedMonths = payData.getPayAmount() / 10_000;

        if (payData.getFirstBillingDate() != null) {
            return expiryDateUsingFirstBillingDate(payData, addedMonths);
        } else {
            return payData.getBillingDate().plusMonths(addedMonths);
        }
    }

    private LocalDate expiryDateUsingFirstBillingDate(PayData payData, int addedMonths) {
        LocalDate candidateExp = payData.getBillingDate().plusMonths(addedMonths);
        final int dayOfFirstBilling = payData.getFirstBillingDate().getDayOfMonth();
        if (isSameDayOfMonth(candidateExp, dayOfFirstBilling)) {
            return candidateExp;
        } else {
            final int dayLenOfCandiMon = lastDayOfMonth(candidateExp);
            if (dayLenOfCandiMon < dayOfFirstBilling) {
                return candidateExp.withDayOfMonth(dayLenOfCandiMon);
            }
            return candidateExp.withDayOfMonth(dayOfFirstBilling);
        }
    }

    private boolean isSameDayOfMonth(LocalDate candidateExp, int dayOfFirstBilling) {
        return dayOfFirstBilling == candidateExp.getDayOfMonth();
    }

    private int lastDayOfMonth(LocalDate candidateExp) {
        return YearMonth.from(candidateExp).lengthOfMonth();
    }
}
```
테스트를 돌려보면 성공한다.

### 다음 테스트
10개월 요금을 납부하면 1년을 제공한다.

```java
public class ExpiryDateCalculatorTest {

    @Test
    void 십만원을_납부하면_1년_제공() throws Exception{
        assertExpiryDate(
                PayData.builder()
                        .billingDate(LocalDate.of(2019,1,28))
                        .payAmount(100_000)
                        .build(),
                LocalDate.of(2020,1,28)
        );
    }
    ...
}
```
테스트는 실패하니 로직을 수정하자.
```java
public class ExpiryDateCalculator {
    public LocalDate calculateExpiryDate(PayData payData) {
        // 여기만 삼항 연산자로 변경
        int addedMonths = payData.getPayAmount() == 100_000 ? 12 : payData.getPayAmount() / 10_000;
        
        if (payData.getFirstBillingDate() != null) {
            return expiryDateUsingFirstBillingDate(payData, addedMonths);
        } else {
            return payData.getBillingDate().plusMonths(addedMonths);
        }
    }
    ...   
}
```
테스트를 돌리면 성공한다.

## 테스트할 목록 정리하기 <a name = "7"></a>
막상 이렇게 테스트를 체계적으로 하기는 정말 힘들다. 그래서 TDD를 시작할 때 테스트할 목록을 미리 정리하는 것이 좋다.  
예를 들어 만료일 계산 기능을 구현할 때에는 다음과 같이 테스트할 목록을 정리할 수 있다.
+ 1만원 납부 후 한 달 뒤가 만료일
+ 달의 마지막 날에 납부하면 다음달 마지막 날이 만료일
+ 2만원 납부하면 2개월 뒤가 만료일
+ 3만원 납부하면 3개월 뒤가 만료일
+ 10만원 납부하면 1년 뒤 만료일

테스트할 내용을 정리했다면 이 중에 어떤 테스트가 구현이 쉬울지 또는 어떤 테스트가 예외적인지 고민한다.  
시간을 들여 구현의 난이도나 구조를 검토하면 다음 테스트를 선택할 때 도움이 된다.  
처음부터 모든 사례를 정리하면 시간도 오래 걸릴뿐더러 쉽지 않다.  
따라서 테스트 과정에서 새로운 테스트 사례를 발견하면 그 사례를 목록에 추가해서 놓치지 않도록 한다.  
TDD는 리팩토링을 통해 지속해서 코드를 정리하는데 개발을 진행하다 보면 변경 범위가 매우 큰 리팩토링 거리를 발견할 때도 있다.  
범위가 큰 리팩토링은 시간이 오래 걸리므로 TDD흐름을 깨기 쉽다. 이때는 리팩토링을 진행하지 말고 테스트를 통과시키는 데 집중한다.  
대신 범위가 큰 리팩토링은 다음 할 일 목록에 추가해서 놓치지 않고 진행할 수 있도록 한다.  

> 리팩토링 범위가 크면 리팩토링에 실패할 수도 있다. 그러니 범위가 큰 리팩토링을 진행하기 전에 코드를 커밋하는 것을 잊지 말자. 또는 별도 브랜치에서 리팩토링을 진행한다. 그래야 실패했을 때 마지막 상태로 쉽게 돌아올 수 있다.

