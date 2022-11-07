## 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

매개변수화 타입은 불공변이다.
즉, 서로 다른 타입 Type1과 Type2가 있을 때 List<Type1> 은 List<Type2>의 하위타입도, 상위타입도 아니다.

즉 List<String>은 List<Object>의 하위타입이 아니라는 것이다.

List<String>은 List<Object>가 하는 일을 제대로 수행하지 못하기 때문에 하위타입이 될 수 없다.

하지만 때론 불공변방식을 깬 유연한 무언가가 필요하다.

Stack클래스를 가정해보자

```javan3
public class Stack<E> {
	public Stack();
	public void push(E e);
	public E pop();
	public boolean isEmpty();
}
```

여기에 원소를 스택에 넣는 메서드를 추가해야 한다고 가정해보자\

ex) 와일드카드 타입을 사용하지 않은 pushAll메서드. - 결함이 존재한다.

```java
public void pushAll(Iterable<E> src){
	for(E e : src){
		push(e);
	}
}
```

컴파일되나 완벽하지 않다.

Iterable src의 원소 타입이 스택의 원소와 일치하면 잘 동작한다.

하지만 Stack<Number>로 선언 후 pushAll(Intetger intVal)을 호출하면?
Integer는 Number의 하위타입이므로 잘 동작해야할 것 같지만 매개변수화 타입이 불공변이기 때문에 오류가 발생한다.

이런 문제를 해결하기 위해 한정적 와일드카드를 사용할 수 있다.
pushAll의 입력매개변수 타입이 **E의 Iterable이 아니라 E의 하위타입의 Iterable이어야 함**을 와일트 카드 타입 **Iterable<? extends E>**로 나타낼 수 있다.

ex) 한정적 와일드카드 타입을 적용

```java
public void pushAll(Iterable<? extends E> src){
	for(E e : src){
		push(e);
	}
}
```

반대로 Stack안의 모든 원소를 주어진 컬렉션으로 옮겨담는 popAll메서드를 생각해보자

ex) 와일드카드 타입을 사용하지 않은 popAll

```java
public void popAll(Collection<E> dst){
	while(!isEmpty()){
		dst.add(pop());
	}
}
```

마찬가지로 이번엔 Stack<Number>의 원소를 Collection<Object>인 dst에 add()한다고 가정해보면 Collection<Object>는 Collection<Number>의 하위 타입이 아니다”라는 오류가 발생한다.

**여기서 가장 중요한건 Object랑 Number를 따로 봐선 안된다는 것이다. 불공변인 매개변수화 타입인 Collection<Object>와 Collection<Number>의 관계에서 발생하는 오류를 생각하자**

입력매개변수의 타입이 E의 상위타입 Collection임을 나타내는 매개변수화 타입을 적용하여 해결할 수 있다.

```java
public void popAll(Collection<? super E> dst){
	while(!isEmpty()){
		dst.add(pop());
	}
}
```

결론은, 유연성을 극대화하려면 생산자, 소비자용 입력매개변수에 와일드카드 타입을 사용하라는 것이다.

**(일반적으로 producer는 extends를 사용한 와일드카드 타입, consumer는 super를 사용한 와일드카드 타입을 사용한다. PECS공식)**

두 Set을 Union하는 union메서드를 가정해보자

PECS공식에 따라 다음처럼 변경할 수 있다.

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2);

public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2); // PECS적용
```

(반환타입에선 한정적 와일드카드 타입을 사용해선 안된다. 클라이언트코드에서도 와일드카드 타입을 사용해야하기 때문이다)

다음처럼 사용할 수 있다.

```java
Set<Integer> inegers = ...
Set<Double> doubles = ...
Set<Number> numbers = union(inegeter, doubles);
```

위 코드는 자바 8부터 제대로 컴파일되지만 자바 7까지는 타입추론능력이 강하지 못해서 문맥에 맞는 반환타입을 명시해야했다.

예로, 위의 코드에서는 union의 목표 반환타입이 Set<Number>이다

이렇게 컴파일러가 타입추론을 하지 못할때면 **명시적 타입 인수**를 사용하여 타입을 알려주면 된다.

ex) 명시적 타입 인수 사용

```java
Set<Number> numers = Union.<Number>union(integeres, doubles);
```

---

컬렉션에서 최댓값을 반환하는 메서드의 예를 보자
마찬가지로 와일드카드 타입을 사용해서 다듬을 수 있다.

ex) 최댓값을 반환하는 메서드 max()의 함수시그니처

```java
public static <E extends Comparable<E>> E max(Collection<E> c)
public static <E extends Comparable<? super E>> E max(Collection<? extends E> c) //PECS
```

- Comparable은 언자나 소비자이므로 일반적으로 Comparable<E>보단 Comparable<? super E>를 사용하는 편이 낫다. (Comparator도 마찬가지)

위와 같이 PECS를 적용한 메서드가 필수적인 경우가 있다.
다음과 같은 리스트는 오직 수정된 max로만 처리할 수 있다.

```java
List<ScheduledFuture<?>> scheduledFutures = ...;
```

수정전 max가 이 리스트를 처리할 수 없는 이유는 ScheduledFuture가 Comparable<SchduledFuture>를 구현하지 않았기 때문이다.

ScheduledFuture는 Delayed하위 인터페이스이고 Delayed는 Comparable<Delayed>를 확장했기 때문에 ScheduledFuture의 인스턴스는 다른 ScheduledFuture뿐 아니라 Delayed인스턴스와도 비교할 수 있어서 수정 전 max는 사용할 수 없다.

간단히 말하면 ScheduledFuture가 Comparable을 직접 구현하지 않고 직접 구현한 다른 타입인 Delayed를 extends한 것을 지원하기 위해 와일드카드가 필요한 것이다.

---

타입매개변수와 와일드카드에는 공통되는 부분이 있어서 메서드를 정의할 때 어느 것을 사용해도 괜찮은 경우가 있다.

ex)

```java
public static <E> void swap(List<E> list, int i, int j);  //비한정적 타입 매개변수 사용
public static void swap(List<?> list, int i, int j);  // 비한정적 와일드카드 사용
```

기뵨규칙은 다음과 같다.

메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라. (2번째 방식)

이 때 비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다.

하지만 두 번째 swap은 문제가 있다.

다음 코드가 컴파일되지 않는다.

ex) 리스트에서 꺼낸 원소 바로 넣을때

```java
public static void swap(List<?> list, int i, int j){
	list.set(i, list.set(j, list.get(i)));
}

//incompatible types : Object cannot be converted to ...
```

원인은 리스트의 타입이 List<?>인데 List<?>엔 null외에 아무 값도 넣을 수 없다는데에 있다.

형변환이나 리스트의 로 타입을 사용하지 않고도 해결할 수 있다.

와일드카드의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 따로 작성하여 활용하면 된다. 실제타입을 알아내려면 도우미메서드는 제네릭이어야 한다.

ex) 도우미메서드 swapHelper

```java
public static void swap(List<?> list, int i, int j){
	swapHelper(list, i, j);
}

public static <E> void swapHelper(List<E> list, int i, int j){
	list.set(i, list.set(j, list.get(i)));
}
```

swapHelper는 리스트가 List<E>임을 알고 있기 때문에 꺼낸 값도 E이고 set했을 때 안전하다는 것을 알고 있기 때문에 깔끔하게 컴파일된다.

이렇게 하면 외부에서는 와일드카드 기반의 선언으로 유연성을 유지하면서 swapHelper 메서드의 존재를 모르는 채 그 혜택을 누릴 수 있다.

**결론**

유연성을 높이기 위해 와일드카드 타입을 적용할 수 있다.

PECS공식을 기억하자. producer-extends-consumer-super