# Chapter 19. 함수형 프로그래밍 기법
***내용***

- 일급 시민, 고차원 함수, 커링, 부분 적용
- 영속 자료구조
- 자바 스트림을 일반화하는 게으른 평가와 게으른 리스트
- 패턴 매칭, 자바에서 패턴 매칭을 흉내내는 방법
- 참조 투명성과 캐싱

고급적인 함수형 프로그래밍 기법 소개

고차원 함수, 커링, 영구 자료구조, 게으른 리스트, 패턴 매칭, 참조 투명성을 이용한 캐싱, 콤비네이터 등 학습

## 19.1 함수는 모든 곳에 존재한다.

함수형 프로그래밍 : 함수나 메서드가 수학의 함수처럼 부작용 없이 동작하게 하는 프로그래밍 기법

일급함수 : 일반 값처럼 인수로 전달하거나 결과로 받거나 자료구조에 저장할 수 있는 대상으로 취급할 수 있는 함수

자바 8이후에는 일급함수를 지원한다.
::연산자로 메서드 참조를 만들거나 람다표현식을 사용하여 메서드를 함숫값으로 사용할 수 있다.
또한 메서드 참조로 함수를 일반 값처럼 다룰 수 있다.

```java
Function<String, Integer> strToInt = Integer::parseInt;
```

### 19.1.1 고차원 함수

고차원 함수 : 하나 이상의 함수를 인수로 받거나, 또는 함수를 결과로 반환하는 함수를 고차원 함수라고 부른다.

ex) 함수를 인수로 받아서 함수를 반환하는 고차원 함수의 시그치너

```java
Function<Function<double, Double>, Function<Double, Double>>
```

자바 8에서는 함수를 인수로 전달할 수 있고 결과로 받을 수 있다. 그리고 지역변수로 할당하거나 구조체로 삽입할 수 있으므로 자바8의 함수를 고차원함수라고 할 수 있다.

### 19.1.2 커링

커링을 알기 전 예제를 본다.

예) 국제화를 지원하기 위해 온도의 단위 변환을 하는 문제

변환요소와 기준치 조정 요소가 단위 변환 결과를 좌우한다.

ex) 섭씨를 화씨로 변환하는 공식

```java
cToF(x) = x * 9/5 + 32;
```

메서드로 변환 패턴을 표현했을때는 다음과 같다.

```java
static double converter(double x, double f, double b){
	return x * f + b;
}
```

x : 변환하려는 값

f : 변환 요소

b : 기준치 조정 요소

세 개의 인수를 받는 converter메서드를 사용하여 온도, 킬로미터 등의 단위 변환 문제를 해결할 수 있지만 인수에 변환 요소를 넣는 것은 귀찮고 오타 발생 여지가 크다.

다음과 같이 커링이라는 개념을 사용하여 한 개의 인수를 갖는 변환 함수를 생산하는 팩토리 메서드를 정의할 수 있다.

```java
static DoubleUnaryOperator curriedConverter(double f, double b){
	return (double x) -> x * f + b;
}
```

위처럼 팩토리 메서드를 정의하면 변환요소(f), 기준치(b)만 넘겨주면
변환하려는 값(x) 만 인수로 받아서 값을 변환하는 함수를 얻을 수 있다.

ex)

```java
DoubleUnaryOperator convertCtoF = curriedConverter(9.0/5, 32);
DoubleUnaryOperator convertUSDtoGBP = curriedConverter(0.6, 0);
DoubleUnaryOperator convertKmtoMi = curriedConverter(0.6214, 0);
```

DoubleUnaryIOperator의 applyAsDouble 메서드로 변환하려는 값을 전달하여 사용할 수 있다

```java
double gbp = convertUSDtoGBP.applyAsDouble(1000);
```

결과적으로 기존의 변환 로직을 재활용하는 유연한 코드를 얻는다.

**커링의 이론적 정의**

커링 : x와 y를 인수로 받는 함수 f를 한 개의 인수를 받는 g라는 함수로 대체하는 기법.

g라는 함수는 하나의 인수를 받는 함수를 반환한다.

```java
f(x,y) = (g(x))(y);
```

더 적은 인수를 받는 함수로 반환하며 결과를 얻는 과정에서

과정이 끝까지 완료되지 않은 상태를 함수가 **부분적으로** 적용되었다고 말한다.

## 19.2 영속 자료구조

함수형 프로그램에서 사용되는 자료구조를 학습한다.

함수형 프로그램에서는 보통 영속 자료구조라고 부른다. (DB에서 말하는 데이터영속의 의미와 다름)

함수형 메서드에서는 전역 자료구조나 인수로 전달된 구조를 갱신할 수 없다.

이유는, 자료구조를 바꾼다면 같은 인수를 받는 같은 메서드 호출에 대해 결과가 달라지며 참조 투명성에 위배되고 인수를 결과로 단순하게 매핑할 수 있는 능력이 상실되기 때문.

### 19.2.1 파괴적인 갱신과 함수형

함수에 전달된 인수를 함수 내에서 변형시키는 문제. (파괴적인 갱신)

이 함수 외에 다른 곳에서 전달된 인수를 참조하고 있다면 다른 버그를 발생시킬 수 있다..

ex) 파괴적인 갱신

```java
static TrainJourney link(TrainJourney a, TrainJourney b){
	if(a == null){
		return b;
	}
	TrainJourney t = a;
	while(t.onward != null){
		t = t.onward;
	}
	t.onward = b;
	return a;
}
```

함수형에서는 이러한 부작용을 수반한 메서드를 제한하는 방식으로 문제를 해결한다.

전달된 기존 자료구조를 변형하고 다시 반환하는 것이 아니라,
기존의 자료구조를 변형하지 않도록 새로운 자료구조를 만든다.

ex) 갱신 없이 새로운 자료구조 반환

```java
static TrainJourney append(TrainJourney a, TrainJourney b){
 return a == null ? b : new TrainJourney(a.price, append(a.onward, b));
}
```

또한 함수의 호출자 역시 반환된 자료구조의 append 결과를 갱신해서는 안된다.

다른 곳에서 참조하고 있는 자료구조가 변형될 수 있기 때문이다.

### 19.2.2 트리를 사용한 다른 예제
19.2.3 함수형 접근법 사용

함수형 자료구조의 영속 : 저장된 값이 다른 누군가에 의해 영향을 받지 않는 상태

p.593~596 참고

## 19.3 스트림과 게으른 평가

스트림은 한 번만 소비할 수 있다는 제약이 있어서 재귀적으로 정의할 수 없다.

이런 제약때문에 발생하는 문제를 학습.

### 19.3.1 자기 정의 스트림

정수 스트림으로 소수 스트림을 생성하는 예

```java
public static Stream<Itneger> primes(int n){
	return Stream.iterate(2, i -> i + 1)
						.filter(MyMathUtils::isPrime)
						.limit(n);
}

public static boolean isPrime(int candidate){
	int candidateRoot = (int) Math.sqrt((double) candidate);
		
	return IntStream.rangeClosed(2, candidateRoot)
								.noneMatch(i -> candidate % i == 0);
}
```

위 코드보다 효율적으로 코드를 작성할 수 있다.

1. 소수를 선택할 숫자 스트림이 필요하다.
2. 스트림에서 첫 번째 수(스트림의 머리, head)를 가져온다. 이 숫자는 소수다. (2부터 시작..)
3. 스트림의 꼬리(헤드를 제외한 나머지) 에서 가져온 수로 나누어 떨어지는 모든 수를 걸러 제외시킨다.
4. 제외되고 남은 숫자만 포함하는 새로운 스트림에서 소수를 찾는다.
   1 번부터 다시 재귀적으로 반복한다.

   ![img.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4bef26e3-4d95-4dde-969c-9158d114466e/img.gif)


아리스토테네스의 체

**1단계 : 스트림 숫자 얻기**

IntStream.iterate 메서드를 사용하여 2에서 시작하는 무한 숫자 스트림을 생성한다.

```java
static IntStream numbers(){
	return IntStream.iterate(2, n -> n + 1);
}
```

**2단계 : 머리 획득**

IntStream에서 첫 번째 요소를 반환하는 findFirst 메서드로 머리값을 획득한다.

```java
static int head(IntStream numbers){
	return numbers.findFirst().getAsInt();
}
```

**3단계 : 꼬리 필터링**

스트림의 꼬리를 얻는 메서드를 정의한다.

```java
static IntStream tail(IntStream numbers){
	return numbers.skip(1);
}
```

획득한 머리값으로 숫자를 필터링할 수 있다.

```java
IntStream numbers = numbres();
int head = head(numbers);
IntStream filtered = tail(numbers)
```

**4단계 : 재귀적으로 소수 스트림 생성**

반복적으로 머리를 얻어서 스트림을 필터링

```java
static IntStream primes(IntStream numbers){
	int head = head(numbers);

	return IntStream.concat(IntStream.of(head),
						primes(tail(numbers).filter(n -> n % head != 0)));
}
```

**나쁜 소식**

4단계 코드를 실행하면 java.langIllegalStateException:stream has already been operated upon or closed 에러가 발생한다.

머리와 꼬리로 분리하는 중 최종 연산 findFirst와 skip을 사용했는데

최종연산을 스트림에 호출하면 스트림이 완전 소비되기 때문에 에러가 발생한다.

**게으른 평가**

IntStream.concat은 두 개의 스트림을 받는다.

두 번째 인수가 primes를 직접 재귀적으로 호출하면서 무한재귀에 빠진다.

두 번째 인수에서 prines를 게으르게 평가하는 방식으로 문제를 해결할 수 있다.

소수를 처리할 필요가 있을 때만 스트림을 실제로 평가한다는 것이다.

### 19.3.2 게으른 리스트 만들기

자바 8의 스트림은 게으르다. 요청을 할 때만 값을 생성한다.

연산을 적용해도 연산이 수행되지 않고 **최종 연**산 을 적용했을 때에만 실제 연산이 이루어진다.

게으른 특성때문에 각 연산별로 스트림을 탐색할 필요 없이 한 번에 여러 연산을 처리할 수 있다.

**기본적인 연결 리스트**

p.600~601 참고

**기본적인 게으른 리스트**

Supplier<T>를 이용해서 게으른 리스트를 만들면 꼬리가 모두 메모리에 존재하지 않게 할 수 있다.

Supplier<T>로 리스트의 다음 노드를 생성한다.

```java
class LazyList<T> implemenets MyList<T>{
	final T head;
	final Supplier<MyList<T>> tail;
	
	public LazyList(T head, Supplier<MyList<T>> tail){
		this.head = head;
		this.tail = tail;
	}
	
	public T head(){
		return head;
	}
	
	public MyList<T> tail(){
		return tail.get();
	}
	
	public boolean isEmpty(){
		return false;
	}
}
```

Supplier의 get메서드를 호출하면 마치 팩토리로 새로운 객체를 생성하듯이 LazyList의 노드가 만들어진다.

연속적인 숫자의 다음 요소를 만드는 LazyList의 생성자에 tail인수로 Supplier를 전달하는 방식으로 n으로 시작하는 무한히 게으른 리스트를 만들 수 있다.

```java
public static LazyList<Integer> from(int n) {
	 return new LazyList<Itneger>(n, () -> from(n + 1));
}
```

다음과 같이 요소를 가져올 수 있다.

```java
LazyList<Integer> numbers = from(2);
int two = numbers.head();
int three = numbers.tail().head();
int four = numbers.tail().tail().head();
```

**소수 생성으로 돌아와서**

스트림 API로는 불가능했던 소수 생성 작업을 위의 게으른 소수 리스트로 생성한다면  다음과 같이 코드를 작성할 것이다.

```java
public static MyList<Integer> primes(MyList<Integer> numbres){
	return new LazyList<>(
					numbers.head(),
					() -> primes(numbers.tail()
															.filter(n -> n % numbers.head() != 0)
											)
					);
}	
```

하지만 LazyList 클래스(정확히는 MyList 인터페이스)에는 filter 메소드가 없다. ⇒ 컴파일 에러

filter 메소드를 정의하자.

**게으른 필터 구현**

```java
public MyList<T> filter(Predicate<T> p){
	return isEmpty() ? this : 
						p.test(head()) ?
							new LazyList<>(head(), () -> tail().filter(p)) :
							tail().filter(p);
}															
```

이제 다음과 같이 tail, head 메서드를 호출하여 n번째 소수를 구할 수 있다.

```java
LazyList<Integer> numbers = from(2);
int two = numbers.head();
int three = numbers.tail().head();
int four = numbers.tail().tail().head();
```

- 모든 소수 출력. 재귀적으로 호출이 가능하다. (스택 오버플로우는 발생함)

```java
static void printAll(MyList<T> list){
	if(list.isEmpty()){
		return;
	}
	sout(list.head());
	printAll(list.tail());
}
```

## 19.4 패턴 매칭

함수형 프로그래밍을 구분하는 특징인 패턴매칭은 정규표현식 패턴 매칭과는 다르다.

패턴매칭을 사용하여 불필요한 코드를 줄일 수 있다.

예를 들어 표현식을 단순화하는 메서드를 구현한다고 가정할 때 5 + 0 는 5로 단순화할 수 있다.

이 때 연산자, 왼쪽 수, 오른쪽 수를 모두 검사해야만 표현식을 단순화 할 수 있는지 알 수 있다.

메서드에 조건 검사를 모두 작성하면 코드가 깔끔하지 않게 된다.

### 19.4.1 방문자 디자인 패턴

자바에선 방문자 디자인 패턴으로 자료형을 언랩할 수 있다.

특히 특정 데이터형식을 ‘방문’하는 알고리즘을 캡슐화하는 클래스를 따로 만들 수 있다.

방문자 클래스를 지정된 데이터 형식의 인스턴스를 입력으로 받는다.

그리고 인스턴스의 모든 멤버에 접근한다.

ex)

```java
class BinOp extends Expr {
	 ...
	public Expr accept(SimplifyExprVisitor v){
		return v.visit(this);
	}
}

//
public class SimplifyExprVisitor{
	...
	public Expr visit(BinOp e){
		if("+".equals(e.opname) && e.right instanceof Number && ...){
			return e.left;
		}
		return e;
	}
}
```

### 19.4.2 패턴 매칭의 힘

자바는 패턴 매칭을 지원하지 않는다.

스칼라에서는 객체 생성자의 인수값과 대조하여 조건에 따른 수식 단순화를 간결하게 작성할 수 있다. p.607

**자바로 패턴 매칭 흉내 내기**

스칼라의 패턴 매칭은 다수준이나 자바 8의 람다를 이용한 패턴 매칭 흉내 내기는 단일 수준의 패턴 매칭만 지원한다.

람다식과 삼항연산자를 사용하여 패턴매칭을 통해 연산을 단순화하는 메서드 patternMatchExpr을 정의할 수 있다.

```java
static <T> T patternMatchExpr(
	Expr e,
	TriFunction<String, Expr, Expr, T> binopcase,
	Function<Integer, T> numcase,
	Supplier<T> defaultcase){
	
	return
		(e instanceof BinOp) ?
			binopcase.apply((BinOp)e).opname, ((BinOp)e).left, ((BinOp)e).right) :
			(e instanceof Number) ?
				numcase.apply(((Number)e).val) "
				defaultcase.get();
}
```

```java
patternMatchExpr(e, (op, l, r) -> {return binopcode;},
	(n) -> {return numcode;}.
	() -> {return defaultcode;});
```

위 코드는 e의 타입에따라서 전달된 람다를 실행하여 리턴하는 코드이다.

patternMatchExpr을 이용해서 아래와 같이 표현식을 단순화할 수 있다.

ex) 덧셈, 곱셈 표현식 단순화

```java
public static Expr simplify(Expr e){
  // BinOp 표현식을 처리하는 함수 정의
	TriFunction<String, Expr, Expr, Expr> binopcase = 
		(opname, left, right) -> {
			if("+".equals(opname)){ //덧셈 처리
				if(left instanceof Number && ((Number) left).val == 0){
					return right;
				}
				if(right instanceof Number && ((Number) right).val == 0){
					return left;
				}
			}
		}
		(opname, left, right) -> {
			if("*".equals(opname)){ //곱셈 처리
				if(left instanceof Number && ((Number) left).val == 1){
					return right;
				}
				if(right instanceof Number && ((Number) right).val == 1){
					return left;
				}
			}
		}
		return new BinOp(opname, left, rught);
	};

	Function<Integer, Expr> numcase = val -> new Number(val); //숫자 처리
	Supplier<Expr> defaultcase = () -> new Number(0); //수식을 인식할 수 없을 때 기본 처리

	return patternMatchExpr(e, binopcase, numcase, defaultcase); // 패턴 매칭 적용
}
		
		
```

## 19.5 기타 정보

함수형, 참조투명성과 관련된 내용 학습(효율성, 같은 결과를 반환하는 것과 관련된 염려사항)

두 개 이상의 함수를 인수로 받아서 다른 함수를 반환하는 메서드나 함수를 가리키는 콤비네이터 개념 학습

### 19.5.1 캐싱 또는 기억화

네트워크 범위 내에 존재하는 노드의 수를 계산하는 computeNumberOfNodes() 라는 메서드가 있다고 가정하면,

computeNumberOfNodes()함수를 호출했을 때 재귀 탐색이 필요하므로 노드 계산 비용이 비싸다.

이 때 참조 투명성이 유지되는 상황이라면 오버헤드를 피할 수 있는 방법이 생긴다.

**기억화**기법을 사용한다.

기억화는 메서드에 래퍼로 HashMap같은 캐시를 추가하는 기법이다.

다수의 호출자가 캐시 자료구조를 갱신하는 기법이므로 순수 함수형 해결방식은 아니지만computeNumberOfNodes메서드 코드 내에서는 참조 투명성을 유지할 수 있다.

```java
final Map<Range, Integer> numberOfNodes = new HashMap<>());
Integer computeNumberOfNodeUIsingCache(Range range){
	Integer result = nubmerOfNodes.get(range)p;
	if(result != null){
		return result;
	}{
	result = computeNumberOfNodes(range);
	numberOfNodes.put(range, result);
	return result;
}
```

### 19.5.2 ‘같은 객체를 반환함’은 무엇을 의미하는가?

참조투명성이란 인수가 같다면 결과도 같아야 한다는 규칙을 만족하는 것을 의미한다.

참조가 다르면 같지 않으므로 참조 투명성을 갖지 않는다고 판단할 수 있다.
하지만 자료구조를 변경하지 않는 상황에서는 참조가 다르다는 것은 의미가 없고 논리적으로 같다고 판단할 수 있다.

함수형 프로그래밍에서는 데이터가 변경되지 않으므로 같다는 의미는 참조가 같다는 것을 의미하는 것이 아니라 주조적인 값이 같다는 것을 의미한다.

### 19.5.3 콤비네이터

콤비네이터 : 두 함수를 인수로 받아서 다른 함수를 반환하는 등 함수를 조합하는 기능.

## 19.6 마치며

- 일급함수란 인수로 전달하거나, 결과로 반환하거나, 자료구조에 저장될 수 있는 함수
- 고차원 함수란 한 개 이상의 함수를 인수로 받아서 다른 함수를 반환하는 함수.
- 커링은 함수를 모듈화하고 코드를 재사용할 수 있도록 지원하는 기법
- 영속 자료구조는 갱신될 때 기존 버전의 자신을 보존함.
- 패턴매칭. 자바의 switch문의 기능을 일반화 가능
- 참조 투명성을 유지하는 경우 계산 결과를 캐시할 수 있음
- 콤비네이터는 둘 이상의 함수나 자료구조를 조합하는 함수형 개념

## 질문

p.608 : 스칼라의 패턴 매칭은 다수준이나 자바 8의 람다를 이용한 패턴 매칭 흉내 내기는 단일 수준의 패턴 매칭만 지원한다. (다수준, 단일수준이 무슨 의미인지? 스칼라에서는 연산자, 숫자 등을 case문 내에서 동시에 모두 비교한 반면 자바에선 연산의 각 부분을 따로 비교한 이 부분을 말한 것인가?)