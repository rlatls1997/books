# Chapter 5. 스트림 활용
**내용**

- 필터링, 슬라이싱, 매칭
- 검색, 매칭, 리듀싱
- \특정 범위의 숫자와 같은 숫자 스트림 사용하기
- 다중 소스로부터 스트림 만들기
- 무한 스트림

스트림 API가 지원하는 다양한 연산을 살펴본다.

## 5.1 필터링

### 5.1.1 프레디케이트로 필터링

filter메서드는 프레디케이트를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환한다.

- 프레디케이트를 인수도 받아 필터링하는 예

```java
List<Dish> vegetarianMenu = menu.stream()
		.filter(Dish::isVegetarian)
		.collect(toList());
```

### 5.1.2 고유 요소 필터링

고유 요소로 이루어진 스트림을 반환하는 distinct메서드를 지원한다.
고유 여부는 스트림에서 만든 객체의 hashcode, equals로 결정된다.

- distinct활용 예

```java
numbers.stream()
	.filter(i -> i % 2 == 0)
	.distinct()
	.forEach(system.out::println);
```

중복값을 걸러준다.

## 5.2 스트림 슬라이싱

스트림 요소를 선택하거나 스킵하는 방법을 배운다.

### 5.2.1 프레디케이트를 이용한 슬라이싱

자바 9부터 스트림 요소를 효과적으로 선택할 수 있도록 takeWhile, dropWhile 두 개의 새 메서드를 지원한다.

**TakeWhile활용**

만약 320칼로리 이하의 요리를 선택한다고 가정하면 아래와 같이 filter메서드를 떠올릴 수 있다.

```java
spectialMenu.stream()
	.filter(dish -> dish.getCalories() < 320)
	.collect(toList());
```

위의 sepcialMenu가 이미 칼로리순으로 정렬되어 있다고 하자.

filter연산을 이용하면 전체 스트림을 반복하면서 각 요소에 프레디케이트를 적용하게 된다. 따라서 리스트가 이미 정렬되어 있다는 사실을 이용하면 320칼로리 이상인 요리가 나왓을 때 반복 작업을 중단할 수 있을 것 같다.

위와 같이 조건대상으로 이미 정렬된 스트림의 반복 작업을 중단하려면 어떻게 하는가? ⇒ takeWhile 연산을 이용한다.

```java
spectialMenu.stream()
	.takeWhile(dish -> dish.getCalories() < 320)
	.collect(toList());
```

칼로리가 320보다 작은 동안 dish를 가져온다고 생각하면 되겠다.

takeWhile을 이용하면 무한 스트림을 포함한 모든 스트림에 프레디케이트를 적용해서 스트림을 슬라이스할 수 있다.

**DropWhile 활용**

위처럼 조건을 대상으로 정렬된 스트림의 나머지 요소를 선택하려면? ⇒ dropWhile

```java
spectialMenu.stream()
	.dropWhile(dish -> dish.getCalories() < 320)
	.collect(toList());
```

칼로리가 320보다 작은 동안은 drop한다고 생각하면 되겠다.

프레디케이트가 거짓이 되면 그 지점에서 작업을 중단하고 남은 모든 요소를 반환한다.

dropWhile도 마찬가지로 무한스트림에서도 동작한다.

### 5.2.2 스트림 축소

주어진 값 이하의 크기를 갖는 새로운 스트림을 반환할 수 있다 ⇒ limit(n) 메서드 사용

최대 n개의 요소를 반환하게 만들 수 있다.

```java
spectialMenu.stream()
	.filter(dish -> dish.getCalories() < 320)
	.limit(3)
	.collect(toList());
```

스트림으로 전달되는 선착순 3까지의 요소를 반환한다.

### 5.2.3 요소 건너뛰기

처음 n개 요소를 제외한 스트림을 반환하는 경우 ⇒ skip(n) 메서드 사용

만약 스트림의 개수가 n개 이하라면 빈 스트림이 반환된다.

```java
spectialMenu.stream()
	.filter(dish -> dish.getCalories() < 320)
	.skip(2)
	.collect(toList());
```

## 5.3 매핑

특정 객체에서 특정 데이터를선택하는 작업에서 매핑이 필요하다.

### 5.3.1 스트림의 각 요소에 함수 적용하기

스트림은 함수를 인수로 받는 amp메서드를 지원한다.

인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑된다. 과정이 기존의 값을 고친다는 개념보단 새로운 버전을 만든다는 개념에 가까우므로 매핑이라는 단어를 사용한다.

```java
menu.stream()
	.map(Dish::getName)
	.collect(toList());
```

### 5.3.2 스트림 평면화

[”hello”,”world”]라는 리스트가 있을 경우
[”h”,”e”,”l”,”o”,”w”,”r”,”d”]처럼 고유문자로 이루어진 리스트를 반환하는 경우를 생각해보자.

다음과 같이 매핑을 생각할 수 있다.

```java
words.stream()
	.map(word -> word.split(""))
	.distinct()
	.collect(toList());
```

결과는 어떻게 될까?

1. 리스트의 각 원소가 한 문자로 나눠진 배열로 매핑된다.
   [”h”,”e”,”l”,”l”,”o”], [”w”,”o”,”r”,”l”,”d”]
2. 스트링 배열 스트림이 반환되고 distint()가 수행되므로 걸러지는 내용이 없다.
3. 최종연산에서 list로 collect하면 문자열 배열 리스트가 반환된다.

예상 결과와 다르다.

**map과 [Arrays.stream](http://Arrays.stream) 활용**

위와 같은 경우 매핑 결과의 평탄화를 위해 반환된 배열을 다시 스트림으로 매핑하는 것을 생각해볼 수 있다.

```java
words.stream()
	.map(word -> word.split("")) //문자열 배열 반환
	.map(Arrays::stream) //각 배열을 별도의 스트림으로 매핑
	.distinct()
	.collect(toList());
```

하지만 위는 문자열 stream의 stream이 반환되기 때문에(각 문자열 배열의별도의 스트림) distinct()에서 원하는 중복값을 거르지 못할 뿐더러 최종 결과물은 List<Stream<String>>이다.

**flatMap활용**

flatMap은 각 배열을 스트림이 아닌 스트림의 콘텐츠로서 매핑한다. map(Array::stream)과 달리 flatMap은 개별적이지 않은 하나의 스트림으로 스트림요소를 평탄화해준다.

```java
words.stream()
	.map(word -> word.split(""))
	.flatMap(Arrays::stream) // 생성된 스트림을 하나의 스트림으로 평탄화
	.distinct()
	.collect(toList());
```

## 5.4 검색과 매칭

특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리.
allMatch, anyMatch, nonMatch, findFirst, findAny 등의 유틸리티 메서드를 제공한다.

### 5.4.1 프레디케이트가 적어도 한 요소와 일치하는지 확인

anyMatch 메서드를 이용한다.

```java
if(menu.stream().anyMatch(Dish::isVegetarian)){
	...
}
```

### 5.4.2 프레디케이트가 모든 요소와 일치하는지 검사

allMatch 메서드를 이용한다.

```java
boolean isHealthy = menu.stream()
											.allMatch(dish -> dish.getCalories() < 1000);
```

**noneMatch**

allMatch와 반대 연산을 수행한다.
즉, 주어진 프레디케이트와 일치하는 요소가 없는지 확인한다.

```java
boolean isHealthy = menu.stream()
											.nonMatch(dish -> dish.getCalories() >= 1000);
```

anyMatch, allMatch, noneMatch 세 메서드는 스트림 **쇼트서킷**기법, 즉, 자바의 &&, ||과 같은 연산을 활용한다.

**쇼트서킷 평가**

때론 전체 스트림을 처리하지 않았더라도 결과를 반환할 수 있다. 예시로 여러 and 연산으로 이어진 불리언 표현식을 평가하는 경우 표현식에서 하나라도 거짓이 나오면 나머지 표현식의 결과는 필요가 없어진다. 이런 상황을 **쇼트서킷**이라고 한다.

allMatch, nonMatch, findFirst, findAny등의 연산 또한 조건에 따라 모든 스트림 요소를 처리하지 않고 결과를 반환할 수 있다. limit또한 모든 요소를 처리하지 않고 주어진 크기의 스트림만 처리하므로 쇼트서킷 연산에 해당된다.

### 5.4.3 요소 검색

findAny : 현재 스트림에서 임의의 요소를 반환한다.

```java
Optional<Dish> dishj = menu.stream()
											.filter(Dish::isVegetarian)
											.findAny();
```

filter와 findAny를 이용하여 조건에 해당하는 임의의 요소를 반환할 수 있다.

findAny메서드는 쇼트서킷 연산이다. 결과를 찾는 즉시 스트림을 종료한다.

**Optional**

Optional<T> 클래스는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스이다. findAny 연산이 아무 요소도 반환하지 않을 수 있고 null을 반환하여 NPE를 일으킬 수 있다.

Optional 클래스를 통해 null처리를 확실하게 할 수 있다.

- isPresent()는 Optional이 값을 포함하면 참을 반환하고 포함하지 않으면 거짓을 반환한다.
- ifPresent(Consumer<T> block)은 값이 있으면 주어진 블록을 실행한다.
- T get()은 값이 존재하면 값을 반환하고 값이 없으면 NoSuchElementException을 일으킨다.
- T orElse(T other)는 값이 있으면 값을 반환하고 값이 없으면 기본값을 반환한다.

### 5.4.4 첫 번째 요소 찾기

정렬된 데이터로부터 생성된 스트림처럼 일부 스트림에는 **논리적인 아티템 순서**가 정해져 있을 수 있다. 이런 스트림에서 첫 번째 요소를 찾을 때 findFirst 메서드를 사용한다.

```java
List<Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> firstSquareDivisibleByThree = 
	someNumbers.stream()
		.map(n -> n*n)
		.filter(n -> n % 3 == 0)
		.findFirst(); //9
```

**병렬성과 findFirst, findAny**

이렇게 보면 findFirst와 findAny의 역할이 중복되는 것처럼 보인다.
findFirst와 findAny가 모두 필요한 이유는 뭘까?

병렬성 때문이다. 병렬 실행에서는 첫 번째 요소를 찾기 어렵다. 따라서 요소의 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 findAny를 사용한다.

## 5.5 리듀싱

스트림 요소를 조합해서 더 복잡한 질의를 표현하는 방법.

**리듀싱 연산** : 모든 스트림 요소를 처리해서 값으로 도출하는 연산)

함수형 프로그래밍 언어 용어로는 이 과정이 종이를 작은 조각이 될 때까지 반복해서 접는 것과 비슷하다는 의미로 **폴드**라고 부른다.

### 5.5.1 요소의 합

for-each 루프로 리스트의 숫자 요소를 더하는 코드는 아래와 같다

```java
int sum = 0;
for( int x : numbers) {
	sum += x;
}
```

numbers의 각 요소는 결과에 반복적으로 더해진다. 리스트에서 하나의 숫자가 남을 때까지 reduce과정을 반복한다. 코드에는 파라미터를 두 개 사용했다.

- sum 변수의 초깃값 0
- 리스트의 모든 요소를 조합하는 연산 +

reduce를 사용하면 애플리케이션의 반복되는 패턴을 추상화 할 수 있다.
reduce를 사용하여 스트림의 모든 요소를 더할 수 있다.

```java
int sum = numbers.stream().reduce(0, (a, b) -> a+b);
```

reduce는 두 개의 인수를 갖는다

- 초깃값 0
- 두 요소를 조합해서 새로운 값을 만드는 BinaryOperator<T>. ( (a,b) → a+b )

reduce의 연산과정은 스트림이 하나의 값으로 줄어들 때까지 람다는 각 요소를 반복해서 조합한다.

**초깃값이 없을 경우**

초깃값을 받지 않도록 오버로드된 reduce로 있다.
이 reduce메서드는 Optional객체를 반환한다.

Optional을 반환하는 이유는 스트림에 아무 요소도 없는 상황에서 초깃값까지 없다면 reduce메서드는 결과를 반환할 수 없기 때문이다.

### 5.5.2 최댓값과 최솟값

최댓값과 최솟값을 찾을 때도 reduce를 활용할 수 있다.

```java
Optional<Integer> max = numbers.stream().reduce(Integer::max);
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```

**reduce 메서드의 장점과 병렬화**

기존 for-each로 합계를 구하는 것과 reduce를 이용해서 합계를 구하는 것의 차이는?

- reduce를 이용하면 내부 반복이 추상화되면서 내부 구현에서 병렬로 reduce를 실행할 수 있게된다. 반면 반복적인 합계에서는 sum변수를 공유해야 하므로 쉽게 병렬화하기 어렵다. 강제적으로 동기화하더라도 병렬화로 얻어야할 이득이 스레드 간의 소모적인 경쟁 때문에 생쇄된다.
  가변 누적자 패턴은 병렬화와 거리가 먼 기법이다.

**스트림연산 : 상태 없음과 상태 있음**

스트림을 통해 모든 연산을 쉽게 구현할 수 있고 parallelStream으로 쉽게 병렬성을 얻어낼 수 있다. filter, map, reduce등의 연산을 병렬로 실행할 수 있다. 하지만 이 메서드들은 각각 다양한 연산을 수행하기 때문에 각각의 연산은 내부적인 상태를 고려해야 한다.

map, filter등은 입력스트림에서 각 요소를 받아 0또는 결과를 출력스트림으로 보낸다. 이런 메서드들은 보통 상태가 없는, 즉 내부 상태를 갖이 않는 연산이다.

반면 reduce, sum, max같은 연산은 결과를 누적할 내부 상태가 필요하다. 스트림에서 처리하는 요소의 수와 관계없이 **내부 상태의 크기는 한정되어있다. (int, double등의 데이터 타입 크기로 한정)**

sorted나 distinct같은 연산은 filter나 map처럼 스트림을 입력으로 ㅂ다아 다른 스트림을 출력하는 것처럼 보일 수 있으나 filter와 map과는 다르다. 스트림의 요소를 정렬하거나 중복을 제거하려면 과거의 이력을 알고 있어야 한다. 어떤 요소를 출력 스트림으로 추가하려면 **모든 요소가 버퍼에 추가되어 있어야 한다**
연산을 수행하는데 필요한 저장소 크기는 정해져있지 않다. 따라서 데이터 스트림의 크기가 크거나 무한이라면 문제가 생길 수 있다. 이러한 연산을 **내부 상태를 갖는 연산**이라고 한다.

## 5.6 실전 연습

## 5.7 숫자형 스트림

reduce메서드로 다음과 같이 스트림 요소의 합을 구할 수 있다.

```java
menu.stream()
	.map(Dish::getCalories)
	.reduce(0, Integer::sum);
```

위 코드는 박싱 비용이 숨어있다.
내부적으로 함계를 계산하기 전에 Integer를 기본형으로 언박싱 해야한다.

스트림 API는 숫자 스트림을 효율적으로 처리할 수 있도록 **기본형 특화 스트림**을 제공한다.

### 5.7.1 기본형 특화 스트림

자바 8에서 박싱 비용을 피할 수 있는 세가지 기본형 특화 스트림을 제공한다.

int요소에 특화된 IntStream,

double요소에 특화된 DoubleStream

long요소에 특화된 LongStream을 제공한다.

각각의 인터페이스는 숫자 스트림의 합계를 계산하는 sum, 최댓값을 검색하는 max등 자주 사용되는 숫자 관련 리듀싱 연산 수행 메서드를 제공한다.

특화스트림은 박싱 과정에서 일어나는 효율성과 관련있고 스트림에 추가 기능을 제공하지는 않는다.

**숫자스트림으로 매핑**

mapToInt, mapToDouble, mapToLong 세 가지 메서드를 사용하여 특화스트림으로 변환
map과 같은 역할을 하지만 Stream<T>타입 대신 특화된 스트림을 반환한다.

```java
menu.stream()
	.mapToInt(Dish::getCalories) //IntStream반환
		.sum(); //특화스트림인 IntStream이 반환되었기 때문에 sum 메서드를 사용할 수 있다.
```

**객체스트림 복원하기**

숫자스트림을 만든 뒤 boxed() 메서드를 사용하여 원상태인 특화되지 않은 스트림으로 복원할 수 있다.

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
Stream<Integer> stream = intStream.boxed(); //숫자스트림을 스트림으로 변환
```

**기본값 : OptionalInt**

sum메서드는 기본값이 0이라서 stream요소가 없더라도 문제가 없다.

반면 IntStream에서 최댓값을 찾는 max메서드와 같은 경우에 기본값이 존재한다면 문제가 될 수 있다. 여기서 Optional 개념이 사용된다.

ex)

```java
OptionalInt maxCalories = menu.stream()
														.mapToInt(Dish::getCalories)
														.max();
```

### 5.7.2 숫자 범위

특정 범위의 숫자를 이용해야 하는 경우
IntSteam과 LongStream에서 range와 rangeClosed라는 두 가지 정적 메서드를 사용할 수 있다.

두 메서드 모두 첫 번째 인수는 시작값, 두 번째 인수는 종료값을 갖는다.

range메서드는 시작값과 종료값이 결과에 포함되지 않는다.

rangeClosed는 시작값과 종료값이 결과에 포함된다.

```java
IntStream evenNumbers = IntStream.rangeClosed(1, 100) //1부터 100까지의 int stream생성	
```

### 5.7.3 숫자 스트림 활용 : 피타고라스 수

a^2 + b^2 = c^2

**세 수 표현하기**

배열로 나타내고 인덱스로 접근할 수 있다.

```java
new int[]{3, 4, 5}
```

**좋은 필터링 조합**

a, b, c 세 수 중 a, b 두 수만 알고 있을 때 두 수가 피타고라스 수의 일부가 될 수 있는 조합인지 확인하는 방법

a^2 + b^2의 제곱근이 정수인지 확인한다.

```java
Math.sqrt(a*a + b*b) % 1 == 0;
```

이 식을 filter로 다음처럼 활용할 수 있다.

```java
filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
```

a와 함꼐 피타고라스 수를 구성하는 모든 b를 필터링할 수 있다.

**집합 생성**

필터를 이용해서 좋은 조합을 갖는 a, b를 선택할 수 있게 되었다

이제 마지막 c를 찾아야 한다.

map을 이용해서 각 요소를 피타고라스 수로 변환할 수 있다.

```java
stream.filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
	.map(b-> new int[]{a, b, (int)Math.sqrt(a*a + b*b)});
```

**b 값 생성**

b값을 생성하기 위해 IntStream의 rangeClosed메서드를 사용할 수 있다.

```java
IntStream.rangeClosed(1, 100)
	.filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
	.boxed()
	.map(b-> new int[]{a, b, (int)Math.sqrt(a*a + b*b)});
```

boxed()로 박싱을 해야하는 이유는 IntStream의 map 결과는 int가 반환되길 기대하기 때문에 map을 통해 int가 아닌 다른 형식을 반환하려면 이전에 박싱을 해줘야 한다.

아래처럼 mapToObj를 사용하면 int가 아닌 다른 형식의 값을 리턴할 수 있다.

```java
IntStream.rangeClosed(1, 100)
	.filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
	.mapToObj(b-> new int[]{a, b, (int)Math.sqrt(a*a + b*b)});
```

**a값 생성**

a값을 생성하는 코드를 추가한다.

```java
Stream<int [] > pythagoreanTriples = 
	IntStream.rangeClosed(1, 100).boxed()
	.flatMap( a->
		IntStream.rangeClosed(1, 100)
			.filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
			.mapToObj(b-> new int[]{a, b, (int)Math.sqrt(a*a + b*b)})
	);
```

위 코드로 피타고라스 수에 해당하는 배열 스트림을 얻어낼 수 있다.

**개선**

위 코드는 filter와 mapToObj 메서드에서 제곱근을 두 번 계산한다.

얻어내려는 배열의 2번째 원소는 항상 정수이어야 하므로 이 값을 사용하여 제곱근 계산을 한 번만 수행하도로 ㄱ한다.

```java
Stream<int [] > pythagoreanTriples = 
	IntStream.rangeClosed(1, 100).boxed()
	.flatMap( a->
		IntStream.rangeClosed(1, 100)
			.mapToObj(b-> new int[]{a, b, (int)Math.sqrt(a*a + b*b)})
			.filter(t -> t[2] % 1 == 0));
```

## 5.8 스트림 만들기

stream 메서드로 컬렉션에서 스트림을 얻을 수 있다.
또한 원하는 범위의 수에 해당하는 스트림을 만들수도 있다.

그 외 일련의 값, 배열, 파일, 함수를 이용한 무한 스트림 만들기 등 다양한 방식으로 스트림을 만들 수 있다.

### 5.8.1 값으로 스트림 만들기

임의의 수를 인수로 받는 정적 메서드 Stream.of를 이용해서 스트림을 만들 수 있다.

ex)

```java
Stream<String> stream = Steram.of("Modern ", "Java ", "In ", "Action");
stream.map(String::toUpperCase).forEach(System.out.println); 
```

empty메서드를 사용해서 빈 스트림을 생성할 수 있다.

```java
Stream<String> emptyStream = Stream.empty();
```

### 5.8.2 null이 될 수 있는 객체로 스트림 만들기

자바 9에서 null이 될 수 있는 개체를 스트림으로 만들 수 있는 메서드가 추가됨.

예시로 System.getProperty는 제공된 키에 대응하는 속성이 없다면 null을 반환한다.
이런 메소드를 스트림에 활용하려면 다음처럼 null을 명시적으로 확인해야 했다.

```java
String homeValue = System.getProperty("home");
Stream<String> homeValueStream
	= homeValue == null? Stream.empty() : Stream.of(homeValue);
```

Stream.ofNullable을 이용해서 다음처럼 코드를 구현할 수 있다.

```java
Stream<String> homeValueStream
	= Stream.ofNullable(Syste.getProperty("home"));
```

null이 될 수 있는 객체를 포함하는 스트림값을 flatMap과 함께 사용하는 상황에서 이 패턴을 더 유용하게 사용할 수 있다.

```java
Stream<String> values = 
	Stream.of("config", "home", "user")
		.flatMap(key -> Stream.ofNullable(System.getProperty(key)));
```

### 5.8.3 배열로 스트림 만들기

배열을 인수로 받는 정적메서드 Arrays.stream을 이용하여 스트림을 생성할 수 있다.

```java
int [] numbers = {2, 3, 5, 7, 11};
IntStream intStream = Arrays.stream(numbers)
```

### 5.8.4 파일로 스트림 만들기

파일 처리등의 I/O연산에 사용되는 자바의 NIO API도 스트림 API를 활용할 수 있도록 업데이터 되었다.

java.nio.file.Files의 많은 정적 메서드가 스트림을 반환한다.

예로 files.lines는 주어진 파일의 행 스트림을 문자열로 반환한다.

ex) 파일에서 고유한 단어 수를 찾는 예

```java
long uniqueWords = 0;

try(Stream<String> lines = 
			Files.lines(Paths.get("data.txt"), Charset.defaultCharset()))) {
	uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
										.distinct()
										.count();
} catch(IOException e){
	...
}
```

메모리 누수를 막으려면 자원을 닫아야한다

기존에는 finally 블록에서 자원을 닫았다.

Stream인터페이스는 AutoCloseable인터페이스를 구현하기 때문에 try블록 내의 자원은 자동으로 관리된다.

### 5.8.5 함수로 무한 스트림 만들기

스트림 API는 함수에서 스트림을 만들 수 있는 두 정적 메서드

Stream.iterate
Stream.generate

를 제공한다.

두 연산을 이용해서 **무한스트림, 크기가 고정되지 않은 스트림을 만들 수 있다.**

iterate와 generate에서 만든 스트림은 요청할때마다 주어진 함수를 이용해서 값을 만든다.
보통 무한한 값을 출력하지 않도록 limit가 수반됨.

```java
Stream.iterate(0, n -> n + 2)
	.limit(10)
	.forEach(System.out::println);
```

iterate메서드는 (초깃값, 람다) 를 인수로 받아서 새로운 값을 끊임없이 만든다.
예제의 경우는 0, 2, 4, ......

이와 같이 iterate는 요청할때마다 값을생산할 수 있으며 끝이 없으므로 무한스트림을 만들고 이러한 스트림을 **언바운드 스트림**이라고 표현한다.

자바 9에서의 iterate메서드는 프레디케이트를 지원한다.
만약 100보다 크면 숫자 생성을 중단하는 코드는 다음처럼 구현할 수 있다.

```java
IntStream.iterate(0, n -> n < 100, n -> n + 4)
	.forEach(System.out::println);
```

filter로 같은 결과를 얻을 수 있을것이라 생각할 수 있지만 filter메서드는 언제 작업을 중단해야 하는지 알 수 없다.

```java
IntStream.iterate(0, n -> n + 4)
	.filter(n -> n < 100)
	.forEach(System.out::println);
```

스트림 쇼트서킬을 지원하는 메서드를 사용해야 한다

```java
IntStream.iterate(0, n -> n + 4)
	.takeWhile(n -> n < 100)
	.forEach(System.out::println);
```

**generate 메서드**

generate도 무한 스트림을 만들 수 있다.

iterate와는 달리 generate는 생산된 각 값을 연속적으로 계산하지 않는다. Supplier<T>를 인수로 받아서 새로운 값을 생산한다.

```java
Stream.generate(Math::random)
	.limit(5)
	.forEach(System.out::println);
```

박싱을 피하기 위해 IntStream의 generate를 사용할 수 있다.
IntStream의 generate는 Supplier<T> 대신 IntSupplier를 인수로 받는다.

```java
IntStream ones = IntStream.generate(() -> 1);
```

람다로 함수형 인터페이스의 인스턴스를 바로 만들어 전달할 수도 있다.
다음처럼 IntSuipplier 인터페이스에 정의된 getAsInt를 구현하는 객체를 명시적으로 전달할 수도 있다.

```java
IntStream tows = IntStream.generate(new IntSupplier(){
	public int getAsInt(){
		return 2;
	}
});
```

generate메서드는 주어진 발행자를 이용해서 2를 반환하는 getAsInt메서드를 반복적으로 호출할 것이다. 여기서 사용한 익명클래스와 람다는 비슷한 연산을 수행하지만 익명 클래스에서는 getAsInt메서드의 연산을 커스터마이즈할 수 있는 상태 필드를 정의할 수 있다는 점이 다르다.

⇒ 부작용이 생길 수 있다!

지금까지의 람다는 상태를 바꾸지 않았다. (부작용이 없었다)

피보나치 수열 작업을 getAsInt메서드의 연산을 커스터마이즈 할 수 있는 상태필드를 가진 형태로 만들어보자(부작용이 있는 상태로 만들어보자)

```java
IntSupplier fib = new IntSupplier(){
	private int previous= 0;
	private int current = 1;
	public int getAsInt() {
		int oldPrevious = this.previous;
		int nextValue = this.previous + this.current;
		this.previous = this.current;
		this.current = nextValue;
		return oldPrevious;
	}
};

IntStream.generate(fib).limit(10).forEach(System.out::println);	
```

위 fib인스턴스는 내부에 previous와 current라는 상태필드를 정의하여 getAsInt메서드를 커스터마이징 했다.

fib인스턴스 객체는 기존 피보나치 요소와 두 인스턴스 변수에 어떤 피보나치 요소가 들어있는지 추적하므로 **가변**상태 객체이다. getAsInt를 호출하면 객체 상태가 바뀌며 새로운 값을 생산한다.

```java
Stream.iterate(new int[]{0, 1}, t-> new int[]{t[1], t[0] + t[1]}_
			.limit(10)
			.forEach(System.out::println);
```

반면 iterate를 사용했을 때는 새로운 값을 생성하면서도 기존 상태를 바꾸지 않는 순수한 **불변**상태를 유지했다. 스트림을 병렬로 처리하면서 올바른 결과를 얻으려면 **불변상태기법**을 고수해야한다.

## 5.9 마치며

- filter, distinct, takeWhile, dropWhile, skip, limit 메서드로 스트림을 필터링하거나 자를 수 있다.
- 정렬된 스트림에서는 takeWhile, dropWhile이 효과적으로 사용될 수 있다.
- findFirst, findAny 메서드로 스트림 요소를 검색
- allMatch, noneMach, anyMatch 메서드로 주어진 프레디케이트와 일치하는 요소 검색
- 이런 메서드는 쇼트서킷으로 결과를 찾는 즉시 반환하며 전체 스트림을 처리하지 않음
- reduce
- filter, map 등은 상태를 저장하지 않는 상태없는 연산
- sorted, distinct등의 메서드는 앞선 스트림의 모든 요소를 버퍼에 저장해야함. 상태 있는 연산
- 기본형 특화 스트림 IntStream, DoubleStream, LongStream
- 값, 배열 파일, iterate, generate메서드 등으로 스트림을 만들 수 있다.
- 무한스트림 = 언바운드 스트림. 무한스트림을 끊어서 언바운드를 해제한다.