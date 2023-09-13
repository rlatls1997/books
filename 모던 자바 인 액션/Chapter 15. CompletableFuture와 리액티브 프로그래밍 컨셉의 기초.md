# Chapter 15. CompletableFuture와 리액티브 프로그래밍 컨셉의 기초
**내용**

- Thread, Future, 자바가 풍부한 동시성 API를 제공하도록 강요하는 진화의 힘
- 비동기 API
- 동시 컴퓨팅의 박스와 채녈 뷰
- CompletableFuture 콤비네이터로 박스를 동적으로 연결
- 리액티브 프로그래밍용 자바 9 플로 API의 기초를 이루는 발행 구독 프로토콜
- 리액티브 프로그래밍과 리액티브 시스템

## 15.1 동시성을 구현하는 자바 지원의 진화

초기 자바는 Runnable과 Thread를 동기화된 클래스와 메서드를 이용해서 잠갔다.

자바 5는 조금 더 표현력있는 동시성을 지원하는, 특히 스레드 실행과 태스크 제출을 분리하는 ExecutorService인터페이스, 높은 수준의 결과 즉, Runnable, Thread의 변형을 반환하는 Callble<T>, Future<T>, 제네릭 등을 지원했다.

ExecutorServices는 Runnable과 Callable 둘 다 실행할 수 있다. 이후 멀티코어 CPU에서 쉽게 병렬 프로그래밍을 구현할 수 있게 되었다.

멀티코어 CPU를 효과적으로 활용하기 위해 자바 7에서는 포크/조인 구현을 지원하는 RecursiveTask가 추가되었고 자바 8에서는 스트림과 새로 추가된 람다 지원에 기반한 병렬 프로세싱이 추가됨.

자바는 Future를 조합하는 기능을 추가하면서 동시성을 강화했고,

자바 9에서는 분산 비동기 프로그래밍을 명시적으로 지원함.

이 API들은 다양한 웹 서비스를 이용한 매쉬업 애플리케이션에서 실시간으로 정보를 조합하여 제공하는 등의 작업을 할 때 필수적이다.

이러한 과정을 리액티브 프로그래밍이라고 부르며 자바 9에서는 발행-구독 프로토콜(Flow 인터페이스)로 이를 지원한다

CompletableFuture와 java.util.concurrent.Flow의 목표는 동시에 실행할 수 있는 독립적인 태스크를 가능하게 만들면서 멀티코어를 사용한 병렬성을 쉽게 이용하는 것이다.

### 15.1.1 스레드와 높은 수준의 추상화

딘일 CPU이더라도 여러개의 프로세스에 CPU자원을 번갈아서 할당하여 멀티 프로세싱이 가능하다.

각 프로세스는 해당 프로세스와 같은 주소 공간을 공유하는 한 개 이상의 스레드로 작업을 요청하여 태스크를 동시에, 또는 협력적으로 실행할 수 있다.

멀티코어 CPU에서 각 코어는 한 개 이상의 프로세스나 스레드에 할당될 수 있지만 프로그램이 스레드를 사용하지 않는다면 효율성을 고려해서 여러 프로세서 코어 중 한 개만을 사용할 것이다.

스트림을 이용해서 명시적으로 사용하는 대신 병렬 스트림을 사용하면 내부 반복을 통해 에러 없이 동시성을 이뤄낼 수 있다.

스트림을 이용해서 스레드 사용 패턴을 추상화할 수 있다.

### 15.1.2 Executor와 스레드 풀

자바 5의 Executor 프레임워크와 스레드 풀 : 프로그래머가 태스크 제출(?)과 실행을 분리할 수 있는 기능을 제공하여 스레드의 힘을 높은 수준으로 끌어올릴 수 있게 해준다.

**스레드의 문제**

자바 스레드는 직접 운영제체 스레드에 접근한다.

운영체제 스레드를 만들고 종료하려면 비싼 비용을 치러야 하며 운영체제 스레드의 숫자는 제한적이라는 문제가 있다.

운영체제가 지원하는 스레드의 수를 초과하여 사용하면 자바 애플리케이션이 예상치 못하게 크래시될 수 있으므로 기존 스레드를 유지한 채 새로운 스레드를 만드는 상황이 발생하지 않도록 해야한다.

보통 운영체제와 자바의 스레드 개수가 하드웨어 스레드 개수보다 많으므로,

일부 운영체제 스레드가 블록되거나 자고있는 상황에서도 모든 하드웨어 스레드는 코드를 실행중인 상황에 놓일 수 있다.

하드웨어 코어의 개수에 따라 최적의 자바 스레드 개수는 달라질 수 있기 때문에 다양한 기기에서 프로그램이 동작하는 경우에는 하드웨어 스레드 개수를 미리 추측하지 않는 것이 좋다.

**스레드 풀 그리고 스레드 풀이 더 좋은 이유**

ExecutorService는 태스크를 제출하고 나중에 결과를 수집할 수 있는 인터페이스를 제공한다.

프로그램은 newFixedThreadPool 같은 팩토리 메서드 중 하나를 이용하여 스레드 풀을 만들어 사용할 수 있다.

```java
ExecutorService newFixedThreadPool(int nThreads)
```

위 메서드는 워커 스레드라고 불리는 nThraed를 포함하는 ExecutorService를 만들고 이들을 스레드 풀에 저장한다.

스레드 풀에서 사용하지 않은 스레드로 제출된 태스크를 먼저 온 순서대로 실행한다.

이들 태스크 실행이 종료되면 스레드를 스레드풀로 반환한다.

위 방식의 장점은, 하드웨어에 맞는 수의 태스크를 유지함과 동시에 수 천개의 태스크를 스레드 풀에 아무 오버헤드 없이 제출할 수 있다는 점이다. 큐의 크기 조정, 거부 정책, 우선순의 등 다양한 설정도 가능하다.

프로그래머가 태스크(Runnable or Callable)을 제공하면 스레드가 이를 실행한다.

**스레드 풀 그리고 스레드 풀이 나쁜 이유**

스레드를 직접 사용하는 것보다 스레드 풀이 더 좋아보인다.

하지만 주의해야할 부분이 있다.

- 잠을 자거나 I/O작업 또는 네트워크 작업을 하여 결과를 기다리는 태스크가 있다면 주의해야 한다. 이들 태스크는 워커 스레드에 할당된 상태를 유지하지만 아무런 작업도 수행하지 않는다.

이러한 경우 작업효율이 크게 떨어질 수 있고 태스크끼리 서로의 결과에 의존적인 경우 데드락에 걸릴 수도 있다. block(자거나 이벤트를 기다리는)할 수 있는 태스크는 스레드 풀에 제출하지 않는 것이 좋다.
- 자바 프로그램은 main이 반환되기 전 중요한 코드를 실행하는 스레드가 죽는 일이 없도록 모든 스레드의 작업이 끝나길 기다린다. 따라서 프로그램을 종료하기 전 모든 스레드 풀을 종료하는 것은 중요하다.

### 15.1.3 스레드의 다른 추상화 : 중첩되지 않은 메서드 호출

엄격한 포크/조인 : 태스크나 스레드가 메서드 호출 안에서 시작되면 그 메서드 호출은 반환하지 않고 작업이 끝나기를 기다린다.

여유로은 포크/조인 : 시작된 태스크를 내부 호출이 아니라 외부 호출에서 종료하도록 기다리는 조금 더 여유로운 방식. 여유로운 포크/조인을 사용해도 비교적 안전한다.

비동기 메서드 : 메서드 호출자에게 기능을 제공하도록 메서드가 반환된 후에도 만들어진 태스크 실행이 계속되는 메서드

**비동기 메서드의 위험성**

비동기 메서드를 사용할 때는 다음의 위험성이 따를 수 있다.

- 스레드 실행은 메서드를 호출한 다음의 코드와 동시에 실행되므로 데이터 경쟁 문제를 일으키지 않도록 주의해야 한다.
- 실행중이던 스레드가 종료되지 않은 상태에서 main() 메서드가 반환되면 다음과 같은 두 가지 방법으로 처리할 수 있다.
    - 애플리케이션을 종료하지 못하고 모든 스레드가 실행을 끝낼 때까지 기다린다.
    - 애플리케이션 종료를 방해하는 스레드를 강제종료하고 애플리케이션을 종료한다.

  하지만 두 방법 모두 안전하지 못하다.

  첫 번째 방법은 종료를 못한 스레드에 의해 애플리케이션이 크래시될 수 있고, 스레드를 강제종료하면 I/O, DB관련 작업등을 할 때 데이터일관성이 파괴될 수 있다.

  따라서 스레드풀을 포함한 모든 스레드를 종료한 후 애플리케이션을 종료하는 것이 안전하다.


자바 스레드는 setDaemon()메서드를 이용하여 데몬 또는 비데몬으로 구분시킬 수 있다.

- 데몬 스레드 : 애플리케이션이 종료될 때 강제 종료되므로 디시크(디스크?)의 데이터 일관성을 파괴하지 않는 동작을 수행할 때 유용하게 활용가능
- 비데몬 스레드 : main() 메서드가 종료되기전 정상적으로 종료되어야 하는 스레드. 비데몬 스레드가 종료되기 전까지는 프로그램이 종료되지 못하고 기다리게 된다.

### 15.1.4 스레드에 무엇을 바라는가?

모든 하드웨어 스레드를 활용하여 병렬성의 장점을 극대화하는 프로그램 구조를 만드는 것이 목표이다. 프로그램을 작은 태스크 단위로 구조화하는 것이 목표.

## 15.2. 동기 API와 비동기 API

자바 8 스트림을 이용하여 명시적으로 병렬 하드웨어를 이용할 수 있다.

먼저 외부 반복(명시적 for 루프)을 내부 반복(스트림 메서드 사용)으로 바꾸고 스트림에 parallel()메서드를 이용하여 자바 런타임 라이브러리가 복잡한 스레드 작업을 하지 않고 병렬로 요소가 처리되도록 할 수 있다.

루프 계산 외의 ㄷ른 상황에서도 병렬성이 유용할 수 있따.

예로 다음의 시그니처를 갖는 두 메서드의 반환값을 합하는 예제를 살펴보자.

```java
// 메서드 시그니처
int f(int x);
int g(int x);

// 두 메서드의 반환값을 합하는 예제
int y = f(x);
int z = g(x);
sout(y+z);
```

메서드 f와 g를 완전히 수행하는데에 오랜 시간이 걸린다고 가정한다.

만약 f와 g가 서로 상호작용하지 않는다는 사실을 알고 있다면 f와 g를 별도의 CPU코어로 실행함으로 f와 g중 오래 걸리는 작업의 시간으로 합을 구하는 시간을 단축할 수 있따.

(f는 5초, g는 10초라고 가정하면 합을 구하는데에 걸리는 시간은 10초)

별도의 스레드로 f와 g를 실행하여 이를 구현할 수 있으나 다음처럼 코드가 복잡해진다.

```java
class ThreadExample{
	public static void main(String[] args) thorws InterruptedException{
		int x = 1337;
		Result result = new Result();
	
		Thread t1 = new Thread(() -> {result.left = f(x);});
		Thread t2 = new Thread(() -> {result.right = g(x);});

		t1.start();
		t2.start();
	
		t1.join();
		t2.join();
	
		sout(result.left + result.right);
	}	
	
	private static class Result {
		private int left;
		private int right;
	}
}
```

Runnable대신 Future API인터페이스를 사용하면 코드를 단순화할 수 있따.

이미 ExecutorService로 스레드풀을 설정했다고 가정하면 다음처럼 코드를 구현할 ㅜㅅ 있다.

```java
public class ExecutorServiceExample{
	public static void main(String[] args) throws ExecutionException, InterruptedException{
		int x = 1337;
		
		ExecutorService execitorService = Executors.newFixedThreadPool(2);
		
		Future<Integer> y = executorService.submit((0 -> f(x));
		Future<Integer> z = executorService.submit((0 -> g(x));

		sout(y.get() + z.get());
	
		executorService.shutdown();
	}
}
```

하지만 위 코드도 명시적인 submit 메서드 호출 같은 불필요한 코드로 오염되었다.

명시적 반복으로 병렬화를 수행하던 코드를 스트림을 이용해여 내부 반복으로 바꾼것처럼 문제를 해결해야 한다.

이러한 문제는 **비동기 API** 기능으로 API를 바꿔서 해결할 ㅜㅅ 있다.

### 15.2.1 Future 형식 API

대안을 이용하면 f, g의 시그니처가 다음처럼 바뀐다.

```java
Future<Integer> f(int x);
Future<Integer> g(int x);
```

그리고 다음처럼 호출이 바뀐다.

```java
Future<Integer y = f(x);
Future<Integer z = g(x);

sout(y.get() + z.get());
```

메서드 f와 g는 호출 즉시 바디를 수행하는 태스크를 포함하는 Future를 반환한다.

그리고 get() 메서드를 이용하여 두 Future의 결과가 합쳐지기를 기다린다.

예제에서는 API를 그대로 유지하고 g를 그대로 호출하면서 f에만 Future를 적용할 수 있었다. (이게 무슨말??)

하지만 조금 더 큰 프로그램에서는 다음 두 가지 이유로 이런 방식을 사용하지 않는다.

- 다른 상황에서는 g에도 Future형식이 필요할 수 있으므로 API형식을 통일하는 것이 바람직하다.
- 병렬하드웨어로 프로그램 실행 속도를 극대화하려면 여러개의 작으면서도 합리적인 크기의 태스크로 나누는 것이 좋다.

### 15.2.2 리액티브 형식 API

두 번째 대안은 f, g의 시그니처를 바꿔서 콜백 형식의 프로그래밍을 이용하는 것이다.

```java
void f(int x, IntConsumer dealWithResult);
```

함수 시그니처를 보면 두 번째 대안이 이상해보일 수 있다. f가 값을 반환하지 않기 때문이다.

f에 추가 인수로 콜백을 전달해서 f의 바디에서는 return문으로 결과를 반환하는 것이 아니라 결과가 준비되면 이를 람다 콜백으로 호출하는 태스크를 만들기 때문에 return문이 필요 없다.

즉 f는 바디를 실행하면서 태스크를 만든 다음 즉시 반환되므로 코드 형식은 다음처럼 바뀐다.

```java
public class CallbackStyleExample{
	public static void main(String[] args){
		int x = 1337;
		Result result = new Result();
	
		f(x, (int y) -> {
			result.left = y;
			sout(result.left + result.right);
		});

		g(x, (int z) -> {
			result.right = z;
			sout(result.left + result.right);
		});
	}
}
```

하지면 결과가 달라졌다.

f와 g의 호출에 대한 합계를 정확하게 출력하지 않고 상황에 따라 먼저 계산된 함수의 콜백이 실행되어 결과가 출력된다.

락을 사용하지 않으므로 값을 두 번 출력할 수 있을 뿐더러 + 연산에 제공된 두 피연산자가 printLn이 호출되기 전에 업데이트될 수도 있다.

다음 두 방법으로 이 문제를 보완할 수 있다.

- if-then-else로 적절한 락을 이용하여 두 콜백이 모두 호출되었는지 확인한 다음 println을 한 번 호출한다.
- 리액티브 형식의 API는 한 결과가 아니라 일련의 이벤트에 반응하도록 설계되었으므로 Future를 이용하여 해결한다.

### 15.2.3 잠자기(그리고 기타 블로킹 동작)는 해로운 것으로 간주

어떤 일이 일정 속도로 제한되어 일어나는 상황의 경우를 만들 때 sleep() 메서드를 사용할 수 있다.

하지만 스레드는 잠들어도 여전히 시스템 자원을 점유하고, 스레드풀에서 잠을 자는 태스크는 다른 태스크가 시작되지 못하게 막으므로 자원을 소비한다.

잠자는 스레드 뿐만 아니라 모든 블록 동작도 마찬가지로 자원을 소비할 수 있다.

이런 상황에선

- 태스크에서 기다리는 일을 만들지 말게 하거나
- 코드에서 예외를 일으키는 방법

으로 문제를 처리할 수 있다.

ex) 코드 A와 코드 B의 비교

- 한 개의 작업을 갖는 코드 A

```java
work1();
Thread.sleep(10000); //10초동안 잔다.
work2();
```

- 코드 B

```java
public class ScheduledExecutorServiceExample{
	public static void main(String[] args){
		ScheduledExecutorService scehduledExecutorService
								 = Executors.newScheduledThreadPool(1);

		work1();
		// work1() 이 끝난 다음 10초 뒤에 work2()를 개별 태스크로 스케줄함
		scheduledExecutorService.schedule(ScheduledExecutorServiceExample::work2,
													 10, TimeUnit.SECONDES);

		scheduledExecutorService.shutdown();
	}

	public static void work1(){
		sout("hello work1");
	}

	public static void work2(){
		sout("hello work2");
	}
}
```

**코드 A의 실행과정**

코드는 스레드 풀 큐에 추가되며 나중에 차례가 되면 실행된다.

하지만 코드가 실행되면 워커스레드를 점유한 상태에서 아무것도 하지 않고 10초를 잔다.

그리고 깨어난 뒤 work2()를 실행한 다음 작업을 종료하고 워커 스레드를 해제한다.

**코드 B의 실행과정**

work1()를 실행하고 종료한다.

work2()가 10초 뒤에 실행될 수 있도록 큐에 추가한다.

A와 B의 동작은 같으나,
A는 자는 동안 스레드 자원을 점유하는 반면 B는 다른 작업이 실행될 수 있도록 스레드 자원을 놓아준다.

따라서 잠을 자거나 블록해야하는 여러 태스크가 있을 때엔 코드 B 형식을 따르는 것이 좋다.

### 15.2.4 현실성 확인

시스템을 동시 실행 가능한 태스크로 설계하여 블록할 수 있는 모든 동작을 비동기 호출로 구현하면 병렬 하드웨어를 최대한으로 활용할 수 있다.

개선된 동시성 API를 이용하여 얻을 수 있는 이점을 알아보는 것도 중요하다.

### 15.2.5 비동기 API에서 예외는 어떻게 처리하는가?

Future나 리액티브 형식의 비동기 API에서 호출된 메서드의 실제 바디는 별도의 스레드에서 호출된다. 따라서 이때 발생하는 어떤 에러는 이미 호출자의 실행 범위와는 관계가 없는 상황이 된다. 그러면 예외에 따라서 호출자의 동작 변경이 필요할 때는 어떻게 해야할까?

Future를 구현한 CompletableFuture에서는 런타임 get()메서드에 예외를 처리할 수 있는 기능을 제공하며 예외에서 회복할 수 있도록 execptiuonally()같은 메서드도 제공한다.

리액티브 형식의 비동기 API에서는 return 대신 기존 콜백이 호출되므로 예외가 발생했을 때 실행될 콜백을 추가하도록 인터페이스를 변경해야한다.

ex)

```java
void f(int x, Consumer(Integer> dealWithResult,
							Consumer<Throwable> dealWithException);
```

메서드 f의 바디는 다음을 수행할 수 있다.

```java
dealWithException(e);
```

만약 콜백이 여러개라면 각 콜백을 따로 제공하는 것보다는 한 객체로 이 메서드들을 감싸는 것이 좋다.

자바 9 플로 API에서는 여러 콜백을 한 객체(Subscriber<T> 클래스)로 감싼다.

```java
void onComplete() // 값을 다 소진했거나 에러가 발생하여 더이상 처리할 데이터가 없을 때
void onError(Throwable throwable) // 도중에 에러가 발생했을 때
void onNext(T item) // 값이 있을 때
```

메서드 f는 다음과 같이 바뀔 수 있다.

```java
void f(int x, Subscriber<Integer> s);
```

메서드 f의 바디에서는 다음처럼 Throwable을 가리키는 t로 예외콜백인 onError(t)로 전달할 수 있다.

```java
s.onError(t);
```

이런 방식의 메서드 호출을 메시지 또는 이벤트라고 부른다.

## 15.3 박스와 채널 모델

박스와 채널 모델 : 동시성 모델을 잘 설계하고 개념화 하기 위한 다이어그램

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8ec206ed-8c70-425f-978b-d996233b20bf/Untitled.png)

위 그림을 다음과 같이 구현할 수 있다

```java
int t = p(x);
sout( r(q1(t), q2(t));
```

이 코드는 깔끔해 보이지만 하드웨어 병렬성의 활용과는 거리가 멀다.

Future를 이용해서 f, g를 병렬로 평가하는 방법도 있다.

```java
int t = p(x);
Future<Integer> a1 = executorService.submit(() -> q1(t));
Future<Integer> a2 = executorService.submit(() -> q2(t));

sout( r(a1.get(), a2.get()));
```

위 코드는 다이어그램의 모양 상 함수 p와 r을 Future로 감싸지 않았다.
p는 다른 작업보다 먼저 처리되어야 하며 r은 모든 작업이 끝난 후에 처리되어야 한다.

병렬성을 극대화하기 위해선 모든 함수를 Future로 감싸야 한다.

만약 많은 태스크가 get() 메서드를 호출하여 Future를 기다리게 되는 상태에 놓일 경우 데드락에 걸릴 수 있다. 그리고 대규모 시스템 구조에서 얼마나 많은 수의 get()을 감당할 수 있는지도 이해하기 어렵다.

CompletableFuture와 콤비네이터로 문제를 해결할 수 있다.

두 Function이 있을 때 compose(), andThen() 등의 메서드를 적용하면

```java
p.thenBoth(q1, q2).thenCombine(r)
```

처럼 구현할 수 있다.

(thenBoth, thenCombine은 자바 Function, BiFunction 클래스의 일부가 아님)

## 15.4 CompletableFuture와 콤비네이터를 이용한 동시성

Future 를 통해 연산을 실행하고 종료되길 기다리는 동작을 수행할 수 있다.

자바 8에서는 Future인터페이스의 구현인 CompletableFuture를 이용하여 Future를 조합할 수 있는 기능을 추가했다.

일반적으로 Future는 실행해서 get()으로 결과를 얻을 수 있는 Callable로 만들어지지만

CompletableFuture는 실행할 코드 없이 Future를 만들 수 있도록 허용하고 complete() 메서드를 이용해서 어떠한 값을 이용하여 다른 스레드가 이를 완료할 수 있고 get()으로 값을 얻을 수 있도록 허용한다.

- CompletableFuture를 사용한 f(x)와 g(x)를 동시 실행하여 합계를 구하는 코드 예

```java
public class CFComplete{
	public static void main(String [] args) throws ... {
		ExecutorService executorService = Executors.newFixedThreadPool(10);
		int x = 1337;
	
		CompletableFuture<Integer> a = new CompletableFuture<>();
		executorService.submit(() -> a.complete(f(x)));
		int b = g(x);
		
		sout(a.get() + b);
		executorService.shutdown();
	}
}
```

또는

```java
public class CFComplete{
	public static void main(String [] args) throws ... {
		ExecutorService executorService = Executors.newFixedThreadPool(10);
		int x = 1337;
	
		CompletableFuture<Integer> b = new CompletableFuture<>();
		executorService.submit(() -> b.complete(g(x)));
		int a = f(x);
		
		sout(a + b.get());
		executorService.shutdown();
	}
}
```

위 두 코드는 get()의 호출로 비동기적으로 실행되는 함수가 끝나기를 기다려야 하기 때문에 프로세싱 자원을 낭비할 수 있다.

자바의 completableFuture를 사용하여 이를 해결할 수 있다.

CompletableFuture<T>의 thenCombine메서드를 사용하여 두 연산 결과를 더 효과적으로 더할 수 있다.

```java
CompletableFuture<V> thenCombine(CompletableFuture<U> other, BiFunction<T, U, V> fn)
```

thenCombine 메서드는 두 개의 CompletableFuture값을 받아서 한 개의 새 값을 만든다.

첫 두 작업이 끝나면 두 결과 모두에 fn을 적용하고 블록하지 않은 상태로 결과 Future를 반환한다.

다음과 같이 구현할 수 있다.

```java
public class CFComplete{
	public static void main(String [] args) throws ... {
		ExecutorService executorService = Executors.newFixedThreadPool(10);
		int x = 1337;
	
		CompletableFuture<Integer> a = new CompletableFuture<>();
		CompletableFuture<Integer> b = new CompletableFuture<>();
		CompletableFuture<Integer> c = a.thenCombine(b, (y, z) -> y+z);

		executorService.submit(() -> a.complete(f(x)));
		executorService.submit(() -> b.complete(g(x)));

		sout(c.get());
		executorService.shutdown();
	}
}
```

Future a와 Future b의 결과를 알지 못한 상태에서 thenCombine은 두 연산이 끝났을 때 스레드풀에서 실행된 연산을 만든다.

결과를 추가하는 세 번째 연산 c는 다른 두 작업이 끝날 때까지는 스레드에서 실행되지 않는다. 따라서 기존의 두 가지 버전의 코드에서 발생했던 블록 문제가 어디서도 일어나지 않는다.
(? 블록/논블록 개념먼저 다시 이해할 것)

Future의 연산이 두 번째로 종료되는 상황에서 실제 필요한 스레드는 한 개이지만 스레드 풀의 두 스레드가 여전히 활성 상태이다.

이전 버전의 합계를 구하는 경우는 f(x), g(x)를 실행한 같은 스레드에서 덧셈을 수행했으나

thenCombine을 이용하면 f(x)와 g(x)가 끝난 다음에야 덧셈 계산이 실행된다.

많은 수의 Future를 사용해야 하는 경우 CompletableFuture와 콤비네이터를 이용하여 get()에서 블록하지 않을 수 있고 이로 인해 병렬 실행의 효율성은 높이면서 데드락을 피할 수 있다.

## 15.5 발행-구독 그리고 리액티브 프로그래밍

Future, CompletableFuture는 독립적 실행과 병렬성이라는 정식적 모델에 기반한다.

연산이 끝나면 get()으로 결과를 얻을 수 있고 한 번만 실행하여 결과를 제공한다.

반면 리액티브 프로그래밍은 시간이 흐르면서 여러 Future같은 객체를 통해 여러 결과를 제공한다.

(매 초 온도값을 반복적으로 제공하는 등.)

리액티브 프로그래밍이라 이름붙여진 이유는 여러 번의 결과를 받아들이고 어떤 경우에는 이에 대해 **반응**을 하는 부분이 존재하기 때문이다.

자바 9에선 java.util.concurrent.Flow 인터페이스에 발행-구독 모델(pub-sub)을 적용하여 리액티브 프로그래밍을 제공한다. Flow API는 다음 세 가지로 정리 가능하다.

- **구독자**가 구독할 수 있는 **발행자**
- 구독자와 발행자의 연결을 **구독**이라고 한다.
- 구독자와 발행자의 연결을 통해 **메시지**를 전달한다.

### 15.5.1 두 플로를 합치는 예제

두 정보 소스로부터 발생하는 이벤트를 합쳐서 다른 구독자가 볼 수 있도록 발행하는 예로 스프레드 시트 동작을 볼 수 있다. “=c1+c2” 식을 c3에 적용한다고 할 때 c1, c2 셀의 값이 변경되면 c3에도 새로운 값이 반영된다.

이를 구현하기 위해 값을 포함하는 셀을 만든다.

```java
private class SimpleCell{
	private int value = 0;
	private String name;
	public SimpleCell(String name){
		this.name= name;
	}
}
```

다음처럼 스프레드 시트의 셀에 해당하는 객체를 초기화할 수 있다.

```java
SimpleCell c2 = new SimpleCell("C2");
SimpleCell c1 = new SimpleCell("C1");
```

c1나 c2의 값이 바뀌었을 때 c3가 두 셀의 값을 더하도록 하려면,

c1, c2에 이벤트가 발생했을 때 c3를 구독하도록 만들어야 한다.

그러기 위해 Publisher<T> 인터페이스가 필요하다.

```java
interface Publisher<T> {
	void subscribe(Subscriber<? super T> subscriber);
}
```

이 인터페이스는 통신할 구독자를 인수로 받는다.

Subscriber<T> 인터페이스는 onNext라는 정보를 전달할 단순 메서드를 포함하며 구현자가 필요한대로 이 메서드를 구현할 수 있따.

```java
interface Subscriber<T> {
	 void onNext(T t);
}
```

Cell은 Publisher이자 Subscriber이므로 둘 모두를 구현한다.

```java
private class SimpleCell implements Publisher<Integer>, Subscriber<Integer> {
	private int value = 0;
	private String name;
	private List<Subscriber> subscribers = new ArrayList<>();
	
	public SimpleCell(String name){
		this.name = name;
	}
	
	@Override
	public void subscribe(Subscriber? super Inetger> subscriber){
		subscribers.add(subscriber);
	}

	// 새로운 값이 있음을 모든 구독자에게 알리는 메서드	
	private void notifyAllSubscribers() {
		subscribers.forEach(subscriber -> subscriber.onNext(this.value));
	}

	// 구독한 셀에 새 값이 생겼을 때 값을 갱신해서 반응함
	@Override
	public void onNext(Integer newValue){
		this.value = newValue;
		System.out.println(this.name + ":" + this.value);
		notifyAllSubscribers(); //값이 갱신되었음을 모든 구독자에게 알림
	}

```

ex) 실행

```java
SimpleCell c3 = new SimpleCell("C3");
SimpleCell c2 = new SimpleCell("C2");
SimpleCell c1 = new SimpleCell("C1");

c1.subscribe(c3); // c1의 구독자에 c3 추가

c1.onNext(10); // 값 갱신 및 발행
c2.onNext(20);

// 출력 결과
C1:10
C3:10
c2:20
```

“C3=C1+C2”의 구현

왼쪽과 오른쪽의 연산 결과를 저장할 수 있는 별도의 클래스가 필요하다.

- SimpleCell을 상속한 ArtihmeticCell 구현

```java
public class ArithmeticCell extends SimpleCell{
	private int left;
	private int right;
	
	public ArithmeticCell(String name){
		super(name);
	}
		
	public void setLeft(int left){
		this.left = left;
		onNext(left + this.right); //셀 값을 갱신하고
	}

	public void setRight(int right){
		this.right = right;
		onNext(right + this.left);
	}
}
```

다음처럼 c3의 타입을 ArtihmeticCell로 사용할 수 있다.

```java
ArtihmeticCell c3 = new ArtihmeticCell("C3");
SimpleCell c2 = new SimpleCell("C2");
SimpleCell c1 = new SimpleCell("C1");

c1.subscribe(c3::setLeft); 
// 이게 되는게 맞나? subscribe 메서드의 인수 타입이 Subscriber<? super T> subscriber)인데
// 어떻게 메서드참조를 인수로 전달할 수 있는지?
c2.subscribe(c3::setRight);

c1.onNext(10); // 값 갱신
c2.onNext(20); 
c1.onNext(15); 

//출력 결과
C1:10
C3:10
C2:20
C3:30
C1:15
C3:35

```

데이터가 발생자(생산자)에서 구독자(소비자)로 흐름에 착안하여 이를 **업스트림**또는 **다운스트림**이라고 부른다.

위 예제어서 데이터는 업스트림 onNext()메서드로 전달되고,

nofifyAllSubscribers() 호출을 통해 다운스트림 onNext() 호출로 전달된다.

**플로 인터페이스의 압력과 역압력**

압력 : 업스트림 onNext()의 호출이 빠르게, 많이 발생하여 과부하가 발생하는 상황

역압력 : 압력상황에서 출구로 추가될 공간의 숫자를 제한하는 기법(구독자의 수를 제한하여 부하 줄이기)

지바 9 Flow API에선 요청했을 때만 아이템을 보내도록 하는 request() 메서드(Sbuscription 인터페이스에 포함)를 제공한다.

### 15.5.2 역압력

자바 9 플로 API의 Subscriber 인터페이스는 onNext, onError, onComplete 외에 onSubscribe 메서드를 포함한다.

```java
void onSubscribe(Subscription subscription);
```

이 메서드는 Publisher와 Subscriber가 연결되면 첫 이벤트로 호출된다.

Subscription객체는 다음처럼 subscriber, Publisher와 통신할 수 있는 메서드를 포함한다.

```java
interface Subscription {
	void cancel();
	void requrest(long n);
}
```

콜백을 통한 역방향 소통을 한다.

Publisher는 Subscription객체를 만들어서 Subscriber로 전달하면,

Subscriber는 이 객체를 이용하여 Publisher로 정보를 보낼 수 있다.

### 15.5.3 실제 역압력의 간단한 형태

한 번에 한 개의 이벤트를 처리하도록 발행-구독 연결을 구성하기 위한 작업

- Subscriber가 onSubscribe로 전달된 Subscription 객체를 subscription같은 필드에 저장한다.
- Subscriber가 수많은 이벤트를 받지 않도록 onSubscribe, onNext, onError의 마지막 동작에 channel.request(1)을 추가하여 오직 한 이벤트만 요청한다.
- 요청을 보낸 채널에만 onNext, onError 이벤트를 보내도록 Publisher의 notifyAllSubscribers 코드를 변경한다. (Publisher는 새 Subscription을 만들어서 각 Subscriber와 연결한다.)

역압력 구현에 있어서 고려해야할 사항

- 여러 Subscriber가 있을 때 이벤트를 가장 느린 속도로 보낼 것인가? (위 방식으로 역압력을 구현하면 Subscriber가 요청할 때 이벤트를 보내므로 데이터 싱크가 맞지 않을 수 있다.)
  아니면 각 Subscriber에게 보내지 않은 데이터를 저장할 별도의 큐를 가질 것인가?
- 큐가 너무 커지면 어떻게 해야할까?
- Subscriber가 준비가 되지 않았다면 큐의 데이터를 폐기할 것인가?

## 15.6 리액티브 시스템 vs 리액티브 프로그래밍

리액티브 시스템 : 런타임 환경이 변화에 대응하도록 전체 아키텍처가 설계된 프로그램.
반응성, 회복성, 탄력성은 액티브 시스템이 가져야 할 속성이다.

- 반응성 : 큰 작업을 처리하느라 간단한 질의의 응답을 지연하지 않는 상황 없이 실시간으로 입력에 반응하는 것을 의미
- 회복성 : 한 컴포넌트의 실패로 전체 시스템이 실패하는 경우가 없음을 의미
- 탄력성 : 시스템이 자신의 작업 부하에 맞게 적응하며 작업을 효율적으로 처리함을 의미

java.util.concurrent.Flow 와 관련된 자바 인터페이스에서 제공하는 **리액티브 프로그래밍** 형식을 이용하여 위의 속성을 따라갈 수 있다.

이들 인터페이스 설계는 메시지 주도 속성을 반영한다.

메시지 주도 시스템에서 컴포넌트는 처리할 입력을 기다리고 결과를 다른 컴포넌트로 보내면서 시스템이 반응한다.

## 15.7 마치며

- 스레드풀은 유용하지만 블록되는 태스크가 많아지만 문제가 발생
- 메서드를 비동기로 만들면 병렬성을 추가할 수 있다.
- 플로 API는 발행-구독 패턴, 역압력 기법을 제공한다. 이를 이용하여 자바의 리액티브 프로그래밍의 기초를 제공
- 리액티브 프로그래밍으로 리액티브 시스템을 구현할 수 있따.

### 질문

p.474 마지막 문단 ‘예제에서는 API는 그대로 유지하고 g를 그대로 호출하면서 f에만 Future를 적용할 수 있었다’ 이 문장이 무슨말인지? 예제에서는 g, f 둘 다 Future를 적용한 것으로 보이는데 아닌가?