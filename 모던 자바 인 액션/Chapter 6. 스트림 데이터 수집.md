# Chapter 6. 스트림 데이터 수집
- Collectors 클래스로 컬렉션 만들고 사용하기
- 하나의 값으로 데이터 스트림 리듀스하기
- 특별한 리듀싱 요약 연산
- 데이터 그룹화의 분할
- 자신만의 커스텀 컬렉터 개발

스트림의 연산

- 중간 연산 : 한 스트림을 다른 스트림으로 변환하는 연산. 스트림의 요소를 소비하지 않는다.
- 최종 연산 : 스트림의 요소를 소비해서 최종 결과를 도출한다.

toList외에 다양한 요소 누적 방식을 사용해본다.

일반적인 List형식 외에도
Map<T, Z>, Map<T, List<Z>> 등 다양한 형식으로 그릅화할 수 있다.

## 6.1 컬렉터란 무엇인가

collect 메서드로 Collector 인터페이스 구현을 전달한다.

Collector 인터페이스 구현은 스트림의 요소를 어떤식으로 도출할지 지정한다.

### 6.1.1 고급 리듀싱 기능을 수행하는 컬렉터

명령형 프로그래밍에서는 문제를 해결하는 과정에서 다중 루프와 조건문을 추가해야 하기 때문에 가독성과 유지보수성이 떨어진다.

함수형 API는 높은 수준의 조합성과 제사용성을 갖고 있다. collct로 결과를 수집하는 과정을 간단하고 유현하게 정의할 수 있다.

스트림에 collect를 호출하면 스트림의 요소에 리듀싱 연산이 수행된다.collect에서는 리듀싱 연산을 이용해서 스트림의 각 요소를 방문하면서 컬렉터가 작업을 처리한다.

Collector 인터페이스의 메서드를 어떻게 구현하느냐에 따라 스트림에 어떤 리듀싱 연산을 수행할지 결정된다. Collectors유틸리티 클래스는 자주 사용되는 컬렉터 인스턴스를 쉽게 생성할 수 있는 정적 팩토리 메서드를 제공한다. toList()도 여기에 포함된다.

```java
List<Transaction> transactions = transactionStream.collect(Collectors.toList());
```

### 6.1.2 미리 정의된 컬렉터

Collectors 에서 제공하는 메서드의 기능은 크게 세가지로 구분된다.

- 스트림 요소를  하나의 값으로 리듀스하고 요약
- 요소 그룹화
- 요소 분할

## 6.2 리듀싱과 요약

컬렉터로 스트림의 항목을 컬렉션으로 재구성할 수 있다. 즉 컬렉터로 스트림의 모든 항목을 하나의 결과로 합칠 수 있다.

counting()이라는 팩토리 메서드가 반환하는 컬렉터로 메뉴에서 요리 수를 계산하는 예제는 아래와 같다.

```java
 ... = menu.stream().collect(Collectors.counting());
or
 ... = menu.stream().count();
```

Collectors 클래스의 정적 팩토리 메서드를 임포트했다고 가정하면 메서드만 명시하여 사용할 수 있다.

```java
import static java.util.stream.Collectors.*;
...
 ... = menu.stream().collect(counting());
```

### 6.2.1 스트림값에서 최댓값과 최솟값 검색

Collectors의 maxBy, minBy 두 개의 메서드를 이용하여 스트림의 최댓값과 최솟값을 계산할 수 있다.

이 두 컬렉터는 스트림 요소를 비교하는 데 사용할 Comparator를 인수로 받는다.

ex)

```java
Comparator<Dish> dishCaloriesComparator = 
	Comparator.comparingInt(Dish::getCalories); // 인수로 받을 Comparator 정의

Optional<Dish> mostCaloriesDish = 
	menu.stream().collect(maxBy(dishCaloriesComparator)); // Comparator를 인수로 전달
```

Optional인 이유는 menu가 비어있을 경우가 있을 수 있기 때문이다.

스트림에 있는 객체의 숫자나 필드의 합계나 평균 등을 반환하는 연산에도 리듀싱 기능이 자주 사용된다. 이러한 연산을 **요약연산**이라고 부른다.

### 6.2.2 요약 연산

Collectors 클래스는 Collectors.summingInt라는 요약 팩토리 메서드를 제공한다.

summingInt는 객체를 int로 매핑하는 함수를 인수로 받는다.
summingInt의 인수로 전달된 함수는 객체를 int로 매핑한 컬렉터를 반환한다.
summingInt가 collect 메서드로 전달되면 요약 작업을 수행한다.

ex)

```java
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

각 요소의 int로 매핑된 값을 리듀싱, 계속 누적해나가며 최종 결과를 도출한다.

summingLong, summingDouble도 같은 방식으로 동작하나 다른 데이터 형식으로 요약한다.

**평균값**

평균을 구할때는 averagingInt, averagingLong, averagingDouble 메서드를 사용할 수 있다.

```java
double avgCalories = menu.stream().collect(averagingInt(Dish::getCalories));
```

**합계, 갯수, 평균 등의 통계요소**

여러가지 정보가 담긴 통계를 구할 때에는 summarizingInt가 반환하는 컬렉터를 사용할 수 있다.

```java
IntSummaryStatistics menuStatistics = 
	menu.stream().collect(summarizingInt(Dish::getCalories));
```

IntSummaryStatistics객체는 다음 정보를 담는다.

```java
IntSummaryStatistics{count, sum, min, average, max}
```

마찬가지로 long, double에 대응하는 메서드 및 클래스도 존재한다.

### 6.2.3 문자열 연결

켈럭터에 joining 팰토리 메서드를 이용하면 문자열 요소들을 하나의 문자열로 연결하여 반환할 수 있다.

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining());
```

joining 메서드는 내부적으로 StringBuilder를 이용하여 문자열을 하나로 만든다.

만약 위의 Dish클래스가 요리명을 반환하는 toString메서드를 포함하고 있다면 위 코드에서 각 요리명을 가져오는 map 연산을 생략할 수 있다.

```java
String shortMenu = menu.stream().collect(joining());
```

연결되는 요소 사이에 구분자 String을 넣을 수 있는 오버로드된 joining 메서드도 존재한다.

```java
String shortMenu - menu.stream().collect(joining(", "));
```

### 6.2.4 범용 리듀싱 요약 연산

위에서 본 모든 컬렉터는 Collectors.reducing 팰토리 메서드로도 정의할 수 있다.
위의 특화된 컬렉터를 사용하는 이유는 편의성과 가독성 때문이다.

다음 코드처럼 reducing 메서드로 만들어진 컬렉터로도 멘의 모든 칼로리 합계를 계산할 수 있다.

```java
int totalCalories = menu.stream()
	.collect(reducing(0, Dish::getCalories, (i, j) -> i + j));
```

reducing 메서드는 세 개의 인수를 받는다.

- 첫 번째 인수 : 리듀싱 연산의 시작값. (스트림에 인수가 없을때는 반환값의 의미)
- 두 번째 인수 : 변환 함수 (mapping)
- 세 번째 인수 : 같은 종류의 두 항목을 하나의 값으로 도출하는 BinaryOperator

한 개의 인수만을 받는 reducing 메서드도 있다.

```java
Optional<Dish> mostCalorieDish = menu.stream()
.collect(reducing(d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
```

위의 reducing 메서드에서는 한 개의 인수를 받는다.

인수가 세 개인 reducing 메서드로 나타내면

- 첫 번째 인수 = 스트림의 첫 번째 요소
- 두 번째 인수 = 자신을 그대로 받환하는 **항등 함수**
- 세 번째 인수 : 전달받는 인수( 같은 종류의 두 항목을 하나의 값으로 도출하는 BinaryOperator)

**collect와 reduce**

educe메서드는 다음과 같이 collect메서드에서 toList()로 컬렉트하는 것과 같은 기능을 구현할 수 있다.

```java
List<Integer> numbers = stream.reduce(
		new ArrayList<Integer>(),
		(List<Integer> l, Integer e) -> {
			l.add(e);
			return l;
		},
		(List<INteger> l1, List<Integer> l2 -> {
			l1.addAll(l2);
			return l1;
		});
```

reduce가 collect의 기능을 감당할 수 있음에도 collect가 존재하는 이유는?

collect메서드는 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드인 반면,
reduce는 두 값을 하나로 도출하는 불변형 연산이라는 점에서 의미론적인 문제가 일어난다.

즉 위 예제에서 reduce 메서드는 누적자로 사용된 리스트를 변환시키므로 reduce를 잘못 활용한 예시에 해당한다.

reduce메서드를 잘못 사용하면 실용적인 문제도 발생한다.
여러 스레드가 동시에 같은 데이터 구조체를 고치면 리스트 자체가 망가져 버리므로 리듀싱 연산을 병렬로 수행할 수 없다는 점도 문제이다. 이 문제를 해결하려면 매번 새로운 리스트를 할당해야 하고 이는 성능 저하로 직결된다.

가변 컨테이너 관련 작업이면서 병렬성을 확보하기 위해서는 collect메서드로 리듀싱 연산을 구현하는 것이 바람직하다.

**컬렉션 프레임워크의 유연석 : 같은 연산도 다양한 ㅂ아식으로 수행할 수 있다.**

reducing 컬렉터를 사용한 예제에서 람다 표현식 대신 Integer클래스의 sum메서드 참조를 이용하면 코드를 더 단순화할 수 있다.

```java
menu.stream().collect(reducing(0, //초깃값
							Dish::getCalories, //변환 함수
							Integer::sum)); //합계 함수
```

counting 컬렉터도 reducing 팩토리 메서드를 사용하여 아래와 같이 구현할 수 있다.

```java
public static <T> Collector<T, ?, Long> counting(){
	return reducing(0L, e -> 1L, Long::sum);
}
```

**제네릭 와을드카드 ‘?’ 사용법**

웨에서 countng 팩토리 메서드가 반환하는 컬렉터 시그니처의 두 번째 제네릭 형식으로 와일드카드 ?이 사용되었다. ?는 컬렉터의 누적자 형식이 알려지지 않았음을, 즉 누적자의 형식이 자유로움을 의미한다.

**자신의 상황에 맞는 최적의 해법 선택**

함수형 프로그래밍에서는 하나의 연산을 다양한 방법으로 해결할 수 있다.

문제를 해결할 수 있는 다양한 해결 방법을 확인한 다음에 가장 일반적으로 문제에 특화된 해결책을 고르는 것이 바람직하다. 가독성과 성능을 챙기자. 예시로 **자동 언박싱**연산을 피할 수 있는 IntStream을 사용하는 경우가 있다.

## 6.3 그룹화

자바 8의 함수형을 이용하면 가독성 있는 코드로 그룹화를 구현할 수 있다.

다음과 같이 팩토리 메서드 Collectors.groupingBy를 이용하여 그룹화할 수 있다.

```java
Map<Dish.Type, List<Dish>> dishesByType = 
	menu.stream().collect(groupingBy(Dish::getType));
```

그룹핑된 map의 결과 예시는 다음과 같다.

```java
{
	FISH=[prawns, salmon],
	OTHER=[rice, pizze],
	MEAT=[pork, beef, chicken]
}
```

스트림 각 요소에서 Dish.Type과 일치하는 모든 요리를 추출하는 함수를 groupingBy 메서드로 전달했다. 이 함수를 기준으로 스트림이 그룹화되므로 이를 **분류함수**라도 부른다.

그룹화 연산은 그 결과로

- 그룹화 함수가 반환하는 키
- 각 키에 대응하는 스트림의 모든 항목 리스트

를 갖는 맵이 반환된다.

단순한 속성 접근자 대신 더 복잡한 분류 기준이 필요한 상황에서는 메서드 참조를 분류 함수로 사용할 수 없다.

예로 400 칼로리 이하를 ‘diet’, 400~700은 ‘normal’, 700초과는 ‘fat’이라는 키로 분류하고 싶어도 Dish클래스에는 이러한 연산에 필요한 메서드가 없으므로 메서드 참조를 분류 함수로 사용할 수 없다.

따라서 위와 같이 복잡한 기준이 필요한 경우에는 메서드참조 대신 람다 표현식으로 필요한 로직을 구현할 수 있다.

```java
public enum CaloricLevel { DIET, NORMAL, FAT}
	
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream()
	.collect(groupingBy(dish -> {
		if(dish.getCalories() <= 400) return CaloricLevel.DIET;
		else if(dish.getCalories() <= 700) return CaloricLevel.NORMAL;
		else return CaloricLevel.FAT;
	}));
```

### 6.3.1 그룹화된 요소 조작

요소를 그룹화 한 다음에는 각 결과 그룹의 요소를 조작하는 연산이 필요하다.

예를 들어 500칼로리가 넘는 요리만 필터한다고 가정하면 아래와 같이 filter메서드에 프레디케이트로 필터링 할 수 있다.

```java
Map<Dish.Type, List<Dish>> dishesByType =
	menu.stream()
	.filter(dish -> dish.getCalories() > 500)
	.collect(groupingBy(Dish::getType));
```

문제는 해결이 되지만 단점이 존재한다.

만약 특정 키의 모든 요소가 filter메서드에 의해 필터되는 경우 결과 맵에서 해당 키 자체가 사라지게 된다.

```java
{
	//FISH라는 key가 존재조차 하지 않음
	OTHER=[rice, pizze],
	MEAT=[pork, beef, chicken]
}
```

위 문제를 두 번째 인수를 갖는 오버로드된 groupingBy메서드로 해결할 수 있다.

```java
Map<Dish.Type, List<Dish>> dishesByType =
	menu.stream()
	.collect(groupingBy(Dish::getType),
					filtering(dish -> dish.getCalories() > 500, toList())_);
```

위의 filtering 메서드는 collectors의 정적 팩토리 메서드로 프레디 케이트를 인수로 받는다. 이 프로디케이트로 각 그룹의 요소와 필터링된 요소를 재그룹화 한다. 따라서 value가 없는 key도 반환된다.

```java
{
	FISH=[] //요소가 없더라도 key가 포함됨
	OTHER=[rice, pizze],
	MEAT=[pork, beef, chicken]
}
```

---

그룹화된 항목을 조작하는 유용한 기능 중 또 다른 하나로
맵핑 함수를 이용해 요소를 변환하는 작업이 있다.

filtering 컬렉터와 같은 이유로 Collectors 클래스는 매핑 함수와 각 항목에 적용한 함수를 모으는 데 사용하는 또 다른 컬렉터를 인수로 받는 mapping 메서드를 제공한다.

이 함수를 이용하여 그룹의 각 요리를 관련 이름 목록으로 변환하는 코드는 아래와 같다.

```java
Map<Dish.Type, List<String>> dishesByType =
	menu.stream()
	.collect(groupingBy(Dish::getType), mapping(Dish::getName, toList());
```

---

groupingBy와 연계하여 일반 맵이 아닌 flatMap 변환을 수행할 수 있다.

다음처럼 태그 목록을 가진 각 요리로 구성된 맵이 있을 경우

```java
Map<String, List<String>> dishTags = new HashMap<>();
```

flatMapping 컬렉터를 이용하면 각 형식의 요리 태그를 추출할 수 있다.

```java
Map<Dish.Type, Set<String>> dishNamesByType = 
	menu.stream()
	.collect(groupingBy(Dish::getType,
		flatMapping(dish -> dishTags.get(dish.getName()).stream(), toSet())));
```

1. 각 요리에서 태그 리스트를 얻고
2. 두 수준의 리스트를 한 수준으로 평면화하기 위해 flatMap을 수행한다.
3. 평면화하여 반환된 값엔 중복이 있을 수 있으므로 Set으로 그룹화한다.

### 6.3.2 다수준 그룹화

두 인수를 받는 팩토리 메서드 Collectors.groupingBy를 이용해서 항목을 다수준으로 그룹화할 수 있다. Collectors.groupingBy는 일반적인 분류 함수와 컬렉터를 인수로 받는다. 즉 바깥쪽 groupingBy 메서드에 스트림의 항목을 분류할 두 번째 기준을 정의하는 groupingBy를 전달해서 두 수준으로 스트림의 항목을 그룹화 할 수 있다.

ex)

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = 
	menu.stream()
	.collect(groupingBy(Dish::getType, //첫 번째 수준의 분류 함수
						groupingBy(dish -> { //두 번째 수준의 분류 함수
							if(dish.getCalories() <= 400)
								return CaloricLevel.DIET;
							else if(dish.getCalories() <= 700)
								return CaloricLevel.NORMAL;
							else 
								return CaloricLevel.FAT;
						})
					})
				)
			);
```

groupingBy 연산을 버킷(bucket) 개념으로 생각하면 된다

컷 번째 groupingBy는 각 키의 버킷을 반든다.
그리고 준비된 각각의 버킷을 서브스트림 컬렉터로 채워가기를 반복하면서 n수준의 그룹화를 달성한다.

### 6.3.3 서브그룹으로 데이터 수집

첫 번째 groupingBy로 넘겨주는 컬렉터의 형식은 제한이 없다. 예로 다음 코드처럼 groupingBy컬렉터에 두 번째 인수로 counting 컬렉터를 전달하여 메뉴에서 요리의 수를 종류별로 계산할 수 있다.

```java
Map<Dish.Type, Long> typesCount = menu.stream()
																			.collect(groupingBy(Dish::getType, counting()));
```

분류 함수 한 개의 인수를 갖는 groupingBy(f)는 groupingBy(f, toList())의 축약형이다.

요리의 종류를 분류하는 컬렉터로 메뉴에서 가장 높은 칼로리를 가진 요리를 찾는 코드는 다음과 같다.

```java
Map<Dish.Type, OIptional<Dish>> mostCaloricByType = 
	menu.stream()
			.collect(groupingBy(Dish::getType,
														maxBy(comparingInt(Dish::getCalories))));
```

maxBy는 값이 Optional이라 value로 Optional값이 들어가게 되었다.

하지만 위 예시에서 요리는 Optional.empty()를 값으로 갖는 요리는 존재하지 않는다.

처음부터 존재하지 않는 요리의 키는 맵에 추가되지 않기 때문이다.

groupingBy컬렉터는 스트림의 첫 번째 요소를 찾은 이후에야 그룹화 맵에 새로운 키를(게으르게) 추가한다. 따라서 리듀싱 컬렉터가 반환하는 형식을 사용하는 상황이므로 굳이 Optional 래퍼를 사용할 필요가 없다.

Optional로 감쌀 필요가 없으므로 Optional을 삭제할 수 있다.
팩터리 메서드 Collectors.collectiingAndThen으로 컬렉터가 반환한 결과를 다른 형식으로 활용할 수 있다.

```java
Map<Dish.Type, Dish> mostCaloricByType = 
	menu.stream()
			.collect(groupingBy(Dish::getType,
													collectingAndThen(
														maxBy(comparingInt(Dish::getCalories)), //컬렉터
														Optional::get) //변환함수
												 )
							);
```

동작과정은 다음과 같다.

- groupingBy로 인해 Dish.Type을 기준으로 각각의 서브스트림으로 그룹화된다.
- groupingBy 컬렉터는 collectingAndThen 컬렉터를 감싼다. 두 번째 컬렉터는 그룹화된 각각의 서브스트림에 적용된다.
- collectingAndThen 컬렉터는 세 번째 컬렉터 maxBy를 감싼다.
- maxBy컬렉터가 서브스트림에 연산을 수행한 결과(칼로리가 가장 높은 요리)에 Optional::get 변환함수가 적용된다.
- groupingBy로 인해 생긴 Dish.Type 키 분류에 값이 대응된다.

**groupBy와 함께 사용하는 다른 컬렉터 예제**

일반적으로 스트림에서 같은 그룹으로 분류된 모든 요소에 리듀싱 작업을 수행할 때는
팩토리 메서드 groupingBy에 두번째 인수로 전달된 컬렉터를 사용한다.

예로 메뉴에 있는 모든 요리의 칼로리 함계를 구혀로고 만든 컬렉터를 다음과 같이 groupingBy의 두 번째 인수로 전달할 수 있다.

```java
Map<Dish.Type, Integer> totalCaloriesByType = 
	menu.stream()
	.collect(groupingBy(Dish::getType,
				summingInt(Dish::getCalories)));
```

다양한 형식의 값이 필요할 때 mapping 컬렉터를 사용할 수도 있다.

```java
Map<Dish.Type, Set<CaloricLevel>> caloricLevelsByType = 
	menu.stream().collect(
			groupingBy(Dish::getType, mapping(dish ->{
				if(dish.getCalories() <= 400)
					return CaloricLevel.DIET;
				else if(dish.getCalories() <= 700)
					return CaloricLevel.NORMAL;
				else 
					return CaloricLevel.FAT; },
			toSet() ))));
```

위 예시는 Set의 형식이 정해져있지 않다.
toCollection을 이용하면 원하는 방식으로 결과를 제어할 수 있따.

아래는 HashSet으로 결과를 얻어내는 예이다.

```java
Map<Dish.Type, Set<CaloricLevel>> caloricLevelsByType = 
	menu.stream().collect(
			groupingBy(Dish::getType, mapping(dish ->{
				if(dish.getCalories() <= 400)
					return CaloricLevel.DIET;
				else if(dish.getCalories() <= 700)
					return CaloricLevel.NORMAL;
				else 
					return CaloricLevel.FAT; },
			toCollection(HashSet::new) ))));
```

## 6.4 분할

분할은 **분할 함수**라 불리는 프레디케이트를 분류 함수로 사용하는 그룹화 기능.

분할함수는 불리언을 반환한다. 따라서 맵의 키 형식은 Boolean.
따라서 맵은 최대 두 개의 그룹으로 분류된다. (boolean ⇒ 참, 거짓)

ex)

```java
Map<Boolean, List<Dish>> partitionedMenu = 
	menu.stream().collect(partitioningBy(Dish::isVegetarian)); //분할함수
```

결과는 boolean에 해당하는 키 값에 매핑된다.

```java
{
	false=[pork, beef ...],
	true=[rice, fruit...]
}
```

### 6.4.1 분할의 장점

분할함수가 반환하는 참, 거짓 두가지 요소의 스트림 리스트를 모두 유지한다는 것이 분할의 장점.

filter연산은 하나의 boolean값에 대한 요소만 얻게됨

**오버로드된 버전의 partitioningBy**

컬렉터를 두 번째 인수로 전달받는 partitioningBy 메서드도 존재한다.

```java
Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType = menu.stream()
	.collect(
		// (분할함수, 컬렉터) 
		partitioningBy(Dish::isVegetarian, groupingBy(Dish::getType)));
```

오버로드된 partitioningBy를 사용하여 각 분할그룹에서의 특정 요소를 갖도록 구현할 수 있다.

ex) 채식요리와 채식이 아닌 요리 중 각 요리그룹에서의 칼로리가 가장 높은 요리

```java
Map<Boolean, Dish> mostCaloricPartitionedByVegetarian = 
	menu.stream().collect(
		partitioningBy(Dish::isVegetarian,
			collectingAndThen(maxBy(comparingInt(Dish::getCalories)), Optional::get)));
```

### 6.4.2 숫자를 소수와 비소수로 분할하기

정수 n을 인수로 받아서 2에서 n까지의 자연수를 소수와 비소수로 나누는 프로그램

1. 자연수가 소수인지 판단하는 프레디케이트 구현하기
   2부터 (candidate-1)까지 순회하며 값을 검사

```java
public boolean isPrime(int candidate){
	return InsStream.range(2, candidate)
		// 스트림의 모든 정수로 candidate를 나눌 수 없으면 참을 반환.
		.noneMatch(i -> candidate % i == 0);
}
```

소수판별은 해당 수의 제곱근 이하까지의 수로만 검사해도 된다

```java
public boolean isPrime(int candidate){
	int candidateRoot = (int) Math.sqrt((double)candidate);

	return InsStream.range(2, candidateRoot)
		.noneMatch(i -> candidate % i == 0);
}
```

1. n까지의 숫자를 포함하는 스트림 생성 후 isPrime프레디케이트 사용

```java
public Map<Boolean, List<Integer>> partitionPrimes(int n){
	return IntStream.rangeClosed(2, n).boxed()
					.collect(
						partitioningBy(candidate -> isPrime(candidate));
}
```

Collectors의 정적 팩토리 메서드로 List, Set, Map.. 등의 다양한 형식으로 스트림 요소를 모을 수 있다. 모든 컬렉터는 Collector 인터페이스를 구현한다.

정적 팩토리 메소드 외의 집합이 필요한 경우 커스텀 컬렉터를 구현할 수 있다.

## 6.5 Collector 인터페이스

Collector 인터페이스는 리듀싱연산을 어떻게 구현할지 제공하는 메서드 집합으로 구성된다.

Collector 인터페이스를 구현하는 리듀싱 연산을 만들어서 요소를 수집할 수 있지만 Collector 인터페이스를 직접 구현해서 더 효율적으로문제를 해결하는 컬레거를 만들 수 있따.

**Collector 인터페이스**

```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator()
    BinaryOperator<A> combiner();
    Function<A, R> finisher();
    Set<Characteristics> characteristics();
```

T : 수집될 스트림 항목의 제네렉 형식

A : 누적자. 수집 과정에서 중간 결과를 누적하는 객체 형식

R : 수집 연산 결과 객체의 형식(보통 컬렉션)

ex) Stream<T>의 모든 요소를 List<T>로 수집하는 ToListCollector<T> 클래스 구현 예

```java
public class ToListCollector<T> implements Collector<T, List<T> List<T>>
```

누적과정에서 사용되는 객체가 수집 과정의 최종 결과로 사용된다.

### 6.5.1 Collector 인터페이스 메서드 사렾보기

characteristics 메서드는 collect 메서드가 어떤 최적화(ex : 병렬화)를 이용해서 리듀싱 연산을 수행할 것인지 결정하도록 돕는 힌트 특성 집합을 제공

그 외 4개의 메서드는 collect메서드에서 실행하는 함수를 반환하는 메서드

**supplier 메서드 : 새로운 결과 컨테이너 만들기**

supplier 메서드는 빈 결과로 이루어진 Supplier를 반환해야 한다.
즉, supplier는 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수이다.

ToListCollector처럼 누적자를 반환하는 컬렉터에서는 빈 누적자가 비어있는 스트림의 수집 과정의 결과가 될 수 있다. (요소가 없으면 빈 누적자를 반환하면 되니까)

ToListCollector에서의 supplier는 다음처럼 빈 리스트를 반환한다.

```java
public Supplier<List<T>> supplier(){
	return () -> new ArrayList<T>;
}

//생성자 참조 반환도 가능
public Supplier<List<T>> supplier(){
	return ArrayList::new;
}
```

**accumulator 메서드 : 결과 컨테이너에 요소 추가하기**

accumulator 메서드는 리듀싱 연산을 수행하는 함수를 반환한다.
스트림에서 n번째 요소를 탐색할 때 두 인수, 즉 누적자(스트림의 첫 n-1개 항목을 수집한 상태)와 n번째 요소를 함수에 적용한다.

함수의 반환값은 void, 요소를 탐색하면서 적용하는 함수에 의해 누적자 내부 상태가 바뀌므로 누적자가 어떤 값일지 단정할 수 없다. (??? mapping 함수에 의해 요소 타입이 바뀌는 경우를 말하는건지?)

ToListCollector에서의 accumulator는 이전에 탐색한 항목을 포함하는 리스트에 현재 항목을 추가한다.

```java
public BiConsumer<List<T>, T> accumumlator(){
	return List::add;
}
```

**finisher 메서드 : 최종 반환값을 결과 컨테이너로 적용하기**

finisher 메서드는 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환하면서 누적 과정을 끝낼 때 호출할 함수를 반환한다.

때로는 누적자 객체가 이미 최종 결과인 상황도 있다.(요소가 없는 경우?)
이련경우는 변환과정이 불필요하므로 finisher메서드는 항등 함수를 반환한다.

```java
public Function<List<T>, List<T>> finisher(){
	retrun Function.identity()
}
```

**combiner 메서드 : 두 결과 컨테이너 병합**

combiner는 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할지 정의한다.

toList의 combiner를 예시로 들면 스트림의 두 번째 서브 파트에서 수집한 항목 리스트를 첫 번째 서브파트 결과 리스트의 뒤에 추가하면 된다.

```java
public BinaryOperator<List<T>> combiner(){
	return (list1, list2) ->{
		list1.addAll(list2);
		return list1;
	}
}
```

combiner메서드를 사용하여 스트림의 리듀싱을 병렬로 수행할 수 있다.

스트림의 리듀싱을 병렬로 수행할 때 자바 7의 포크/조인 프레임워크와 Spliterator를 사용한다.

스트림의 병렬 리듀싱 수행 과정

1. 스트림을 분할해야 하는지 정의하는 조건이 거짓으로 바뀌기 전까지 원래 스트림을 재귀적으로 분할한다. (분산 작업의 크기가 너무 작아지면 순차수행보다 느려질 수 있다. 프로세싱 코어의 개수를 초과하는 병렬 작업은 보통 비효율적이다)
2. 모든 서브스트림의 각 요소에 리듀싱 연산을 순차적으로 적용해서 서브스트림을 병렬로 처리할 수 있다.
3. combiner메서드가 반환하는 함수로 모든 부분결과를 쌍으로 합친다.
   모든 서브스트림의 결과를 합치면 연산이 완료된다

**Characteristics 메서드**

Characteristics메서드는 컬렉터의 연산을 정의하는 Characteristics형식의 불변 집합을 반환한다.

Characteristics는 스트림을 병렬로 리듀스할 것인지, 병렬로 리듀스한다면 어떤 최적화를 선택해야 할지 힌트를 제공한다.

Characteristics는 다음 세 항목을 포함하는 열거형이다.

- UNORDERED : 리두싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않는다.
- CONCURRENT : 다중 스레드에서 accumulator함수를 동시에 호출할 수 있으며 이 컬렉터는 스트림의 병렬 리듀싱을 수행할 수있다. 컬렉터의 플래스에 UNORDERED를 함께 설정하지 않았다면 데이터소스가 정렬되어있지 않은 상황(요소의 순서에 의미가 없는 상황)에서만 병렬 리듀싱을 수행할 수 있다.
- IDENTITY_FINISH : finisher 메서드가 반환하는 함수는 단순히 identity를 적용할 뿐이므로 이를 생략할 수 있다. 따라서 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용할 수 있다.
  또한 누적자 A를 결과 R로 안전하게 형변환할 수 있다. (?)

ToListCollector 예시에서 스트림의 요소를 누적하는데 사용한 리스트가 최종 결과 형식이므로 추가 변환이 필요없다. 따라서 ToListCollector는 IDENTITY_FINISH이다.

하지만 리스트의 순서는 상관이 없으므로 UNORDERED이다.

마지막으로 ToListCollector는 CONCURRENT이다.

요소의 순서가 무의미한 데이터소스여야 병렬로 실행할 수 있다.

### 6.5.2 응용하기

ex) 커스텀 ToListCollector 구현

```java
import ...

public class ToListCollector<T> implements Collector<T, List<T>, List<T>> {
	@Override
	public Supplier<List<T>> supplier(){
		return ArrayList::new;
	}

	@Override
	public BiConsumer<List<T>, T> accumulator(){
		return List:add;
	}

	@Override
	public Function<List<T> ,List<T>> finisher(){
		return Function.identity(); // 항등함수
	}
	
	@Override
	public BinaryOperator<List<T>> combiner(){
			return (list1, list2) -> {
			list1.addAll(list2); //두 번째 콘텐츠를 첫번째 누적제하 합치기
			return list1; //첫번째 누적자 반환
		};
	}

	@Override
	public Set<Characteristics> characterisctics(){
		//컬렉터의 플레그 설정. 컬렉터가 어떻게 연산될 것인지에 대한 힌트 개념
		return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH, CONCURRENT));
	}
}
	
```

```java
//자바 컬렉터 팩토리 사용
List<Dish> dishes = menuStream.collect(new ToListCollector<Dish>());

//커스텀 컬렉터 사용
List<Dish> dishes = menuStream.collect(toList());
```

**켈럭터 구현체를 만들지 않고 커스텀 수집 수행하기**

IDENTITY_FINISH 수집 연산에서는 Collector 인터페이스를 새로 구현하지 않고도 같은 결과를 얻을 수 있다.

Stream은 세 함수(발행, 누적, 합침)를 인수로 받는 collect메서드를 오버로드하며 각각의 메서드는 Collector 인터페이스의 메서드가 반환하는 함수와 같은 기능을 수행한다

ex)

```java
List<Dish> dishes = menuStream.collect(
	ArrayList::new, //누적자 발행
	List::add, //누적
	List::addAll); //병합
```

코드가 간결하지만 가독성이 떨어진다.

또한 Characteristics를 전달할 수 없다.(IDENTITY_FINISH와 CONCURRENT지만 UNORDERED는 아닌 컬렉터로만 동작)

## 6.6 커스텀 컬렉터를 구현하여 성능 개선하기

아래와 같이 자연수를 소수와 비소수로 분할하는 컬렉터를 커스텀 컬렉터로 만들기

```java
public Map<Boolean, List<Integer>> partitionPrimes(int n){
	return IntStream.rangeClosed(2, n).boxed()
					.collect(
						partitioningBy(candidate -> isPrime(candidate));
}
```

### 6.6.1 소수로만 나누기

제수(devisor)가 소수가 아니라면 나누어서 검사를 하는 의미가 없다.
따라서 제수를 현재 숫자 이하에서 발견된 소수로 제한한다.

이 때 주어진 숫자가 소수인지 아닌지 판단해야한다.

지금까지 발견한 소수 리스트에 접근해서 알 수 있다.

하지만 위의 컬렉터 수집과정에서는 수집된 소수의 부분결과에 접근할 수 없기 때문에 커스텀 컬렉터로 이 문제를 해결한다.

중간 결과리스트가 있다면 isPrime 메서드로 중간 결과 리스트를 전달하도록 다음과 같이 코드구현 가능 ⇒ 소수 list를 파라미터로 추가

```java
public static boolean isPrime(List<Integer> primes, int candidate){
	return primes.stream().noneMatch(i -> candidate % i == 0);
}
```

검사 대상의 제곱근보다 작은 소수만 사용하게 할 수 있다.
filter는 쇼트서킷이 아니므로 takeWhile을 사용하여 최적화한다.

```java
public static boolean isPrime(List<Integer> primes, int candidate){
	int candidateRoot = (int)Math.sqrt((double) candidate);
	return primes.stream()
						.takeWhile(i -> i<= candidateRoot)
						.noneMatch(i -> candidate % i == 0);
}
```

### 커스텀 컬렉터의 구현

### 1. Colllector 클래스 시그니처 정의

public interface Collector<T, A, R> 형식에 맞춰서 클래스 시그니처 정의

T : 스트림 요소의 형식

A : 중간 결과를 누적하는 객체 형식

R : collect 연산의 최종 결과 형식

ex) 소수를 판별하여 분할하기 위한 컬렉터 정의 예시

```java
public class PrimeNumbersCollector 
	implements Collector<Integer, Map<Boolean, List<Integer>>, Map<Boolean, List<Integer>>>
```

### 2. 리듀싱 연산 구현

supplier 메서드는 누적자를 만드는 함수를 반환하도록 구현해야 한다.

```java
public Supplier<Map<Boolean, List<Integer>>> supplier(){
	return () -> new HashMap<Boolean, List<Integer>>(){{
		put(true, new ArrayList<Itneger>());
		put(false, new ArrayList<Integer>());
	}};
}
```

누적자로 사용할 맵을 만듦과 동시에 true, false 키와 빈 배열로 초기화한다.

accumulator 메서드에는 요소를 어떻게 수집할 것인지 정의해야 한다

```java
public BiConsumer<Map<Boolean, List<Integer>>, Integer> accumulator(){
	return (Map<Boolean, List<Integer>> acc, Integer candidate) ->{
		acc.get( isPrime(acc.get(true), candidate) )
			.add(candidate);
	};
};
```

소수만 들거있는 true값의 value를 isPrime메서드에 전달하여 제수로 소수만 사용할 수 있다

### 3. 뱡랼 실행할 수 있는 컬렉터 만들기

병렬 수집과정에서 두 부분 누적자를 합칠 수 잇는 메서드 만들기

```java
public BinaryOperator<Map<Boolean, List<Integer>>> combiner() {
	return (Map<Boolean, List<Integer>> map1, Map<Boolean, List<Integer>> map2) ->{
		map1.get(true).addAll(map2.get(true));
		map1.get(false).addAll(map2.get(false));
		return map1;
	};
}
```

### 4. finisher 메서드와 컬렉터의 characteristics 메서드

accumulator의 형식이 컬렉터 결과 형식과 같으므로 변환 과정이 필요 없을 때에는 항등 함수 identity를 반환하도록 finisher메서드를 구현한다

```java
public Function<Map<boolean, List<Integer>>,
	Map<Boolean, List<Integer>>>> finisher(){
	return Function.identity();
}
```

## 6.7 마치며

- collect는 스트림의 요소를 요약결과로 누적하는 다양한방법(컬렉터)을 인수로 갖는 최종 연산
- 많은 컬렉터들이 미리 정의되어 있음
- stream 요소를 groupingBy로 그룹화, partitioningBy로 분할 가능
- Colelctor 인터페이스를 구현하여 커스텀 컬렉터 생성 가능