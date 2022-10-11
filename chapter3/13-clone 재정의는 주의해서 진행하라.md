## 13. clone 재정의는 주의해서 진행하라.

Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이다.

하지만 의도한 목적을 제대로 이루지 못했는데, 가장 큰 문제가 clone메서드가 선언된 곳이 Cloneable이 아닌 object이고 그 마저도 protected라는 데 있다.

따라서 cloneable을 구현하는 것만으로는 외부 객체에서 clone메서드를 호출할 수 없다.

이런 문관례상 이 메서드가 반환하는 객체는 super.clone을 호출해서 얻어야 한다.

Cloneable 인터페이스가 하는 일은,

Object의 protected 메서드인 clone의 동작 방식을 결정한다.

Cloneable 인터페이스를 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 봇가한 객체를 반환하며 그렇지 않은 클래스의 인스턴스에서 clone()을 호출하면 CloneNotSupportedException을 던진다.

일반적으로 인터페이스를 구현한다는 의미는,

해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 것이다.

반면 Cloneable은 상위 클래스에 정의된 protected메서드의 동작 방식을 변경한 것이다.

**실무에서 Cloneable을 구현한 클래스는 clone메서드를 public으로 제공하며 사용자는 당연히 복제가 제대로 이루어질 것이라고 기대한다**

이 기대를 만족하려면 그 클래스와 모든 상위 클래스는 복잡하고 허술한 프로토콜을 지켜야만 하는데 이는 결과적으로 깨지기 쉽고 위험한 메커니즘의 탄생으로 이루어진다.

생성자 호출 없이 객체를 생성할 수 있게 되는 것이다.

### clone 메서드 재정의 규약

- x.clone() ≠ x = true (필수는 아니다)
- x.clone().getClass() == x.getClass() (필수는 아니다)
- x.clone().equals(x) (필수는 아니다)
- 관례상 이 메서드가 반환하는 객체는 super.clone()을 호출해서 얻어야 한다.
  이 클래스와 모든 상위 클래스(Object를 제외한)가 이 관례를 따른다면 다음 식은 참이다.

x.clone().getClass() == x.getClass()
- 관례상 반환된 객체와 원본 객체는 독립적이어야 한다.
  이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

강제성이 없기 때문에

clone메서드가 super.clone이 아닌 생성자를 호출해서 얻은 인스턴스를 반환해도 컴파일러는 불평하지 않을 것이다.

하지만 만약 이 클래스(새 인스턴스를 반환하도록 한 클래스) 의 하위 클래서에서 super.clone을 호출한다면 잘못된 클래스의 객체가 만들어져서 하위 클래스의 clone메서드가 제대로 동작하지 않게 된다.

```java
클래스 B가 클래스 A를 상송할 때 B의 clone은 B타입 객체를 반환해야 한다.

하지만 A의 clone이 생성자(new A(...))로 생성한 객체를 반환한다면 
B의 clone도 A를 반환할 수 밖에 없다.

달리 말하면 super.clone을 연쇄적으로 호출하도록 구현해두면 
clone이 처음 호출된 상위 클래스의 객체가 만들어진다. 

(왜 super.clone을 반환해야 할까? => 부모클래스 인스턴스도 복사해야하기 때문인듯?)
```

상위 클래스가 clone을 제대로 구현한 경우에 그 클래스를 상속해서 Coneable을 구현한다고 해보자.

먼저 clone을 재정의하고 super.clone을 호출할 것이다.

만약 모든 필드가 기본타입이거나 불변 객체를 참조한다면 더 이상 건드릴 필요가 없다.

(쓸데없는 복사를 지양한다는 관점에서 보면 불변클래스는 clone 메서드를 제공할 필요가 없긴 함.

값을 변경해서 사용할 것도 아닌데 clone이 무슨 의미가 있는건지라는 의문에서부터..)

```java
@Override
public PhoneNumber clone(){
	try{
		return (PhoneNumber) super.clone();
	} catch(CloneNotSupportedException e){
		throw new AssertionError();
	}
}
```

자바 공변 타이핑으로 PhoneNumber의 clone은 PhoneNumber를 반환하도록 변경했다.

catch를 추가한 이유는 clone메서드가 checked exception인 CloneNotSupportedException을 던지도록 선언되었기 때문이다. (잘못 설계된 clone. CloneNotSupportedException은 unchecked excpetion이어도 충분하다. PhoneNumber는 Cloneable을 구현하기 때문에..)

위처럼 불변 객체만 존재하는 경우는 간단하다.

하지만 가변 객체를 참조하는 경우 복잡해진다.

ex) 가변 객체를 갖고있는 Stack 클래스

```java
public class Stack{
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;
	...
}
```

만약 clone메서드가 super.clone의 결과를 그대로 반환한다면

Stack인스턴스의 size필드는 올바른 값을 가지겠지만

element필드는 원본 Stack인스턴스와 같은 배열을 참조할 것이다.

이는 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해치게 된다.

**clone 메서드는 생성자와 같은 효과를 낸다. clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다**

따라서 가변객체에 대한 복사도 다음과 같이 철저하게 이루어져야 한다

ex) 가변 상태를 참조하는 클래스의 clone 메서드

```java
@Override
public Stack clone(){
	try{
		Stack result = (Stack) super.clone();
		result.elements = elements.clone();
		return result;
	} catch (CloneNotSupportedException e)_{
		throw new AssertionError();
	}
}
```

배열의 clone은 런타임 타입, 컴파일타임 타입 모두 원본 배열과 같은 배열을 반환하므로 배열 복제는 배열의 clone을 사용한다.

만약 elements 필드가 final이었다면 새 값을 할당할 수 없기 때문에 위 방법은 작동하지 않을 것이다.

이는 **Cloneable 아키텍쳐는 ‘가변 객체를 참조하는 필드는 final로 선언하라’는 용법과 충돌한다**

따라서 clone을 재정의하기 위해선 final 한정자를 제거해야 할 수도 있다.

clone을 재귀적으로 호출하는 것만이 충분하지 않을 때가 있다.

해시테이블용 clone을 생각해보자

해시테이블 내부는 버킷들의 배열이고 버킷은 키-값 쌍을 담는 연결리스트의 첫 번째 엔트리를 참조한다.

```java
public class HashTable implements Cloneable {
	private Entry[] buckets = ...'

	private static class Entry{
		final Object key;
		Object value;
		Entry next;

		Entry(Object key, Object value, Entry next){
			this.key = key;
			this.value = value;
			this.next = next;
		}
	}
}
```

단순히 버킷 배열의 clone을 재귀적으로 호출하도록 clone메서드를 재정의해보자.

```java
@Override
public HashTable clone(){
	try{
		HashTable result = (HashTable) super.clone();
		result.buckets = buckets.clone();
		return result;
	}catch(CloneNotSupportedException e){
			throw new AssertionError();
	}
}
```

이렇게 buckets만 clone하면

buckets의 원소들은 원본과 같은 연결리스트를 참조하기 때문에 예상치못한 동작을 할 수 있다.

따라서 버킷을 구성하는 연결리스트 자체를 복사해야 한다.

ex) 복잡한 가변 상태를 갖는 클래스용 재귀적 clone메서드

```java
public class HashTable implements Cloneable{
	private Entry[] buckets = ...;
	...
	
	private static class Entry{
		...
		Entry deepCopy(){
			return new Entry(key, value, next == null? null : next.deepCopy());
		}
	}
	
	@Override public HashTable clone(){
		try{
			HashTable result = (HashTable) super.clone();
			result.buckets = new Entry[buckets.length];
			for(int i = 0; i<buckets.length; i++){
				if(buckets[i] != null){
					result.buckets[i] = vuckets[i].deepCopy();
			}
			
			return result;
		}catch(CloneNotSupportedException e){
			throw new AssertionError();
		}
	}
}
```

버킷이 너무 길지 않다면 버킷의 연결리스트에 포함된 모든 객체를 deepCopy하여 간단하게 문제를 해결할 수 있다.

하지만 이 방법은 좋지 않다.

원소의 수만큼 스택 프레임을 소비하여 리스트가 길면 스택 오버플로를 일으킬 수 있다.

이 문제를 피하기 위해 deepCopy를 재귀호출대신 반복자를 써서 순회하는 방향으로 수정해야 한다.

ex) 엔트리 자신이 가리키는 연결 리스트를 반복적으로 복사한다.

```java
Entry deepCopy(){
	Entry result = new Entry(key, value, next); // 자신 Entry를 먼저 copy하고
	for(Entry p = result; p.next != null; p = p.next){ // 링크된 Entry를 반복으로 copy
		p.next = new Entry(p.next.key, p.next.value, p.next.next);
	}
	
	return result;
}
```

마지막 방법으로는 다음 방법이 있다.

---

super.clone을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정한다.
그리고 원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출한다.

HashTable 예에서라면 buckeys 필드를 새로운 버킷 배열로 초기화한 다음 원본 테이블에 담긴 모든 키-값 쌍 각각에 대해 복제본 테이블의 put(key, value) 메서드를 호출하여 둘의 내용이 같게 해주면 된다.

---

이렇게 고수준 API를 활용하면 코드가 간단해지지만

저수준에서 바로 처리하는 것보다는 느리다.

또한 필드단위객체복사를 우회하기 때문에 전체 Cloneable아키텍처와는 어울리지 안흔ㄴ다.

생성자는 재정의될 수 있는 메서드를 호출하지 않아야 하는데 이는 clone도 마찬가지이다. 이유는 clone이 하위 클래스에서 재정의한 메서드를 호출하면 원본과 복제본의 상태가 달라질 수 있기 때문이다. 따라서 마지막 방법에서 사용하는 put(key, value) 메서드는 final이거나 private 이어야 한다.

위처럼 복잡한 경우는 많지 않지만..

Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone을 잘 작동하도록 구현해야 한다.

그렇지 않은 상황에서는 **복사생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다**

ex)

```java
public Yum(Yum yum){
	...
}

public static Yum newInstance(Yum yum){
	...
}
```

복사 생성자, 복사팩터리의 장점은 언어 모순적이고 위험한 객체 생성 메커니즘(생성자를 쓰지 않는)을 사용하지 않으며 정상적인 final 필드 용법과도 충돌하지 않고 불필요한 checked 예외를 던지지도 않고 형변환도 필요없다.

또한 복사 생성자와 복사 팩터리는 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다.

이의 예시로 컬렉션 구현체는 Collection이나 Map타입을 받는 생성자를 제공한다.
(ex, HashSet객체 s를 TreeSet타입으로 복제할 때 ⇒ new TreeSet<>(s))

인터페이스 기반 복사 생성자와 복사 팩터리의 더 정확한 이름은 변환 생성자와 변환 팩터리 이다.

**결론**

Cloneable의 문제점을 짚어볼 때 새로운 인터페이스를 만들 때 Cloneable을 확장해선 안되며 새로운 Class를 만들 때에도 이를 구현해선 안된다.

복제의 기본 원칙은 ‘생성자와 팩터리를 이용하는 것이 최고’라는 것

단 배열은 clone메서드 방식이 가장 깔끔한 예외적인 케이스이다.