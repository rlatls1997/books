# Chapter 10. 람다를 이용한 도메인 전용 언어
**내용**

- 도메인전용 언어(domain-specific languages, DSL)란 무엇이며 어떤 형식으로 구성된느지?
- DSL을 API에 추가할 때의 장단점
- JVM에서 활용할 수 있는 자바 기반 DSL을 깔끔하게 만드는 대안
- 효과적인자바 기반 DSL을 구현하는 패턴과 기법
- 이들 패턴을 자바 라이브러리와 도구에서 얼마나 흔히 사용하는가

DSL은 작은, 범용적인 것이 아니라 특정도메인을 대상으로 만들어진 특수 프로그래밍 언어.

DSL은 도메인의 많은 특성 용어를 사용한다.

Maven, Ant 등을 빌드 과정을 표현하는 DSL로 간주할 수 있다.

예로 어떤 쿼리를 프로그램으로 구현한다고 가정한다면,
전문가들은 저수준의 코드로 빠르게 구현할 수 있다.

ex)

```java
while(block != null){
	read(block, buffet)
	...
}
```

위와 같이 구현하려면 locking과 I/O, 디스크할당 등과 같은 지식이 필요하고 애플리케이션 수준이 아닌 시스템 수준의 개념을 다루어야 하기에 어렵다.

이를 SQL 형식처럼 SELECT name FROM … 과 같이 표현하는 것은 쉽고 효과적이다.
이는 자바가 아닌 DSL을 이용하여 데이터베이스를 조작하자는 의미로 통한다.

이런 종류를 **외부적 DSL**이라고 하는데,
이는 데이터베이스가 텍스트로 구현된 SQL표현식을 파싱하고 평가하는 API를 제공하는 것이 일반적이기 때문이다. (???)

위 코드를 스트림 API로 간단하게 표현할 수 있다.

ex)

```java
menu.stream()
	.filter(..)
	...
	forEach(....)
```

스트림의 특성인 메서드 체이닝을 보통 자바 루프의 복잡함 제어와 비교하여 유창함을 의미하는 **플루언트 스타일**이라고 부른다.

위 예제 DSL은 내부적이다.

**내부적 DSL**에서는 SQL의 SELECT … 구문처럼 애플리케이션 수준의 기본값이 자바 메서드가 사용할 수 있도록 데이터베이스를 대표하는 한 개 이상의 클래스 형식으로 노출된다. (자바 + 플루언트 스타일이 내부적 DSL???)

## 10.1 도메인 전용 언어

DSL은 특정 비즈니스 도메인의 문제를 해결하려고 만든 언어이다.

자바에서는 도메인을 표현할 수 있는 클래스와 집합이 필요하다.
DSL이란 특정 비즈니스 도메인을 인터페이스로 만든 API라고 생각할 수 있다.
(DSL이 특정 기능만을 수행하는 라이브러리의 느낌인건지??)

DSL을 이용하여 저수준 구현 세부 사항은 클래스의 private으로 만들어서 숨길 수 있고 이렇게 하여 사용자 친화적인 DSL을 만들 수 있따.

DSL을 구현할 때는

- 프로그래머가 아닌 사람도 코드의 의도를 이해할 수 있도록 해야 한다.
  코드가 비즈니스 요구사항에 부합하는지 확인할 수 있도록 해야 한다.
- 둉료가 쉽게 이해할 수 있도록 가독성 높은 코드를 구현해야 한다.

### 10.1.1 DSL의 장점과 단점

DSL은 코드의 비즈니스 의도를 명확하게 하고 가독성을 높인다. 반면 DSL 구현은 코드이므로 올바로 검증하고 유지보수해야 하는 책임이 따른다.

DSL의 장점과 비용을 확인하고 이득인지 아닌지 판단하여 사용하자.

**장점**

- 간결함 : API는 비즈니스 로직을 간편하게 캡슐화하므로 ㅂ나복을 피할 수 있고 코드를 간결하게 만들 수 있다.
- 가독성 : 도메인 영역의 용어를 사용하므로 비 도메인 전문가도 코드를 쉽게 이해할 수 있다.
- 유지보수 : 잘 설계된 DSL로 구현한 코드는 쉽게 유지보수하고 바꿀 수 있다.
- 높은 수준의 추상화 : 도메인의 문제와 직접적으로 관련되지 않은 세부사항을 숨길 수 있다.
- 집중 : 비즈니스 도메인의 규칙을 표현할 목적으로 설계된 언어이므로 프로그래머가 특정 코드에 집중할 수 있다. ⇒ 생산성이 좋아진다.
- 관심사 분리 : 애플리케이션의 인프라구조와 관련된 문제와 독립적으로 비즈니스 관련된 코드에서 집중하기가 용이하다. 결과적으로 유지보수가 쉬운 코드를 구현한다.

**단점**

- DSL 설계의 어려움 : 간결하게 제한적인 언어에 도메인 지식을 담는 것은 쉽지 않다
- 개발 비용 : 코드에 DSL을 추가하는 작업은 초기에 많은 비용이 든다. 또한 유지보수와 변경은 프로젝트에 부담을 준다.
- 추가 우회 계층 : DSL은 추가적인 계층으로 도메인 모델을 감싸며 이 때 계층을 최대한 작게 만들어 성능 문제를 회피한다.
- 새로 배워야 하는 언어 : DSL을 추가하면 배워야 하는 언어가 한 개 더 늘어난다는 것을 의미한다.
- 호스팅 언어 한계 : 자바같은 범용 프로그래밍언어를 기반으로 만든 DSL은 성가신 문법의 제약을 받고 읽기가 어려워진다. 자바 8이 해결책

### 10.1.2 JVM에서 이용할 수 있는 다른 DSL 해결책

일반적으로 DSL의 카테고리는 두 가지로 나눌 수 있다.

- 내부 DSL(임베디드 DSL) : 순수 자바 코드 같은 기존 호스팅 언어를 기반으로 구현
- 외부 DSL(stnadalone) : 호스팅 언어와는 독립적으로 자체의 문법을 가짐

JVM으로 인해 내부 DSL, 외부 DSL의 중간 카테고리에 해당하는 DSL이 만들어질 가능성.

스칼라나 그루비처럼 JVM에서 실행되며 유현하고 표현이 강력한 언어도 있다.

이들을 다중 DSL이라는 세 번째 카테고리로 칭한다.

**내부 DSL**

내부 DSL이란 자바로 구현한 DSL을 의미한다.

기존 자바는 표현력 있는 DSL을 만드는데 한계가 있었으나 람다의 등장으로 일부 해결될 수 있다.

람다를 사용하여 신호대 잡음 비를 적정 수준으로 유지하는 DSL을 만들 수 있다. (?)

자바 7 문법으로 문자열을 출력하는 상황을
자바 8의 새 forEach메서드를 이용하는 예제로 신호대 잡음 비가 무엇을 의미하는지 보자.

ex)

```java
List<String> numbers= Arrays.asList("one", "two", "three");

numbers.forEach(new Consumber<String>(){
	@Override
	public void accpet(String s){
		sout(s);
	}
});
```

위 코드에는 잡음이 존재한다.
다음처럼 익명 내부 클래스를 람다 표현식으로 바꿀 수 있다.

```java
numbers.forEach(s -> System.out.println(s));
```

메서드참조로 더 간단하게 만들 수 있따.

```java
nubmers.forEach(System.out::println);
```

순수 자바로 DSL을 구현해서 얻는 이점

- 외부 DSL에 비해 새로운 기술과 패턴을 배워 DSL을 구현하는 노력이 현저하게 줄어든다.
- 나머지 코드와 함께 DSL을 컴파일할 수 있따. 외부 DSL을 만드는 도구를 사용할 필요가 ㅇ벗으므로 추가 비용이 들지 않는다.
- 개발 팀 일원이 새로운 언어나 외부 도구를 배울 필요가 ㅇ벗다.
- IDE의 자동완성, 리팩터링등을 그대로 사용할 수 있다.
- 여러 도메인에 대응하지 못하여 추가로 DSL을 개발하는 상황에서 추가 DSL을 쉽게 합칠 수 있다.

같은 자바 바이트코드를 사용하는 JVM기반 프로그래밍 언어를 이용해서 DSL합침 문제를 해결하는 방법도 있다. 이런 언어를 다중 DSL이라고 부른다.

**다중 DSL**

JVM에서 실행되는 언어는 100개 이상. 스칼라 루비, 코틀린 등..
이런 언어들은 자바보다 제약이 덜하며 간편한 문법을 지향하도록 설계되었다.

문법적 잡음을 없앨 수 있으며 개발자가 아닌 사람도 코드를 쉽게 이해할 수 있따.

다중 DSL의 단점

- DSL을 만들기 위한 충분한 추가 학습 필요
- 두 개 이상의 언어 혼재로 인해 여러 컴파일러로 소스를 필드하도록 빌드 과정 개선 필요
- 자바와의 호환성 문제

**외부 DSL**

자신만의 문법과 구문으로 새 언어를 설계해야 한다.
새 언어를 파싱하고, 파서의 결과를 분석하고 외부 DSL을 실행할 코드를 만들어야한다.
이는 일반적인 작업도 아니고 쉽게 기술을 얻을 수도 없는 아주 큰 작업이다.

외부 DSL의 가장 큰 장점은 무한한 유연성이다.
비즈니스에 필요한 특성을 완벽하게 제공하는 언어를 설계할 수 있다.
또한 인프라구조 코드와 외부 DSL로 구현한 비즈니스 코드를 명확하게 분리한다.

하지만 분리로 인해 DSL과 호스트 언어 사이에 인공 계층이 생기므로 장단이 있다.(어떤 인공계층을 말하는 것인지??, 외부 DSL과 코드를 잇는 정해진 틀을 말하는 것?)

## 10.2 최신 자바 API의 작은 DSL

자바의 새로운 기능의 장점을 적용한 첫 API는 네이티브 자바 API 자신.

자바 8 이전의 네이티브 자바 API는 이미 한 개의 추상 메서드를 가진 인터페이스를 갖고 있었지만 무명 내부 클래스를 구현하려면 불필요한 코드가 추가되어야 했다.

이는 람다와 메서드참조가 등장하면서 해결되었다.(특히 DSL 관점에서)

자바 8의 Comparator 인터페이스에 새 메서드가 추가되었다.
Comparator 인터페이스 예제를 통해 람다가 어떻게 네이티브 자바 API의 재사용성과 메서드 결합도를 높였는지 확인하자.

ex) 사람의 나이를 기준으로 객체를 정렬한다고 가정한다.
람다가 없으면 다음과 같이 내부클래스로 Comparator 인터페이스를 구현해야 한다.

```java
Collections.sort(persons, new Comparator<Person>(){
	public int compare(Person p1, Person p2){
		return p1.getAge() - p2.getAge();
	}
});
```

내부클래스를 아래와 같이 람다표현식으로 바꿀 수 있다.

```java
Collections.sort(people, (p1, p2) -> p1.getAge() - p2.getAge());
```

람다 표현식은 신호대 잡음 비를 줄이는데 유용하다.

자바는 Comparator객체를 좀 더 가독성 있게 구현할 수 있는 정적 유틸리티 메서드 집합도 제공한다. 이 메서드들은 Comparator 인터페이스에 포함되어 있다. 정적으로 Comparator.comparing 메서드를 임포트해서 다음과 같이 구현할 수 있다.

```java
Collections.sort(peresons, comparing(p->p.getAge()));

// 메서드 참조 활용
Collections.sort(persons, comparing(Person::getAge));
```

자바 8에 추가된 reverse() 메서드를 사용하여 역순으로 정렬할 수도 있다.

```java
Collections.sort(Persons, comparing(Person::getAge).reverse());
```

나이가 같을 경우 사람들을 알파벳 순으로 정렬하는 코드는 다음과 같이 구현할 수있다.

```java
Collections.sort(persiojns, comparing(Person::getAge)
															.thenComparing(Person::getName));
```

마지막으로 List인터페이스에 추가된 새 sort메서드를 이용해 코드를 깔끔하게 정리할 수 있다.

```java
persons.sort(comparing(Person::getAge)
						.thenComparing(Person::getName));
```

이 API는 컬렉션 정렬 도메인의 최소 DSL이다.

람다와 메서드 참조를 이용한DSL이 코드의 가독성, 재사용성, 결합성을 높일 수 있는 것을 보여준다.

### 10.2.1 스트림 API는 컬렉션을 조작하는 DSL

Stream 인터페이스는 네이티브 자바 API에 작은 내부 DSL을 적용한 것이다.

컬렉션의 항목을 필터, 정렬, 변환, 그룹화, 조작하는 작지만 강력한 DSL인 것이다.

ex) “ERROR”라는 단어로 시작하는 파일의 첫 40행을 수집하는 작업

반복문으로 구현할 경우 코드가 장황하여 한 눈에 파악하기 어렵고 문제가 분리되지 않아서 가독성과 유지보수성이 저하된다. 같은 의미를 지닌 코드가 여러 행에 분산되어 있다.

- FileReader가 만들어짐
- 파일이 종료되었는지 확인하는 while문의 조건
- 파일의 다음 행을 읽는 while루프의 마지막 행
- …..

Stream 인터페이스를 이용하여 함수형으로 구현하면 간결하게 구현할 수 있다.

```java
List<String> errors = Files.lines(Paths.get(fileName)) //파일을 열어서 문자열 스트림 생성
													.filter(line -> line.startsWith("ERORR")) //필터링
													.limit(40) //결과를 첫 40행으로 제한
													.collect(toList()); //결과 문자열을 리스트로 수집
```

스트림 API의 플루언트 형식은 잘 설계된 DSL의 특징이다.

중간 연산은 게으르며 다른 연산으로 파이프라인될 수 있는 스트림으로 반환되고
최종 연산은 적극적이며 전체 파이프라인이 계산을 일으킨다.

### 10.2.2 데이터를 수집하는 DSL인 Collectors

Colelctor 인퍼에스를 데이터 수집을 수행하는 DSL로 간주할 수 있다.

Collector 인터페이스를 통해 스트림의 항목 수집, 그룹화, 파티션을 할 수 있다. 또한 정적 팩토리 메서드를 이용하여 Collector 객체를 만들고 합칠 수 있다.

DSL 관점에서의 메서드 설계 확인

Comparator 인터페이스는 다중 필드 정렬을 지원하도록 합쳐질 수 있으며 Collectors는 다중 수준 그룹화를 달성할 수 있도록 합쳐질 수 있다.

ex) 자동차를 브랜드별, 색상별로 그룹화하는 예

```java
Map<String, Map<Color, List<Car>>> carsByBrandAndAolor = 
	cars.stream().collect(groupingBy(Car::getBrand,
													groupingBy(Car::getColor)));

```

두 Comparator를 연결하는 것과 비교해서 다른점은?

아래는 두 Comparator를 플루언트 방식으로 연결해서 다중 필드 Comparator를 정의했다.

```java
Comparator<Person> comparator = 
	comparing(Person::getAge).thenComparing(Person::getName);
```

(위 예시는 첫 비교 필드가 같을 경우 다른 비교 필드를 사용하는 반면 아래 예시는 중첩된 grouping을 하는 차이? 이 예시의 존재 이유를 모르겠다.)

반면 Collectors API를 이용해서 Collectors를 중첩함으로서 다중 수준 Collector를 만들 수 있다.

ex)

```java
Collector<? super Car, ?, Map<Brand,Map<Color, List<Car>>>> carGroupingCollector = 
	groupingBy(Car::getBrand, groupingBy(Car::getColor));
```

셋 이상의 컴포넌트를 조합할 때는 보통 플루언트 형식이 중첩 형식에 비해 가독성이 좋다.

또한 플루언트 형식처럼 Collector를 연결하지 않고 Collector생성을 여러 정적 메서드로 중첩함으로써 안쪽 그룹화가 처음 평가되고 코드에서는 반대로 가장 나중에 등장하게 된다.

(=안쪽 그룹핑이 먼저 이루어지지만 코드는 가장 마지막에 작성되어 있음을 의미하는듯)

---

groupingBy 팩터리 메서드에 작업을 위임하는 GroupingBuilder를 만들면 더 쉽게 문제를 해결할 수 있다.
GroupingBuklder는 유연한 방식으로 여러 그룹화 작업을 만든다.

ex)

```java
public class GroupingBuilder<T, D, K> {
	private final Collector<? super T, ?, Map<K, D>> collector;

	private GroupingBuilder(Collector<?  super T, ?, Map<K, D>> collector){
		this.collector = collector;
	}

	public Collector<> super T, ?, Map<K, D>> get(){
		return collector;
	}
	
	public 
<J> GroupingBuilder<T, Map<K, D>, J> after(Function<? super T, ? extends K> classifier){
		return new GroupingBuilder<>(groupingBy(classifier, collector));
	}
	
	public static <T, D, K> GroupingBuilder<T, List<T>, K>
		groupOn(Function<? super T, ? extends K> classifier){
			return new GroupingBuilder<>(groupingBy(classifier));
	}
}
```

플루언트 형식 빌더에서의 문제

ex)

```java
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>> carGroupingCollector = 
	groupOn(Car::getColor).after(Car::getBrand).get()
```

중첩된 그룹화 수준에 반대로 그룹화 함수를 구현해야 하므로 유틸리티 사용 코드가 직관적이지 않다.

== 안쪽 그룹먼저 그룹화하는 함수를 명시하고 바깥쪽 그룹화 함수를 명시해야 한다.;;

## 10.3 자바로 DSL을 만드는 패턴과 기법

DSL은 특정 도메인 모델에 적용할 친화적이고 가독성 높은 API를 제공한다.

### 10.3.1 메서드 체인

메서드 체인을 이용하여 비개발자도 코드를 쉽게 이해하도록 할 수 있다.

ex)

```java
Order order = forCustomer("BogBank")
	.buy(80)
	.stock("IBM")
	.on("NYSE")
	.at(125.00)
	.sell(50)
	.....
	.end();
```

위처럼 코드를 작성하기 위해선
도메인 객체를 만드는 빌더를 구현해야 한다.

그리고 빌더를 계속 이어나가기 위해 적절한 빌더나 객체를 반환하는 메서드를 구현해야한다.

### 10.3.2 중첩된 함수 이용

중첨된 함수 DSL 패턴은 다른 함수 안에 함수를 이용하여 도메인 모델을 만든다.

ex)

```java
Order order = order("BigBank"
										buy(80, 
											stock("IBM", on("NYSE")), at(125.00)),
										sell(50,
											stock("GOOGLE", on("NASDAQ")), at(375.00))
										);
```

메서드체인에 비하여 중첩 방식이 도메인 객체 계층 구조에 그대로 반영된다는 것이 장점.

하지만 더 많은 괄호를 사용해야 하고 인수 목록을 정적 메서드에 넘겨줘야한다.

### 10.3.3 람다 표현식을 이용한 함수 시퀀싱

람다 표현식으로 정의한 함수 시퀀스를 사용한다.

ex)

```java
Order order = order( o->{
	o.forCustomer("BigBang");
	o.buy(t->{
		t.quantity(80);
		t.price(125.00);
		t.stock(s->{
			s.symbol("IBM");
			s.market("NYSE");
		});
	});
	o.sell(t->{
		t.quantity(50);
		t.price(475.00);
		t.stock(s->{
			s.symbol("GOOGLE");
			s.market("NASDAQ");
		});
	});
});
```

위와 같은 DSL을 만들기 위해서는 람다 표현식을 받아서 실행하여 도메인 모델을 만들어내는 여러 빌더를 구현해야 한다.

이 패턴은

메서드 체인 패턴처럼 플루언트 방식으로 거래 주문을 정의할 수 있고,
중첨 함수 형식처럼 다양한 람다 표현식의 중첩 수준과 비슷하게 도메인 객체의 계층 구조를 유지한다.

대신 많은 설정코드가 필요하고 DSL 자체가 자바 8 람다 표현식 문법에 의한 잡음의 영향을 받는다.

### 10.3.4 조합하기

한 DSL에 두 개 이상의 패턴을 조합하여 사용할 수 있다.

ex) 여러 DSL패턴을 이용한 예

```java
Order order = forCustomer("BigBank", // 주문의 속성을 지정하는 중첩함수
								buy(t-> t.quantity(80) // 한 개의 주문을 만드는 람다표현식
													.stock("IBM") // 거래 객체를 만드는 람다 바디의 메서드체인
													.on("NYSE")
													.at(125.00)),
								sell(t-> t.quantity(60)
													.stock("GOOGLE")
													.on("NASDAQ")
													.at(125.00)));
```

위와 같이 중첩 함수 패턴과 람다 표현식을 혼용하여 사용할 수 있다.

장점을 모을 수 있지만,
여러가지 패턴의 혼용으로 사용자가 배우는 데오래걸릴 수 있다.

### 10.3.5 DSL에 메서드 참조 사용하기

주식 주문에 세금을 추가하여 최종값을 계산하는 기능이 추가된다고 가정해보자.

- 주문의 총 합에 적용될 세금

```java
public class Tax{
	public static double regional(double value){
		return value * 1.1;
	}

	public static double general(double value){
		return value * 1.3;
	}

	public static double surcharge(double value){
		return value * 1.05;
	}
```

그리고 boolean 플래그 집합을 이용하여 주문에 세금을 적용하는 메서드를 다음과 같이 구현할 수 있다.

```java
public static double calculate(Order order, boolean useRegional,
															boolean useGeneral, boolean useSurcharge){
	double value = order.getValue();
	if(useRegional) value = Tax.regional(value);
	if(useGeneral) value = Tax.general(value);
	if(useSurcharge) value = Tax.surcharge(value);
	return value;
}
```

그리고 다음과 같이 세금을 적용하는 메서드를 사용할 수 있다.

```java
double value = calculate(order, true, false, true);
```

위 구현은 문제가 있다.

메서드를 사용할 때 몇 번째 boolean값이 어떤 플래그를 의미하는지 한 번에 파악할 수 없다.

다음과 같이 유착하게 boolean 플래스를 설정하는  최소(?) DSL을 제공하는 것이 더 좋다

ex) 좀 더 명확한 세금 계산 방식 코드

```java
public class TaxCalculator{
	private boolean useRegional;
	private boolean useGeneral;
	private boolean useSurcharge;
	
	public TaxCalculator withTaxRegional(){
		useRegional = true;
		return this;
	}

	public TaxCalculator withTaxGeneral(){
		useGeneral = true;
		return this;
	}
	
	public TaxCalculator withTaxSurcharge(){
		useSurcharge = true;
		return this;
	}
	
	public double calculate(Order order){
		return calculate(order, useRegional, useGeneral, useSercharge);
	}
}
```

아래와 같이 어떤 세율을 적용할 것인지 한 눈에 파악하기 쉽게 코드를 작성할 수 있다.

```java
double value = new TaxCalculator().withTaxRegional()
																	.withTaxSurcharge()
																	.calculate(order);
```

아쉬운점은 코드가 장황하다는 것이고
각 세금에 해당하는 불리언 필드가 필요하므로 확장성도 제한적이다.

자바의 함수형 기능을 사용하여 다음과 같이 더 간결하고 유연한 방식으로 변경하면서 같은 가독성을 달성할 수 있다.

```java
public class TaxCalculator{
	public DoubleUnaryOperator taxFunction = d-> d;// 주문값에 적용된 모든 세금을 계산하는 함수

	public TaxCalculator with(DoubleUnaryOperator f){
		taxFunction = taxFunction.andThen(f); //새로운 세금 계한 함수를 현재 함수와 이어줌
		return this; //세금 함수가 계속 연결될 수 있도록 결과를 반환
	}
	
	public double calculate(Order order){
		return taxFunction.applyAsDouble(order.getValue()); // 세금 적용한 주문값 계산
	}
}	
```

주문에 사용할 함수 한 개의 필드만 필요로 하며
아래와 같이 세금을 적용하는 함수를 체이닝 형식으로 적용할 수 있다.

```java
double value = new TaxCalculator().with(Tax::regional)
																	.with(Tax::surcharge)
																	.calculate(order);
```

## 10.4 실생활의 자바 8 DSL

DSL 패턴 별 장점과 단점

- 메서드 체인
    - 장점
        1. 정적 메서드를 최소화하거나 없앨 수 있다.
        2. 문법적잡음을 최소화한다.
        3. DSL 사용자가 정해진 순서로 메서드를 호출하도록 강제할 수 있다.
    - 단점
        1. 구현이 장황하다.
        2. 빌드를 연결하는 접착코드가 필요하다.
- 중첩 함수
    - 장점
        1. 구현의 장황함을 줄일 수 있다
        2. 함수 중첩으로 도메인 객체 계층을 반영한다.
    - 단점
        1. 정적 메서드의 사용이 빈번하다.
        2. 이름이 아닌 위치로 인수를 정의한다.
        3. 선택형 파라미터를 처리할 메서드 오버로딩이 필요하다.
- 람다를 이용한 함수 시퀀싱
    - 장점
        1. 정적메서드를 최소화하거나 없앨 수 있다.
        2. 람다 중첩으로 도메인 객체 계층을 반영한다.
        3. 빌더의 접착 코드가 없다.
    - 단점
        1. 구현이 장황하다.
        2. 람다 표현식으로 인한 문법적 잡음이 DSL에 존재한다.

### 10.4.1 JOOQ

SQL을 구현하는 내부적 DSL로 자바에 직접 내장된 형식 안전 언어.

ex) JOOQ DSL을 이용하여 SQL질의를 구현한 예

```java
create.selectFrom(BOOK)
	.where(BOOK.PUIBLISHED_IN.eq(2016))
	.orderBy(BOOK.TITLE)
```

JOOQ DLS은 스트림 API와 조합하여 사용할 수 있다는것이 장점이다.

이때문에 SQL 질의 실행으로 나온 결과를 한 개의 플루언트 구문으로 데이터를 메모리에서 조작할 수 있다.

ex) JOOQ DSL을 이용한 데이터베이스에서의 책 선택

```java
Class.for("org.h2.Driver");
try(Connection c = getConnection("jdbc:h2:~/sql-goodies-with-mapping", "sa", "")){
	DSL.using(c)
			.select(BOOK.AUTHOR, BOOK.TITLE)
			... //JOOQ SQL문 시작
	.fetch() //JOOQ DSL로 SQL문 정의
	.stream() //데이터베이스에서 데이터 가져오기, JOOQ문 중료
	.collection(groupingBy( // 스트림 API로 데이터베이스에서 가져온 데이터 처리 시작
			r-> r.getValue(BOOK.AUTHOR)
					...
	
```

위 예시와 같이 JOOQ를 통한 SQL 질의 실행으로 나온 결과를 바로 stream API와 이어서 처리할 수 있다.

JOOQ DSL 을 구현하는 데 메서드 체인 패턴을 사용했음을 알 수 있다.

### 10.4.2 큐컴퍼

동작주도개발(BDD)은 테스트 주도 개발의 확장으로, 다양한 비즈니스 시나리오를 구조적으로 서술하는 간단한 도메인 전용 스크립팅 언어를 사용한다.

BDD는 비즈니스 가치를 전달하는 개발 노력에 집중하며 비즈니스 어휘를 공유함으로 도메인 전문가와 프로그래머 사이의 간격을 줄인다.

큐컴버는 다른 BDD프레임워크와 마찬가지로 이들 명령문을 실행할 수 있는 테스트 케이스로 변환한다. 따라서 이 개발 기법으로 만든 스크립트 결과물은 실행할 수 있는 테스트임과 동시에 비즈니스 기능의 수용 기준이 된다.

ex) 큐컴버 스크립팅 언어로 비즈니스 시나리오를 정의한 예제

```java
Feature: Buy stock
	Scenario: Buy 10 IBM stocks
		Given the price of a "IBM" stock is 125$
		When I buy 10 "IBM"
		Then the order value should be 1250$
```

큐컴버는 다음 세 가지로 구분되는 개념을 사용한다.

전제 조건 정의(Given), 시험하려는 도메인 객체의 실질 호출(When), 테스트 케이스의 결과를 확인하는 어셜션(Then)

위 스크립트는 테스트 케이스 변수를 캡처하는 정규표현식으로 매칭되며 테스트 자체를 구현하는 메서드로 이를 전달한다.

ex) 큐컴버 어노테이션을 이용한 테스트 시나리오 구현 예

```java
public class BuyStocksSteps{
	private maP<Strinmg, Integer> stockUInitPrices = new HashMap<>();
	private Order order = new Order();
	
	@Given("^the price of a \"(.*?)\" stock is (\\d+)\\$$") //시나리오 전제 조건 주식 단가 정의
	public void setUnitPrice(String stockName, int unitPrice){
		stockUnitvalues.put(stockName, unitPrice);
	}
	
	@When(...~~)
	public void buyStocks(int quantity, String stockName){
		...
	}
	
	@Then(...~~)
	public void checkOrderValue(int expectedValue){
		...
	}	
```

람다표현식의 지원으로 두 개의 인수 메서드

(어노테이션 값을 포함한 정규 표현식, 테스트 메서드를 구현하는 람다)

를 이용하여 어노테이션을 제거한 다른 문법을 큐컴버로 개발할 수 있다.

ex)

```java
public class BuyStocksSteps implements cucumber.api.java8.En {
	...
	public BuyStocksSteps(){
		Given("^the price of a \"(.*?)\" stock is (\\d+)\\$$", 
			(String stockName, int unitPrice) -> {
				stockUnitValues.put(stockName, unitPrice);
			});
		When(...
		Then(...
	}
...
```

두 번째 문법은 코드가 더 단순하다.

큐컴버의 DSL은 외부적 DSL과 내부적 DSL이 어떻게 효과적으로 합쳐질 수 있으며 람다와 함께 가독성 있는 함축된 코드를 구현할 수 있는지를 보여준다.(;)

### 10.4.3 스프링 통합

스프링통합은 유명한 엔터프라이즈 통합 패턴(?)을 지원할 수 있도록 의존성 주입에 기반한 스프링 프로그래밍 모듈을 확장한다.

스프링통합의 핵심 목표는 복잡한 엔터프라이즈 통합 솔루션을 구현하는 단순한 모델을 제공하고 비동기, 메시지주도 아키텍처를 수비게 적용할 수 있게 돕는 것.

스프링통합은 스프링 기반 애플리케이션 내의 경량의 원격, 메시징, 스케줄링을 지원한다. 기존의 스프링 XML설정 파일 기반에도 이들 기능을 지원한다.

스프링통합은 채널, 엔드포인트, 폴러, 채널 인터셉터 등 메시지 기반의 애플리케이션에 필요한 공통 패턴을 모두 구현한다.
가독성이 높아지도록 엔드포인트는 DSL에서 동사로 구현하며 여러 엔드포인트를 한 개 이상의 메시지 흐름으로 조합해서 통합 과정이 구성된다.(?)

ex) 스프링 통합 DSL을 이용해서 스프링 통합 흐름 설정하기

```java
@Configuration
@EnableIntegration
public class MyConfiguration{
	
	@Bean
	public MessageSource<?> integerMessageSource(){
		// 호출 시 AtomicInteger(?)를 증가시키는 새 MessageSource를 생성
		MethodInvokingMessageSource source = new MethodInvokingMessageSource(); 

		source.setObject(new AtomicInetger());
		source.setMethodName("getAndIncrement");
		return source;
	}
	
	@Bean 
	public DirectChannel inputChannel(){
		// MessageSource에서 도착하는 데이터를 나르틑 채널
		return new DirectChannel();
	}
	
	@Bean
	public IntegeratrionFlow myFlow(){
		// IntegrationFlows 리턴
		return IntegrationFlows
							.from(this.integerMessageSource(),
								c -> c.poller(Pllers.fixedRate(10)))
							.channel(this.inputChannel())
							.filter((Integer p) -> p % 2 == 0)
							.transform(Object::toString)
							.chennel(MessageChannels.queue("queueChannel"))
							.get();
	}
}
```

스프링 통합 DSL을 이용해서 myFlow()는 IntegerationFlow를 만든다.

예제는 메서드 체인 패턴을 구현하는 IntegrationFlows클래스가 제공하는 유연한 빌더를 사용한다.

## 10.5 마치며

- DSL의 주 기능은 개발자와 도메인 전문가 사이의 간격을 좁히는 것.
- DSL은 크게
  내부적(DSL이 사용될 애플리케이션을 개발한 언어 그대로 호라용) DSL,
  외부적(직접 언어를 설계해서 사용함) DSL 로 구분할 수 있다.

내부적 DSL은 개발 노력이 적게 드는 반면 호스팅 언어의 문법 제약을 받는다.
외부적 DSL은 높은 유연성을 제공하지만 구현하기가 어렵다.
- JVM에서 이용가능한 언어로 다중 DSL을 개발할 수 있다.
- 자바는 장황하고 문법적으로 엄격하기 때문에 내부적 DSL을 개발하는 언어로는 부적합하지만 람다 표현식과 메서드 참조 덕분에 많이 개선됨.
- 최신 자바는 자체 API에 작은 DSL을 제공한다.
  Stram, Collectors클래스 등에서 제공하는 작은 DSL은 특히 컬렉션 데이터의 정렬, 필터링, 변환, 그룹화에 유용하다.
- 자바로 DSL을 구현할 때 보통 메서드 체인, 중첩 함수, 함수 시퀀싱 세 가지 패턴이 사용된다.

### 궁금했던 것

- p.340 buy(), sell() 메서드가 private인 TradeBuilder생성자를 호출하는 건 무엇?
  마찬가지로 TradeBuilder.stock()도 StockBuilder 생성자를 호출하는데 private
- DSL이 뭐라고 생각하는지? 하나의 특정 도메인을 처리할 수 있는 읽기 쉬운 언어라고 이해하면 되는건가?
  ⇒ 특정 영역을 타겟하고 있는 언어. 예를 들어 SQL. 내부적 DSL은 그 도메인만을 다루는 java언어로 만들어진 새로운 형식의 무언가..
- 내부, 외부 DSL

  ⇒ 내부 DSL은 호스트 언어 구문을 이용하여 자체적으로 의존하는 무언가를 만드는 경우에 해당.
  JAVA 내장 API와 내부 DSL은 차이가 뭘까..?

  ⇒ 외부 DSL은 호스트 언어와 다른 언어(SQL, XML 등..)에서 생성된 DSL

- 스프링통합?

  ⇒ [https://rura6502.tistory.com/entry/Spring-Integration-기본-개념?category=723090](https://rura6502.tistory.com/entry/Spring-Integration-%EA%B8%B0%EB%B3%B8-%EA%B0%9C%EB%85%90?category=723090)

- stream api

  스트림 api는 coolection 등 목록 집합에 특화되었고 메서드 체이닝과 람다 표현식 등으로 비개발자 또한 읽고 이해하기 쉽다. 스트림 api는 dsl에 포함되는건가?

  java 내장이므로 내부 dsl이라고 할 수 있는 건가?
- 