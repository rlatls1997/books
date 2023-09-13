# Chapter 9. 리팩터링, 테스팅, 디버깅
**내용**

- 람다 표현식으로 코드 리팩터링하기
- 람다 표현식이 객체지향 설계 패턴에 미치는 영향
- 람다 표현식 테스팅
- 람다 표현식과 스트림 API 사용 코드 디버깅

람다 표현식을 이용하여 가독성과 유연성을 높일 수 있다.

또한 람다 표현식으로 전략, 템플릿 메서드, 옵저버, 의무 체인, 팩토리 등의 객체 지향 디자인 패턴을 간소화할 수 있다.

람다 표현식과 스트림 API를 사용하는 코드를 테스트하고 디버깅하는 방법도 설명

## 9.1 가독성과 유연성을 개선하는 리팩터링

람다, 메서드 참조, 스트림 등의 기능을 이용하여 더 가독성이 좋고 유연한 코드로 리팩터링 할 수 있다.

### 9.1.1 코드 가독성 개선

코드 가독성 개선 : 코드를 다른 사람도 쉽게 이해하고 유지보수할 수 있도록 만드는 것을 의미.

### 9.1.2 익명 클래스를 람다 표현식으로 리팩터링하기

하나의 추상 메서드를 구현하는 익명 클래스는 람다 표현식으로 리팩터링할 수 있다.

ex)

```java
Runnable r1 = new Runnable() {
	public void run() {
		System.out.println("Hello");
	}
};

// 리팩터링
Runnable r2 = () -> System.out.println("Hello");
```

모든 익명클래스를 위와 같이 람다 표현식으로 변활할 수 있는 것은 아니다.

1. 익명 클래스에서 사용한 this와 super는 람다 표현식에서 다른 의미를 갖는다. 익명 클래스에서 this는 익명 클래스 자신을 가리키지만 람다에서 this는 람다를 감싸는 클래스를 가리킨다.
2. 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다(섀도 변수, shadow variable). 하지만 람다 표현식으로는 변수를 가릴 수 없다.

ex)

```java
int a = 10; // 상위 클래스의 변수

Runnable r1 = () ->{
	int a = 2; // 컴파일 에러
	System.out.println(a);
};

Runnable r2 = new Runnable(){
	public void run(){
		int a = 2; // 정상 작동
		System.out.println(a);
	}
};
```

1. 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다. 익명 클래스는 인스턴스화할 때 명시적으로 형식이 정해지는 반면 람다의 형식은 콘텍스트에 따라 달라지기 때문이다.

ex)

```java
interface Task{
	public void execute();
}
public static void doSomething(Runnable r){ r.run(); }
public static void doSomething(Task a){ a.execute(); }

doSomething(new Task(){
	public void execute(){
		sout("Danger");
	}
});
```

위와 같이 Task를 구현하는 익명 클래스를 전달할 수 있다.

위 익명 클래스를 아래와 같이 람다 표현식으로 바꿀 수 있다.

```java
doSomthing(() -> sout("Danger"));
```

위의 경우 람다표현식으 Runnable과 Task 모두의 대상 형식이 될 수 있으므로 문제가 발생한다.

아래와 같이 명시적 형변환을 이용하여 모호함을 제거할 수 있다.

```java
doSomthing((Task)() -> sout("Danger"));
```

### 9.1.3 람다 표현식을 메서드 참조로 리팩터링하기

람다 표현식 대신 메서드 참조를 이용하면 가독성을 높일 수 있다.

메서드 참조의 메서드명으로 코드의 의도를 명확하게 알릴 수 있기 때문이다.

### 9.1.4 명령형 데이터 처리를 스트림으로 리팩터링하기

반복자를 이용한 컬렉션 처리 코드를 스트림 API로 바꿔서 리팩토링할 수 있다.

스트림 API는 데이터처리 파이프라인의 의도를 명확하게 보여준다.

또한 쇼트서킷과 게으름이라는 최적화뿐 아니라 멀티코어 아키텍쳐를 활용할 수 있는 쉬운 방법도 제공한다.

스트림 API의 parallelStream을 사용하여 코드를 쉽게 병렬화할 수 있다.

### 9.1.5 코드 유연성 개선

람다표현식을 사용하면 동작 파라미터화를 쉽게 구현할 수 있다.

다양한 람다를 전달하여 다양한 동작을 표현할 수 있고 변화하는 요구사항에 대응할 수 있는 코드를 구현할 수 있다.

**함수형 인터페이스 적용**

람다 표현식을 이용하기 위해 함수형 인터페이스를 코드에 추가해야한다.

조건부 연기 실행과 실행 어라운드 패턴으로 람다 표현식 리팩터링을 할 수 있다. (자세한건 8.2절..)

**조건부 연기 실행**

선행하는 특정 조건 이후 어떤 코드를 실행할 수 있을 때 해당하는 특정 조건에서만 코드가 실행될 수 있도록 실행 과정을 연기할 수 있다.

ex) Logger의 log 메서드

```java
// log 메서드 시그니처. Supplier를 인수로 받는다.
public void log(Level level, Supplier<String> msgSupplier)

// 다음처럼 log 메서드를 호출할 수 있다.
// logger level이 다음과 같을 때 인수로 전달된 람다를 내부적으로 실행한다.
logger.log(Level.FINER, () -> "test");

// log메서드의 내부 구현 코드. 조건문에서 level을 검사하고 전달된 람다를 실행한다.
public void log(Level level, Supplier<String> msgSupplier){
	if(logger.isLoggable(level)){
		log(level,msgSupplier.get());
	}
}
```

이 기법으로 매번 객체의 상태를 체크하는 과정을 없앨 수 있다.

객체의 상태를 캡슐화할 수 있다는 장점이 있다.

**실행 어라운드**

매번 같은 준비, 종료 과정을 반복적으로 수행하는 코드를 람다로 변환할 수 있다.

ex)

```java
String oneLine = processFile((BufferedReader b) -> b.readLine());
String twoLine = processFile((BufferedReader b) -> b.readLine() + b.readLine());

public static String processFile(BufferedReaderProcessor p) throws IOException{
	try(BufferedReader br = new BufferedReader(new FileReader("/c/data.txt"))){
		return p.process(br); //같은 과정 내에서 람다만 인수로 분리하여 전달
	}
}

public interface BufferedReaderProcessor{
	String process(BufferedReader b) throws IOException;
}
```

반복되는 코드는 하나로 묶고 다르게 동작하는 부분은 따로 분리하여 람다로 개별적인 동작을 결정할 수 있다.

## 9.2 람다로 객체지향 디자인 패턴 리팩터링하기

디자인 패턴 : 다양한 패턴을 유형별로 정리한 것. 공통적인 소프트웨어 문재를 해결할 때 재사용할 수 있는 검증된 청사진을 제공.

디자인패턴에 람다 표현식을 더해서 문제를 더 쉽고 간단하게 해결할 수 있다.

다음 5가지의 디자인 패턴을 학습.

- 전략
- 템플릿 메서드
- 옵저버
- 의무 체인
- 팩토리

### 9.2.1 전략

전략 패턴 : 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법.

ex) 사과를 색깔로 필터링할 것인지,무게로 필터링할 것인지 선택했던 예

**전략 디자인 패턴 구성요소**

- 알고리즘을 나타내는 인터페이스(Strategy 인터페이스)
- 다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현(StrategyA, StrategyB …)
- 전략 객체를 사용하는 한 개 이상의 클라이언트

전략을 멤버로 갖는 객체의 생성자에 전략 객체를 전달하여 원하는 전략을 정할 수 있다.

**람다 표현식**

람다 표현식을 사용하면 전략 각각의 인터페이스를 구현할 필요 없이 익명 함수로써 전략을 전달할 수 있다.

ex)

```java
Validator numericValidator = new Validator((String s) -> s.matches("[a-z]+"));
Validator lowerCaseValidator = new Validator((String s) -> s.matches("\\d+"));
...

boolean b1 = numericValidator.validate("aaaa");
boolean b2 = lowerCaseValidator.validate("aaaa");
...
```

### 9.2.2 템플릿 메서드

알고리즘의 개요를 제시한 다음에 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야 할 때 템플릿 메서드 디자인 패턴 사용.

알고리즘을 사용하고싶으나 약간의 수정이 필요할 때 사용.

ex) 온라인 뱅킹 애플리케이션의 동작을 정의하는 추상메서드.

```java
abstract class OnlineBanking {
	public void processCustomer(int id){
		Customer c = Database.getCustomerWithId(id);
		makeCustomerHappy(c)
	}
	
	abstract void makeCustomerHappy(Customer c);
}
```

은행의 각 지점은 OnlineBanking 클래스를 상속받아 makeCustomerHappy 메서드를 원하는 동작을 수행하도록 구현할 수 있다.

**람다 표현식 사용**

makeCustomerHappy메서드의 시그니처와 일치하도록 Consumer<Customer> 형식을 갖는 두 번째 인수를 processCustomer에 추가한다.

```java
public void processCustomer(int id, Consumer<Customer> makeCustomerHappy){
	Customer c = Database.getCustomerWithId(id);
	makeCustomerHappy.accpet(c);
}
```

이제 클래스를 상속받지 않고도 람다표현식을 전달해서 다양한 동작을 추가할 수 있다.

ex)

```java
new OnlineBankingLambda().processCustomer(1337, (Customer c) ->
	sout("hello" + c.getName));
```

### 9.2.3 옵저버 패턴

어떤 이벤트가 발생했을 때,

한 객체(주체라 불리는)가 다른 객체 리스트(옵저버라 불리는)에 자동으로 알림을 보내야 하는 상황에서 옵저버 디자인 패턴을 사용한다.

**옵저버 디자인 패턴 구성**

- 주체 : 옵저버에 알림을 전달한다.
- 옵저버 : 옵저버를 추가한 객체에 알림을 보낸다.
- 옵저버 구독자들? : 옵저버의 알림을 받는다.

ex) 옵저버 패턴 예시. 신문 매체들이 뉴스 트윗을 구독하며 트윗이 등록되면 알림을 받는다.

다양한 옵저버를 구룹화할 Observer 인터페이스가 필요하다.
nofify() 메서드는 새 트윗이 등록될 때 주제(Feed)가 호출할 수 있도록 제공된다.

```java
interface Observer {
	void notify(String tweet);
}
```

이제 트윗에 서로 다른 동작을 수행하는 여러 옵저버를 정의할 수 있다.

```java
class NYTimes implements Observer{
	public void notify(String tweet){
		...
	}
}

class Guardian implements Observer{
	public void notify(String tweet){
		...
	}
}

class LeMonde implements Observer{
	public void notify(String tweet){
		...
	}
}
...

```

주제, Subject를 구현한다.

```java
interface Subject {
	void registerObserver(Observer o);
	void notifyObservers(String tweet);
}
```

registerObserver메서드로 새로운 옵저버를 등록할 수 있다.

notifyObservers 메서드로 트윗의 옵저버에 이를 알린다.

```java
class Feed implements Subject {
	private final List<Observer> observers = new ArrayList<>();

	public void registerObserver(Observer o){
		this.observers.add(o);
	}
	
	public void notifyObservers(String tweet){
		observers.forEach(o -> o.notify(tweet));
	}
}
```

다음과 같이 사용할 수 있다.

```java
Feed f = new Feed();
f.registerObserver(new NYTimes());
f.registerObserver(new Guardian());
f.registerObserver(new LeMonde());

f.notifyObservers("hello world");
```

**람다 표현식 사용**

Observer는 함수형 인터페이스이고 따라서 Observer 인터페이스를 구현하는 클래스를 전달하는 대신 notify()라는 함수 시그니처에 맞는 람다를 전달할 수 있다.

따라서 세 개의 옵저버를 인스턴스화하지 않고 람다 표현식을 직접 전달하여 실행할 동작을 지정할 수 있다.

ex)

```java
f.registerObserver((String tweet) ->{
	sout("New Tweet : " + tweet);
});
```

항상 람다표현식이 좋은건 아님.

옵저버가 상태를 가지며 여러 메서드를 정의하는 등의 경우라면 클래스를 구현하는 것이 더 바람직

### 9.2.4 의무 체인

작업 처리 객체의 체인(동작 체인 등)을 만들 때는 의무 체인 패턴을 사용한다.

한 객체가 어떤 작업을 처리한 다음에 다른 객체로 결과를 전달하고,
그 객체도 작업을 처리한 다음 또 다른 객체로 전달하는 방식의 패턴

일반적으로 다음으로 처리할 객체 정보를 유지하는 필드를 포함하는 작업 처리 추상 클래스로 의무 체인 패턴을 구성한다.

ex)

```java
public abstract class ProcessingObject<T>{
	protected ProcessingObject<T> successor;
	
	public void setSuccessor(ProcessingObject<T> successor){
		this.successor = successor;
	}
	
	public T handle(T input){
		T r = handleWork(input);
		if(successor != null){
			return successor.handle(r);
		}
		return r;
	}
	
	abstract protected T handleWork(T input);
```

- handle 메서드는 일부 작업을 어떻게 처리해야 할지 전체적으로 기술한다.
- handleWork메서드를 구현하여 다양한 종류의 작업 처리 객체를 만들 수 있다.

**람다 표현식 사용**

작업 처리 객체를 Function<String, String>,
정확히는 UnaryOperator<String> 형식의 인스턴스로 표현할 수 있다.

andThen 메서드로 함수를 조합해서 체인을 만들 수 있다.

ex)

```java
// 작업 처리 객체 1
UnaryOperator<STring> process1 = (String text) -> "Mario : " + text;
// 작업 처리 객체 2
UnaryOperator<STring> process2 = (String text) -> text.replaceAll("labda", "lambda");

// 동작 체인으로 두 함수를 조합
Function<String, String> pipeline = 
	process1.andThen(process2);

String result = pipeline.apply("Aren't labdas really sexy?");

```

### 9.2.5 팩토리

인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 팩토리 디자인 패턴을 사용.

생성자와 설정을 외부로 노출하지 않음으로써 클라이언트가 단순하게 객체를 생성할 수 있다.

ex)

```java
public class ProductFactory{
	public static Product createProduct(String name){
		switch(name){
			case "loan" : return new Loan();
			case "stock" : return new Stock();
			default : throw new RuntimeException();
		}
	}
}

Product p = ProductFactory.createProduct("load");
```

**람다 표현식 사용**

생성자도 메서드 참조처럼 접근할 수 있다.

Loan생성자를 사용하는 코드는 다음과 같다.

```java
Supplier<Product> loanSupplier = Loan::new;
Loan loan = loanSupplier.get();
```

다음과 같이 상품명을 생성자로 연결하는 Map을 만들 수 있다.

```java
final static Map<String, Supplier<Product>> map = new HashMap<>();
static {
 map.put("loan", Loan::new);
 map.put("stock", Stock::new);
}	

// 위의 map을 사용한 팩토리메서드
public static Product createProduct<String name){
	Supplier<Product> p = map.get(name);
	if(p != null) return p.get();
	throw new IllegalArgumentException();
}
```

만약 생성자에 인수가 필요한 경우 그에 맞는 함수형 인터페이스의 사용이 필요하다.

ex) 세 인수를 받는 생성자의 경우 map

```java
// TriFunction 함수형 인터페이스
public interface TriFunction<T, U, V, R>{
	R apply(T t, U u, V v);
}

Map<String TriFunction<Integer, Integer, String, Product>> map = new HashMap<>();
```

## 9.3 람다 테스팅

제대로 동작하는 코드를 구현하는 것이 최종 목표.
이를 위해 테스트가 필요하다.

### 9.3.1 보이는 람다 표현식의 동작 테스팅

람다는 익명 함수이므로 테스트 케이스 내부에서 호출하기가 불가능하다.

필요하다면 람다를 필드에 저장해서 재사용할 수 있으며 람다의 로직을 테스트할 수 있다.

ex)

```java
public class Point{
	public final static Comparator<Point> compareByXAndThenY = 
		comparing(Point::getX).thenComparing(Point::getY);
	...
}
```

람다 표현식은 함수형 인터페이스의 인스턴스를 생성한다.

따라서 생성된 인스턴스의 동작으로 람다 표현식을 테스트할 수 있다.

```java
@Test
public void textComparingTwoPoints() throws Exception {
	Point p1 = new Point(10, 15);
	Point p2 = new Point(10, 20);
	int result = Point.compareByXAndThenY.compare(p1, p2);
	assertTrue(result < 0);
}
	
```

### 9.3.2 람다를 사용하는 메서드의 동작에 집중하라.

람다의 목표는 정해진 동작을 다른 메서드에서 사용할 수 있도록 하나의 조각으로 캡슐화하는 것.

그러려면 세부 구현을 포함하는 람다 표현식을 공개하지 말아야 한다.

람다 표현식을 사용하는 메서드의 동작을 테스트함으로써 람다를 공개하지 않으면서도 람다 표현식을 검증할 수 있다.

⇒ 람다를 감싸고 있는 메서드를 테스트하여 람다 표현식의 동작도 검증할 수 있다.

### 9.3.3 복잡한 람다를 개별 메서드로 분할하기

테스트코드에선 람다 표현식을 참조할 수 없다.
그러면 복잡한 람다 표현식은 어떻게 테스트할 것인지?

⇒ 람다를 새로운 일반 메서드로 선언하여 분할한다.

### 9.3.4 고차원 함수 테스팅

고차원함수 : 함수를 인수로 받거나 다른 함수를 반환하는 메서드

메서드가 람다를 인수로 받는다면 다른 람다로 메서드의 동작을 테스트할 수 있다.

ex) 다양한 프레디케이트로 filter메서드 테스트

```java
@Test
public void testFilter() throws Exception{
	...
	List<Integer> even = filter(numbers, i -> i % 2 == 0);
	List<Integer> smallerThanThree = filter(numbers, i -> i < 3);

	assertEquals(Arrays.asList(2,4), even);
	assertEquals(Arrays.asList(1,2), smallerThanThree);
}
```

테스트해야할 메서드가 다른 함수를 반환한다면?
⇒ 함수형 인터페이스의 인스턴스로 간주하고 함수의 동작을 테스트할 수 있다.

## 9.4 디버깅

코드를 디버깅할 때 다음 두 요소를 확인

- 스택 트레이스
- 로깅

하지만 람다 표현식과 스트림은 기존의 디버깅 기법을 무력화한다.
람다 표현식과 스트림 디버깅 방법을 살펴보자.

### 9.4.1 스택 트레이스 확인

메서드를 호출할 때 호출 위치, 인수값, 메서드의 지역변수등을 포함한 호출 정보가 생성되며 스택 프레임에 저장된다.

스택트레이스를 통해 프로그램이 어떻게 멈추었는지에 대한 정보를 얻을 수 있다.

**람다와 스택 트레이스**

람다표현식은 이름이 업기 때문에 복잡한 스택 트레이스가 생성된다.

ex) 람다에서 에러가 발생할 경우

```java
public class Debugging{
	public static void main(String [] args){
		List<Point> points = Arrays.asList(new Point(12, 2), null);
		points.stream().map(p->p.getY()).forEach(System.out::println());
	}
}

...

Exception in thread "main" java.lang.NullPointerException
	at Debugging.lambda$main$0(Debugging.java:6) // $0의 의미는?
	at Debugging$$Lambda$5/28472968.apply(Unknown Source)
	...
```

람다 표현식은 이름이 없으므로 컴파일러가 람다를 참조하는 이름을 만들어낸 것이다.

메서드 참조를 사용해도 스택 트레이스에는 메서드 명이 나타나지 않는다.

메스드 참조를 사용할 때

메서드 참조를 사용하는 클래스와 같은 곳에 선언되어 있는 메서드를 참조할 때는 메서드 참조 이름이 스택 트레이스에 나타난다.

ex) 같은 클래스에 존재하는 메서드를 메서드참조로 사용하는 경우

```java
public class Debugging{
	public static void main(String[] args){
		List<Integer> numbers = Arrays.asList(1,2,3);
		
		numbers.stream().map(Debugging::divideByZero).forEach(System.out::println);
	}
	
	public static int divideByZero(int n){
		return n/0;
	}
}
```

이 경우에는 스택트레이스에 divideByZero라는 메서드 이름이 제대로 표시된다.

### 9.4.2 정보 로깅

스트림의 파이프라인 연산을 디버깅한다고 가정할 경우 다음처럼 forEach로 스트림 결과를 출력하거나 로깅할 수 있다.

```java
numbers.steram()
	.map(x -> x + 17)
	.filter(x -> x % 2 == 0)
	.limit(3)
	.forEach(System.out::println);
```

위와 같이 로깅을 하면 forEach를 호출하는 순간 전체 스트림이 소비된다.

map, filter, limit 등 각각의 중간 연산이 어떤 결과를 호출하는지 확인하고 싶다면?
⇒ peek이라는 스트림 연산을 사용한다.

peek은 스트림의 각 요소를 소비한 것처럼 동작을 실행하지만 forEach와 같은 최종연산처럼 스트림의 요소를 소비하지는 않는다. peek은 자신이 확인한 요소를 파이프라인의 다음 연산에 그대로 전달한다.

ex) peek연산 사용 예

```java
numbers.steram()
	.peek(x -> sout("from stream: " + x));
	.map(x -> x + 17)
	.peek(x -> sout("after map: " + x));
	.filter(x -> x % 2 == 0)
	.peek(x -> sout("after filter: " + x));
	.limit(3)
	.peek(x -> sout("after limit: " + x));
	.collect(toList());
```

위와 같이 peek 연산을 사용하여 파이프라인의 각 단계별 스트림 요소 상태를 출력할 수 있다.

## 9.5 마치며

- 람다 표현식으로 유연한 코드를 만들 수 있다.
- 메서드 참조로 람다표현식보다 더 가독성 좋은 코드를 구현할 수 있다.
- 람다 표현식으로 전략, 템플릿메서드, 옵저버, 의무 체인, 팩토리 등의 객체지향 디자인 패턴에서 발생하는 불필요한 코드를 제거할 수 있다.
- 람다 표현식 자체의 테스트보단 람다 표현식을 사용하는 메서드의 동작을 테스트하는 것이 바람직.
- 람다 표현식 사용 시 스택트레이스 이해가 어려워진다.
- peek연산을 사용하여 중간연산의 결과를 디버깅할 수 있다.
- 