## 8. finalizer와 cleaner 사용을 피하라

자바는 두 가지 객체소멸자를 제공한다.

그중 **finalizer는 예측할 수 없고 상황에 따라 위험할 수 있어서 일반적으로 불필요하다**
오동작, 낮은 성능, 이식성 문제의 원인이 되기도 한다. 그래서 자바 9부터는 deprecated api로 지정되었고 cleaner를 대안으로 소개한다.

**cleaner는 fianlizer보단 덜 위험하지만 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.**

finalizer와 cleaner는 즉시 수행된다는 보장이 없다.

객체에 접근할 수 없게 된 후 finalizer나 cleaner가 실행되기까지 얼마나 걸릴지 알 수 없다.

**즉 finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다.**

- Ssstem.gc나 System.runFinalization 메서드는 finalizer와 cleaner가 실행될 가능성을 높여주지만 보장하지 않는다.

실행을 보장하는 메서드가 있지만(System.runFinalizersOnExit, Runtime.,runFinalizersOnExit) 결함이 있어서 사용되지 않는다(ThreadStop)

- finalizer동작 중 발생한 예외는 무시되며 처리할 작업이 남았더라도 그 순간 종료된다. 스택트레이스도 남기지 않고 경고조차 출력하지 않는다. (cleaner는 자신의 스레드를 통제하기 때문에 이 문제에선 괜찮다.)

- finalizer와 cleaner는 심각한 성능 문제를 갖고 있다. 어떤 객체를 생성하고 가비지 컬렉터가 수거하기까지 비교적 오랜 시간이 걸린다.

- finalizer를 사용한 클래스는 finalizer공격에 노출되어 심각한 보안 문제를 일으킬수도 있따.

finalizer공격원리는 생성자나 직렬화 과정에서 예외가 발생하면,
이 생성되다가 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 한다.
이 상황은 발생해선 안된다. (?? )

**객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만 finalizer가 있다면 그렇지도 않다**(???)

**final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalizer메서드를 만들고 final로 선언하자**(???)

그렇다면 finalizer와 cleaner의 대안은??
⇒ **AutoCloseable을 구현해주고 클라이언트에서 인스턴스를 다 쓰고 나면 close메서드를 호출하면 된다**(일반적으로 예외가 발생해도 제대로 종료되도록 try-with-resources를 사용해야한다)

finalizer와 cleaner가 아주 쓸모없는 것은 아니다.
호출되리라는 보장도 없고 호출된다 하더라도 즉시 호출된다는 보장도 없지만,
클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 것이 아예 안하는 것보다 나으니 안전망 역할로 사용할 때도 있다.

또 네이티브 피어와 연결된 객체에서 사용할 수 있따. 네이티브 피어는 자바 객체가 아니기 때문에 gc가 존재를 알지 못하고, 자바 피어를 회수할 때 네이티브 객체까지 회수하지 못한다.

성능저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 갖고 있지 않다면 cleaner나 finalizer로 처리할 수 있다.

네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 close메서드를 사용해야 한다.

ex) cleaner를 안전망으로 활용하는 AutoCloseable 클래스

```java
import java.lang.ref.Cleaner;

public class Room implements AutoCloseable{

	private static final Cleaner cleaner = Cleaner.create();

	private static class State implements Runnable{
		int numJunkPiles; // 방 안의 쓰레기 수

		State(int numJunkPiles){
			this.numJunkPiles = numJunkPiles;
		}

		@Override
		public void run() {
			System.out.println("방 청소");
			numJunkPiles = 0;
		}
	}

	// 방의 상태. cleanable과 공유한다.
	private final State state;

	// cleanable객체. 수거 대상이 되면 방을 청소한다.
	private final Cleaner.Cleanable cleanable;

	public Room(int numJunkPiles){
		state = new State(numJunkPiles);
		cleanable = cleaner.register(this, state);
	}

	@Override
	public void close() throws Exception {
		cleanable.clean();
	}
}
```

run메서드가 호출되는 경우는 다음과 같다.

- Room의 close()메서드가 호출될 때.
  close()메서드 내부의 cleanable.clean() 메서드 안에서 run() 메서드가 호출된다.
- 가비지컬렉터가 Room을 회수할 때까지 close()를 호출하지 않는다면 cleaner가 State의 run메서드를 호출한다. (그니까 이 경우가 안전망인거다.)

이 때 State인스턴스는 절대로 Room인스턴스를 참조해서는 안되는데 만약 참조할 경우 순환참조가 생겨서 가비지컬렉터가 Room인스턴스를 회수해갈 기회가 오지 않는다.

State 정적 중첩 클래스인 이유도 이와 같다. 정적이 아닌 중첩클래스는 자동으로 바깥 객체의 참조를 갖게 되기 때문이다. (람다도 바깥 객체의 참조를 갖기 쉽다)

만약 클라이언트가 모든 Room생성을 try-with-resources블록으로 감쌌다면 자동 청소는 전혀 필요하지 않다.

ex) 잘 짜인 클라이언트 코드

```java
public class Adult{
	psvm(String [] args){
		try(Room myRoom = new Room(7)){
			sout("안녕");
		}
	}
}
```

위 코드는 “안녕”을 출력한 후 “방 청소”를 출력한다.

ex) clean이 되지 않은 경우

```java
public class Teenager{
	psvm(String [] args){
		new Room(99);
		sout("안녕");
	}
}
```

“안녕”이 출력되고 “방 청소”는 출력되지 않는다. (예측할 수 없는 상황)

System.exit 이 호출될 때도 clean은 보장되지 않는다. (System.gc()를 호출해도 마찬가지)

**결론**

cleaner, finalizer는 안전망 역할, 중요하지 않는 네이티브 자원 회수 용으로만 사용하자.

불확실성과 성능 저하에 유의하자.