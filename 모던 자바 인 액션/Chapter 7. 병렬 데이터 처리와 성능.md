# Chapter 7. 병렬 데이터 처리와 성능
**내용**

- 병렬 스트림으로 데이터 병렬처리하기
- 병렬 스트림의 성능 분석
- 포크/조인 프레임워크
- Spliterator로 스트림 데이터 쪼개기

외부 반복을 내부 반복으로 바꾸면 네이티브 자바 라이브러리가 스트림 요소의 처리를 제어할 수 있다. 멀티코어를 활용한 파이프라인 연산을 실행할 수 있다.

**병렬처리**

- 자바 7 등장 이전

데이터를 서브 파트로 분할하고 분할된 서브 파트를 각각의 스레드로 할당한다. 스레드에 할당한 다음, 의도치않은 레이스 컨디션(race condition, 경쟁상태라고도 함)이 발생하지 않도록 적절한 동기화를 추가해야 하며 마지막으로 부분 결과를 합쳐야 한다.

- 자바 7 이후

더 쉽게 병렬화를 수행하면서에러를 최소화할 수 있도록 **포크/조인 프레임워크** 기능을 제공한다.

## 7.1 병렬 스트림

스트림 인터페이스의 parallelStream을 호춣하면 **병렬스트림**을 생성하여 요소들을 쉡게 병렬처리할 수 있다.

병렬스트림은 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림이다.

병렬스트림을 이용하면 모든 멀티코어 프로세서가 각각의 청크를 처리하도록 할당할 수 있다.

숫자 n을 인수로 받아서 1부터 n까지의 모든 숫자의 합계를 반환하는 메서드 코드는 다음과 같다.

```java
public long sequentialSum(long n){
	return Stream.iterate(1L, 1 -> i + 1)
		.limit(n)
		.reduce(0L, Long::sum);
}
```

n이 커질경우 병렬로 처리하는 것이 좋다.

결과 변수의 동기화, 스레드 생성 개수, 숫자 생성 방식, 숫자 더하기 등의 동작을

병렬스트림을 사용하여 해결할 수 있다.

### 7.1.1 순차 스트림을 병렬 스트림으로 변환하기.

순차스트림에 parallel() 메서드를 호출하면 기존의 함수형 리듀싱 연산이 병렬로 처리된다.

```java
public long sequentialSum(long n){
	return Stream.iterate(1L, 1 -> i + 1)
		.limit(n)
		.parallel()
		.reduce(0L, Long::sum);
}
```

이전 코드와 다른 점은 스트림이 여러 청크로 분할되어 있다는 것이다.

따라서 리듀싱 연산을 여러 청크에 병렬로 수행할 수 있다.

리듀싱 연산 수행 이후 리듀싱 연산으로 생성된 부분 결과를 다시 리듀싱 연산으로 합쳐서 전체 스트림의 리듀싱 결과를 도출한다.

순차스트림에 parallel을 호출한다고 해서 스트림 자체에 변화가 일어나는 것은 아니다.
내부적으로는 parallel을 호출하면 이후 연산이 병렬로 수행해야 함을 의미하는 불리언 플래그가 설정된다.

반대로 sequential()메서드를 사용하면 병렬 스트림을 순차 스트림으로 만들 수 있다.

위 두 메서드를 사용하여 아래와 같이 병렬로 처리되어야 할 연산과 순차적으로 처리되어야 할 연산을 지정할 수 있다.

```java
stream.parallel()
	.filter(...)
	.sequential()
	.map(...)
	.parallel()
	.reduce();
```

parallel, sequential 메서드가 같이 사용될 경우 최종적으로 호출된 메서드가 전체 파이프라인에 영향을 미친다.

**병렬 스트림에서 사용하는 스레드 풀 설정**

parallel메서드에서 병렬 작업을 수행하는 스레드는 어디서, 몇 개나 생성되며 어떻게 커스터마이징 하는가?

병렬 스트림은 내부적으로 ForkJoinPool을 사용한다. ForkJoinPool은 프로세서 수(Runtime.getRuntime().availableProcessors()가 반환하는 값)만큼의 스레드를 갖는다.

생성하는 스레드 수를 다음과 같이 전역으로 커스텀하게 설정할 수 있다.

```java
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "12");
```

다만, 전역 설정만 가능하기에 이 코드 이후의 모든 병렬 스트림 연산에 영향을 준다. 병렬스트림 하나하나마다 개별적으로 생성될 특정 개수의 스레드 수는 지정할 수 없다.

### 7.1.2 스트림 성능 측정

병렬화한다고 해서 무조건적으로 성능이 향상될 것이라는 추측은 섣부르다. 성능은 직접 측정해본다. 자바 마이크로 벤치마크 하니스(JMH) 라이브러리를 사용하여어노테이션 기반 방식으로 JVM을 대상으로 하는 벤치마크를 구현할 수 있다.

```xml
...
	<groupId>org.openjdk.jmh</groupId>
	<artifactId>jmh-core</artifactId>
...
	<groupId>org.openjdk.jmh</groupId>
	<artifactId>jmh-generator-annprocess</artifactId>
...
```

- jmh-core : 핵심 JMH 구현을 포함
- jmh-generator-annprocess : 자바 아카이브(JAR) 파일을 만드는데 사용되는 어노테이션 프로세서를 포함

(JMH라이브러리를 사용한 성능측정은 page. 245 ~ 참고)

병렬처리를 수행한다고 해서 항상 성능에서 이득을 얻는 것은 아니다. 다음과 같은 문제들을 고려해야 한다.

- 언박싱에 소모되는 비용
- 반복작업은 병렬로 수행할 수 있는 독립 단위로 나누기 어렵다.

위 예시의 경우 이전 연산의 결과에 따라 다음 함수의 입력이 달라지기 때문에 iterate연산을 청크로 분할하기가 어렵다. 이런 경우 스트림이 청크로 나뉘어져서 개별적인 리듀싱 연산을 수행하는 것이 불가능하다.

리듀싱 과정을 시작하는 시점에 전체 숫자 리스트가 준비되지 않았으므로 스트림을 병렬로 처리할 수 있도록 청크로 분할할 수 없다.

스트림이 병렬로 처리되도록 지시했고 각각의 합계가 다른 스레드에서 수행되었지만 결론적으로 순차처리방식과 크게 다른접이 없고 스레드를 할당하는 오버헤드만 증가하게 된다.

**더 특화된 메서드 사용**

멀티코어 프로세서를 활용해서 효과적으로 합계 연산을 병렬로 실행하려면?

LongStream.rangeClosed메서드는 iterate에 비해 다음과 같은 장점이 있다.

- 기본형 long을 직접 사용하므로 박싱, 언박싱 오버헤드가 사라진다.
- 쉽게 청크로 분할할 수 있는 숫자 범위를 생성한다.

병렬화는 공짜가 아니다.

- 스트림을 재귀적으로 분할해야 한다.
- 각 서브스트림을 서로 다른 스레드의 리듀싱 연산으로 할당해야 한다.
- 각 스레드의 연산 결과를 하나로 합쳐야 한다.

멀티코어 간의 데이터 이동 비용은 비싸기 때문에 이러한 비용을 감수해도 병렬화로 얻는 이득이 더 클 경우에만 사용해야 한다.

### 7.1.2 병렬 스트림의 올바른 사용법

병렬스트림에서 굥유된 상태를 바꾸는 알고리즘을 사용하면 문제가 발생할 수 있다.

ex) n까지의 자연수를 더하면서 공유된 누적자를 바꾸는 프로그램

```java
public long sideEffectSum(long n){
	Accumulator accumulator = new Accumulator();
	LongStream.rangeClosed(1, n)
					.forEach(accumulator::add);
	
	return accumulator.total;
}

public class Accumulator {
	public long total = 0;'
	public void add(long value) {
		total += value;
	}
}
```

위 코드는 병렬로 수행될 때 문제가 발생한다.

tatal 변수에 접근할 때마다(다수의 스레드에서 동시에 total에 접근) 데이터 레이스 문제가 일어난다.

여러 스레드에서 동시에 누적자, 즉 total += value 를 실행하면서 문제가 발생한다.

병렬스트림이 올바르게 ㄷㅇ작하려면 공유된 가변 상태를 피해야 한다.

### 7.1.4 병렬 스트림 효과적으로사용하기

- 순차 스트림, 병렬 스트림 중 어느 것이 성능이 더 좋을지 모르겠다면 직접 성능을 측정하라.
- 박싱을 주의하라. 자동 박싱, 언박싱은 성능저하 요소이다. 기본형 특화 스트림을 사용하자
- limit, findFirst처럼 요소의 순서에 의존하는 연산을 병렬 스트림에서 수행하려면 비싼 비용을 치러야 한다.
- 스트림에서 수행하는 전체 파이프 라인 연산 비용을 고려하라
- 소량의 데이터에서는 병렬 스트림이 도움이 되지 않는다. 병렬화 고정에서 생기는 부가 비용만큼의 추가이득을 보기 힘들다.
- 스트림을 구성하는 자료구조가 적절한지 확인하라. LinkedList같은 경우 분할하려면 모든 요소를 탐색해야하기 때문에 비효율적이다.


    **요소의 분해성**
    
    ArrayList : 분해성 매우좋음
    
    LinkedList : 나쁨
    
    IntStream.range : 좋음
    
    Stream.iterate : 나쁨
    
    HashSet, TreeSet : 좋음

- 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정과 성능이 달라질 수 있다. SIZED스트림은 분할하기가 좋으나 필터연산같은 경우 스트림의 결과를 예측할 수 없어서 효과적인 병렬처리가 가능한지 알 수 없다.
- 최종 연산의 병합 과정 비용을 살펴보자(ex : Collector의 combiner)

## 7.2 포크/조인 프레임워크

포크/조인 프레임워크는 병렬화할 수 잇는 작업을 재귀적으로 분할한 다음에 서브태크스 각각의 결과를 합쳐서 전체 결과를 ㅁ나들도록 설계되었다.

포크/조인 프레임워크에서는 서브태스크를 스레드 풀의 작업자 스레드에 분산 할당하는 ExceutorService 인터페이스를 구현한다.

### 7.2.1 Recursive Task 활용

스레드 풀을 이용하려면 RecursiveTask<R>의 서브클래스를 만들어야 한다.
R은 병렬화된 태스크가 생성하는 결과 형식 또는 결과가 없을 때는 RecursiveAction 형식이다.

RecursiveTask를 정의하려면 추상 메서드 compute를 구현해야한다.

```java
protected abstract R compute();
```

compute 메서드는 태스크를 서브태스크로 분할하는 로직과 더 이상 분할할 수 없을 때 개별 서브태스크의 결과를 생산할 알고리즘을 정의한다

따라서 compute메서드는 보통 다음 의사코드 형식을 유지한다.

```java
if(태스크가 충분히 작거나 더 이상 분할할 수 없으면){
	순차적으로 태스크 계산
} else {
	태스크를 두 서브태스크로 분할
	재귀적 호출
  연산이 완료되면 각 서브태스크의 결과를 합침
}
```

분할-정복 알고리즘의 병렬화 버전이다.

ex) 포크/조인 프레임워크를 사용한 병렬 합계 수행

```java
import java.util.concurrent.RecursiveTask;

public class ForkJoinSumCalculator extends RecursiveTask<Long> {
	private final long[] numbers; // 더할 숫자 배열
	private final int start; //서버태스크에서 처리할 배열의 초기 위치와 최종 위치
	private final int end;
	public static final long THRESHOLD = 10_000; //서브태스크에서 처리할 숫자의 최대 갯수

	public ForkJoinSumCalculator(long[] numbers) {
		this(numbers, 0, numbers.length);
	}

	private ForkJoinSumCalculator(long[] numbers, int start, int end) {
		this.numbers = numbers;
		this.start = start;
		this.end = end;
	}

	@Override
	protected Long compute() {
		int length = end - start; // 이 태스크에서 더할 배열의 길이

		// 더 분할할 수 없는 경우
		if (length <= THRESHOLD) {
			return computeSequentially();
		}
		
		// 분할이 가능하다면 배열 왼쪽 절반을 대상으로한 서브태스크 생성
		ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length / 2);
		leftTask.fork(); // ForkJoinPool의 다른 스레드로 새로 생성한 태스크를 비동기로 실행한다.
		
		ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length/2, end);
		Long rightResult = rightTask.compute();	// 두 번째 서브태스크를 동기적으로 실행한다.
		Long leftResult = leftTask.join(); //첫 번째 서브태스크의 결과를 읽거나 결과가 없으면 기다린다.
		
		return leftResult + rightResult; // 두 서브태스크의 결과를 조합
	}

	private long computeSequentially() {
		long sum = 0;
		for (int i = start; i < end; i++) {
			sum += numbers[i];
		}
		return sum;
	}
}
```

```java
public static long forkJoinSum(long n){
	long[] numbers = LongStream.rangeClosed(1, n).toArray();
	ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
	return new ForkJoinPool().invoke(task);
}
```

배열을 생성해여 생성자의 인수로 전달해서 ForkJoinTask 객체를 생성할 수 있다.
invoke메서드의 반환값은 FockJoinSumCalculator에서 정의한 태스크의 결과가 된다.

일반적으로 둘 이상의 ForkJoinPool을 사용하지 않는다. ForkJoinPool을 한 번만 인스턴스화해서 정적 필드에 싱글턴으로 저장한다.

### 7.2.2 포크/조인 프레임워크를 제대로 사용하는 방법

- join메서드를 태스크에 호출하면 태스크가 생산하는 결과가 준비될 때까지 호출자를 블록시킨다. 따라서 두 서브태스크가 모두 시작된 다음에 join을 호출해야한다. 그렇지 않으면 각각의 서브태스크가 다른 태스크가 끝나길 기다리는 일이 발생한다.
- RecursiveTask내에서는 ForkJoinPool의 invoke메서드를 사용하지 말아야 한다. 대신 compute나 fork메서드를 직접 호출한다.
- 두 작업 모두에 fork메서드를 호출하는 것이 좋아보이지만 한쪽 작업에만 fork를 호출하는 것이 효율적이다. 이래야 두 서브 태스크의 한 태스크에는 같은 스레드를 재사용할 수 있으므로 풀에서 불필요한 태스크를 할당하는 오버헤드를 피할 수 있다.
- 포크/조인 프레임워크를 이용하는 병렬 계산은 디버깅하기 어렵다.
- 포크/조인 프레임워크를 사용하는 것이 순차처리보다 무조건 빠를 것이라는 생각은 버려야 한다.

### 7.2.3 작업 훔치기

위의 예제처럼 연산을 수행할 요소의 개수로 서브태스크를 분할할 경우 서브태스크는 지원을 낭비하는 꼴이 되는 것 처럼 보일 수 있다. cpu는 4개인데 서브태스크가 많다고 성능이 좋아질까?

하지만 코어 개수와는 관계 업이 적절한 크기로 분할된 많은 태스크를 포킹하는 것이 바람직하다. 만약 4코어 기기에 대량의 요소를 단 4개의 서브태스크로 나누었다고 가정해보자. 예상하기로는 각 코어에 할당된 각각의 태스크가 동시에 처리되어 종료될 것 같지만 실제로는 디스크 접근 속도 저하나 외부 서비스 등의 요인으로 인해 작업 완료 시간에 큰 차이가 발생할 수 있다.

포크/조인 프레임워크에서는 **작업 훔치기**라는 기법으로 이러한 문제를 해결한다. 작업훔치기 기법에서는 ForkJoinPool의 모든 스레드를 거의 공정하게분할한다. 각각의 스레드는 자신에게 할당된 태스크를 포함하는 이중 연결 리스트를 참조하면서 작업이 끝날 때마다 큐의 헤드에서 다른 태스크를 가져와서 작업을 처리한다.

이 때 한 스레드가 다른 스레드보다 자신에게 할당된 작업을 더 빨리 처리하면 해당 스레드는 유휴상태로 바뀌지 않고 다른 스레드 큐의 꼬리에서 작업을 훔쳐온다. 모든 태스크 큐가 빌 때까지 이 과정이 반독된다. 따라서 태스크의 크기를 적절히 작게 나누어야 작업자 스레드 간의 작업 부하를 비슷한 수준으로 유지할 수 있다.

## 7.3 Spliterator 인터페이스

Spliterator : 분할할 수 있는 반복자를 뜻한다. Iterator처럼 소스에 대한 요소 탐색 기능을 제공하지만 병렬 작업에 특화되어 있다.

자바 8은 컬렉셔ㅑㄴ 프레임워크에 포함된 모든 자료구조에 사용할 수 있는 디폴트 Spliterator 구현을 제공한다. 컬렉션은 spliterator라는 메서드를 제공하는 Spliterator인터페이스를 구현한다.

Spliterator 인터페이스의 형태

```java
public interface Spliterator<T> {
	boolean tryAdvance(Consumer<? super T> action);
	Spliterator<T> trySplit();
	long estimateSize();
	int characteristics();
}
```

- tryAdvance : Spliterator의 요소를 하나씩 순차적으로 소비하면서 탐색해야할 요소가 남아있으면 참을 반환
- trySplit : Spliterator의 일부 요소를 분할해서 두 번째 Spliterator를 생성한다.
- estimateSize : 탐색해야할 요소 수 정보를 제공한다. 제공된 값을 이용해서 더 쉽고 공평하게 Spliterator를 분할할 수 있다.

### 7.3.1 분할 과정

스트림을 여러 스트림으로 분할하는 과정은 재귀적으로 일어난다.

1. 첫 번째 Spliterator에 trySplit메서드를 호출하여 두 번째 Spliterator가 생성된다.
2. 두 Spliterator에 trySplit메서드를 호출하여 네 개의 Spliterator가 생성된다.
3. trySplit의 결과가 null이 될 때까지 반복한다.
4. 모든 trySplit의 결과가 null이면 재귀 분할이 종료된다.

이러한 분할과정은 characteristics메서드로 정의하는 Spliterator의 특성에 영향을 받는다.

**Spliterator 특성**

Spliterator는 characteristics라는 추상 메서드도 정의한다. characteristics메서드는 Spliterator자체의 특성 집합을 포함하는 int를 반환한다.

Spliterator를 이용하는 프로그램은 특성을 참고해서 Spliterator를 더 잘 제어하고 최적화할 수 있다.

Spliterator특성

- ORDERED : 요소에 정해진 순서가 있으므로 Spliterator는 요소를 탐색하고 분할할 때 이 순서에 유의해야한다는 의미
- DISTINCT : x, y 두 요소를 방문했을 때 x.equals(y)는 항상 false를 반환해야한다. 중복값 x
- SORTED : 탐색된 요소는 미리 정의된 정렬 순서를 따른다는 의미
- SIZED : 크기가 알려진 소스로 Spliterator를 생성했으므로 estimatedSize()는 정확한 값을 반환함.
- NON-NULL : 탐색하는 모든 요소는 null이 아님을 의미
- IMMUTABLE : 해당 Spliterator의 소스는 불변이므로 요소를 탐색하는 동안 요소의 수정이 불가능하다는 의미
- CONCURRENT : 동기화 없이 Spliterator의 소스를 여러 스레드에서 동시에 고칠 수 있음을 의미
- SUBSIZED : 분할되는 모든 Spliterator도 SIZED특성을 갖는다.

### 7.3.2 커스텀 Spliterator 구현하기

문자열의 단어 수를 계산하는 메서드

ex) 반복형으로 단어 수를 세는 메서드

```java
public int countWordsIteratively(String s){
	int counter = 0;
	boolean lastSpace = true;
	for(char c : s.toCharArray()){
		if(Character.isWhitespace(c)){ // 공백문자를 기준으로 단어를 구분하여 개수를 세는..
			lastSpace = true;
		} else { 
			if(lastSpace) counter++; 
			lastSpace = false;
		}
	}
	
	return counter;
}
```

반복형 대신 함수형을 이용하면 직접 스레드를 동기화하지 않고도 병렬 스트림으로 작업을 병렬화할 수 있다.

**함수형으로 단어 수를 세는 메서드 재구현하기**

String을 스트림으로 변환해야한다. 사용할 수 있는 기본형이 따로 없으므로 일반 스트림을 사용한다

```java
Stream<Character> stream = IntStream.range(0, SENTENCE.length())
																		.mapToObj(SENTENCE::charAt);
```

스트림에 리듀싱 연산을 실행하면서 단어 수를 계한할 수 있다.

지금까지 발견한 단어 수를 계산하는 int변수와 마지막 문자가 공백이었는지 여부를 기헉하는 boolean 변수, 두 가지의 변수가 필요하다.

변수 상태를 캡슐화하는 새로운 클래스가 필요하다. ⇒ WordCounter 클래스 생성

```java
class WordCounter{
	private final int counter;
	private final boolean lastSpace;
	
	public WordCounter(int counter, boolean lastSpace){
		tihs.counter = counter;
		this.lastSpace = lastSpace;
	}
	
	public WordCounter accumulate(Character c){
		// 현재 문자가 공백일 경우
		if(Character.isWhitespace(c)){
			// 이전 마지막 분자가 공백일 경우 this.
			// 마지막 문자가 공백이 아닐 경우 새로운 WordCounnter 객체 시작.
			return lastSpace? this : new WordCounter(counter, true);
		} 
		//현재 문자가 공백이 아닌 경우
		else {
			// 이전 마지막 문자가 공백일 경우 새로운 단어 추가 및 WordCounter객체 시작.
			// 이전 마지막 문자가 공백이 아닌 경우 단어의 끝을 계속 탐색
			return lastSpace? new WordCounter(counter+1, false) : this;
		}
	}

	// 두 WorkCounter객체 병합.
	public WordCounter combine(WordCounter wordCounter){
		return new WordCounter(counter + wordCounter.counter, wordCounter.lastSpace);
	}

	public int getCounter(){
		return counter;
	}
}
```

accumulate 메서드는 문자열의 문자를 검증하여 해당 문자열의 단어 개수와 이전 마지막 문자에 대한 정보를 갖고 있는 WordCounter객체를 리턴한다.

combine 메서드는 문자열 서브스트림을 처리한 WordCounter의 결과를 합친다.

위 WordCounter객체를 이용하여 리듀싱 연산을 다음처럼 구현할 수 있다.

```java
private int countWords(Stream<Character> stream){
	WordCounter wordCounter = stream.reduce(new WordCounter(0, true),
																					WordCounter::accumulate,
																					WordCounter::combine);
	return wordCounter.getCounter();
}
```

**WordCounter 병렬로 수행하기**

병렬로 수행하면 문자열을 분할하는 위치에 따라 원하지 않는 경과를 얻게 될 수 있다.

따라서 문자열을 임의의 위치에서 분할하지 말고 단어가 끝나는 위치에서만 분할해야 한다.

단어 끝에서 문자열을 분할하는 문자 Spliterator가 필요하다.

ex) 문자 Spliterator를 구현한 다음 병렬 스트림으로 전달하는 코드

```java
class WordCounterSpliterator implements Spliterator<Character>{
	private final String string;
	private int currentChar = 0;
	
	public WordCounterSpliterator(String string){
		this.string = string;
	}

	@Override
	public boolean tryAdvance(Consumer<? super Character> action){
		action.accept(string.charAt(currentChar++); // 현재 문자를 소비한다.
		return currentChar < string.length(); // 소비할 문자가 남아있으면 true를 반환한다.
	}
	
	@Override
	public Spliterator<Character> trySplit(){
		int currentSize = string.length() - currentChar;
		
		// 파싱할 문자열을 순차처리할 수 있을 만큼 충분히 작아졌음을 알리는 null을 반환한다.
		if(currentSize < 10){
			return null;
		}

		// 파싱할 문자열의 중간을 분할 위치로 설정한다.
		for(int splitPos = currentSize / 2 + currentChar;
					splitPos < string.length(); splitPos++){

			// 다음 공백이 나올 때까지 splitPos값을 증가시키며 분할 위치를 조정한다.
			if(Character.isWhitespace(string.charAt(splitPos))){
				
				// 처음부터 분할위치까지의 문자열을 파싱할 새로운 WordCounterSpliterator를 생성
				Spliterator<Character> spliterator = 
					new WordCounterSpliterator(string.substring(currentChar, splitPos));

				// 이 WordCounterSpliterator의 시작 위치를 분할 위치로 설정한다.
				currentChar = splitPos;
				
				// 문자열을 분리했으므로 루프 종료.
				return spliterator;
			}
		}
		
		return null;
	}

	@Overrude
	public long estimateSize(){
		return string.length() - currentChar;
	}
	
	@Override
	public int characteristics(){
		return ORDERED + SIZED + SUBSIZED + NON-NULL + IMMUTABLE;
	}
}
```

분석 대상 문자열로 Spliterator를 생성한 다음 현재 탐색중인 문자를 가리키는 인덱스를 이용해서 모든 문자를 반복 탐색한다.

Spliterator를 구현하는 WordCounterSpliterator의 메서드

- tryAdvance : 문자열에서 현재 인덱스에 해당하는 문자를 Consumer에 제공한 다음 인덱스를 증가시킨다. 인수 Consumer는 스트림을 탐색하면서 적용해야 하는 함수 집합이 작업을 처리할 수 있도록 소비한 문자를 전달하는 자바 내부 클래스.
- trySplit : 반복될 자료구조를 분할하는 로직을 포함한다. 태스크 한계값 이하라면 분할이 필요없으므로 null을 반환한다. 분할이 필요하다면 분할 위치(단어와 단어 사이)를 찾아서 새로운 Spliterator를 만들어 반환한다.
- estimateSize : Spliterator가 파싱할 문자열 전체 길이 = string.length와 currentChar의 차
- characteristic : 프레임워크에 Spliterator의 특성을 알려준다.

**WordCounterSpliterator 활용**

WordCounterSpliterator를 병렬스트림에 사용할 수 있다.

```java
Spliterator<Character> spliterator = new WordCounterSpliterator(SENTENCE);
Stream<Character> stream = StreamSupport.stream(spliterator, true);
```

[StreamSupport.stream](http://StreamSupport.stream) 팩토리 메서드의 두 번째 인수는 병렬 스트림 생성 여부를 나타낸다.

병렬스트림을 countWords 메서드로 전달한다.

생성된 병렬 스트림을 countWords 메서드로 전달하면 원하는 위치에서 분할하여 병렬처리가 가능하다.

```java
System.out.println(countWords(stream));
```

## 7.4 마치며

- 내부 반복을 이용하면 명시적으로 다른 스레드를 사용하지 않고도 스트림을 병렬로 처리할 수 있다.
- 병렬처리가 항상 빠른 것은 아니다. 때론 성능을 직접 측정해보는 것이 좋다
- 특히 처리해야 할 데이터가 아주 많을 때, 또는 각 요소를 처리하는 데 오랜 시간이 걸릴 때 병렬스트림으로 이득을 볼 수 있다.
- 기본형 특화 스트림 사용 등 올바른 자료구조의 선택 또한 성능에 큰 영향을 미친다.
- 포크/조인 프레임워크에서는 병렬화할 수 있는 태스크를 잘게 나누어 분할된 태스크를 각각의 스레드로 실행하며 서브태스크 각각의 결과를 합쳐서 최종 결과를 생산한다.
- Spliterator는 탐색하려는 데이터를 포함하는 스트림을 어떻게 병렬화할 것인지 정의한다.