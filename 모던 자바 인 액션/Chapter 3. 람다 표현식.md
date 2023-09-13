# Chapter 3. 람다 표현식
익명 클래스로 다양한 동작을 구현할 수 있지만 만족할만큼 코드가 깔끔하지는 않다.
람다 표현식을 어떻게 만드는지, 어떻게 사용하는지, 어떻게 코드를 간결하게 만들 수 있는지설명한다.

## 1 람다란 무엇인가?

**람다 표현식**은 메서드로 전달할 수 있는 익명함수를 단순화한 것

람다의 특징

- 익명 : 보통의 메서드와 달리 이름이 없으므로 익명이라고 표현한다. 네이밍에 대한 고민을 하지 않아도 됨
- 함수 : 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 부른다. 하지만 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함한다.
- 전달 : 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.
- 간결성 : 익명클래스처럼 자질구레한 코드를 구현할 필요가 없다.

람다를 통해 간결하고 깔끔한 코드를 작성할 수 있다.

람다 표현식은 파라미터, 화살표, 바디로 이루어진다.

```java
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
|----람다 파라미터---|   |----------------람다 바디----------------|
								|--화살표--|
```

자바8에서 지원하는 5가지 람다 표현식 예시

```java
(String s) -> s.length()

(Apple a) -> a.getWeight() > 150

(int x, int y) -> {
	System.out.println("Result :");
	System.out.println(x + y);
}

() -> 42

(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())
```

## 2 어디에, 어떻게 람다를 사용할까

함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다.

### 1. 함수형 인터페이스

**함수형 인터페이스**는 정확히 하나의 추상 메서드를 지정하는 인터페이스이다. 자바 API의 함수형 인터페이스로 Comparator, Runnable 등이 있다.

ex)

```java
public interface Predicate<T> {
	boolean test(T t);
}
```

만약 인터페이스에 기본 구현을 제공하여 바디를 포함할 수 있는 메서드인 디폴트메서드가 많이 존재하더라도 추상 메서드가 오직 하나이면 함수형 인터페이스이다.

함수형 인터페이스로 뭘 할 수 있을까?

람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 **전체 표현식을 함수형 인터페이스의 인스턴스로 취급**기술적으로 따지면 함수형 인터페이스를 구현한 클래스의 인스턴스)할 수 있다.

### 2. 함수 디스크립터

함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가리킨다.
람다 표현식의 시그니처를 서술하는 메서드를 **함수 디스크립터**라고 부른다.

예를 들어서 Runnable 인터페이스의 유일한 추상 메서드 run은 인수와 반환값이 없으므로 Runnable 인터페이스는 인수와 반환값이 없는 시그니처로 생각할 수 있다.

람다 표현식은 변수에할당하거나 함수형 인터페이스를 인수로 받는 메서드로 전달할 수 있으며 함수형 인터페이스의 추상 메서드와 가은 시그니처를 갖는다.

예를 들어 process메서드에 직접 람다 표현식을 전달하는 경우

```java
public void process(Runnable r){
	r.run();
}

process(() -> system.out.println("tis is awesome!!"));
```

위 코드를 실행하면 “This is awesom!!” 이 출력된다.

() → System.out.println(”This is awesome!!”)은 인수가 없으며 void를 반환하는 람다 표현식이다.
이는 Runnable 인터페이스의 run 메서드 시그니처와 같다.

**람다와 메소드 호출**

```java
process(() -> system.out.println("tis is awesome!!"));
/* 위 코드는 정상적인 람다 표현식이다.
위 코드에서는 중괄호를 사용하지 않았고 System.out.println은 void를 반환하므로
완벽한 표현식이 아닌 것처럼 보인다. 그럼 코드를 중괄호로 감싸면?*/

process(()->{System.out.println("This is awesome"); });

/*중괄호는 필요가 없다. 자바 언어 명세에서는 void를 반환하는 메서드와
관련된 특별한 규칙을 정하고 있기 때문이다.
즉, 한 ㄱ의 void 메소드 호출은 중괄호로 감쌀 필요가 없다.*/
```

왜 함수형 인터페이스를 인수로 받는 메스데아만 람다 표현식을 사용할 수 있을까?

언어 설계자들이 자바에 함수 형식을 추가하는 방법도 고려했지만 더 복잡하게 만들기 않는 현재 방법을 선택했고 대부분의 자바 프로그래머가 하나의 추상 메서드를 갖는 인터페이스에 익숙하다는 점도 고려되었다.

@**FunctionalInterface 애너테이션**

해당 인터페이스가 함수형 인터페이스임을 가리키는 애너테이션이다. @FunctionalInterface로 인터페이스를 선언했지만 실제로 함수형 인터페이스가 아니라면 컴파일러가 에러를 발생시킨다.

## 3. 람다 활용 : 실행 어라운드 패턴

자원 처리(ex : 데이터베이스의 파일 처리)에 사용되는 순환 패턴은 자원을 열고 처리한 다음에 자원을 닫는 순서로 이루어진다. 설정과 정리 과정은 대부분 비슷하다. 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는다.

**실행 어라운드 패턴**이란 어떤 작업에 대하여 설정(setup) 과정과 정리(cleanup) 과정이 둘러싸고 있는 형태의 패턴을 말한다.

(자바 7에 추가된 try-with-resources 구문을 사용하면 자원을 명시적으로 닫을 필요가 없어서 코드가 간결해진다.)

```java
public String processFile() throws IOException {
	try(BufferedReader br = 
					new BufferedReader(new FileReader("data.txt"))){
			return br.readLine();
	}
}
```

### 1. 1단계 : 동작 파라미터화를 기억하라

위 코드는 파일에서 한 번에 한 줄만 읽을 수 있다.
만약 한 번에 두 줄을 읽거나 자주 사용되는 단어를 반환하려면 어떻게 해야 할까?

설정, 정리 과정은 재사용하고 processFile 메서드만 다른 동작을 수행하도록 명령할 수 있으면 된다.

processFile의 동작을 파라미터화하면 된다. processFile 메서드가 BufferedReader를 이용해서 다른 동작을 수행할 수 있도록 processFile메서드로 동작을 전달한다.

메서다가 한 번에 두 행을 읽는 동작을 전달하려면 BuffredReader를 인수로 받아서 String을 반환하는 람다가 필요하다.

```java
	String result = processFile(BuffredReader br) ->
												br.readLine() + br.readLine());
```

### 2. 2단계 : 함수형 인터페이스를 이용해서 동작 전달

함수형 인터페이스 자리에 람다를 사용할 수 있다. 따라서 BuffredReader → String과 IOException을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스를 만들어야 한다.

```java
@FunctionalInterface
public interface BuffredReaderProcessor {
	String process(BuffredReader b) throws IOException;
}
```

이제 정의한 인터페이스를 processFile메서드의 인수로 전달할 수 있다.

```java
public String processFile(buffredReaderProcessor p) throws IOException {
}
```

### 3. 3단계 : 동작 실행

이제 BufferedReaderProcessor에 정의된 process메서드의 시그니처 (BuffredReader → String)와 일치하는 람다를 전달할 수 있다. 람다 표현식으로 함수형 인터페이스의 추상메서드 구현을 직접 전달할 수 있으며 전달된 코드는 함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리한다.

따라서 processFile바디 내에서 BufferedReaderProcessor 객체의 process를 호출할 수 있다.

```java
public String processFile(BuffredReaderProcessor p) throws IOException {
	try (BuffredReader br = 
				new BuffredReader(new FileReader("data.txt"))){
			//BuffredReader객체를 전달한다.
			return p.process(br);
	}
}
```

### 4. 4단계 : 람다 전달

이제 람다를 이용해서 다양한 동작을 processFile 메서드로 전달할 수 있다.

```java
String oneLine = 
		processFile((BufferedReader br) -> br.readLine());

String twoLines = 
		processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

## 4. 함수형 인터페이스 사용

함수형 인터페이스에서는 오직 하나의 추상메서드를 지정한다. 함수형 인터페이스의 추상 메서드는 람다 표현식의 시그니처를 묘사한다. 함수형 인터페이스의 추상 메서드 시그니처를 **함수 디스크립터**라고 한다. 다양한 람다 표현식을 사용하려면 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요하다. 자바 API는 이미 Comparable, Runnable, Callable 등의 ㅎ마수형 인터페이스를 포함하고 있다.

java.util.function 패키지로 여러 가지 새로운 함수형 인터페이스를 제공한다.

### 1. Predicate

java.util.function.Predicate<T> 인터페이스는 test라는 추상 메서드를 정의하며 test는 제네릭 형식의 T의 객체를 인수로 받아 불리언을 반환한다. T형식의 객체를 사용하는 불리언 표현식이 필요한 경우라면 새로 인터페이스를 만들 필요 없이 Precidate 인터페이스를 사용할 수 있다

```java
@FunctionalInterface
public interface Predicate<T> {
	boolean test(T t);
}

public <T> List<T> filter(List<T> list, Predicate<T> p) {
	List<T> results = new ArrayList<>();
	for(T t : list){
		if(p.test(t)){
			results.add(t);
		}
	}
	return results
}
```

Predicate 인터페이스 명세를 보면 and나 or같은 메서드도 있음

### 2. Consumer

java.util.functionConsumer<T>인터페이스는 T객체를 받아서 void를 반환하는 accept라는 추상 메서드를 정의한다. T향식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 Consumer 인터페이스 사용한다.

ex)

```java
@FunctionalInterface
public interface Consumer<T> {
	void(accpet(T t);
}

public <T> void forEach(List<T> list, consumer<T> c){
	for(T t : list) {
		c.accept(t);
	}
}

forEach(
	Arrays.asList(1,2,3,4,5),
	// Consumer의 accept메서드를 구현하는 람다
	(Integer i) -> System.out.println(i);
}
```

### 3. Function

java.util.function.Function<T, R> 인터페이스는 제레닉 형식 T를 인수로 받아서 제네릭 형식 R객체를 반환하는 추상 메서드 apply를 정의한다. 입력을 출력으로 매핑하는람다를 정의할 때 Function 인터페이스를 활용한다.

```java
@FunctionalInterface
public interface Function<T, R>{
	R apply(T t);
}

public <T, R> List<R> map(List<T> list, Function<T, R> f){
	List<R> result = new ArrayList<>();
	for(T t : list){
		result.add(f.apply(t);
	}
	return result;
}

List<Integer> l = map(
	Arrays.asList("lambdas", "in", "action"),
	//Function의 apple메서드를 구현하는 람다
	(String s) -> s.length()
);
```

**기본형 특화**

앞의 세 가지 제네릭 함수형 인터페이스 말고도 특화된 형식의 함수형 인터페이스 있다.

위와 같이 제네릭 파라미터를 받는 경우에는 제네릭의 내부 구현 때문에 어쩔 수 없이 참조형 타입을 사용해야 한다.

- 박싱 **:** 기본형을 참조형으로 변환
- 언박싱 : 참조형을 기본형으로 변환
- 오토박싱 : 박싱과 언박싱을 자동으로

ex)

```java
List<Integer> list = new ArrayList<>();
for(int i = 300; i< 400; i++){
	list.add(i);
}
```

하지만 이런 과정에는 비용이소모된다.
박싱한 값은 기본형을 감싸는 래퍼이며 힙에 저장된다. 따라서 박싱한 값은 메모리를 더 소비하며 기본형을 가져올 때에도 메모리를 탐색하는 과정이 필요하다.

자바 8에는 기본형을 입출력으로 사용하는 상황에서 오토박싱 동작을 피할 수 있도록 특별한 버전의 함수형 인터페이스를 제공한다.

ex)

```java
public interface IntPredicate{
	boolean test(int t);
}

IntPredicate evenNumbers = (int i ) -> i % 2 == 0;
evenNumbers.test(1000); // 박싱과정이 없음

Predicate<Integer> oddNumbers = (Integer i) -> i % 2 != 0;
oddNumbers.test(1000); // 박싱과정이 있음
```

**예외, 람다, 함수형 인터페이스의 관계**

함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다. 예외를 던지는 람다를 표현식으로 만들려면 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의하거나 람다를 try/catch블록으로 감싸야 한다.

ex)

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
	String process(BufferedReader b) throws IOException;
}
BufferedReaderProcessor p = (Bufferedreader br) -> br.readLine();

/*
하지만 만약 function<T, R> 형식과 같이 이미 정의된 함수형 인터페이스를
파라미터로 받는 API를 사용하고 있으며 직접 함수형 인터페이스를 만들기 어려운 상황일 때는
try/catch로 예외를 잡는다. */
Function<BufferedReader, String> f = (BufferedReader b) -> {
	try {
		return b.readLine();
	}
	catch (IOException e) {
		throw new RuntimeException(e);
	}
}
```

## 5. 형식 검사, 형식 추론, 제약

람다로 함수형 인터페이스의 인스턴스를 만들 수 있다.
람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지의 정보가 포함되어 있지 않다.

### 1. 형식 검사

람다가 사용되는 콘텍스트를 이용해서 람다의 형식을 추론할 수 있다.

어떤 콘텍스트(ex : 람다가 전달될 메서드 파라미터나 람다가 할당되는 변수 등)에서 기대되는 람다 표현식의 형식을 **대상 형식**이라고 부른다.

```java
List<Apple> heavierThan150g = 
		filter(inventory, (Apple apple) -> apple.getWeight() > 150);
```

위와 같은 람다식이 있을 때 다음의 순서로 형식 확인 과정이 진행된다.

1. filter 메서드의 선언을 확인한다.
2. filter메서드는 두 번째 파라미터로 Predicate<Apple>형식을 기대한다.
3. Predicate<Apple>은 test라는 한 개의 추상 메서드를 정의하는 함수형 인터페이스이다.
4. test메서드는 Apple을 받아 boolean을 반환하는 함수 디스크립터를 묘사한다.
5. filter 메서드로 전달된 인수는 이와 같은 요구사항을 충족해야 한다.

위 예제의 람다 표현식은 Apple을 인수로 받아 boolean을 반환하는 유효한 코드이다.

### 2. 같은 람다, 다른 함수형 인터페이스

**대상 형식**의 특징 때문에 같은 람다 표현식이더라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스를 사용할 수 있다.

예로 Callable과 PrivilegedAction 인터페이스는 모두 인수를 받지 않으며 제네릭 형식을 반환하는 함수를 정의한다. 따라서 다음 두 문장은 모두 유효하다.

```java
Callable<Integer> c = () -> 42;
PriviliegedAction<Integer> p = () -> 42;
```

즉, 하나의 람다 표현식을 다양한 함수형 인터페이스에 사용할 수 있다.

**다이아몬드 연산자**

자바 7에서도 다이아몬드 연산자로 콘텍스트에 따른 제네릭 형식을 추론할 수 있다. 인스턴ㅇ스 표현식의 형식 인수는 콘텍스트에 의해 추론된다.

```java
List<String> listOfStrings = new ArrayList<>();
List<Integer> listOfIntegers = new ArrayList<>();
```

**특별한 void 호환 규칙**

람다의 바디에 일반표현식이 있으면 void를 반환하는 함수 디스크립터와 호환된다.

```java
// Predicate는 불리언 반환값을 갖는다.
Predicate<String> p = s-> list.add(s);

// Consumer는 void 반환값을 갖는다.
Consumer<String> b = s -> list.add(s);
```

### 3. 형식 추론

자바 컴파일러는 람다 표현식이 사용된 컨텍스트(대상 형식)를 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다.

대상 형식을 이용해서 함수 디스크립터를 알 수 있으므로 컴파일러는 람다의 시그니처도 추론할 수 있다. 결과적으로 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다.

자바 컴파일러는 다음처럼 형식을 추론할 수 있다.

```java
// 형식을 명시해줬기 때문에 형식을 추론할 필요가 없다.
Coomparator<Apple> c = 
	(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

// 형식을 명시해주지 않아도 함수 디스크립터를 알기때문에 람다 시그니처도 추론할 수 있다.
Coomparator<Apple> c = 
	(a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

### 4. 지역 변수 사용

람다 표현식에서는 익명함수가 하는 것처럼 **자유 변수**(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다. 이와 같은 동작을 **람다 캡처링**이라고 부른다.

```java
int port = 1337;
Runnable r = ()-> System.out.println(port);
```

하지만 약간의 제약이 있다.

람다는 인스턴스 변수와 정적 변수는 자유롭게 캡쳐(람다 바디에서 참조)할 수 있다. 하지만 그러려면 지역변수는 명시적으로 final로 선언되어 있거나 final로 선언된 변수와 똑같이 사용되어야 한다.

```java
// 에러 : 람다에서 참고하는 변수는 final 또는 final처럼 취급되어야 한다.
int port = 1337;
Runnable r = ()-> System.out.println(port);
port = 31331;
```

**지역 변수의 제약**

일단 내부적으로 인스턴스 변수와 지역 변수는 다르다.

인스턴스 변수는 힙에 저장되는 반면 지역 변수는 스택에 위치한다. 람다에서 지역 변수에 바로 접근할 수 있다는 가정하에 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었음에도 람다를 실행하는 스레드에서는 해당 변수에 접근하려고 할 수 있다.

따라서 자바구현에선 원래 변수에 접근하는 것을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다.

또한 지역 변수의 제약때문에 외부 변수를 변화시키는 일반적인 명령형 프로그래밍 패턴(병렬화를 방해하는 요소)에 제동을 걸 수 있다.

**클로저**

클로저란 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스를 가리킨다. 클로저는 클로저 외부에 정의된 변수의 값에 접근하고 값을 바꿀 수 있다. 자바 8의 람다와 익명클래스는 클로저와 비슷한 동작을 수행한다.

## 6. 메서드 참조

메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다.

```java
inventory.sort(a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));

에서 메서드 참조를 사용하면 아래처럼 코드를 작성할 수 있다.

inventory.sort(comparing(Apple::getWeight));
```

### 1. 요약

메서드 참조는 특정 메서드만을 호출하는 람다의 축약형이라고 생각할 수 있다.

메서드 참조를 이용하면 기존 메서드 구현으로 람다 표현식을 만들 수 있다. 명시적으로 메서드명을 참조함으로써 가독성을 높일 수 있다.

Apple::getWeight는 Apple 클래스의 getWeight 메서드 참조이다. 람다표현식

(Apple a) → a.getWeight() 를 축약한 것이다.

**메서드 참조를 만드는 방법**

1. 정적 메서드 참조
   ex) Integer의 parseInt 는 Integer::parseInt와 같이 표현
2. 다양한 형식의 인스턴스 메서드참조
   ex) String의 length 메서드는 String::length로 표현
3. 기존 객체의 인스턴스 메서드 참조
   ex) Apple::getWeight

세 번째 유형의 메서드 참조는 비공개 헬퍼 메서드를 정의한 상황에서 유용하다.

다음처럼 isValidName이라는 헬퍼 메서드가 있을 때

```java
private boolean isValidNAme(String string){
	return Character.isUpperCase(string.charAt(0));
}
```

Predicate<String>를 필요로 하는 상황에서도 위의 메서드를 참조할 수 있다.

```java
filter(words, this::isValidName)
```

### 2. 생성자 참조

ClassName::new와 같이 new 키워드를 사용해서 생성자의 참조를 만들 수도 있다.

생성자 파라미터가 없을 경우

```java
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get(); //Supplier의 get메서드를호출해서 새로운 객체를 만들 수 있다.
```

생성자 파라미터가 1개일 경우

```java
Function<Integer, Apple> c2 = Apple::new
Apple a2 = c2.apply(110); // Function의 apple메서드에 무게를 인수로 호출하여 새 객체 생성
```

2개일 경우는 BiFunction, 3개 이상일 경우는 새로운 함수형 인터페이스를 정의한다.

## 7. 람다, 메서드 참조 활용하기

동작 파라미터화, 익명 클래스, 람다 표현식, 메서드 참조 등을 사용하여 코드 개선

### 1. 1단계 : 코드 전달

sort메서드는 다음과 같은 시그니처를 갖는다.

```java
void sort(Comparator<? super E> c)
```

객체 안에 동작을 포함시키는 방법으로 다양한 전략을 전달핧 수있다.

```java
pyblic class AppleComparator implements Comparator<Apple> {
	public int compare(Apple a1, Apple a2){
		return a1.getWeight().compareTo(a2.getWeight())
	}
}
inventory.sort(new AppleComparator());
```

### 2. 2단계 : 익명 클래스 사용

익명클래스를 사용하면 한 번만 사용하는 인터페이스를 구현하지 않아도 된다.

```java
inventory.sort(new Comparator<Apple>(){
	public int compare(Apple a1, Apple a2){
		return a1.getWeight().compareTo(a2.getWeight())
	}
});
```

### 3. 3단계 : 람다 표현식 사용

```java
inventory.sort((Apple a1, Apple a2) ->
	a1.getWeight().compareTo(a2.getWeight())
);
```

자바 컴파일러는 람다 표현식이 사용된 컨텍스트를 활용해서 람다의 파라미터 형식을 추론할 수 있다.

```java
inventory.sort((a1,a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

Comparator는 Comparable 키를 추출해서 Comparator 객체로 만드는 Function 함수를 인수로 받는 정적 메서드 comparing을 포함한다.

comparing 메서드를 사용하여 코드를 간소화할 수 있다. 사과를 비교하는 데에 사용할 키를 어떻게 추출할 것인지 지정하는 한개의 인수만 포함한다.

```java
import static java.util.Comparator.comparing;
inventory.sort(comparing(apple->apple.getWeight());
```

### 4. 4단계 : 메서드 참조 사용

```java
inventory.sort(comparing(Apple::getWeight));
```

## 8. 람다 표현식을 조합할 수 있는 유용한 메서드

자바 8 API의 몇몇 함수형 인터페이스는 다양한 유틸리티 함수를 제공한다. 함수형 인터페이스의 정의를 벗어나지 않게 하는 디폴트 메서드로 가능.

### 1. Comparator 조합

**역정렬**

내림차순 정렬을 하고 싶다면 Comparator에서 제공하는 reversed 디폴트 메서드를 사용하면 됨.

```java
inventory.sort(comparing(Apple::getWeight).reversed()); //무게 내림차순
```

**Comparator 연결**
무게가 같은 사과가 존재할 경우 원산지지 국가별로 사과를 정렬하는 예시

thenComparing 메서드를 사용해서 두 번째 비교자를 만들 수 있다.

```java
inventory.sort(comparing(Apple::getWeight)
	.reversed()
	.thenComparing(Apple::getCountry));
```

### 2. Predicate 조합

Predicate 인터페이스는 복잡한 프레디케이트를 만들 수 있도록 negate, and, or 세 가지 메서드를 제공한다.

ex) 빨간색이 아닌 사과 조건

```java
Predicate<Apple> notRedApple = redApple.negate(); //redApple의 결과를 반전시킨 객체를 만든다.
```

ex) 빨간색이면서 무거운 사과. and 메서드 사용

```java
Predicate<Apple> redAndHeavyApple = 
	redApple.and(apple -> apple.getWeight() > 150);
// 두 프레디케이트를 연결하여 새 프레디케이트 객체를 만든다.
```

ex) or 메서드를 사용하여 조건 추가

```java
Predicate<Apple> redAndHeavyAppleOrGreen = 
	redApple.and(apple->apple.getWeight() > 150)
					.or(apple->GREEN.equals(a.getColor()));
```

메서드 연결의 장점은 단순한 람다 표현식의 조합으로 더 복잡한 람다 표현식을 만들 수 있고 코드 자체가 문제를 잘 설명한다.

### 3. Function 조합

Function 인터페이스는 Function 인스턴스를 반환하는 andThen, compose 두 가지 디폴트 메서드를 제공한다.

andThen 메서드는 주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달하는 함수를 반환한다.

ex)

```java
Function<Integer, Integer> f = x -> x+1;
Function<Integer, Integer> g = x -> x*2;
Function<Integer, Integer> h = f.andThen(g);
int result = h.apply(1); //4를 반환
```

compose메서드는 g(f(x)) 가 아니라 f(g(x))라는 수식이 된다.

```java
Function<Integer, Integer> f = x -> x+1;
Function<Integer, Integer> g = x -> x*2;
Function<Integer, Integer> h = f.compose(g);
int result = h.apply(1); //3을 반환
```

## 9. 비슷한 수학적 개념

### 1. 적분

### 2. 자바 8 람다로 연결

## 10. 마치며

- **람다 표현식**은 익명함수의 일종. 이름은 없지만 파라미터 리스트, 바디, 반환 형식을 가지며 예외를 던질 수 있다.
- **함수형 인터페이스**는 하나의 추상 메서드만을 정의하는 인터페이스이다.
- 함수형 인터페이스를 기대하는 곳에서만 람다 표현식을 사용할 수 있다.
- 람다 표현식의 기대 형식을 대상 형식 이라고 한다.