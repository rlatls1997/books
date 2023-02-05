### 14. Comparable을 구현할지 고려하라

Comparable 인터페이스엔 compareTo 메서드가 정의되어 있다.

compareTo는 Object의 메서드가 아니다.

compareTo는 equals와 비슷하지만,

compareTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며 제네릭하다.

Compareble을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 의미한다. Comparable을 구현한 객체들의 배열은 다음처럼 쉽게 정렬할 수 있다.

```java
Arrays.sort(a);
```

알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 Comparable 인터페이스를 구현하자

### compareTo 메서드 일반 규약

- 객체끼리의 순서를 비교한다.
  현 객체가 주어진 객체보다 작으면 음의 정수,
  같으면 0,
  현 객체가 주어진 객체보다 크면 양의 정수를 반환한다.

비교할 수 없는 타입의 객체라면 ClassCastException을 던진다.
- sgn(표현식) 표기는 수학에서 말하는 부호 함수(signum finction, 부호를 판별하는 함수)을 의미한다
- Comparable을 구현한 클래스는 모든 x, y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))
- Comparable을 구현한 클래스는 추이성을 보장해야 한다.
- Comparable 을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0 이면
  sgn(x.compareTo(z)) == sgn(y.compareTo(z)) 다.
- 이 절은 필수가 아니지만 지키는 것이 좋다.
  (x.compareTo(y) == 0) == (x.equals(y)) 여야 한다.

Comparable을 구현하고 이 권고를 지키지 않는다면 순서가 equals메서드와 일관되지 않다는 사실을 명시해야 한다.

equals 의 규약과 마찬가지로 반사성, 대칭성, 추이성을 충종해야 한다.

hashCode 규약을 지키지 못했을 때 Hash를 사용하는 클래스와 상호작용할 수 없던 것처럼,

compareTo 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못하게 된다. (ex, TreeSet, TreeMap, Collections, Arrays …)

equals와 마찬가지로,

기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 compareTo 규약을 지킬 방법은 없다. 우회법도 equals와 같다.(확장대신 독립된 클래스를 만들고 필드에 추가 후 내부 인스턴스를 반환하는 ‘뷰’메서드 추가)

마지막 규약(equals와 compareTo의 결과가 일관되게 하는 것)은 필수는 아니지만 지키는 것이 좋다. 그 이유는 일관되지 않아도 동작은 하지만 객체를 정렬된 컬렉션에 넣으면 해당 컬렉션이 구현한 인터페이스(Collection, Set, Map…)에 정의된 동작과 다른 결과를 낼 것이다.

이 인터페이스들은 equals 규약을 따르도록 되어 있지만 정렬된 컬렉션들은 동치성을 비교할 때 compareTo를 사용하기 때문이다.

### compareTo 작성 요령

- Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo메서드의 인수 타입은 컴파일타임에 정해진다. 따라서 입력인수 타입을 확인하거나 형변환할 필요가 없다.
- compareTo메서드드는 각 필드의 동치가 아닌 순서를 비교한다. 객체 참조 필드를 비교하려면 compareTo메서드를 재귀적으로 호출한다. Comparable을 구현하지 않은 필드의 경우 Comparator를 사용한다.

ex) 객체 참조 필드가 하나뿐인 Comparator

```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
	public int compareTo(CaseInsensitiveString cis){
		return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
	}
	...
}

```

자바 7이전에서 compareTo 메서드에서 정수 기본 타입 필드를 비교할 때 관계 연산자인 <, >,
실수 기본타입 비교에서는 Double.compare, Float.compare을 사용했었다.

자바 7부터는 박싱된 기본타입 클래스에 추가된 compare을 사용하면 된다.

**compareTo 메서드에서 관계 연산자 <, >를 사용하는 방식은 거추장스럽고 오류를 유발하여 추천되지 않는다**

- 핵심필드가 여러개라면 가장 핵심적은 필드부터 비교하자. 비교 결과가 0이 아닐 경우 즉시 결과를 반환할 수 있다.
- 자바 8부터 Comparator 인터페이스에서 제공하는 비교자 생성 메서드를 사용하면 메서드 체이닝으로 비교자를 생성할 수 있다.

ex) 비교자 생성 메서드를사용한 비교자

```java
private static final Comparator<PhoneNumber> COMPARATOR = 
	comparingInt((PhoneNumber pn) -> pn.areaCode)
		.thenComparingInt(pn -> pn.prefix)
		.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn){
	return COMPARATOR.compare(this. pn);
)
```

약간의 성능 저하가 있다.

객체 참조용 비교자 생성 메서드도 존재한다.

hashCode값의 차를 기준으로 하는 비교자를 만들어보자

ex) hashCode값의 차를 기준으로 하는 비교자 - 추이성 위배 버전

```java
static Comparator<Object> hashCodeOrder = new Comprarator<>(){
	public int compare(Object o1, Object o2){
		return o1.hashCode() - o2.hashCode();
	}
};
```

위 방식은 정수 오버플로를 일으키거나 부동소수점 계산 방식에 따른 오류를 낼 수 있다.

대신에 다음의 방법을 사용하자.

ex) 정적 compare 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = new Comparator<>(){
	public int compare(Object o1, Object o2){
		return Integer.compare(o1.hashCode(), o2.hashCode());
	}
};
```

ex) 비교자 생성 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o → o.hashCode());
```

**결론**

순서를 고려해야 하는 값 클래스의 경우 꼭 Comparable인터페이스를 구현하자.

compareTo 메서드에서 필드값을 비교할 땐 <. > 대신 박싱된 기본타입 클래스의 정적메서드를 사용하거나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.