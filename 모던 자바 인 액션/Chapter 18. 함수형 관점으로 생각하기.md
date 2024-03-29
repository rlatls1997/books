# Chapter 18. 함수형 관점으로 생각하기
**내용**

- 함수형 프로그래밍을 사용하는 이유
- 함수형 프로그래밍을 어떻게 정의하는가
- 선언형 프로그래밍과 참조 투명성
- 함수형 스타일의 자바 구현 가이드라인

## 18.1 시스템 구현과 유지보수

함수형 프로그래밍이 제공하는 **부작용 없음**, **불변성**이라는 개념이 유지보수 중 발생할 수 있는 코드 크래시 디버깅 문제를 해결하는데에 도움을 준다.

### 18.1.1 공유된 가변 데이터

변수가 예상하지 못한 값을 갖는 이유는 시스템에서 공유된 가변 데이터 구조를 읽고 갱신하기 때문.공유 가변 데이터 구조를 사용하면 프로그램 전체에서 데이터 갱신 사실을 추적하기가 어려워진다.

자신을 포함하는 클래스의 상태, 다른 객체의 상태를 바꾸지 않으며 return문을 통해서만 자신의 결과를 반환하는 메서드를 **순수 메서드라고 부른다.**

순수 메서드가 아닌 메서드에선 다음과 같은 부작용이 발생할 수 있다.

- 자료구조를 고치거나 필드에 값을 할당
- 예외 발생
- 파일에 쓰기 등의 I/O 동작 수행

이렇게 함수 내에 포함되지 못한 기능을 부작용이라 한다.

**불변객체**를 이용해서 이러한 부작용을 없앨 수 있다.
불변객체는 객체의 상태를 바꿀 수 없는 객체이므로 함수 동작에 영향을 받지 않기 때문에 공유가 가능하며 스레드 안전성을 제공한다.

부작용을 없애면 lock없이 멀티코어 병렬성을 사용할 수 있고, 프로그램의 어떤 부분이 독립적인지 바로 이해할 수 있다.

### 18.1.2 선언형 프로그래밍

주어진 문제를 **어떻게** 구현할지에 집중하는 프로그래밍 방식을 명령형 프로그래밍이라고 부른다.

아래와 같이 일련의 명령을 실행하는 방식

ex) 가장 비싼 트랜잭션 구하기

```java
// 트랜잭션 하나를 가져온다.
Transaction mostExpensive = transactions.get(0);

if(mostExpensive == null)
	throw new IllegalArgumentExceptuon("Empty list of transactions");

// 모든 트랜잭션을 비교하며 가장 비싼 트랜잭션을 갱신한다.
for(transaction t : transactions.subList(1, transactions.size())){
	if(t.getValue() > mostExpensive.getValue()){
		mostExpensive = t;
	}
}
```

**무엇을**에 집중하는 방식도 있다. 이러한 방식의 프로그래밍을 선언형 프로그래밍이라 부른다.

원하는 것이 무엇인지, 어떻게 목표를 달성할 것인지가 코드로 명확하게 나타난다는게 강점이다.

ex)

```java
Optional<Transaction> mostExpensive = transactions.stream()
																				.max(comparing(Transaction::getValue));
```

### 18.1.3 왜 함수형 프로그래밍인가?

함수형 프로그래밍은 선언형 프로그래밍을 따르는 대표적인 방식이며 부작용이 없는 계산을 지향한다.

선언형 프로그래밍을 따르고 부작용이 없다는 점은 시스템 구현과 유지보수를 더 쉽게 해준다.

함수형 프로그래밍의 사용으로 부작용이 없는 복잡하고 어려운 기능을 수행하는 프로그램을 구현할 수 있다.

## 18.2 함수형 프로그래밍이란 무엇인가?

함수형 프로그래밍에서의 함수란? **부작용이 없는** 함수(다른 객체의 변수 등 가변 변수를 갱신하는 함수)

인수가 같은 호출에 대해 항상 같은 결과를 반환하는 수학적 함수와 같다.

함수, if-then-else 등의 수학적 표현만 사용하는 방식을 순수 함수형 프로그래밍이라 하며

시스템의 다른 부분에 영향을 미치지 않는다면 내부적으로는 함수형이 아닌 기능도 사용하는 방식을 함수형 프로그래밍이라 함.

### 18.2.1 함수형 자바

Scanner.nextLine() 의 경우 호출마다 다른 결과가 반환될 가능성이 있기에 순수하지 않다.
하지만 시스템 컴포넌트가 순수한 함수형인 것처럼 동작하도록 코드를 구현할 수 있다.

실제 부작용이 있으나 아무도 보지 못하게 함으로써 **함수형**을 달성할 수 있다.

예를 들어 함수가 실행되면서 값을 변경하나 함수가 끝날 때 값을 원래대로 돌려놓는다면 단일스레드로 실행되는 프로그램의 경우 이 메서드는 아무 부작용도 일으키지 않으므로 함수형이라 간주할 수 있다.

하지만 다른 스레드에서 동시에 이 메서드를 호출하는 경우에는 함수형이 아니게 된다. 이 경우 메서드 바디에 lock을 걸어서 해결할 수 있으나 메서드의 병렬 호출이 불가능해져서 효율이 떨어지게 된다.

함수나 메서드는 지역 변수만을 변경해야 함수형이라고 할 수 있다.

만약 참조하는 객체가 있다면 그 객체는 불변객체여야한다.

메서드 내에서 생성한 객체가 필드 갱신이 외부에 노출되지 않고 다음 메서드 호출에 대해 영향을 끼치지 않는다면 변경이 가능하다.

또한 함수형이라면 **함수나 메서드가 어떤 예외도 일으키지 않아야 한다.**

예외가 발생하면 return으로 결과를 반환할 수 없게 될 수 있기 때문이다.(0으로 수를 나누거나 제곱근을 구해야하는 수가 음수일 경우 등)

이런 경우 Optional 클래스를 사용해서 예외사용을 없앨 수 있다.

그리고 함수형에서는 비함수형 동작을 감출 수 있는 상황에서만 부작용을 포함하는 라이브러리 함수를 사용해야한다. (먼저 자료구조를 복사하는 등 발생가능한 문제를 내부적으로 처리하여 호출자가 자료구조의 변경을 알 수 없게 할떄)

### 18.2.2 참조 투명성

참조 투명성 : 같은 인수로 함수를 호출했을 때 항상 같은 결과를 반환하는 함수를 참조적으로 투명한 함수라고 한다.

Random.nextInt()같은 경우는 매번 다른 값을 생성하므로 함수형이 될 수 없다.

참주투명성은 비용이 많으 드는 연산을 **기억화** 또는 **캐싱**을 통해서 연산을 반복하지 않고 결과값을 저장하는 최적화 기능도 제공한다.

함수가 List를 생성하여 반환하는 경우를 생성해보면

같은 인수를 사용하여 호출하고 List에 같은 요소를 포함하더라도 결과적으로 서로 다른 메모리 공간에 생성된 List를 참조하게 된다.

결과 List가 가변 객체라면 리스트를 반환하는 메서드는 참조적으로 투명한 메서드가 아니라는 결론이 나온다.

결과 List를 불변한 순수값으로 사용할 것이라면 두 리스트가 같다고 볼 수 있으므로 List 생성 함수를 참조적으로 투명한 것으로 간주할 수 있다. 함수형 코드에서는 이런 함수를 참조적으로 투명한 것으로 가눚한다.

### 18.2.3 객체지향 프로그래밍과 함수형 프로그래밍

익스트림 객체지향 : 모든 것을 객체로 간주하고 프로그램이 객체의 필드를 갱신하고, 메서드를 호출하고, 관련 객체를 갱신하는 방식으로 동작

함수형 프로그래밍 : 참조적 투명성을 중시하는, 변화를 허용하지 않는 동작

하드웨어의 변경(멀티코어 등), 프로그래머의 기대치(SQL과 비슷한 방식의 프로그래밍)등으로 프로그래밍 형식이 점차 함수형으로 다가갈 것임.

### 18.2.4 함수형 실전 연습

p..577 어떤 집합으로 만들 수 있는 모든 부분집합을 List<List<Integer>> 형태로 나타내는 예

결론은, 메서드를 작성할 때 인수를 변형하는 로직을 지양하고 순수함수로써 동작할 수 있도록 코드를 작성하는 것이 사이드이펙트를 줄일 수 있게 한다.

## 18.3 재귀와 반복

함수형 스타일에서는 호출자가 변화를 알아차리지 못한다면 괜찮기 때문에 지역변수는 자유롭게 갱신가능하고 while이나 for문의 조건을 변경할 수 있다.

```java
Iterator<Apple> it = apples.iterator();
while(it.hasNext()){
	Apple apple = it.next();
	...
}
```

위 코드는 호출자가 변화를 알 수 없으므로 문제가 없다.

```java
public void searchForGold(List<String> l, Stats stats){
	for(String s : l){
		if("gold".equals(s)){
			stats.incremenetFor("gold");
		}
	}
}
```

위 코드는 문제가 될 수 있다.

루프의 바디에서 다른 부분과 공유될 수 있는 인수인 stats의 상태를 변변화시킨다.

반복문은 재귀로 구현할 수 있고

재귀를 이용하면 변화가 일어나지 않는다.(?, 결국은 incrementFor메서드 호출이 필요할텐데 어떻게)

재귀를 이용하면 루프마다 갱신되는 반복 변수를 제거할 수 있다.

ex) 재귀 방식의 팩토리얼 계산

```java
static long factorialRecursive(long n){
	return n == 1 ? 1 : n * factorialRecursive(n-1);
}
```

그렇다고 무조건 재귀가 좋은건 아니다. 재귀는 일반적으로 반복코드보다 비싸다.

재귀함수를 호출할 때마다 호출 스택에 각 호출시 생성되는 정보를 저장할 새로운 스택 프레임이 만들어지기 때문에 메모리 사용량이 증가하고 StackOverflow 발생 가능성이 높아진다.

**꼬리 호출 최적화**

```java
static long factorialTailRecursive(long n){
	return factorialHelper(1, n);
}

static long factorialHelper(long acc, long n){
	return n == 1? acc : factorialHelper(acc * n, n-1);
}
```

factorialHelper에서 재귀호출이 가장 마지막에 이루어지므로 꼬리재귀라 부른다.

일반 재귀인 factorialRecursive메서드는 마지막으로 수행한 연산이 n과 재귀호출 결과값의 곱셈이다.

중간 결과를 각 스택 프레임으로 저장해야하는 일반 재귀와 달리

꼬리 재귀에서는 컴파일러가 하나의 스택프레임을 재활용할 가능성이 생긴다.

자바는 이와 같은 최적화를 제공하지 않는다.

하지만 여러 컴파일러 최적화 여지를 남겨둘 수 있는 꼬리 재귀를 적용하는 것이 좋다.

스칼라, 그루비 등 최신 JVM언어는 꼬리재귀를 반복으로 변환하는 최적화를 재공한다.

## 18.4 마치며

- 공유된 가변 자료구조를 줄이는 것은 프로그램을유지보수하고 디버깅하는데에 도움이 된다.
- 함수형 프로그래밍은 부작용없는 메서드와 선언형 프로그래밍 방식을 지향한다.
- 꼬리재귀를 사용하여 컴파일러 최적화를 기대할 수 있다.

### 궁금했던 부분

p.573 : 내부적으로는 함수형이 아닌 기능도 사용하는 방식이란게 random값 등의 사용을 말하는건가?