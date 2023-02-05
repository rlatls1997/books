## 88. readObject 메서드는 방어적으로 작성하라

다음처럼 가변인 Date필드에 대해 불변식을 지키고 불변식을 유지하기 위해서 방어적 복사랄 사용하는 불변클래스가 있다.

ex) 방어적 복사를 사용하는 불변클래스

```java
public final class Period {
	private final Date start;
	private final Date end;

	public Period(Date start, Date end){
		this.start = new Date(start.getTime());
		this.end = new Date(end.getTime());

		if(this.start.compareTo(this.end) > 0){
			throw new IllegalArgumentException(start+ " 가"+ end + "보다 늦다.");
		}
	}

	public Date start() {
		return new Date(start.getTime());
	}

	public Date end() {
		return new Date(end.getTime());
	}

	public String toSTring() {
		return start + " - " + end;
	}
}
```

이 클래스를 직렬화하기로 해보자.

Period객체의 물리적 표현이 논리적 표현에 부합하므로 기본 직렬화를 사용해도 괜찮을 것 같고 Serializable만 구현해주면 될 것 같지만 이렇게 하면 불변식이 보장되지 못한다.

이유는 readObject메서드가 실질적으로 또 다른 public 생성자로 볼 수 있기 때문이다. 따라서 다른 생성자와 같은 수준으로 주의를 기울여야 한다.

불변식을 깨뜨릴 의도를 가진 임의의 바이트스트림을 건네면 정상적인 생성자를 우회하여 객체를 생성할 수 있기 때문이다.

위 클래스에선 end Date가 start Date보다 빠른 Period인스턴스를 생성할 수 없지만 Serializable 인터페이스를 구현한 순간부터 이 규칙을 깬 인스턴스를 생성할 수 있다.

```java
public class BogusPeriod {
	// 실제론 만들어 질 수 없는 케이스의 Period인스턴스 바티으트스림
	private static final byte[] serializedForm = { ......};

	public static void main(String [] args){
		Period p = (Period) deserialize(serializedForm);
		sout(p);
	}

	// 바이트스트림으로부터 객체를 만들어 반환한다.
	static Object  deserialize(byte[] sf){
		try{
			return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
		} catch (IOException | ClassNotFoundException e){
			throw new IllegalArgumentException(e);
		}
	}
}

```

Period에 Serilizable을 구현하면 위 코드로 Period 클래스의 불변식을 깬 인스턴스 객체를 만들 수 있다.

**불변식 저해 문제 해결하기**

이 문제를 방지하기 위해 Period의 readObject 메서드가 defaultReadObject를 호출한다음 역직렬화된 객체가 유효한지 검사해야한다.

ex) 유효성 검사를 수행하는 readObject 메서드 - 아직 부족하다

```java
private void readObject(ObjectInputStream) throws IOException, ClassNotFoundException {
	s.defaultReadObject();
	
	// 불변식을 만족하는지 검사한다
	if(start.compareTo(end) > 0){
		throw new InvalidObjectException (start+"가 "+ end+ "보다 늦다.");
	}
}
```

이렇게 하면 허용되지 않은 Period인스턴스를 생성하는 일을 막을 수 있다.

**남아있는 문제**

아직 문제가 있는데, 정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어낼 수 있다.

공격자는 ObjectInputStream에서 Period인스턴스를 읽은 후 스트림 끝에 추가된 악의적인 객체참조를 읽어서 Period 객체의 내부 정보를 얻을 수 있다.

이 참조로 얻은 Date인스턴스들은 수정될 수 있으므로 불변식이 깨지게 된다.

(예시코드 p.470참고)

결론은, **객체를 역직렬화할 때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 방어적으로 복사해야 한다**는 것이다.

ex) 불변식, 불변 성질을 지키는 readObject메서드

```java
private void readObject(ObjectInputStream) throws IOException, ClassNotFoundException {
	s.defaultReadObject();
	
	//가변요소들을 방어적으로 복사
	start = new Date(start.getTime());
	end = new Date(end.getTime();

	// 불변식을 만족하는지 검사한다
	if(start.compareTo(end) > 0){
		throw new InvalidObjectException (start+"가 "+ end+ "보다 늦다.");
	}
}
```

참고할 점은,

final필드는 방어적 복사가 불가능하므로 readObject메서드를 사용하려면 final한정자를 제거해야한다.