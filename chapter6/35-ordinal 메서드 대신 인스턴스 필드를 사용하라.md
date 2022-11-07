## 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

ordinal 메서드 : 해당 상수가 그 열거타입에서 몇 번째 위치인지를 반환하는 메서드

순서를 매겨주는 ordinal메서드의 기능을 열거타입과 잇고 싶은 생각이 들 수 있다.

ex) ordinal메서드를 사용한 예 - 따라하지말것

```java
public enum Ensemble {
	SOLO, DUET, TRIO, QUARTET, ... , NONET, DECTET;
	
	public int numberOfMusicians(){
		return ordinal() + 1; // 1, 2, 3, ...
	}
}
```

위 코드처럼 ordinal()메서드를 사용해선 안되는 이유는 유지보수가 힘들기 때문이다.

**위 코드의 단점**

- numberOfMusicians() 메서드는 상수 선언순서에 의존적이라서 순서를 바꿀 수 없다.
- 같은 값을 반환하는 상수를 선언할 수 없다.
- 값을 건너뛸 수 없다. (Ensemble에서 TRIO가 빠진다고 가정하면..)

**해결책**

열거타입 상수에 연결된 값은 ordinal메서드의 사용 대신 인스턴스 필드에 저장하자

ex) numberOfMusicians 인스턴스필드 사용

```java
public enum Ensemble {
	SOLO(1), DUET(2), QUARTET(4), ... , NONET(9), DECTET(10);
	
	private final int numberOfMusicians;
	Ensemble(int size) {
			this.numberOfMusicians = size;
	}
	public int numberOfMusicians(){
		return numberOfMusicians;
	}
}
```

ordinal메서드의 설계 목적은 EnumSet, EnumMap 같이 열거 타입 기반의 범용 자료구조에 사용되기 위함이다.

이외의 목적에서는 ordinal메서드를 사용하지 말자.