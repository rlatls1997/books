## 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

다음과 같은 메서드를 흔히 볼 수 있다.

ex) 컬렉션이 비었으면 null을 반환 - 따라하지 말것

```java
private final List<Cheese> cheeseInStock = ...;

public List<Cheese> getCheeses(){
	return cheeseInStock.isEmpty() ? null : new ArrayList<>(cheeseInStock);
}
```

위와 같이 null을 반환한다면 이 메서드를 사용하는 클라이언트는 이 null상황을 처리하는 코드를 추가로 작성해야 한다.

```java
List<Cheese> cheeses = shop.getCheeses();
if(cheese != null && cheeses.contains(Cheese.STILTON)){
	~~~
}
```

컬렉션이나 배열같은 컨테이너가 비었을 때 null을 반환하는 메서드를 사용하면 항시 위와 같은 방어 코드를 넣어줘야한다.

만약 클라이언트가 실수로 빼먹는 경우에는 오류가 발생할 수 있다.

null을 반환하는 쪽에서도 이 상황을 따로 취급해야하므로 코드가 더 복잡해진다.

때론 빈 컨테이너를 할당하는데에도 비용이 있으므로 null을 반환하는 쪽이 좋다는 주장이 있다.

하지만 위 주장은

- 성능차이는 신경쓸 수준이 되지 못한다.
- 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

아래와 같이 빈 컬렉션을 반환할 수 있따

ex) 빈 컬렉션을 반환하는 올바른 예

```java
public List<Cheese> getCheeses(){
	return new ArrayList<>(cheeseInSTock);
}
```

또는 Collections.emptyList나 Collections.emptySet, Collections.emptyMap등의 빈 불변컬렉션을 사용하여 최적화를 할 수 있다.

ex) 최적화 - 빈 컬렉션을 매번 새로 할당하지 않도록 해줌

```java
public List<Cheese> getCheeses(){
	return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
```

배열도 마찬가지이다.

빈 배열의 최적화가 필요할 땐 길이가 0짜리인 배열을 미리 선언해두고 매번 그 배열을 사용하면된다. 길이가 0인 배열은 불변이기 때문이다.

ex) 최적화 - 빈 배열을 매번 새로 할당하지 않도록 해줌

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses(){
	return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

성능개선이 목적이라면 toArray에 넘기는 배열을 미리 할당하는건 추천되지 않는다.
오히려 성능을 하락시킨다.