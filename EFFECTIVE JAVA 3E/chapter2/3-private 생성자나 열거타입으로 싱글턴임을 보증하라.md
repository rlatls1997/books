## 3. private 생성자나 열거타입으로 싱글턴임을 보증하라

싱글턴 : 인스턴스를 오직 하나만 생성할 수 있는 클래스

싱글턴 예시로는 함수와 같은 무상태 객체나 설계상 유일해야 하는 시스템 컴포넌트가 있따.

하지만 클래스를 싱글턴으로 만들면 이를 사용하는 칼라이언트를 테스타하기가 어려워질 수 있다. 만약 어떤 타입을 인터페이스로 정의한다음 그 인터페이스를 구현해서 만든 싱글턴이 아닐 경우 싱글턴 인스턴스를 mock구현으로 대체할 수 없기 때문이다.

실글턴을 만드는 두 방식 모두 생성자는 private으로 감춰두고

유일한 인스턴스에 접근할 수 있는 수단을 제공한다.

ex) 첫 번째 방법. public static 멤버를 하나 마련해둔다.

```java
public class Elvis {
		public static final Elvis INSTANCE = new Elvis();
		
		private Elvis() {...}
		public void leaveTheBuilding() {...}
}
```

private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 때 딱 한번만 호출된다.

public이나 protected 생성자가 없으므로 인스턴스가 전체시스템에서 하나뿐임이 보장된다.

예외가 하나 있긴 하다. 리플렉션 API인 AccessibleObject.setAccessible을 사용해서 private생성자를 호출할 수 있다. 이 경우도 생성자를 수정하여 두 번째 객체가 생성되려고 할 때 예외를 던지게 하면 된다.

ex) 두 번째 방법. 정적 팩터리 메서드를 public static멤버로 제공

```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() {...}
	public static Elvis getInstance() {return INSTANCE;}
	
	public void leaveTheBuilding() {...}
}
```

마찬가지로 Elvis.getInstance는 항상 같은 객체의 참조를 반환하기 때문에 싱글턴이다.

첫번째 방법(public 필드 방식)의 장점

- 해당 클래스가 싱글턴임이 API에 명백히 드러난다
- 간결하다

두 번째 방법의 장점(private필드에 접근할 수 있는 팩터리 메서드 제공)

- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
- 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.(아이템 30 참고)
- 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.
  예로 Elvis::getInstance를 Supplier<Elvis>로 사용하는 식이다.

이런 장점이 필요없을 땐 첫번째 방법이 좋다.

둘 중 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면,

모든 인스턴스 필드를 일시적(transient)라고 선언하고 readResolve메서드를 제공해야 한다.

이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.

가짜 Elvis인스턴스의 생성을 막으려면 readResolve메서드를 추가하면된다.(이게 뭔데)

ex)

```java
private Object readResolve(){
		// 진짜 Elvis를 반환하고 가짜 elvis는 가비치컬렉테에 맡긴다.
	return INSTANCE;
}
```

싱글턴을 만드는 세번째 방법은 원소가 하나인 열거타입을 선언하는 것이다.

ex) 열거 타입 방식의 싱글턴 - 바람직한 방법

```java
public enum Elvis{
	INSTANCE;
	
	public void leaveTheBuilding() { ... }
}
```

public 필드 방식과 비슷하지만 더 간결하고 추가 노력없이 직렬화할 수 있다.

또한 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽히 막아준다.

대부분의 상황에서는 원소가 하나뿐인 열거타입이 싱글턴을 만드는 가장 좋은 방법이다.

(만드려는 싱글턴이 Enum외의 클래스를 상속해야 하는 경우 이 방법은 사용할 수 없다)