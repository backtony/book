## 목차
- [TDD란?](#1)
- [예시 1 - Calculator](#2)
- [예시 2 - 암호 검사기](#3)
- [TDD 흐름](#4)
- [테스트가 개발을 주도](#5)
- [지속적인 코드 정리](#6)
- [빠른 피드백](#7)

---

## TDD란? <a name = "1"></a>
TDD는 테스트부터 시작한다. 구현을 먼저 하고 나중에 테스트하는 것이 아니라 먼저 테스트를 하고 그다음에 구현한다.  
여기서 테스트를 먼저 한다는 것은 기능이 올바르게 동작하는지 검증하는 테스트 코드를 작성한다는 것을 의미한다.  
즉, 기능을 검증하는 테스트 코드를 먼저 작성하고 테스트를 통과시키기 위해 개발을 진행하는 것이다.  

## 예시 1 - Calculator <a name = "2"></a>
덧셈 기능을 검증하기 위한 테스트 코드를 작성해보자.
```java
public class CalculatorTest {

    @Test
    void plus() throws Exception{
        int result = Calculator.plus(1,2);
        Assertions.assertEquals(3,result);
    }
}
```
위 코드 작성하기 위해서는 다음과 같은 고민을 한다.
+ 메서드 이름은 plus가 좋을까? sum이 좋을까?
+ 덧셈 기능을 제공하는 메서드는 파리미터가 몇 개여야 할까? 타입은? 반환값은?
+ 메서드를 정적 메서드로 구현할까? 인스턴스 메서드로 구현할까?
+ 메서드를 제공할 클래스 이름은 뭐가 좋을까?

이렇게 고민하고 테스트를 완성하고 나서야 Calculator 클래스를 생성한다.  
```java
public class Calculator {
    public static int plus(int a1, int a2){
        return a1+a2;
    }
}
```
intelliJ의 도움을 받으면 간단하게 클래스를 생성할 수 있다. 이때 default로 test/java... 위치에 생성되게 된다.  
main 위치에 만들어도 되지만 아직 완성된 기능이 아니기 때문에 test에 놓도록 하고 기능이 최종적으로 완성된 이후에 main 위치로 옮겨 배포 대상에 포함시킨다.  
main으로 이동시켰다면 CalculatorTest 클래스를 실행해서 테스트에 통과하는지 다시 한번 확인한다.

> src/test/java 소스 폴더는 배포 대상이 아니므로 src/test/java 폴더에 코드를 만들면 완성되지 않은 코드가 배포되는 것을 방지하는 효과가 있다.

## 예시 2 - 암호 검사기 <a name = "3"></a>
암호 검사기는 문자열을 검사해서 규칙을 준수하는지에 따라 암호를 약함, 보통, 강함으로 구분하고 규칙은 다음과 같다.
+ 길이가 8글자 이상
+ 0부터 9 사이의 숫자를 포함
+ 대문자 포함
+ 위 세 규칙을 모두 충족하면 강함, 2개 충족 보통, 1개 이하 약함

```java
public class PasswordStrengthMeterTest {
    
    @Test
    void name() throws Exception{
    }
}
```
가장 먼저 테스트할 기능의 이름을 정하고 뼈대를 구성한다.  
__첫 번째 테스트를 선택할 때에는 가장 쉽거나 가장 예외적인 상황을 선택해야 한다.__  
+ 모든 규칙을 충족하는 경우
+ 모든 조건을 충족하지 않는 경우

이 중에서 어떤 경우가 시작하기에 좋을까?  
먼저 모든 조건을 충족하지 않는 경우를 생각해보자.  
모든 조건을 충족하지 않는 테스트를 통과시키려면 각 조건을 검사하는 코드를 모두 구현해야 한다. 한 번에 만들어야 할 코드가 많아지므로 첫 번째 테스트 코드를 통과하는 시간도 길어지고 사실상 구현을 다 하고 테스트를 하는 방식과 다르지 않다.  
모든 규칙을 충족하는 경우에는 어떨까? 각 조건을 검사하는 코드를 만들지 않고 강함에 해당하는 값을 리턴하면 테스트를 통과시킬 수 있다. 그러므로 이것을 먼저 만들자.  

### 첫 번째 테스트
```java
public class PasswordStrengthMeterTest {

    @Test
    void meetsAllCriteria_Then_Strong() throws Exception{
        PasswordStrengthMeter meter = new PasswordStrengthMeter();
        타입 결과 = meter.meter("ab12!@AB");
        assertEquals(기대값, 결과);
    }
}
```
코드를 완성하려면 메서드 리턴 타입을 결정해야 한다. 결과값은 암호의 강도이므로 열거 타입을 사용하면 잘 표현할 수 있을 것 같다.

```java
public class PasswordStrengthMeterTest {

    @Test
    void meetsAllCriteria_Then_Strong() throws Exception{
        PasswordStrengthMeter meter = new PasswordStrengthMeter();
        PasswordStrength result = meter.meter("ab12!@AB");
        assertEquals(PasswordStrength.STRONG, result);
    }
}
```
테스트는 완성되었으니 이제 컴파일 에러를 해결하자.

```java
public enum PasswordStrength {
    STRONG
}
```
```java
public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){
        return null;
    }
}
```
__열거값을 미리 작성할 수 있겠지만 TDD는 테스트를 통과시킬 만큼의 코드만 작성해야한다.__  
코드를 돌려보면 기대값이 STRONG인데 null이어서 테스트에 실패했음을 알 수 있다.  

```java
public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){
        return PasswordStrength.STRONG;
    }
}
```
PasswordStrengthMeter코드를 수정하고 돌리면 성공한다.  
```java
public class PasswordStrengthMeterTest {

    @Test
    void meetsAllCriteria_Then_Strong() throws Exception{
        PasswordStrengthMeter meter = new PasswordStrengthMeter();
        PasswordStrength result = meter.meter("ab12!@AB");
        assertEquals(PasswordStrength.STRONG, result);

        PasswordStrength result2 = meter.meter("ab12!@ABB");
        assertEquals(PasswordStrength.STRONG, result2);
    }
}
```
테스트 코드를 하나 더 추가하고 돌려봐도 통과한다.  


### 두 번째 테스트
길이만 8 미만이고 나머지 조건은 충족하는 경우로 강도는 보통이어야 한다.
```java
public class PasswordStrengthMeterTest {
    @Test
    void meetsOtherCriteria_except_for_Length_Then_Normal() throws Exception{
        PasswordStrengthMeter meter = new PasswordStrengthMeter();
        PasswordStrength result = meter.meter("ab12!@A");
        assertEquals(PasswordStrength.NORMAL, result);
    }
    ...
}
```
NORMAL이 없어 컴파일 에러가 발생하므로 NORMAL을 추가해준다.
```java
public enum PasswordStrength {
    STRONG, NORMAL
}
```
테스트 전체를 돌려보면 앞서 만든 테스트는 통과하지만 현재 테스트는 통과하지 못한다. 따라서 로직을 수정한다.
```java
public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){
        if(s.length() < 8){
            return PasswordStrength.NORMAL;
        }
        return PasswordStrength.STRONG;
    }
}
```
이제 돌리면 성공한다. 테스트를 조금 더 추가해보자.
```java
public class PasswordStrengthMeterTest {
    @Test
    void meetsOtherCriteria_except_for_Length_Then_Normal() throws Exception{
        PasswordStrengthMeter meter = new PasswordStrengthMeter();
        PasswordStrength result = meter.meter("ab12!@A");
        assertEquals(PasswordStrength.NORMAL, result);

        PasswordStrength result2 = meter.meter("aB12!");
        assertEquals(PasswordStrength.NORMAL, result2);
    }
    ...
}
```
통과하므로 다음으로 넘어가자.

### 세 번째 테스트
숫자를 포함하지 않고 나머지 조건은 충족하는 경우로 강도는 보통이어야 한다.
```java
public class PasswordStrengthMeterTest {
    @Test
    void meetsOtherCriteria_except_for_number_then_normal() throws Exception{
        PasswordStrengthMeter meter = new PasswordStrengthMeter();
        PasswordStrength result = meter.meter("ab!@ABqewr");
        assertEquals(PasswordStrength.NORMAL, result);
    }
    ...
}
```
실패하므로 로직을 수정한다.
```java
public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){
        if(s.length() < 8){
            return PasswordStrength.NORMAL;
        }

        boolean containsNum = false;
        for (char ch : s.toCharArray()) {
            if (ch >= '0' && ch <= '9'){
                containsNum = true;
                break;
            }
        }
        if (!containsNum){
            return PasswordStrength.NORMAL;
        }
        
        return PasswordStrength.STRONG;
    }
}
```
테스트를 돌리면 이제 통과하니 가독성을 위해 코드를 리팩토링해보자.
```java
public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){
        if(s.length() < 8){
            return PasswordStrength.NORMAL;
        }

        boolean containsNum = meetsContainingNumberCriteria(s);
        if (!containsNum){
            return PasswordStrength.NORMAL;
        }

        return PasswordStrength.STRONG;
    }

    private boolean meetsContainingNumberCriteria(String s) {
        for (char ch : s.toCharArray()) {
            if (ch >= '0' && ch <= '9'){
                return true;
            }
        }
        return false;
    }
}
```

### 테스트 코드 정리
지금까지 세 개의 테스트 메서드를 추가했고 테스트 코드가 매우 유사한 것을 확인할 수 있다.  
테스트 코드도 코드이기 때문에 유지보수 대상이다. 따라서 중복을 제거하고 의미가 잘 드러나도록 수정할 필요가 있다.  
먼저 Meter 객체 생성 코드의 중복을 제거하자.  
```java
public class PasswordStrengthMeterTest {
    PasswordStrengthMeter meter = new PasswordStrengthMeter();
    ...
}
```
클래스의 필드로 빼내서 처리하고 다시 테스트 코드를 실행해 코드가 깨지지 않았는지 확인한다.  
암호 강도 측정 기능을 실행하고 이를 확인하는 코드 또한 중복으로 제거할 수 있다.
```java
public class PasswordStrengthMeterTest {

    PasswordStrengthMeter meter = new PasswordStrengthMeter();

    private void assertStrength(String password, PasswordStrength expStr) {
        PasswordStrength result2 = meter.meter(password);
        assertEquals(expStr, result2);
    }
    ...
}
```
이렇게 수정하고 다시 돌려서 테스트가 성공하는지 확인한다.

### 네 번째 테스트
값이 없는 경우에는 INVALID를 리턴하도록 한다.
```java
public class PasswordStrengthMeterTest {
    @Test
    void nullInput_then_invalid() throws Exception{        
        assertStrength(null,PasswordStrength.INVALID);
    }

    @Test
    void emptyInput_then_invalid() throws Exception{
        assertStrength("",PasswordStrength.INVALID);
    }
    ...
}
```
```java
public enum PasswordStrength {
    STRONG, NORMAL, INVALID
}
```
테스트를 만들고 INVALID를 추가한다. 테스트를 돌리면 당연히 실패한다. 이제 로직을 수정하자.
```java
public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){

        if(!StringUtils.hasText(s)){
            return PasswordStrength.INVALID;
        }

        ....
    }
}
```
이제 돌려보면 통과한다.

### 다섯 번째 테스트
대문자를 포함하지 않고 나머지 조건을 충족하는 경우 Normal을 반환해야 한다.
```java
public class PasswordStrengthMeterTest {
    @Test
    void meetsOtherCriteria_except_for_uppercase_then_normal() throws Exception{
        assertStrength("ab12!@df",PasswordStrength.NORMAL);
    }
    ...
}
```
돌리면 실패하니 로직을 수정하자.
```java
public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){

       ...

        boolean containsUpp = meetsContainingUppercaseCriteria(s);

        if (!containsUpp){
            return PasswordStrength.NORMAL;
        }

        return PasswordStrength.STRONG;
    }

    private boolean meetsContainingUppercaseCriteria(String s) {
        boolean containsUpp = false;
        for (char ch : s.toCharArray()) {
            if (Character.isUpperCase(ch)){
                containsUpp = true;
                break;
            }
        }
        return containsUpp;
    }

    ...
}
```
로직을 수정하고 테스트를 돌리면 성공한다.

### 여섯 번째 테스트
길이가 8글자 이상인 조건만 충족하는 경우 WEAK를 반환한다.
```java
public class PasswordStrengthMeterTest {
    @Test
    void meetsOnlyLengthCriteria_then_weak() throws Exception{
        assertStrength("abdefghi",PasswordStrength.WEAK);
    }
    ...
}
```
컴파일 에러가 나니 우선 해결한다.
```java
public enum PasswordStrength {
    STRONG, NORMAL, INVALID, WEAK
}
```
테스트를 돌리면 실패한다. WEAK가 아니라 NORMAL를 반환하고 있다.  
테스트를 통과시키려면 세 조건 중에서 길이 조건은 충족하고 나머지 두 조건은 충족하지 않았을 때 WEAK를 리턴하도록 구현해야 한다.  
이를 위해서 먼저 길이가 8 이상인지 여부를 뒤에서 확인할 수 있어야 한다.  
길이가 8 이상인지 여부를 담는 로컬 변수를 추가하고 코드의 위치를 조금 수정하자.
```java
public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){
        
        if(!StringUtils.hasText(s)){
            return PasswordStrength.INVALID;
        }

        boolean lengthEnough = s.length() >= 8;
        boolean containsNum = meetsContainingNumberCriteria(s);
        boolean containsUpp = meetsContainingUppercaseCriteria(s);

        if(!lengthEnough){
            return PasswordStrength.NORMAL;
        }

        if (!containsNum){
            return PasswordStrength.NORMAL;
        }

        if (!containsUpp){
            return PasswordStrength.NORMAL;
        }

        return PasswordStrength.STRONG;
    }
    ...
}
```
이제 새로 추가한 테스트를 해결할 코드를 추가해보자.
```java
public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){

        if(!StringUtils.hasText(s)){
            return PasswordStrength.INVALID;
        }

        boolean lengthEnough = s.length() >= 8;
        boolean containsNum = meetsContainingNumberCriteria(s);
        boolean containsUpp = meetsContainingUppercaseCriteria(s);
        
        if(lengthEnough && !containsNum && !containsUpp){
            return PasswordStrength.WEAK;
        }
        ...
    }

}
```
이제는 테스트가 통과한다. 

### 일곱 번째 테스트
숫자 포함 조건만 충족하는 경우 WEAK이다.
```java
public class PasswordStrengthMeterTest {
    @Test
    void meetsOnlyNumCriteria_then_weak() throws Exception{
        assertStrength("12345",PasswordStrength.WEAK);
    }
    ...
}
```
당연히 테스트는 실패하고 로직을 수정하자.
```java
public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){

        if(!StringUtils.hasText(s)){
            return PasswordStrength.INVALID;
        }

        boolean lengthEnough = s.length() >= 8;
        boolean containsNum = meetsContainingNumberCriteria(s);
        boolean containsUpp = meetsContainingUppercaseCriteria(s);

        ...
        
        if(!lengthEnough && containsNum && !containsUpp){
            return PasswordStrength.WEAK;
        }

        ...
        
    }
    ...
}
```
테스트는 통과한다.

### 여덟 번째 테스트
대문자 포함 조건만 충족하는 경우를 WEAK를 반환한다.
```java
public class PasswordStrengthMeterTest {
    @Test
    void meetsOnlyUpperCriteria_then_weak() throws Exception{
        assertStrength("ABZEF",PasswordStrength.WEAK);
    }
    ...
}
```
테스트는 당연히 실패하고 로직을 수정하자.
```java
public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){

        if(!StringUtils.hasText(s)){
            return PasswordStrength.INVALID;
        }

        boolean lengthEnough = s.length() >= 8;
        boolean containsNum = meetsContainingNumberCriteria(s);
        boolean containsUpp = meetsContainingUppercaseCriteria(s);

        ...
             
        if(!lengthEnough && !containsNum && containsUpp){
            return PasswordStrength.WEAK;
        }

        ...     
    }
    ...
}
```

### 코드 정리
PasswordStrengthMeter 클래스의 meter 메서드가 가독성이 떨어져 보이므로 정리해보자.  
만족하는 조건의 수를 count 변수로 뽑아서 해결해보자.  
```java

public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){

        if(!StringUtils.hasText(s)){
            return PasswordStrength.INVALID;
        }

        int metCounts = 0;
        if (s.length() >= 8)
            metCounts++;
        if(meetsContainingNumberCriteria(s))
            metCounts++;
        if(meetsContainingUppercaseCriteria(s))
            metCounts++;

        if (metCounts == 1)
            return PasswordStrength.WEAK;


        if(metCounts == 2){
            return PasswordStrength.NORMAL;
        }

        return PasswordStrength.STRONG;
    }
    ...
}
```
테스트를 다시 돌려보면 성공하는 것으로 확인된다. 리팩토링이 잘 수행되었다.  

### 아홉 번째 테스트
아무 조건도 충족하지 않는 경우 WEAK를 반환한다.
```java
public class PasswordStrengthMeterTest {
    ...

    @Test
    void meetsNoCriteria_then_weak() throws Exception{
        assertStrength("abc",PasswordStrength.WEAK);
    }
    ...
}
```
당연히 테스트는 실패하고 로직을 수정한다.
```java
public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){
        ...

        if (metCounts <= 1)
            return PasswordStrength.WEAK;
        ...
    }
    ...
}
```
부등호만 추가해주면 된다.  
지금까지 새로운 테스트를 추가하거나 기존 코드를 수정하면 습관처럼 테스트를 실행했다.  
그리고 실패한 테스트가 있다면 그 테스트를 통과시키기 위한 코드를 추가했다.  
__이것이 TDD이고 이를 어색해지지 않을 때까지 반복해야 한다.__  

### 코드 정리
meter 메서드가 아직은 가독성이 떨어져 보이고 함수 라인수도 너무 많아보인다.  
count를 계산하는 부분을 함수로 뽑아내면 조금 더 깔끔해질 것 같다.
```java
public class PasswordStrengthMeter {
    public PasswordStrength meter(String s){

        if(!StringUtils.hasText(s)){
            return PasswordStrength.INVALID;
        }

        int metCounts = getMetCriteriaCounts(s);

        if (metCounts <= 1)
            return PasswordStrength.WEAK;

        if(metCounts == 2){
            return PasswordStrength.NORMAL;
        }

        return PasswordStrength.STRONG;
    }

    private int getMetCriteriaCounts(String s) {
        int metCounts = 0;
        if (s.length() >= 8)
            metCounts++;
        if(meetsContainingNumberCriteria(s))
            metCounts++;
        if(meetsContainingUppercaseCriteria(s))
            metCounts++;
        return metCounts;
    }

    private boolean meetsContainingUppercaseCriteria(String s) {
        boolean containsUpp = false;
        for (char ch : s.toCharArray()) {
            if (Character.isUpperCase(ch)){
                containsUpp = true;
                break;
            }
        }
        return containsUpp;
    }

    private boolean meetsContainingNumberCriteria(String s) {
        for (char ch : s.toCharArray()) {
            if (ch >= '0' && ch <= '9'){
                return true;
            }
        }
        return false;
    }
}
```
이제 해당 코드를 처음 보는 개발자도 다음과 같이 순차적으로 쉽게 코드를 이해할 수 있다.
+ 암호가 null이거나 빈 문자열이면 강도는 INVALID
+ 충족하는 규칙 개수를 구한다.
+ 충족하는 규칙 개수가 1개 이하면 강도는 WEAK
+ 충족하는 규칙 개수가 2개면 강도는 NORMAL
+ 이외의 경우 STRONG

이제 테스트 코드를 src/main 위치로 이동하면 PasswordStrengthMeter 클래스의 구현이 끝난다.

## TDD 흐름 <a name = "4"></a>

> 테스트 -> 코딩 -> 리팩토링

__TDD는 테스트를 먼저 작성하고 테스트를 통과시킬 만큼 코드를 작성하고 리팩토링으로 마무리하는 과정을 반복한다.__  
TDD 사이클은 레드-그린-리팩터로 부르기도 한다.  
+ 레드 : 테스트 코드가 실패하면 빨간색
+ 그린 : 성공한 테스트
+ 리팩터 : 리팩토링 과정

## 테스트가 개발을 주도 <a name = "5"></a>
테스트 코드를 먼저 작성하면 테스트가 개발을 주도한다.  
앞서 예시에서도 가장 먼저 통과해야 할 테스트를 작성했다. 테스트를 작성하는 과정에서 구현을 생각하지 않았다.  
단지 해당 기능이 올바르게 동작하는지 검증할 수 있는 테스트 코드를 만들었을 뿐이다.  
테스트를 추가한 뒤에는 테스트를 통과시킬 만큼 기능을 구현했다. 아직 추가하지 않은 테스트를 고려해서 구현하지 않았다.

## 지속적인 코드 정리 <a name = "6"></a>
구현을 완료한 뒤에는 리팩토링을 진행했다. 테스트 코드 자체도 리팩토링 대상에 포함된다.  
당장 리팩토링을 하지 않더라도 테스트 코드가 있으면 리팩토링을 과감하게 진행할 수 있다.  
TDD 개발은 개발 과정에서 지속적으로 코드 정리를 하므로 코드 품질이 급격히 나빠지지 않게 막아주는 효과가 있다.  
이는 향후 유지보수 비용을 낮추는데 기여한다.

## 빠른 피드백 <a name = "7"></a>
TDD가 주는 이점은 코드 수정에 대한 피드백이 빠르다는 점이다.  
새로운 코드를 추가하거나 기존 코드를 수정하면 테스트를 돌려서 해당 코드가 올바른지 바로 확인할 수 있다.  
이는 잘못된 코드가 배포되는 것을 방지한다.

