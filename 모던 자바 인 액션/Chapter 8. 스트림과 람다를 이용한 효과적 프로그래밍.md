# Chapter 8. 스트림과 람다를 이용한 효과적 프로그래밍
**내용**

- 컬렉션 팩토리 사용하기
- 리스트 및 집합과 사용할 새로운 관용 패턴 배우기
- 맵과 사용할 새로운 관용 패턴 배우기

자바 8, 9에 추가된 컬렉션 API 기능 학습한다.

리스트, 집합, 맵을 쉽게 만들 수 있도록 추가된 컬렉션 팩토리.

리스트와 집합에서 요소를 삭제하거나 바꾸는 관용 패턴을 적용하는 방법 학습.

맵 작업과 관련해 추가된 새로운 편의 기능 학습.

## 8.1 컬렉션 팩토리

작은 컬렉션 객체를 쉽게 만들 수 있는 방법을 제공

왜 필요할까? ⇒ 아래 코드를 보자.

```java
List<String> friends = new ArrayList<>();
friends.add("A");
friends.add("B");
friends.add("C");
```

세 개의 문자열을 저장하는 데에도 많은 코드가 필요하다.

Arrays.asList()팰토리 메서드를 사용하여 코드를 간결하게 줄일 수 있다.

```java
List<String> friends = Arrays.asList("A", "B", "C");
```

고정 크기의 리스트를 만들었으므로 요소를 갱신할 수 있지만,

새 요소를 추가하거나 요소를 삭제할 순 없다.

요소를 추가하는 작업은 UnsupportedOperationException이 발생한다.

```java
friends.add("D");
```

**UnsupportedOperationException 예외 발생**

내부적으로 고정된 크기의 변활할 수 있는 배열로 구현되었기 때문에 이와 같은 일이 발생한다.

집합은? ⇒ asSet()이라는 팩토리메서드는 없기 때문에 리스트를 인수로 받는 HashSet생성자를 사용한다.

```java
Set<String> friends = new HashSet<>(Arrays.asList("A", "B", "C"));
```

또는 stream API를 사용할 수 있다.

```java
Set<String> friends = Stream.of("A", "B", "C")
														.collect(Collectors.toSet());
```

위 두 방법은 내부적으로 불필요한 객체 할당을 필요로 하기 때문에 매끄럽지 못하다.

자바 9에서 작은 리스트, 집합, 맵을 쉽게 만들 수 있도록 제공하는 팩토리 메서드를 알아보자.

**컬렉션 리터럴**

파이썬, 그루비 등 일부언어에서는 컬렉션 리터럴을 사용하여 컬렉션을 만들 수 있는 기능을 지원한다. 자바에서는 이런 기능 대신 컬렉션 API를 개선했다.

### 8.1.1 리스트 팩토리

List.of 팩토리 메서드를 사용하여 간단하게 리스트를 만들 수 있다.

```java
List<String> friends = List.of("A", "B", "C");
```

위 리스트 friends는 변경할 수 없는 리스트로 만들어졌기 때문에 add는 물론 set 메서드를 호출하면 UnsupportedOperationException 에러가 발생한다.

이러한 처리는 컬렉션이 의도치 않게 변하는 것을 막을 수 있지만 요소 자체가 변하는 것을 막을 수 있는 방법은 없다.

리스트를 바꿔야 하는 상황이라면 직접 리스트를 만들면 된다.

또한 null요소는 금지하므로 의도치 않은 버그를 방지하고 조금 더 간결한 내부 구현을 달성할 수 있다.

**오버로딩 vs 가변 인수**

List.of 는 다양한 오버로드 버전이 존재한다.

ex)

```java
static <E> List<E> of(E e1) {
        return new ImmutableCollections.List12<>(e1);
    }
static <E> List<E> of(E e1, E e2) {
        return new ImmutableCollections.List12<>(e1, e2);
    }
static <E> List<E> of(E e1, E e2, E e3) {
        return new ImmutableCollections.ListN<>(e1, e2, e3);
    }
.
.
.
static <E> List<E> of(E... elements) {
	...
```

다중요소를 받을 수 있는 of메서드가 존재하는데 다양한 버전을 만든 이유는?

내부적으로 가변 인수 버전은 추가 배열을 할당해서 리스트로 감싼다.

```java
static <E> List<E> of(E... elements) {
        switch (elements.length) { // implicit null check of elements
            case 0:
                return ImmutableCollections.emptyList();
            case 1:
                return new ImmutableCollections.List12<>(elements[0]);
            case 2:
                return new ImmutableCollections.List12<>(elements[0], elements[1]);
            default:
                return new ImmutableCollections.ListN<>(elements);
        }
    }
```

따라서 배열을 할당하고 초기화하며 나중에 가비지 컬렉션을 하는 비용을 지불해야 한다.

고정된 숫자 요소(최대 10개)를 API로 정의했기에 이런 비용을 제거할 수 있다.

10개 이하일 때는 개수별로 오버로딩된 메서드, 10개 초과일 땐 가변 인수를 받는 메서드를 사용하면 된다.

데이터 처리 형식을 설정하거나 데이터를 변환할 필요가 없다면 ⇒ 팩토리 메서드 사용하자

(구현이 단순하고 목적을 달성하는데에 충분하다)

데이터 변환이 필요하고 크기 등이 가변적이라면 ⇒ Collectors.toList() 를 사용하자

### 8.1.2 집합 팩토리

List.of처럼 바꿀 수 없는 집합도 생성할 수 있다.

```java
Set<String> friends = Set.of("A", "B", "C");
```

중복된 요소를 팩토리 메서드의 인수로 전달하면 IllegalArgumentException이 발생한다.

### 8.1.3 맵 팩토리

두 가지 방법으로 바꿀 수 없는 맵을 초기화할 수 있다.

1. Map.of 팩토리메서드에 키와 값을 번갈에 제공한다.

```java
Map<String, Integer> ageOfFriends = Map.of("A", 1, "B", 2, "C", 3);
```

열 개 이하의 키, 값 쌍에 대해서는 이 메서드가 유용하다.

그 이상의 맵에서는 Map.Entry<K, V> 객체를 인수로 받으며 가변 인수로 구현된 Map.ofEntries 팩토리 메서드를 이용하는 것이 좋다. 키와 값을 감쌀 추가 객체 할당을 필요로 한다.

1. Map.ofEntries 팩토리 메서드에 Entry<K, V> 객체를 전달한다.

```java
import static java.util.Map.entry;

Map<String, Integer> ageOfFriends = Map.ofEntries(entry("A", 1),
																									entry("B", 2),
																									entry("C", 3));	
```

entry는 키와 값을 받아서 Map.Entry객체를 만드는 팩토리 메서드이다.

## 8.2 리스트와 집합 처리

자바 8에서 List, Set 인터페이스에 다음 메서드가 추가됨

- removeIf : 프레디케이트를 만족하는 요소를 제거한다. List, Set을 구현하거나 그 구현을 상속받은 모든 클래스에서 이용 가능
- replaceAll : 리스트에서 이용할 수 있는 기능. UnaryOperator 함수를 이용해서 요소를 바꾼다.
- sort : List 인터페이스에서 제공하는 기능. 리스트를 정렬한다.

위 메서드들은 호출한 컬렉션 자체를 바꾼다. 새로운 결과를 만드는 스트림 동작과 달리 이들 메서드는 기존 컬렉션을 바꾼다.

컬렉션을 바꾸는 동작(컬렉션을 교체한다는 의미?)은 에러를 유발하며 복잡함을 더한다. 이러한 이유때문에 위 메서드가 추가되었다.

### 8.2.1 removeIf 메서드

숫자로 시작되는 참조코드를 가진 트랜잭션을 삭제하는 코드이다.

```java
for(Transaction transaction : transactions){
	if(Character.isDigit(transaction.getReferenceCode().charAt(0)))){
		transactions.remove(transaction);
	}
}
```

위 코드는 ConcurrentModificationException을 일으킨다. 이유는?

내부적으로 for-each 루프는 Iterator 객체를 사용하므로 위 코드는 다음과 같이 해석된다.

```java
for(Interator<Transaction> iterator = transactions.iterator(); iterator.hasNext();){
	Transaction transaction = iterator.next();
	if(Character.isDigit(transaction.getReferenceCode().charAt(0)))){
		transactions.remove(transaction);
	}
}
```

두 개의 개별 객체가 컬렉션을 관리한다

- Iterator 객체가 next(), hasNext()를 이용해서 소스를 질의한다.
- Collection 객체가 remove()를 호출해 요소를 삭제한다.

반복자의 상태는 컬렉션의 상태와 서로 동기화되지 않기 때문에 에러가 발생한다.

Iterator객체를 명시적으로 사용하고 그 객체의 remove()메서드를 호출함으로 이 문제를 해결할 수 있다.

```java
for(Interator<Transaction> iterator = transactions.iterator(); iterator.hasNext();){
	Transaction transaction = iterator.next();
	if(Character.isDigit(transaction.getReferenceCode().charAt(0)))){
		iterator .remove();
	}
}
```

코드가 복잡하다.

자바 8의 removeIf 메서드를 사용할 수 있다. 코드 단순화가 가능하고 버그도 예방할 수 있다.

removeIf메서드는 삭제할 요소를 가리키는 프레디케이트를 인수로 받는다.

```java
transactions.removeIf(transaction->
	Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

### 8.2.2 replaceAll 메서드

요소의 변경이 필요한 경우 List 인터페이스의 replaceAll 메서드를 이용할 수 있다.

스트림 API를 사용하면 다음과 같이 문제를 해결할 수 있다.

```java
referenceCodes.stream()
	.map(code -> Character.toUpperCase(code.charAt(0))
				+ code.substring(1))
	.collect(Collectors.toList())
	.forEach(System.out::println);
```

위 코드는 새 문자열 컬렉션을 만든다.

기존 컬렉션을 바꾸기 위해서 ListIterator객체를 이용할 수 있다.

```java
for(ListIterator<String> iterator = referenceCodes.listIterator(); iterator.hasNext();){
	String code = iterator.next();
	iterator.set(Character.toUpperCase(code.charAt(0)) + code.substring(1));
}
```

코드가 복잡하다.

그리고 컬렉션 객체와 Iteractor객체와 혼용하면 반복과 컬렉션 변경이 동시에 이루어지면서 문제를 야기할 수 있다. replaceAll 메서드를 사용하면 간단하게 구현할 수 있따.

```java
referenceCodes.replaceAll(code -> 
	Character.toUpperCase(code.charAt(0)) + code.substring(1));
```

## 8.3 맵 처리

자바 8에서 Map인터페이스에 몇 가지 디폴트메서드가 추가되었다. 자주 사용되는 패턴에 대해 메서드를 추가함.

### 8.3.1 forEach메서드

맵에서 키와 값을 반복하면서 확인하기 위해서는 Map.Entry<K, V>의 반복자를 이용해야한다.

ex)

```java
for(Map.Entry<String, Integer> entry : ageOfFriends.entrySet()){
	...
}
```

자바 8부터 Map인터페이스는 BiConsumer를 인수로 받는 forEach메서드를 지원한다.

ex)

```java
ageOfFriends.forEach((friend, age) -> {
	...
});
```

### 8.3.2 정렬 메서드

다음의 유틸리티를 이용하여 맵의 항목을 값 또는 키를 기준으로 정렬할 수 있다.

- Entry.comparingByValue
- Entry.comparingByKey

ex)

```java
favouriteMovies
	.entrySet()
	.stream()
	.sorted(Entry.comparingByKey())
	.forEachOrdered(System.out::println); //스트림요소의 순서를 보장하기 위한 forEach

// 결과 예시
Cristina=matrix
Olivia=james bond
Raphael=star wars
```

**HashMap 성능**

자바 8에서는 HashMap의 내부 구조를 바꿔서 성능을 개선했다.

기존에 맵의 항목은 키로 생성한 해시코드로 접근할 수 있는 버켓에 저장했다. 많은 키가 동일한 해시코드를 반환하는 상황이 되면 O(n)의 시간이 걸리는 LinkedList로 버킷을 반환해야 하므로 성능이 저하된다.

최근엔 버킷이 너무 커질 경우 이를 O(log(n))의 시간이 소요되는 정렬된 트리를 이용해서 동적으로 치환하여 충돌이 일어나는 요소 반환 성능을 개선했다.

키가 String, Number 클래스같은 Comparable 형태여야만 정렬된 트리가 지원된다.

### 8.3.3 getOrDefault 메서드

기존에는 츶으려는 키가 존재하지 않으면 null이 반환되므로 NPE를 방지하기 위해 요청 결과가 null인지 확인해야 한다.

getOfDefault메서드를 사용하여 기본값을 반환하는 방식으로 해결가능하다.

첫번째 인수로 key, 두번째 인수로 기본값을 받으며 맵에 키가 존재하지 않으면 두 번째 인수로 받은 기본값을 반환한다.

ex)

```java
Map<String, String> movies = Map.ofEntries(entry("Raphael", "star wards"),
		entry("Olivia", "james bond"));

sout(movies.getOfDefault("Olivia", "matrix")); // james bond 출력
sout(movies.getOfDefault("Thibaut", "matrix")); // key가 없으므로 matrix 출력
```

키가 존재하더라도 값이 null이면 getOrDefault도 null이 반환될 수 있다.

### 8.3.4 계산 패턴

맵에 키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장해야 하는 경우 아래와 같은 연산을 사용할 수 있다.

- computeIfAbsent : 제공된 키에 해당하는 값이 없으면(값이 없거나 null), 키를 이용해서 새 값을 계산하고 맵에 추가한다.
- computeIfPresent : 제공된 키가 존재하면, 새 값을 계산하고 맵에 추가한다.
- compute : 제공된 키로 새 값을 계산하고 맵에 저장한다.

### 8.3.5 삭제 패턴

제공된 키에 해당하는 맵 항목을 제거하는 remove메서드.

자바 8에서는 키가 특정한 값과 연관되었을 때만 항목을 제거하는 오버로드 버전 메서드를 제공한다.

```java
favouriteMovies.remove(key, value);
```

위 코드는

1. key가 map에 존재하고 key로 get한 값이 value와 동일하다면
2. key, value쌍을 remove하고
3. true를 리턴한다.

만약 1번 조건에 해당하지 않는다면 false를 리턴한다.

### 8.3.6 교체 패턴

맵의 항목을 바꾸는데에 사용할 수 있는 두 개의 메서드가 맵에 추가되었다.

- replaceAll : BiFunction을 적용한 결과로 각 항목의 값을 교체한다.
- replace : 키가 존재하면 맵의 값을 바꾼다.

ex) value에 해당하는 movie를 upperCase로 교체한다.

```java
favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```

### 8.3.7 합침

두 개의 맵을 합칠 때 putAll 메서드를 사용할 수 있다.

```java
everyone.putAll(friends); // everyone 맵에 friends 맵의 모든 항목을 복사한다.
```

중복된 키가 없다면 위 코드는 잘 동작한다.

중복된 키를 어떻게 합칠지 결정해야 한다면 merge메서드를 이용할 수 있다.

merge메서드는 BiFuntion을 인수로 받는다.

family와 friends 두 맵에 모두 cristina라는 키가 존재한다고 가정하자.

아래와 같이 forEach와 merge메서드를 이용해서 충돌을 해결할 수 있다.

```java
frends.forEach((k, v) ->
	everyone.merge(k, v, (movie1, movie2) -> movie1));
```

merge 메서드는 널값과 관련된 상황도 처리한다.

값이 없거나 널이면 merge는 키를 널이 아닌 값과 연결한다.

아니면 merge는 연결된 값을 주어진 매핑 함수의 결과값으로 대치하거나 결과가 널이면 항목을 제거한다.

## 8.4 개선된 ConcurrentHashMap

ConcurrentHashMap

동시성 친화적이며 최신 기술을 반영한 HashMap 버전.

내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다. 따라서 동괴화된 Hashtable 버전에 비해 읽기 쓰기 연산 성능이 좋다.

### 8.4.1 리듀스와 검색

세 가지의 새로운 연산을 지원한다.

- forEach : 각 (키, 값) 쌍에 주어진 액션을 실행
- reduce : 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해서 결과로 ㅅ합침
- search : 널이 아닌 값을 반활할 때까지 각 (키, 값) 쌍에 함수를 적용

키에 함수 받기, 값, Map.Entry, (키, 값) 인수를 이용한 네 가지 연산 형태를 지원한다.

- 키, 값으로 연산( forEach, reduce, search)
- 키로 연산(forEachKey, reduceKeys, searchKeys)
- 값으로 연산(forEachValue, reduceValues, searchValues)
- Map.Entry 객체로 연산(forEachEntry, reduceEntries, searchEntries)

이들 연산은 ConcurrentHashMap의 상태를 잠그지 않고 연산을 수행한다.

따라서 이들 연산에 제공된 함수는 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야 한다.

또한 이들 연산에 병렬성 기준값을 지정해야 한다.

맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산을 실행한다.

기준값을 1로 지정하면 공통 스레드 풀을 이용해서 병렬성을 극대화한다.

Long.MAX_VALUE를 기준값으로 설정하면 한 개의 스레드로 연산을 실행한다.

ex)

```java
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
long parallelismThreshold = 1;

Optional<Integer> maxValue = 
	Opional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
```

### 8.4.2 계수

ConcurrentHashMap 클래스에는 맵의 매핑 개수를 반환하는 mappingCount메서드를 제공한다.

기존 size메서드 대신 int를 반환하는 mappingCount메서드를 사용하는 것이 좋다.

그래야 매핑의 개수가 int의 범위를 넘어서는 이후의 상황을 대처할 수 있다. (???)

### 8.4.3 집합뷰

ConcurrentHashMap클래스는 ConcurrentHashMap를 집합 뷰로 반환하는 keySet이라는 새 메서드를 제공한다.

맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받는다.

newKeySet이라는 새 메서드를 이용해서 ConcurrentHashMap으로 유지되는 집합을 만들 수도 있다.

## 8.5 마치며

- 바꿀 수 없는 리스트, 집합 맵을 쉽게 만들 수 있는 List.of, Set.of 등의 컬렉션 팩토리를 지원한다.
- List 인터페이스는 removeIf, replaceAll, sort 세 가지 디폴트 메서드를 지원한다.
- Set 인터페이스는 removeIf 디폴트 메서드를 지원한다.
- ConcurrentHashjMap은 Map에서 상속받은 새 디폴트 메서드를 지원함과 동시에 스레드 안전성도 제공한다.
