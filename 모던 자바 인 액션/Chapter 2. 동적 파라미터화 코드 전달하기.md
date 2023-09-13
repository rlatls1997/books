# Chapter 2. 동적 파라미터화 코드 전달하기
### 내용

- 변화하는 요구사항에 대응
- 동작 파라미터화
- 익명 클래스
- 람다 표현식 미리보기
- 실전 예제 : Comparator, Runnable, GUI

소비자의 요구사항은 항상 바뀐다.
어떤 조건으로 어떤 요청을 할 지, 계속해서 변화하는 요청에 따라 최소한의 비용으로 대응할 수 있도록 해야한다. 장기적인 관점에서 유지보수가 쉬워야 한다.

**동적 파라미터화**를 이용하여 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다. 동적 파라미터화란 아직 어떻게 실행할 것인지 결정되지 않은 코드 블록을 의미한다.

## 2.1 변화하는 요구사항에 대응하기

변화에 대응하는 유연한 코드를 구현하는 것이 좋다.
사과 농장을 예로 들어서 코드를 짜보자.

### 2.1.1 첫 번째 시도 : 녹색 사과 필터링

녹색 사과를 필터링하는 기능이 요구되었다

```java
enum Color { RED, GREEN }

public static List<Apple> fulterGreenApples(List<Apple> inventory){
	List<Apple> result = new ArrayList<>();

	for (Apple apple: inventory){
		if (GREEN.equals(apple.getColor()) {
			result.add(apple);
		}
	}
	
	return result;
}
```

위와 같이 구현할 수 있다.
그런데 만약 여기서 빨간 사과도 필터링하고 싶어졌다.

간단하게는 위 메서드의 Green을 Red로 바꾸고 if문의 조건만 RED로 변경한 메서드를 추가할 수 있다.

위 방법은 코드량도 많지 않고 그냥 복붙을 하면 되지만 만약 다른 수많은 색상에 대한 필터링 요구가 생기게 되면 대응하기가 어려워진다.

> 비슷한 코드가 반복 존재한다면 그 코드를 추상화하자.
>

### 2.1.2 두 번째 시도 : 색을 파라미터화

filterGreenApples 코드를 반복사용하지 않고 filterRedApples를 구현하는 것은 메서드에 파라미터를 추가하여 해결할 수 있다. 색을 파라미터화하여 변화하는 요구사항에 더 유연하게 대응할 수 있다.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color){
	List<Apple> result = new ArrayList<>();

	for (Apple apple: inventory){
		if (apple.getColor().equals(color)) {
			result.add(apple);
		}
	}
	
	return result;
}
```

```java
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
...
```

위와 같이 하나의 메서드에서 다양한 색의 사과를 필터링 할 수 있다.

근데 사과를 색이 아닌 무게로 필터링하는 요청이 생길 수 있다.

그렇다면 아래와 같이 무게를 조건으로 갖는 메서드도 만들 수있다.

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight){
	List<Apple> result = new ArrayList<>();

	for (Apple apple: inventory){
		if (apple.getWeight() > weight) {
			result.add(apple);
		}
	}
	
	return result;
}
```

위 방식은 좋은 해결책일 수 있지만 색으로 필터링하는 메서드와 대부분의 코드가 중복된다.
이는 DRY(같은 것을 반복하지 말 것)원칙을 어기는 것이다.

색과 무게를 하나의 필터링 메서드로 합치는 생각을 할 수 있지만 어떤 기준으로 필터링 할 지 구분을 해야 한다. 하나로 합친다면 색과 무게 중 어떤 기준으로 필터링할지 가리키는 플래그를 추가할 수 있다.

### 2.1.3 세 번째 시도 : 가능한 모든 속성으로 필터링

모든 속성을 합친 메서드를 만들어보자

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color,
																					int weight, boolean flag){
	List<Apple> result = new ArrayList<>();

	for (Apple apple: inventory){
		if (flag && apple.getColor().equals(color)) ||
					(!flag && apple.getWeight() > weight)) {
			result.add(apple);
		}
	}
	
	return result;
}
```

```java
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> redApples = filterApples(inventory, null, 150, false);
...
```

위 코드가 나쁜 이유

- flag의 의미를 알기 어렵다.
- 앞으로 요구사항이 바뀌었을 때 유연하게 대응할 수도 없다.
  ⇒ 사과 크기, 모양 등으로 필터링하고싶어진다면 모든 필터링을 할 수 있는 거대한 필터 메서드가 필요해짐.

> ⇒ 이 방법은 절대 실무에서 사용하면 안된다.
>

사과를 어떤 기준으로 필터링할 것인지를 전달할 수 있으면 더 좋을 것이다.

## 2.2 동작 파라미터화

파라미터를 추가하여 변화하는 요구에 대응하는 것은 한계가 있다.

사과의 어떤 속성에 기초해서 불리언 값을 반환하는 방법이 있다. (?)
참 또는 거짓을 반환하는 함수를 프레디케이트라고 한다.

선택 조건을 결정하는 인터페이스를 정의하자

```java
public interface ApplePredicate {
	boolean test (Apple apple);
}
```

다음 예제처럼 다양한 조건을 대표하는 여러 버전의 프레디케이트를 정의할 수 있다.

```java
public class AppleHeavyWeightPredicate implements ApplePredicate {
	public booleantest(Apple apple){
		return apple.getWeight() > 150;
	}
}

public class AppleHeavyWeightPredicate implements ApplePredicate {
	public booleantest(Apple apple){
		return GREEN.equals(apple.getColor());
	}
}
```

ApplePredicate를 구현하는 클래스에 따라 filter메서드가 다르게 동작할 것이다.

이를 **전략 디자인 패턴**이라고 부흔다.

전략 디자인 패턴 : 각 알고리즘(전략)을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법이다. 여기서 알고리즘 패밀리는 ApllePredicate, 전략은 위의 두 클래스이다.

근데 하나의 메서드에서 어떻게 다양한 동작을 수행하게 하는지?
⇒ ApplePredicate 객체를 받아 애플의 조건을 검사하도록 메서드를 고쳐야 한다.
**동작 파라미터화** 한다. 메서드가 동작을 받아서 내부적으로 다양한 동작을 수행할 수 있도록한다.

### 2.2.1 네 번째 시도 : 추상적 조건으로 필터링

ApplePredicate객체를 인수로 받도록 고치자.
이 방식으로 filterApples 메서드 내에서 컬렉션을 반복하는 로직과 컬렉션의 각 요소에서 적용할 동작을 분리할 수 있다는 큰 이점을 얻을 수 있다.

```java
public static List<Apple> filterApples(List<Apple> inventory, Applepredicate p){
	List<Apple> reuslt = new ArrayList<>();
	
	for(Apple apple: inventory){
		if(p.test(apple)){
			result.add(apple);
		}
	}
	
	return result;
}
```

**코드/동작 전달하기**

위와 같이 구현하므로써 유연한 코드를 얻는 동시에 가독성도 높아지고 사용하기도 쉬워졌다.
ApplePredicate를 구현하는 다양한 메서드를 필요한대로 만들어서 동작을 전달할 수 있다.

위와 같이 가장 중요한 동작은 ApplePredicate를 구현하여 작성할 수 있는 test라는 메서드이다. 하지만 메서드를 인수로 전달할 수 있기 때문에 위와 같이 ApplePredicate객체로 test메서드를 감싸서 전달해야 한다. test메서드를 구현하는 객체를 이용해서 불리언 표현식 등을 전달할 수 있으므로 이는 코드를 전달할 수 있는 것과 같다.

> 동적 파라미터화를 통해 한 개의 파라미터로 다양한 동작을 수행할 수 있다. 유연한 API를 만들 때 동작 파라미터화가 중요하다.
>

## 2.3 복잡한 과정 간소화

위의 동적 파라미터화 예시는 항상 프레디케이트를 구현하는 여러 클래스를 정의한 다음 인스턴스화해야한다. 이는 굉장히 번거로운 작업이고 시간낭비이다.

이런 문제를 해결하기 위해 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 **익명 클래스**를 사용할 수있다.

### 2.3.1 익명 클래스

익명 클래스 : 자바의 지역 클래스와 비슷한 개념. 이름이 없는 클래스이다. 익명클래스를 사용하여 클래스 선언과 인스턴스화를 동시에 할 수 있다. 즉석에서 필요한 구현을 바로 만들어서 사용할 수 있다.

### 2.3.2 다섯 번째 시도 : 익명 클래스 사용

아래와 같이익명 클래스를 이용하여 ApplePredicate를 구현하고 메서드를 재정의할 수 있다.

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate(){
	public boolean test(Apple apple){
		return RED>equals(apple.getColor());
	}
});
```

하지만 익명클래스도 부족한 부분이 많다.

- 코드 공간을 많이 차지하여 지저분하다.
- 사용에 익숙하지 않다.

코드의 장황함은 구현하고 유지보수하는 데 시간이 오래 걸리며 읽기 힘들다.
익명클래스를 사용한 방식에는 이 단점들이 아직 남아있다.

### 2.3.3 여섯 번째 시도 : 람다 표현식 사용

람다 표현식을 사용하여 위 예제를 재구현할 수 있다.

```java
List<Apple> result = filterApple(inventory, (Apple apple) ->RED.equals(apple.getColor()));
```

코드가 간단해지고 문제를 더 잘 설명한다.

### 2.3.4 일곱 번째 시도 : 리스트 형식으로 추상화

사과 뿐 아니라 여거가리 객체에 대한 필터링이 필요 할 수 있다.

```java
public interface Predicate<T> {
	boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
	List<T> result = new ArrayList<>();

	for(T e : list) {
		if(p.test(e)){
			result.add(e);
		}
	}
	
	return result;
}
```

```java
List<Apple> redApple = filter(inventory, (Apple apple) -> RED.quals(apple.getColor()));

List<Integer> evenNumbers = filter(numbers, (Integer i)-> i % 2 == 0);
```

동적 파라미터화, 람다, 추상화를 통해 유연성과 간결함을 얻을 수 있다.

## 2.4 실전 예제

### 2.4.1 Comparator로 정렬하기

java.util.Comparator 객체로 sort의 동작을 파라미터화 할 수 있다.

```java
//java.util.Comparator
public interface Comparator<T> {
	int compare(T o1, T o2);
}
```

Comparator를 구현해서 sort메서드의 동작을 다양화할 수 있다.

```java
inventory.sort(new Comparator<Apple>(){
	public int compare(Apple a1, Apple a2){
		return a1.getWeight().compareTo(a2.getWeight());
	}
});
```

람다를 사용하면

```java
inventory.sort((Apple a1, Appl2 a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

### 2.4.2 Runnable로 코드 블록 실행하기

자바 스레드를 이용하면 병렬로 코드 블록을 실행할 수 있다.
어떤 코드를 실행할 것인지 스레드에게 알려줄 수 있나?

여러 스레드가 각자 다른 코드를 실행할 수 있다. 나중에 실행할 수 있는 코드를 구현할 방법이 필요하다. 자바 8까지는 Thread생성자에 객체만을 전달할 수 있었으므로 결과를 반환하지 않는 void run 메소드를 포함하는 익명 클래스가 Runnable 인터페이스를 구현하도록 하는 것이 일반적

자바에서는 Runnable 인터페이스를 이용해서 실행할 코드 블록을 지정할 수 있다.

```java
// java.lang.Runnable
public interface Runnable{
	void run();
}
```

Runnable을 이용해서 다양한 동작을 스레드로 실행할 수 있다.

```java
Thread t = new Thread(new Runnable() {
	public void run()(
		Syste.out.println("hello world");
	}
});
```

자바8부터 람다표현식 가능

```java
Thread t = new Thread(()->System.out.println("hello world"));
```

### 2.4.3 Callable을 결과로 반환하기

자바 5부터 지원하는 ExecutorService

ExecutorService 인터페이스는 태스크 제출과 실행 과정의 연관성을 끊어준다. ExecutorService를 이용하면 태스크를 스레드 풀로 보내고 결과를 Future로 저장할 수 있다는 점이 Thread와 Runnable을 이용하는 방식과는 다르다.

Callbable 인터페이스를 이용해 결과를 반환하는 태스크를 만들기. → Runnable의 업그레이드 버전

```java
// java.util.concurrent.Callable
public interface Callable<V> {
	V call();
}
```

실행 서비스에 태스크를 제출해서 위 코드를 할용할 수 있음.

아래는 테스크를 실행하는 스레드의 이름을 반환

```java
ExecutorService executorService = Executors.newCachedThreadPool();
future<String> threadName = executorService.submit(new Callable<String>(){
	@Override
		public String call() throws Exception{
			return Thread.urrentThread().getName();
		}
	}
});
```

람다 사용

```java
Future<String> threadName = executorService.submit(()->Thread.currentThread().getName());
```

### 2.4.4 GUI 이벤트 처리하기

gui 프로그래밍은 이벤트에 대응하는 동작을 수행하는 식으로 동작하기 때문에 동적 파라미터를 전달하여 가능한 많은 동작을 커버할 수 있는 유연한 코드가 필요하다.

## 2.5 마치며

- 동작 파라미터화
    - 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다.
    - 요구에 더 잘 대응할 수 있는 코드를 구현할 수 있고 엔지니어링 비용을 줄일 수 있다.
- 람다
    - 인터페이스 구현 및 인스턴스화, 익명클래스 등을 사용하여 동작 파라미터를 전달하는 지저분한 구현 대신 람다를 사용하여 코드를 간결하고 이해하기 쉽게 만들 수 있다.

## 궁금

어떻게 구현할 메서드를 정의하지 않았는데도 람다에서 찾아가는건지?

⇒ 메서드가 하나인 인터페이스만 사용 가능

만약 구현할 메서드가 두개라면?

⇒ 람다 구현부분에서 에러 발생
⇒ 명시적으로 람다에 사용할 것을 알리는 FunctionalInterface 애너테이션 사용

java.lang.FunctionalInterface 애너테이션 사용 가능

```java
@FunctionalInterface
public Interface PrintApple{
	public String print(Apple apple);
}

// 만약 메서드를 2개 정의하면 컴파일 에러가 발생
@FunctionalInterface
public Interface PrintApple{
	public String print(Apple apple);
	public String printElse(T t);
}
```

https://catch-me-java.tistory.com/30