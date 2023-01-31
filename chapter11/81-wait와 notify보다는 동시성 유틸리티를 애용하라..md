## 81. wait와 notify보다는 동시성 유틸리티를 애용하라.

wait: 갖고 있던 고유 락을 해제하고 스레드를 잠들게한다. 호출하는 스레드가 반드시 고유락을 갖고 있어야 한다(즉, synchronized 블록 내에서 호출되어야 한다)

notify : 잠들어있던 스레드 중 임의로 하나를 골라 깨운다

자바 6에 도입된 고수준의 동시성 유틸리티는 wait와 notify로 하드코딩해야했던 전형적인 일들을 대신 처리해준다.

**wait와 notify는 올바르게 사용하기가 아주 까다로우므로 고수준 동시성 유틸리티를 사용하자**

java.util.concurrent의 고수준 유틸리티는 세 범주로 나눌 수 있다.

- 실행자 프레임워크
- 동시성 컬렉션(concurrent collectiuon)
- 동기화 장치(synchronizer)

**동시성 컬렉션**

동시성 컬렉션은 List, Queue, Map같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다. 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다. 따라서 **동시성 컬렉션에서 동시성을 무력화하는것은 붌가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다**

동시성컬렉션에서는 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출하는 일 역시 불가능하다. 그래서 여러 기본 동작을 하나의 원자적 동작으로 묶는 ;상태 의존적 수정; 메서드들이 추가되었다. (이 메서드들은 유용해서 자바8에서 일반컬렉션 인터페이스에도 디폴트메서드 형태로 추가되었다)

예로 Map의 putIfAbsent(key, value).

주어진 키에 매핑된 값이 없을때만 새 값을 집어넣는다. 그리고 기존 값이 있었다면 그 값을 반환하고 없었다면 null을 반환한다.

이 메서드 덕에 스레드 안전한 정규화 맵(canonicalizing map)을 쉽게 구현할 수 있다.

ex) ConcurrentMap으로 구현한 동시성 정규화 맵 - 최적은 아님

```java
private static final ConcurrentMap<String, String> map = new ConcurrentHashMap<>();

public static String intern(String s){
	String previousValue = map.putIfAbsent(s, s);
	return prevuousValue == null? s: previusValue;
}
```

개선할 부분이 있따.

ConcurrentHashMap은 get같은 검색 기능에 최적화되어있다.

따라서 get을 먼저 호출하여 필요할때믄 putIfAbsent를 호출하면 더 빠르다.

```java
public static String intern(String s){
	String result = map.get(s);
	if(result == null){
		result = map.putIfAbsent(s, s);
		if(result == null){
			result = s;
		}
	}
	return result;
}
```

ConcurrentHashMap은 동시성이 뛰어나며 속도도 빠르다.

동시성 컬렉션은 동기화한 컬렉션보다 훨씬낫다.

이제는 **Collections.synchronizedMap보다는 ConcurrentHashMap을 사용하는게 훨씬 좋다**

컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될때까지 기다리도록 확장되었다.

예로, BlockingQueue에 추가된 메서드 중 take는 큐의 첫 원소를 꺼낸다. 만약 큐가 비어있다면 새로운 원소가 추가될 때까지 기다린다.

이런 특성 덕에 BlockingQueue는 작업 큐(생산자-소비자 큐)로 쓰기에 적합하다.

ThreadPoolExecutor를 포함한 대부분의 실행자 서비스 구현체에서 이 BlockingQueue를 사용한다.

**동기화 장치**

스레드가 다른 스레드를 기다릴 수 있게 하며 서로 작업을 조율할 수 있게 해준다.

자주 쓰이는 동기화장치는 CountDownLatch와 Semaphore이다.

가장 강력한 동기화 장치는 Phaser다.

**카운트다운 래치(latch; 걸쇠)**

카운트다운 래치는 일회성 장벽으로, 하나 이상의 스레드가 다른 하나 이상의 스레드 작업이 끝날때까지 기다리게 한다.

CountDownLatch의 유일한 생성자는 int값을 받으며, 이 값이 래치의 countDown메서드를 몇번 호출해야 대기중인 스레드들을 깨우는지를 결정한다.

예로 어떤 동작들을 동시에 시작해 모두 완료하기까지의 시간을 재는 프레임워크를 구축한다고 하자. 프레임워크는 메서드 하나로 구성되며 이 메서드는 동작들을 실행할 실행자와 동작을 몇개나 동시에 ㅅ수행할 수 있는지를 뜻하는 동시성 수준(concurrency)을 매개변수로 받는다.

타이머를 시작하기 전에 모든 작업자 스레드는 동작을 수행할 준비를 마친다.

마지막 작업자 스레드가 준비를 마치면 타이머 스레드가 작업자스레드들에게 작업 시작을 알린다.

마지막 작업자 스레드가 동작을 마치면 타이머 스레드는 시계를 멈춘다.

이를 wait, notify만으로 구현하면 난해하고 지저분하나 CounterDownLatch를 쓰면 직관적으로 구현할 수 있다.

ex) 동시 실행 시간을 재는 간단한 프레임워크

```java
public static long time(Executor executor, int concurrency, Runnable action) 
	throws InterruptedException {
	CountDownLatch ready = new CountDownLatch(concurrency);
	CountDownLatch start = new CountDownLatch(1);
	CountDownLatch done = new CountDownLatch(concurrency);

	for(int i = 0; i<concurrency; i++){
		executor.execute(() ->{
			// 타이머에게 준비를 마쳤음을 알린다.
			ready.countDown();
			try{
				//모든 작업자 스레드가 준비될 때까지 기다린다.
				start.await();
				action.run();
			} catch(InterruptedException e){
				Thread.currentThread().interrupt();
			} finally{
				//타이머에게 작업을 마쳤음을 알린다.
				done.countDown();
			}
		});
	}

	ready.await(); //모든 작업자가 준비될때까지 기다린다.
	long startNanos = System.nanoTime();
	start.countDown(); //작업자들을 깨운다.
	done.await(); //모든 작업자가 일을 끝마치기를 기다린다.
	return System.nanoTime() - startNanos;
}	
```

3개의 카운트다운 래치는 다음을 나타낸다.

ready : 작업자 스레드들이 준비가 완료됐음을 타이머 스레드에 통지할때 사용한다.

start : 마지막 작업자 스레드가 ready.countDown을 호출하면 start.coundDown() 메서드로 작업자 스레드들을 깨운다.

done : 마지막 남은 작업자 스레드가 동작을 마치고 done.countDown을 호출하면 열린다.

주의사항은

- time메서드에 넘겨진 실행자는 concurrency 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 한다. 그렇지 않으면 이 메서드는 끝나지 않을 것이다.
  이런 상태를 스레드 기아 교착상태 라고 한다.
- IterruptedException을 캐치한 작업자 스레드는 Thread.currentThread().interrupt()관용구를 사용하여 인터럽트를 되살리고 자신은 run()메서드에서 빠져나온다. 이렇게해야 실행자가 인터럽트를 적절하게 처리할 수 있다.
- 시간간격을 잴때는 항상 System.currentTimeMillis가 아닌 System.nanoTime을 사용하자. System.nanoTime은 더 정확하고 정밀하며 시스템의 실시간 시계의 시간보정에 영향을 받지 않는다.

**wait 메서드 사용**

동시성유틸리티를 사용하면 좋지만 레거시코드를 다룰때는 그렇지 못할수도 있다.

wait메서드는 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용한다.

락 객체의 wait메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 한다.

ex) wait메서드를 사용하는 표준 방식

```java
synchronized(obj){
	while(<조건이 충족되지 ㅇ낳음>){
		obj.wait(); //락을 놓고 깨어나면 다시 잡는다.
	}
	../조건이 충족됐을때의 동작 수행
}
```

**wait메서드를 사용할때는 반드시 대기 반복문(wait loop)관용구를 사용하라. 반복문 밖에서는 절대로 호출하지 말자**

대기전에 조건을 검사하여 조건이 이무 충족되었다면 wait를 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치이다. 조건이 충족되었는데 스레드가 notify메서드를 먼저 호출한 후 대기상태로 빠지면 그 스레드를 다시 깨울 수 있다고 보장할 수 없다.

대기후에 조건을 검사하여 조건ㅇ ㅣ충족되지 않았다면 다시 대기하게 하는 것은 안전 실패를 막는 조치이다. 만약 조건이 충족되지 않았는데 스레드가 동작을 이어가면 락이 보호하는 불변식을 깨뜨릴 위험이 있다.

조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황이 몇가지 있다.

- 스레드가 notify를 호출한 다음 대기중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다.
- 조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 notify를 호출한다.
- 깨우는 스레드는 지나치게 관대해서 대기중인 스레드 중 일부만 조건이 충족되어도 notifyAll을 호출해 모든 스레드를 깨울 수도 있따.
- 대기중인 스레드가 notify없이도 깨어나는 경우가 있다 ⇒ 허위각성(spurious wakeup);

notify(스레드 하나만깨움)보다는 notifyAll(모든 스레드 깨움)을 사용하는것이 합리적이고 안전하다. 깨어나야하는 모든 스레드가 깨어남을 보장한다.

**결론**

코드를 새로 작성한다면 wait, notify를 사용할 이유가 없다.

이들을 사용하는 레거시 코드를 유지보수해야한다면 wait는 항상 표준 관용구에 따라 while문 안에서 호출하도록 하자.