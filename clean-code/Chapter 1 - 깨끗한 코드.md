## 목차
- [코드는 존재하리라](#1)
- [나쁜 코드](#2)
- [나쁜 코드로 치르는 대가](#3)
- [원대한 재설계의 꿈](#4)
- [태도](#5)
- [원초적 난제](#6)
- [깨끗한 코드라는 예술](#7)
- [우리는 저자다](#8)
- [보이스카우트 규칙](#9)
- [결론](#10)

---

## 코드는 존재하리라 <a name = "1"></a>
앞으로는 프로그램이 자동으로 코드를 생성해 줄 것이기 때문에 모델이나 요구사항에 집중해야 한다고 말하는 사람들이 있다.  
이 말은 프로그램에 요구사항을 모호하게 줘도 우리 의도를 정확히 꿰뚫어 프로그램을 완벽하게 실행한다는 뜻이다.  
하지만 이는 절대로 불가능한 기대이다.  
창의력과 직관을 보유한 우리 인간조차도 고객의 막연한 감정만 갖고는 성공적인 시스템을 구현하지 못한다.  
궁극적으로 __코드는 요구사항을 표현하는 언어__ 라는 사실을 명심해야 한다.  
요구사항에 더욱 가까운 언어를 만들 수도 있고 요구사항에서 정형 구조를 뽑아내는 도구를 만들 수도 있지만, 어느 순간에는 정밀한 표현이 결국 필요하다.  
그 필요성을 없앨 방법은 없으므로 __코드는 항상 존재하리라.__  
<br>

## 나쁜 코드 <a name = "2"></a>
Killer App 하나를 구현한 회사가 있다.  
처음에는 큰 인기를 끌었으나 업데이트 주기가 늦어지고, 이전 버그가 다음 버전에도 그대로 남아있더니 얼마못가 망했다.  
__회사가 망한 원인은 바로 나쁜 코드 탓이다.__  
일정을 맞추기 위해 대충 짠 프로그램이 돌아간다는 사실에 안도감을 느끼기며 __안 돌아가는 프로그램보다 돌아가는 쓰레기가 좋다고 스스로를 위안하며 나중에 정리하겠다고 다짐해서는 안된다.__  
__나중은 결코 오지 않는다.__  
<br>

## 나쁜 코드로 치르는 대가 <a name = "3"></a>
나쁜 코드는 개발 속도를 크게 떨어뜨린다.  
코드를 고칠 때마다 엉뚱한 곳에서 얽혀있어 문제가 생긴다.  
매번 얽히고설킨 코드를 해독해서 얽히고 설킨 코드를 더하면 쓰레기 더미는 점점 높아지고 깊어진다.  
팀의 생산성은 마침내 0에 근접하고 관리층은 생산성을 증가시키기 위해 인력을 보충한다.  
하지만 새 인력은 시스템 설계에 대한 이해가 깊지 않아 설계 의도에 반하는 변경을 구분하지 못한다.  
결국 나쁜 코드를 더 양상하게 되고 생산성은 더더욱 떨어져 거의 0이 된다.  
<br>

## 원대한 재설계의 꿈 <a name = "4"></a>
나쁜 코드로 인한 팀의 반기가 생기고 새로운 타이거 팀이 구성된다.  
모두가 타이거 팀에 합류하고 싶어 하지만 똑똑한 사람들만 타이거 팀으로 차출되고 나머지는 계속 현재 시스템을 유지 보수한다.  
타이거 팀은 기존 시스템 기능을 모두 제공하는 새 시스템을 제공해야 할 뿐만 아니라 기존 시스템에 가해지는 변경도 모두 따라잡아야 한다.  
그래야 기존 시스템 기능을 100% 대체할 수 있으니까..
이 작업은 때때로 아주 오랫동안 이어지고, 새 시스템이 기존 시스템을 따라잡을 즈음이면 초창기 타이거 팀원들은 모두 팀을 떠났고 새로운 팀원들이 새 시스템을 설계하고자 나선다.  
왜? 현재 시스템이 너무 엉망이라서..  
__깨끗한 코드를 만드는 노력이 비용을 절감하는 방법일 뿐만 아니라 전문가로서 살아남는 길이다.__  
<br>

## 태도 <a name = "5"></a>
좋은 코드가 나쁜 코드로 전락하는 순간 우리는 온갖 이유를 들이댄다.  
요구사항이 변했다, 쓸모없는 마케팅 부서와 멍청한 관리자 탓이다 등등..  
하지만 그건 우리의 잘못이다.  
일정과 요구사항을 밀어붙이더라도 그들은 좋은 코드를 원한다.  
그것이 그들의 책임이기 때문이다.  
그와 마찬가지로, __좋은 코드를 지키는 것 또한 우리들의 책임이다.__  
<br>

## 원초적 난제 <a name = "6"></a>
누구나 나쁜 코드가 업무 속도를 늦춘다는 사실은 안다.  
하지만 기한을 맞추려면 나쁜 코드를 양산할 수밖에 없다고 느낀다.  
즉, 빨리 가기 위해서 좋은 코드를 위한 시간을 쓰지 않는다.  
하지만 진짜 전문가는 이 사실이 틀렸다는 것을 안다.  
나쁜 코드는 결과적으로 기한을 맞추지 못하게 된다.  
__기한을 맞추는 유일한 방법은 언제나 코드를 최대한 깨끗하게 유지하는 습관이다.__  
<br>

## 깨끗한 코드라는 예술 <a name = "7"></a>
좋은 코드를 작성하려면 '청결'이라는 힘겹게 습득한 감각을 활용해 자잘한 기법들을 적용하는 절제와 규율이 필요하며 그 열쇠는 '코드 감각'이다.  
좋은 코드와 나쁜 코드를 구분할 줄 안다고 깨끗한 코드를 작성할 줄 안다는 뜻이 아니다.  
좋은 코드와 나쁜 코드를 구분할 뿐만 아니라 나쁜 코드를 보면 좋은 코드로 개선할 방안을 떠올릴 수 있어야 한다.  

### Bjarne Stroustrup (C++ 창시자)
+ 논리가 간단해야 버그가 숨어들지 못한다.
+ 의존성을 최대한 줄여야 유지보수가 쉬워진다.
+ 성능을 최적으로 유지해야 사람들이 원칙 없는 최적화로 코드를 망치려는 유혹에 빠지지 않는다.
+ 깨끗한 코드는 한 가지를 제대로 한다.
+ 오류는 명백한 전략에 의거해 철저히 처리한다.

### Grady booch (Object Oriented Analysis and Design with Application 저자)
+ 깨끗한 코드는 단순하고 직접적이다.
+ 깨끗한 코드는 잘 쓴 문장처럼 읽힌다.
+ 깨끗한 코드는 결코 설계자의 의도를 숨기지 않아 명쾌한 추상화와 단순한 제어문으로 가득하다.

### Dave Thomas (OTI 창립자)
+ 깨끗한 코드는 작성자가 아닌 사람도 읽기 쉽고 고치기 쉽다.
+ 단위 테스트 케이스와 인수 테스트 케이스가 존재한다.
+ 깨끗한 코드에는 의미 있는 이름이 붙는다.
+ 특정 목적을 달성하는 방법은 하나만 제공한다.
+ 의존성은 최소이며 각 의존성을 명확히 정의한다.
+ API는 명확하며 최소로 줄인다.
+ 모든 정보를 코드만으로 명확히 표현할 수 없기에 코드는 문학적으로 표현해야 한다.

### Michael Feathers (Working Effectively with Legacy Code)
+ 깨끗한 코드는 언제나 누군가에게 주의 깊게 짰다는 느낌을 준다.
+ 깨끗하게 고치려고 시도해도 언제나 제자리로 돌아온다.

### Ron Jeffries (Extreme Programming Installed, Extreme Programming Adventure in C# 저자)
+ 모든 테스트를 통과한다.
+ 중복이 없다
+ 시스템 내 모든 설계 아이디어를 표현한다.
+ 클래스, 메서드, 함수 등을 최대한 줄인다.

### Ward Cunningham (위키 창시자, 피트 창시자, 익스트림 프로그래밍 공동 창시자)
+ 코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드이다.
+ 코드가 그 문제를 풀기 위한 언어처럼 보인다면 아름다운 코드이다.

<br>

## 우리는 저자다 <a name = "8"></a>
우리는 저자다.  
__저자에게는 독자와 소통할 책임이 있다.__  
다음부터 코드를 짤 때는 자신이 저자라는 사실을 기억하기 바란다.  
우리는 새 코드를 짜면서 __끊임없이__ 기존 코드를 읽는다.  
따라서 읽기 쉬운 코드를 짜야 새 코드를 짜기도 쉬워진다.  
급하다면, 서둘러 끝내려면, 쉽게 짜려면, __읽기 쉽게 만들면 된다.__  
<br>


## 보이스카우트 규칙 <a name = "9"></a>
> 캠프장은 처음 왔을 때보다 더 깨끗하게 해놓고 떠나라

__잘 짠 코드는 시간이 지나도 언제나 깨끗하게 유지해야 한다.__  
체크아웃할 때보다 좀 더 깨끗한 코드를 체크인한다면 코드는 절대 나빠지지 않는다.  
변수 이름을 개선하고, 긴 함수를 분할하고, 중복을 제거하고, 복잡한 if문을 정리하면 충분하다.  
이는 전문가라면 너무도 당연하지 않은가!  
<br>

## 결론 <a name = "10"></a>
이 책을 읽는다고 뛰어난 프로그래머가 된다, 코드 감각을 얻는다는 보장은 없다.  
단지 뛰어난 프로그래머가 생각하는 방식과 그들이 사용하는 기술과 기교와 도구를 소개할 뿐이다.  
책에서는 수 많은 예제를 제공하고 습득은 당신에게 달렸다.  
__"연습해, 연습"__  



