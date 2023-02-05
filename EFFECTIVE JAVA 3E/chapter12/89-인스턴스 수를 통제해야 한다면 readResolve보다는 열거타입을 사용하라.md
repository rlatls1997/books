## 89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거타입을 사용하라

싱글턴패턴은 생성자를 호출하지 못하게 막는 방식으로 인스턴스가 오직 하나만 만들어짐을 보장한다.

ex) 싱글턴패턴

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() {...}
	...
}
```

싱글턴패턴을 적용한 이 클래스에 Serializable을 구현하면 그 순간부터 이 클래스는 싱글턴이 아니게 된다.

**readResolve**

readResolve기능을 이용하면 readObject가 만들어낸 인스턴스를 다른 것으로 대체할 수 있다.

역직렬화한 객체의 클래스가 readResolve메서드를 적절히 정의해뒀다면, 역직렬화 후 새로 생성된 객체를 인수로 readResolve메서드가 호출되고, 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해서 반환된다.

ex) 인스턴스 통제를 위한 readResolve - 개선이 필요하다

```java
private Object readResolve(){
	// 진짜 Elvis를 반환하고 가짜 Elvis는 가비지 컬렉트
	return INSTNACE;
```

이 메서드는 역직렬화한 객체를 무시하고 클래스 초기화때 만들어진 Elvis인스턴스를 반환한다.

따라서 Elvis인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 이유가 없으므로 모든 인스턴스 필드를 transient로 선언해야 한다.

**사실 readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다.** 그렇지 않으면 역직렬화된 객체의 참조를 공격할 여지가 남기 때문이다.

싱글턴이 transient가 아닌 참조 필드를 가지고 있다면 그 필드의 내용은 readResolve메서드가 실행되기 전에 역직렬화된다. 그렇다면 해당 참조 필드의 내용이 역직렬화되는 시점에 그 역직렬화된 인스턴스의 참조를 훔쳐올 수 있다.

**예시**

도둑(strealer)클래스가 있다.

도둑클래스는 readResolve메서드와 인스턴스 필드 하나를 포함한다.

인스턴스필드는 도둑이 숨길 직렬화된 싱글턴을 참조하는 역할을 한다.

직렬화된 스트림에서 싱글턴의 비휘발성 필드를 이 도둑의 인스턴스로 교체한다.

이제 싱글턴은 도둑을 참조하고 도둑은 싱글턴을 참조하는 순환고리가 만들어졌다. (??)

싱글턴이 도둑을 포함하므로 싱글턴이 역직렬화될 때 도둑의 readResolve메서드가 먼저 호출된다.

도둑의 readResolve메서드가 수행될때 도둑의 인스턴스 필드에는 역직렬화를 진행중인(readResolve가 수행되기 전인) 싱글턴의 참조가 담겨있게 된다.

도둑의 readResolve메서드는 이 인스턴스 필드가 참조한 값을 정적 필드로 복사하여 readResolve가 끝난 후에도 참조할 수 있도록 한다.

그 다음 이 메서드는 도둑이 숨긴 transient가 아닌 필드의 원래 타입에 맞는 값을 반환한다.

- transient가 아닌 참조필드를 가진 잘못된 싱글턴

```java
public class Elvis implements Serializable {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() {}

	private String [] favoriteSong = {"Hound Dog", "Heartbreak Hotel"};

	public void printFavorites(){
		sout(Arrays.toString(favoriteSongs));
	}

	private Object readResolve(){
		return INSTNACE;
	}
}
```

- 도둑 클래스

```java
public class ElvisStrealer implements Serializable{
	static Elvis impersonator;
	private Elvis payload;

	private Object readResolve(){
		// resolve가 되기 전의 Elvis인스턴스의 참조를 저장한다.
		impersonator = payload;
		
		// favoriteSongs 필드에 맞는 타입의 객체를 반환한다.
		return new String[] {"A Fool such as I");
	}
	
	private static final long serialVersionUID = 0;
}
```

- 직렬화의 허점을 이용한 싱글턴객체 2개 생성하기

```java
public class ElvisImpersonator {
	private static final byte[] serializedForm = { ... };

	public static void main(String[] args){
		//ElvisStrealer.impersonator를 초기화하고 진짜 Elvis를 반환한다.
		Elvis elvis = (Elvis) deserialize(serializedForm);
	
		Elvis impersonator = ElvisStrealer.impersonator;

		elvis.printFavorites(); // "Hound Dog", "Heartbreak Hotel"
		impersonator.printFavorites(); //"A Fool such as I"
	}
}
```

**싱글턴이 2개가 생성되는 과정**

1. 직렬화된 싱글턴 바이트 스트림을 조작하여 인스턴스 필드(favoriteSong)을 도둑클래스 인스턴스(ElvisStrealer)로 교체한다.
2. 1번 과정을 거친 바이트스트림을 역직렬화하면 favoriteSong에는 도둑클래스 인스턴스가 들어가 있으므로 도둑클래스 인스턴스도 같이 역직렬화가 진행된다.
3. 도둑클래스 역직렬화 과정에서 readResolve메서드가 호출된다.
4. readResolve메서드에서 impersonator에 역직렬화중인 싱글턴의 참조가 담기게 된다.
5. readResolve메서드에서 원래의 인스턴스필드 favoriteSong의 타입에 맞는 값을 반환하므로 ClassCastException이 발생하지 않고 favoriteSong필드에 그대로 대체된다.
6. 이렇게 favoriteSong필드가 대체된 Elvis인스턴스의 참조는 ElvisStrealer.impersonator를 통해 가져올 수 있다.

**열거타입으로 변경**

직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해서 구현하면 선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장해준다.

ex) 열거타입 싱글턴

```java
public enum Elvis {
	INSTANCE;
	private String [] favoriteSgongs = {"Hound Dog", "Heartbreak Hotel"};

	public void printFavorites(){
		sout(Arrays.toString(favoriteSongs));
	}
}
```

**readResolve메서드의 접근성**

final 클래스라면 readResolve메서드는 private 이어야한다.

final 클래스가 아니라면 다음을 고려해야한다.

private으로 선언하면 하위 클래스에서 사용할 수 없다.

package-private으로 선언하면 같은 패키지에 속한 하위클래스에서만 사용할 수 있다.

protected나 public으로 선언하면 이를 재정의하지않은 모든 하위클래스에서 사용할 수 있다.

protected나 public이면서 하위클래스에서 재정의하지 않았다면 하위클래스의 인스턴스를 역직렬화하면 상위클래스의 인스턴스를 생성하여 ClassCastException을 일으킬 수 있다.