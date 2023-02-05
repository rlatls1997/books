## 58. 전통적인 for문보다는 for-each문을 사용하라

ex) 전통적인 for문

```java
// 컬렉션 순회하기
for(Iterator<Element> i = c.iterator(); i.hasNext();){
	Element e = i.next();
	...
}

// 배열 순회하기
for(int i = 0; i<a.length; i++){
	...
}
```

반복자와 인덱스 변수는 코드를 지저분하게 할 뿐더러 실질적으로 필요한 원소가 아니다.

또한 이런 요소 종류가 늘어나면 오류가 생길 가능성이 높아진다.

잘못된 변수를 사용했을 때 컴파일러가 잡아주리라는 보장도 없다.

그리고 컬렉션이냐, 배열이냐에 따라 코드 형태도 달라진다.

이는 for-each문으로 해결된다.

반복자와 인덱스변수를 사용하지 않으므로 코드가 깔끔해지고 오류가 날 일도 없다.

그리고 하나의 관용구로 컬렉션, 배열 모두를 처리할 수 있어서 어떤 컨테이너를 다루는지 신경쓰지않아도 된다.

```java
for(Element e : elements){
	...
}
```

컬렉션을 중첩해서 순회해야할 때 for-each문의 이점은 더욱 커진다.

중첩된 반복문에서는 다음과 같은 버그를 만들 수 있다.

```java
enum Suit { A, B, C}
enum Rank { ONE, TWO, THREE }
..
static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for(Iterator<Suit> i = suits.iterator(); i.hasNext();){
	for(Iterator<Rank> j = ranks.iterator(); j.hasNext();){
		deck.add(new Card(i.next(), j.next()));
	}
}
```

문제는,

i.next()는 Suit하나당 한 번씩만 불려야 하는데 안쪽 반복문에서 호출되는 바람에

Rank하나당 한 번씩 불리고 있따.

이는 NoSuchElementException을 야기할것이다.

만약 바깥 컬렉션의 크기가 안쪽 컬렉션 크기의 배수라면 이 반복문은 예외를 던지지 않고 잘못된 동작을 한 채로 종료될 것이다.

안쪽 반복문 시작 전에 바깥원소를 저장하는 변수를 추가할 수 있지만 코드가 별로 보기 좋지 않다.

for-each를 사용하면 깔끔하게 해결된다.

```java
for(Suit suit : suits){
	for(Rank rank : ranks){
		deck.add(new Card(suit, rank));
	}
}
```

**for-each문을 사용할 수 없는 경우**

- 파괴적인 필터링 : 컬렉션을 순회하며 원소를 제거해야하는 경우.
- 변형 : 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야하는 경우
- 병렬 반복 : 여러 컬렉션을 병렬로 순회해야하는 경우.(반복다와 인덱스 변수를 사용하여 엄격하기 명시적으로 제어해야함)

for-each문은 컬렉션, 배열 외에도 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다.

```java
public interafce Iterable<E> {
	Iterator<E> iterator();
}
```