## 11. equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의한 클래스 모두에서 hashCode도 재정의해야한다

그렇지 않으면 hashCode 일반규약을 어기게 되고 해당 클래스의 인스턴스를 HashMap이나 HashSet같은 컬렉션의 원소로 사용할 때 문제가 발생한다.

### HashCode 규약

- equals 비교에 사용되는 정보가 변경되지 않았다면 그 객체의 hashCode메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
- equals(Object)가 두 객체를 다르다고 판단했더라도 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 하지만 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

hashCode 재정의를 잘못했을 때 문제가 되는 규약은 두 번째 규약이다.

**논리적으로 같은 객체는 같은 해시코드를 반환해야 한다**

만약 논리적으로 같은 경우 true를 반환하도록 equals를 재정의했을 때

hashCode를 재정의하지 않는다면 HashMap등에서 원하는 결과를 얻을 수 없을 것이다.

hashCode를 재정의해보자.

ex) 최악의 방법

```java
@Override public int hashCode(){
	return 42;
}
```

위 경우는 동치인 모든 객체에 똑같은 해시코드를 반환해주기 때문에 적합하다.

하지만 모든 객체에게 독같은 값을 내어주므로 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결리스트처럼 동작하고 해시테이블의 수행시간이 O(N)으로 느려지게 된다.

따라서 서로 다른 인스턴스에 대해서는 다른 해시코드를 반환하도록 해야한다. (3 번째 규약의 내용)

### 좋은 hashCode를 작성하는 요령

1. int 변수 result를 선언 후 값 c로 초기화한다.
   이때 c는 해당 객체의 첫번째 핵심 필드(equals()비교에 사용되는 필드)를 2.a의 방식으로 계산한 해시코드이다.
2. 해당 객체의 나머지 핵심필드 f각각에 대해 다음 작업을 수행한다.
    1. 해당 필드의 해시코드 c를 계산한다.
        1. 기본타입필드라면 Type.hashCode(f)를 수행한다. (Type은 박싱클래스)
        2. 참조타입필드라면 hashCode() 호출. null이면 0을 사용한다.( 일반적으로 0을 사용)
        3. 배열이라면 핵심원소 각각을 별도의 필드처럼 다룬다.
           위 규칙을 적용해서 해시코드를 계산하고 2.b방식으로 갱신한다.
           원소가 없다면 0을 사용하고 모든 원소가 핵심원소라면 Arrays.hashCode를 사용한다.
    2. 2.a에서 계산한 해시코드 c로 result를 갱신한다.
       result = 31 * result + c;
3. result를 반환한다.

여기서 파생필드(다른 필드로부터 계산해낼 수 있는 필드)는 해시코드 계산에서 제외해도 된다.

또한 equals비교에 사용되지 않는 필드는 반드시 제외해야 한다.(두 번째 규약을 지키기 위해)

31을 곱한 이유

먼저 곱셈을 추가한 이유는 String을 예로 들면 모든 아나그램(anagram, 구성하는 철자가 같고 그 순서만 다른 문자열)의 해시코드가 같아짐.

31로 곱하는 이유는 소수이기 때문. 짝수거나 오버플로가 발생할 경우 정보를 읽게 된다. 그리고 전통적으로 소수를 이용해왔기 때문.

결과적으로 31을 이용하면 곱셈을 시프트연산과 뺄셈으로 대체해 최적화할 수 있다. (31 * i 는 (i << 5) - i와 같음) 요즘 VM은 이런 작업을 자동으로 해준다.

ex) 전형적인 hashCode 메서드

```java
@Override
public int hashCode(){
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashcode(lineNum);
	return result;
}
```

직접 구현하지 않고 라이브러리를 사용해도 된다. (ex, Guava)

만약 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 그리고 해당 객체가 해시의 키로 자주 사용될 것 같다면 해시코드를 캐싱하는 방식을 고려해야 한다.

ex) 해시코드를 지연초기화하는 hashCode 메서드 - 스레드 안정성까지 고려해야한다.

```java
private int hashCode;

@Override
public int hashCode(){
	int result = hashCode;
	if(result == 0){
		result = Short.hashCode(areaCode);
		result = 31 * result + Short.hashCode(prefix);
		result = 31 * result + Short.hashCode(lineNum);
		hashCode = result;
	}
	
	return result;
}
```

### 해시코드 작성 시 유의점

- 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.
  해시품질이 나빠져서 해시테이블의 성능을 떨어뜨릴 수도 있다. (군집화 문제)
- hashCode가 반환하는 값의 생성 규칙을 API사용자에게 자세히 공표하지 말자.
  그래야 클라이언트가 이 값에 의지하지 않게 되고 추후 계산 방식을 바꿀 수도 있다.