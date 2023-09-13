# Chapter 16. CompletableFuture : 안정적 비동기 프로그래밍
**내용**

- 비동기 작업 만들고 결과 얻기
- 비블록 동작으로 생산성 높이기
- 비동기 API 설계와 구현
- 동기 API를 비동기적으로 소비하기
- 두 개 이상의 비동기 연산을 파이프라인으로 만들고 합치기

CompletableFuture가 비동기 프로그램에 얼마나 도움을 주는 지 학습한다.

자바 9에 추가된 내용을 학습한다.

## 16.1 Future의 단순 활용

자바 5부터는 미래 시점에 대한 결과를 얻는 모델을 활용하기 위해 Future인터페이스를 제공한다.

오래걸리는 작업을 Future의 내부로 설정하여 호출자 스레드에서 다른 작업을 수행할 수 있다.

Future는 저수준의 스레드에 비해 직관적으로 이해하기 쉽다.

Future를 사용하기 위해선 시간이 오래걸리는 작업을 Callable객체 내부로 감싼 뒤 ExecutorService에 제출해야한다.

eX)

```java
// 스레드 풀에 테스크를 제출하기 위해 ExecutorService 생성
ExecutorService executor = Executors.newChachedThreadPool();

// Callble을 ExecutorSErvice로 제출
Future<Double> future = excutor.submit(new Callble<Double>(){
	public Double call(){
		return doSomeLongComputation();
	}
});

doSomethingElse(); // 비동기 작업을 수행하는 동안 다른 작업을 수행한다.

try{
	// 비동기 작업의 결과를 가져온다.
	// 결과가 준비되어 있지 않은 경우 호출 스레드를 블록하지만 최댁 1초까지만 블록한다.
	Double result = future.get(1, TimeUnit.SECONDS);
} catch ( ExecutionException ee){
	// 계산 중 예외 발생
} catch ( INterruptedException ie){
	// 현재 스레드에서 대기 중 인터럽트 발생
} catch ( TimeoutException te){
	// Future가 완료되기 전에 타임아웃 발생
}
```

이 예제에서의 문제는,

오래 걸리는 작업이 끝나지 않으면 호출 스레드를 계속 잡고 있게 된다.

따라서 오버로드된 get() 메서드를 사용하여 호출스레드가 대기할 최대 타임아웃을 설정하는 것이 좋다.

### 16.1.1 Future 제한

이번 예제에서는 Future 인터페이스가 비동기 계산이 끝났는지 확인할 수 있는 isDone() 메서드, 계산이 끝나길 기다리는 메서드, 결과 회수 메서드 등을 제공함을 보여준다.

하지만 이런 메서드들 만으로는 간결한 동시 실행 코드를 구현하기에 충분치않다.

예로 ‘A의 계산이 끝나면 그 결과를 다른 비동기 연산인 B연산에 전달하시오’ 등..

Future로 이런 요구사항을 구현하는 것은 쉽지 않다.

다음과 같은 기능이 존재한다면 유용할 것이다.

- 두 개의 비동기 연산을 하나로 합친다. 결과가 독립적일 수도 있고 다른 연산에 의존적일 수도 있다.
- Future 집합이 실행하는 모든 태스크의 완료를 기다린다.
- Future 완료 동작에 반응한다
- …. 그 외 복잡한 Future 처리

이런 기능을 선언형으로 이용할 수 있도록 자바 8에서 새로 제공하는 CompletableFuture 클래스(Future 인터페이스를 구현한 클래스) 를 살펴본다.

CompletableFuture는 Stream과 비슷하게 람다표현식과 파이프라이닝을 활용한다.

### 16.1.2 CompletableFuture로 비동기 애플리케이션 만들기

CompeletableFuture를 사용해보는 애플리케이션을 만들어본다.

예로, 온라인 상점 중 가장 저렴한 가격을 제시하는 상점을 찾는 애플리케이션을 구현한다고 하자.

이 애플리케이션에서 다음 기술을 배운다.

- 고객에게 비동기 API를 제공하는 방법을 배운다.
- 동기 API를 사용해야 할 때 코드를 비블록으로 만드는 방법을 배운다.
  두 개의 비동기 동작을 파이프라인으로 만드는 방법을 배운다.
  두 개의 동작 결과를 하나의 비동기 계산으로 합치는 방법을 배운다.
- 비동기 동작의 완료에 대응하는 방법을 배운다.
  모든 상점에서 가격 정보를 얻을 때까지 기다리는 것이 아니라,
  각 상점에서 가격 정보를 얻을 때마다 즉시 최저가격을 찾는 애플리케이션을 만든다.

**동기 API와 비동기 API**

- 동기 API : 메서드를 호출한 다음에 메서드가 계산을 완료할 떄까지 기다림.
  다른 스레드에서 실행되는 상황이더라도 호출자는 피호출자의 동작 완료를 기다린다.
  이렇게 동기 API를 사용하는 상황을 **블록 호출**이라고 한다.
- 비동기 API : 메서드가 즉시 반환되며 끝내지 못한 나머지 작업을 호출자 스레드와 동기적으로 실행할 수 있도록 다른 스레드에 할당함.
  이렇게 비동기 API를 사용하는 상황을 **비블록 호출**이라 함.

주로 I/O 시스템 프로그래밍에서 이런 방식의 동작 수행

## 16.2 비동기 API 구현

최저 가격 애플리케이션 구현에선,
상점은 특정 상품명에 해당하는 제품의 가격을 반환해야한다.

```java
public class Shop {
	public double getPrice(String product) {
	}

	private double calculatePrice(String product){
		//시간이 걸리는 작업
	}
}
```

getPrice 메서드는 상점 DB를 조회하거나 API통신등을 하여 시간이 걸릴 수 있다.

따라서 getPrice메서드의 작업시간동안 블록하는 것은 바람직하지 않고 비동기 API로의 전환에 필요성이 생겼다.

### 16.2.1 동기 메서드를 비동기 메서드로 변환

```java
public Future<Double> getPriceAsync(String product) {
	CompletableFuture<Double> futurePrice = new CompletableFuture<>(); // Future 생성

	new Thraed( () -> {
		// 시간이 오래걸리는 작업. 다른 스레드에서 비동기적으로 계산된다.
		double price = calculatePrice(produdct); 
		futurePrice.complete(price); // 오랜 시간이 걸리는 계산이 완료되면 Future에 값을 설정한다.
	}).start();

	// 계산 결과를 기다리지 않고 Future를 반환한다.	
	return futurePrice;
}
```

시간이 오래걸리는 작업이후 가격 정보가 계산되면 complete 메서드를 이용하여 CompletableFuture를 종료할 수 있다.

getPriceAsync() 메서드는 작업 결과가 계산되길 기다리지 않고 Future를 반환한다.
따라서 호출자 스레드는 블록되지 않고 다른 작업을 실행할 수 있다.

다음처럼 getPriceAsync() 메서드를 사용할 수 있다.

```java
Shop shop = new Shop("BestShop");

Future<Doule> futurePrice = shop.getPriceAsync("product");

// 상품 가격을 가져오는 동안 수행
doSomethingElse();

// Future에서 상품 가격 정보 불러오기
try{
	//가격 정보가 있으면 읽고, 없으면 상품 가격정보를 받을 떄까지 블록한다.
	double price = futurePrice.get();
}catch(Exception e){
	throw new RuntimeException(e);
}
```

가격 계산이 끝나기 전에 Future에 담겨진 채로 바로 반환받을 수 있다.

### 16.2.2 에러 처리 방법

가격 계산을 하는 동안 에러가 발생한다면 에러는 해당 스레드에만 영향을 끼친다.
즉, 에러가 발생해도 가격 계산은 계속 진행되고 일의 순서가 꼬인다.

결과적으로 호출자에서는 get() 메서드가 반환될때까지 계속 기다리게 된다.

이 때문에 get() 메서드에 오버로드 버전을 만들어서 문제를 해결할 수 있다.
오버로드 버전을 활용하면 타임아웃 시간이 지났을 때 TimeoutException을 받을 수 있고,
영원히 블록할 수 있는 가능성을 없앨 수 있다.

하지만 이 경우 제품 가격 계산중에 왜 에러가 발생했는지 알 수 없다.
CompleteExceptionally 메서드를 사용하여 CompletableFuture 내부에서 발생한 예외를 클라이언트로 전달하여 해결할 수 있다

- CompletableFuture 내부에서 발생한 에러 전파

```java
public Future<double> getPriceAsync(Stirng product){
	CompletableFuture<Double> futurePrice = new CompletableFuture<>();

	new Thread(() -> {
		try{
			double price = calculatePrice(product);
			futurePrice.complete(price);
		} catch (Exception ex){
			// 예외가 발생했을 경우 예외를 포함시켜서 Future를 종료한다.
			futurePrice.completeExceptionally(ex);
		}
	}).start();
	
	return futurePrice;
}

```

위와 같이  completeExceptionally() 메서드를 사용하여 예외를 포함시켜서 Future를 종료할 수 있다.

**팩토리 메서드 supplyAsync로 CompletableFuture 만들기**

팩토리 메서드로 간단하게 CompletableFuture를 만듥 수 있다.

ex)

```java
public Future<Double> getPriceAsync(Stirng product){
	return CompletableFuture.supplyAsync(() -> calculatePrice(product));
```

supplyAsync 메서드는 Supplier를 인수로 받아서 CompletableFuture를 반환한다.
CompletableFuture는 Supplier를 실행해서 비동기적으로 결과를 생성한다.

ForkJoinPool의 Executor중 하나가 Supplier를 실행할 것이다.

두 번째 인수를 반든 오버로드 버전을 사용하면 다른 Executor를 지정할 수 있다.

## 16.3 비블록 코드 만들기

만약 상점 class의 모든 API가 동기, 블록 메서드이고 변경할 수 없는 경우라면..

블록메서드를 사용할 수 밖에 없는 상황에서 비동기적으로 사용하는 방법을 알아본다.

아래와 같이 상점 리스트가 있다고 가정하자.

```java
List<Shop> shops = Arrays.asList....
```

그리고 상품이름을 전달하면 상점 이름과 상품가격 문자열 정보를 포함하는 List를 반환하는 메서드를 구현해야 한다고 가정하자.

```java
public List<Stirng> findPrices(String product);
```

스트림을 사용하면 아래와 같이 구현할 수 있을 것이다.

```java
public List<String> findPrices(String product){
	return shops.stream()
		.map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product))
		.collect(toList())
}
```

getPrice 메서드가 1초 이상 걸리고 상점의 수가 4개라면,
동기적, 블록으로 실행되는 위 메서드는 최소 4초 이상 걸리게 된다.

### 16.3.1 병렬 스트림으로 요청 병렬화하기

다음과 같이 순차 계산을 병렬로 처리해서 성능을 개선할 수 있다.

```java
public List<String> findPrices(String product){
	return shops.parallelStream()
		.map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product))
		.collect(toList())
}
```

parallelStream으로 스트림을 병렬로 처리하여 성능을 개선할 수 있다.

### 16.3.2 CompletableFuture로 비동기 호출 구현하기

팩토리메서드 supplyAsync로 스트림요소를 CompletableFuture를 만들어보자.

```java
List<CompletableFuture<String>> priceFutures = 
		shops.parallelStream()
		.map(shop -> 
			CompletableFuture.supplyAsync( () ->
				String.format("%s price is %.2f", shop.getName(), shop.getPrice(product))))
		.collect(toList());
```

위는 원하는 결과인 List<String> 과 다르다.

원하는 결과를 얻기 위해선 CompletableFuture의 동작이 완료되고 결과를 추출하여 리스트를 만들어야 한다.

두 번째 map 연산을 List<CompletableFuture<String>> 에 적용할 수 있따.

CompletableFuture는 join 을 호출해서 모든 동작이 끝나기를 기다린다.

join메서드는 get 메서드와 같은 의미를 지니지만 join은 아무런 예외도 발생시키지 않는다는 점이 다르다. 따라서 try-catch도 필요없다.

다음처럼 구현할 수 있다.

```java
public List<String> findPrices(String prodcut){
	List<CompletableFuture<String>> priceFutures = 
		shops.parallelStream()
		.map(shop -> 
			CompletableFuture.supplyAsync( () ->
				String.format("%s price is %.2f", shop.getName(), shop.getPrice(product))))
		.collect(toList());

	return priceFutures.stream()
		.map(CompletableFuture::join) // 비동기연산이 끝나기를 기다림
		.collect(toList());
```

두 map 연산을 하나의 스트림 처리 파이프라인이 아닌 두 개의 스트림 파이프라인으로 처리했다는 사실이 중요하다.

스트림연산은 게으른 특성이 있으므로 하나의 파이프라인으로 모든 연산을 처리하면 가격 정보 요청이 동기적, 순차적으로 이루어지는 결과가 되어버린다.

CompletableFuture로 각 상점의 정보를 요청할 때 기존 요청 작업이 완료되어야 join이 결과를 반환하면서 다음 상점으로 정보를 요청할 수 있기 때문이다.

### 16.3.3 더 확장성이 좋은 해결 방법

코드를 실행할 수 있는 기기가 네 개의 스레드를 병렬로 실행할 수 있는 기기라고 가정한다.

이 경우 네 개의 상점에 대해 상품 가격을 조회한다고 가정하면 각 상점마다 하나의 스레드를 할당하고 병렬로 수행하여 작업시간을 최소화할 수 있다.

만약 상점이 다섯개라면 네 개의 스레드에서 어떤 스레드의 작업이 완료되어야 남은 하나의 상점에 대한 조회를 수행할 수 있따.

병렬 스트림을 사용한 버전과 CompletableFuture를 사용한 버전을 비교하면,

두 버전 모두 수행 시간을 비슷하지만

CompletableFuture는 병렬 스트림 버전에 비해 작업에 이용할 수 있는 다양한 Executor를 지정할 수 있다는 장점이 있다.

Executor로 스레드 풀의 크기를 조절하는 등 애플리케이션에 맞는 최적화된 설정을 해서 성능을 향상시킬 수 있따.

### 16.3.4 커스텀 Executor 사용하기

애플리케이션이 실제로 필요한 작업량을 고려한 풀에서 관리하는 스레드 수에 맞게 Executor를 만들면 좋다.

**스레드 풀 크기 조절**

스레드 풀이 너무 크면 CPU와 메모리 자원을차지하기 위해 경쟁하느라 시간이 낭비될 수 있고
스레드 풀이 너무 작으면 CPU의 일부 코어는 활용되지 않을 수 있다.

다음 공식으로 대략적인 CPU활용 비율을 계산할 수 있다.

```java
N(threads) = N(CPU) * U(CPU) * ( 1 + W/C)
```

N(CPU) : Runtime.getRuntime().availableProcessors()가 반환하는 코어 수

U(CPU) : 0과 1사이의 값을 갖는 CPU 활용 비율

W/C : 대기시간과 계산시간의 비율

삼점에서 가격 정보를 가져오는 위의 예제 애플리케이션에선,

상점의 수보다 많은 스레드를 가지고 있는 것은 낭비이다.

또 스레드 수가 너무 많으면 서버가 크래시될 수 있으므로 최대 개수를 적절히 정하는 것이 좋다.

ex) 위 최저가격 검색 애플리케이션에 맞는 커스텀 Executor

```java
private final Executor executor = 
	// 상점의 개수만큼의 스레드를 갖는 풀 생성, 최대 100개
	Executors.newFixedThreadPool(Math.min(shops.size(), 100), new ThreadFactory(){
		public Thread newThread(Runnable r){
			Thread t = new Thread(r);
			t.setDaemon(true); //포로그램 종료를 방해하지 않는 데몬스레드 사용
			return t;
		}
	});

	
```

데몬 스레드 : 자바 프로그램이 종료될 때 스레드도 강제로 실행이 종료될 수 있다. 일반 스레드와 성능은 같다.

이렇게 만든 Executor를 오버로드된 supplyAsync()메서드의 두 번째 인수로 전달할 수 있다.

```java
CompletableFuture.supplyAsync(
	() -> shop.getName() + " price is " + shop.getPrice(product), executor);
```

결과적으로 스레드 개수를 늘려서 4개를 초과한 상점을 검색하더라도 효율적인 검색을 할 수 있다.

⇒ 애플리케이션 특성에 맞는 Executor를 만들어서 CompletableFuture를 활용하는 것이 바람직하다.

**스트림 병렬화와 CompletableFuture 병렬화**

CompletableFuture를 이용하면 전체적인 계산이 블록되지 않도록 스레드풀의 크기를 조절할 수 있다.

- I/O 작업이 포함되지 않은 계산 중심의 동작을 실행할 때에는 스트림 인터페이스가 구현하기 간단하며 효율적일 수 있다.
- 작업이 I/O를 기다리는 작업을 병렬로 실행할 때는 CompletableFuture가 더 많은 유연성을 제공하며 작업에 따라 적합한 스레드 수를 설정할 수 있다.

## 16.4 비동기 작업 파이프라인 만들기

상점이 있고 상점에서는 할인율을 다음과 같이 제공할 수 있따.

```java
public class Discount {
	public enum Code {
		NONE(0),
		SILVER(5),
		GOLD(10),
		PLATINUM(15),
		DIAMOND(20);

		private final int percentage;
	
		Code(int percentage){
			this.percentage = percentage;
		}
	}
}
```

상점에서는 다음과 같이 상품의 가격 정보를 반환하는 getPrice 메서드를 갖는다

```java
public String getPrice(String product){
	double price = calculatePrice(product);
	Discount.Code code = 
		Discount.Code.values()[random.nextInt(Discount.Code.values().length)];
	
	return String.format("%s:%.2f:%s", name, price, code);
}

private double calculatePrice(String product){
	delay();
	return random.nextDouble()*productchatAt(0) + product.chatAt(1);

```

getPrice 예시 :

```java
BestPrice: 123.26:GOLD
```

### 16.4.1 할인 서비스 구현

상점에서 제공한 문자열의 파싱은 Quote 클래스로 캡슐화할 수 있다.

- Quote 클래스

```java
public class Quote {
	private final String shopName;
	private final double price;
	private final Discount.Code discountCode;
	
	public Quote(String shopName, double price, Discount.Code code){
		this.shopName = shopName;
		this.price = price;
		this.discountCode = code;
	}
	
	//상점에서 getPrice로 얻은 문자열을 정적 팩토리메서드로 넘겨서 Quote클래스를 반환받을 수 있다.
	public static Quote parse(String s){
		String[] split = s.split(":");
	
		String shipName = split[0];
		double price = Double.parseDouble(split[1]);
		Discount.Code discountCode = Discount.Code.valueOf(split[2]);
		
		return new Quote(shopName, price, discountCode);
	}

	public String getShopName(){
		return shopName;
	}
	
	public double getPrice(){
		return price;
	}
	
	public Discount.Code getDiscountCode(){
		return discountCode;
	}
}
```

할인 서버에서 할인율을 확인해서 최종가격을 계산할 수 있다.

Quote객체를 인수로 받아서 할인된 가격 문자열을 반환하는 applyDiscount 메서드를 제공하는 Discount 서비스도 만든다.

- Discount 서비스 (apply 메서드가 할인 서버에서 할인율을 확인한 뒤 가격을 계산하는 동작을 한다고 가정하며 아래처럼 delay를 추가하여 응답 지연을 흉내냄)

```java
public class Discount {
	public enum Code{
			...
	}
	
	public static String applyDiscount(Quote quote){
		return quote.getShopNamt() + " price is " + 
			Discount.apply(quote.getPrice(), quote.getDiscountCode());
	}
	
	private static double apply(double price, Code code){
		delay();
		return format(price * (100 - code.percentage) / 100);
	}
}
```

### 16.4.2 할인 서비스 사용

Discount는 원격서비스이므로 지연이 발생한다.

할인 서비스 이용의 가장 간단한 방법(순차적 동기 방식)으로 findPrices 메서드를 구현해보자.

- Discount서비스를 이용하난 순차적 동기방식의 findPrices

```java
public List<String> findPrices(String product){
	return shops.stream()
						.map(shop -> shop.getPrice(product)) // 상점에서 상품 가격 얻기
						.map(Quote::parse) // 상점에서 반환한 문자열을 Quote객체로 변환
						.map(Discount::applyDiscount) // Discount서비스에서 할인적용
						.collect(toList());
}
```

위 코드는 성능최적화와 거리가 멀다.

병렬스트림을 사용하여 쉽게 성능을 개선할 수 있지만 병렬스트림에서는 스레드 풀의 크기가 고정되어 있어서 상점의 수가 늘어났을 때 유연하게 대응할 수 없다.

CompletableFuture에서 수행하는 태스크를 설정할 수 있는 커스텀 Exceutor를 정의함으로써 CPU사용을 극대화할 수 있다.

### 16.4.3 동기 작업과 비동기 작업 조합하기

CompletableFuture에서 제공하는 기능으로 findPrices메서드를 비동기적으로 재구현할 수 있다.

```java
public List<STring> findPrices(String product){
	List<CompletableFuture<STring>> priceFutures = 
		shops.stream()
				.map(shop -> CompeltableFuture.supplyAsync(
						() -> shop.getPrice(product), executor))
				.map(future -> future.thenApply(Quote::parse))
				.map(future -> future.thenCompose(
						quote -> CompletableFuture.supplyAsync(
							() -> Discount.applyDiscount(quote), executor)))
				.collect(toList());
	
		return priceFutures.stream()
				.map(CompletableFuture::join)
				.collect(toList());
}
	
```

위 코드의 내용은 다음과 같다.

**가격 정보 얻기**

첫 번째 map에서는 각 상점에서 product의 가격을 비동기적으로 조회하고 반환값을 매핑한다.

따라서 첫 번째 map에서는 Stream<CompletableFuture<String>> 이 반환된다.

**Quote 파싱하기**

두 번째 map 에서는 첫 번재 결과 문자열을 Quote로 변환한다.

파싱동작에서는 I/O 등의 응답대기시간이 필요한 동작이 없으므로 지연없이 즉시 동작을 수행할 수 있다.

thhnApply 메서드를 호출한 다음에 문자열을 Quote 인스턴스로 변환하는 Function을 전달한다.

thenApply 메서드는 CompletableFuture가 끝날 때가지 블록하지 않는다.
즉, CompletableFuture가 동작을 완전히 완료한 다음에 ThneApply메서드로 전달된 람다표현식을 적용할 수 있다.

따라서 CompletableFuture<String>은 CompletableFuture<Quote>로 변환된다.

**CompletableFuture를 조합해서 할인된 가격 계산하기**

세 번째 map 연산에서는 할인전 가격에 원격 Discount서비스에서 제공하는 할인율을 적용한다.

원격에서의 실행이 포함되므로 동기적으로 작업을 수행해야 한다.(?)

람다표현식으로 이 동작을 팩토리 메서드 supplyAsync에 전달할 수 있다.(supplyAsync. 16.2 참고)

결국 두 가지 CompletableFuture로 이루어진 연쇄적으로 수행되는 두 개 의 비동기 동작을 만들 수 있다.

- 상점에서 가격 정보를 얻어와서 Quote롤 변환하기
- 변환된 Quote를 Discount 서비스로 전달해서 할인된 최종가격 획득하기\\

CompletableFuture API에선 두 비동기 연산을 파이프라인으로 ㅁ나들 수 있도록 thenCompose 메서드를 제공한다.

thenCompose메서드는 첫 번째 연산의 결과를 두 번째 연산으로 전달한다. 두 번째 연산은 첫 번째 연산에서 넘겨준 결과를 계산의 입력으로 사용한다.

최종적으로 세 개의 map 연산을 List로 collect하면 List<CompletableFuture<String>> 이 반환된다.

마지막으로 CompletableFuture가 완료되기를 기다렸다가 join으로 추출하여 결과를 얻어낼 수 있다.

### 16.4.4 독립 COmpletableFuture와 비독립 CompletableFuture 합치기

위 예제에선 첫 CompletableFuture의 결과를 두 번째 CompletableFuture로 전달했다.

이와 달리 독립적으로 실행된 두 개의 CompletableFuture 결과를 합쳐야 하는 상황이 발생할 수 있다.(두 CompletableFuture가 독립적이라서 서로의 동작 완료에 의존하지 않음)

이런 상황에서는 thenCombine 메서드를 사용한다.

thenCombine메서드는 두 CompletableFuture 결과를 어떻게 합칠지 정의하는 BiFunction을 두 번째 인수로 받는다.

ex)

```java
Future<Double futurePriceInUSD = 
	CompletableFuture.supplyAsync(() -> shop.getPrice(product))
		.thenCombine(
			CompletableFuture.supplyAsync(
				() -> exchangeService.getRate(Money.EUR, Money.USD)),
				(price, rate) -> price * rate
		));
```

두 연산을 합치는 동작이 단순 연산이라 별도 태스크에서 수행하여 자원을 낭비하지 않기 위해 thenCombineAsync 대신 thenCombine 사용.

### 16.4.5 Future의 리플렉션과 CompletableFuture의 리플렉션

CompletableFuture는 람다표현식을 사용하고 이 덕분에 동기 태스크, 비동기 태스크를 활용해서 복잡한 연산 수행 방법을 효과적으로 쉽게 정의할 수 있는 선언형 API를 만들 수 있다.

### 16.4.6 타임아웃 효과적으로 사용하기

Future의 계산 결과를 읽을 때는 무한정 기다리는 상황이 발생할 수 있으므로 블록을 하지 않는 것이 좋다. 자바 9의 CompletableFuture에선 orTimeout메서드로 이런 문제를 해결할 수 있다.

orTimeout 메서드는 지정된 시간이 지난 후에 CompletableFuture를 TimeoutException으로 완료하면서 또 다른 CompletableFuture를 반환할 수 있도록 내부적으로 ScheduledThreadExecutor를 활용한다.

orTimeout 메서드를 사용하면 TimeoutExceptuon이 발생했을 때 사용자가 쉽게 이해할 수 있는 메시지를제공할 수 있다.

일시적으로 서비스를 이용할 수 없는 상황에서 꼭 서버에서 얻은 값이 아닌 미리 지정된 값을 사용할 수 있는 상황에선 completeOnTimeout 메서드를 사용할 수 있다.

completeOnTimeout 메서드는 타임아웃이 발생했을 때 미리 지정된 값을 사용하도록 정의한다.

## 16.5 CompletableFuture의 종료에 대응하는 방법

get이나 join으로 CompletableFuture가 완료될 때까지 블록하는 방법 말고 다른 방식으로 대응하는 방법.

I/O 작업 등 원격 서비스는 얼마나 지연될 지 모른다.

### 16.5.1 최저가격 검색 애플리케이션 리팩터링

모든 가격 정보를 포함할 때까지 리스트 생성을 기다리지 않도록 프로그램을 고쳐야한다.

CompletableFuture의 스트림을 직접 제어해야 한다.

- Future 스트림을 반환하도록 findPrices 메서드 리팩터링

```java
public Stream<CompletableFuture<STring>> findPricesStream(String product){
	return shops.stream()
					.map(shop -> CompletableFuture.supplyAsync(
									() -> shop.getPrice(product), executor))
					.map(future -> future.thenApply(Quote::parse))
					.map(future -> future.thenCompose(quote ->
									CompletableFuture.supplyAsync(
										() -> Discount.applyDiscount(quote), executor)));
}
```

각 CompletableFuture에 동작을 등록하여 CompleableFuture의 계산이 끝났을 때 값을 소비할 수 있다.

자바 8 CompletableFuture의 thenAccept 메서드로 연산 결과를 소비할 수 있다.

thenAccept메서드는 Consumer를 인수로 받는다.

ex)

```java
findPricesStream("phone").map(f -> f.thenAccept(System.out::println));
```

thenAccept 메서드는 CompletableFuture가 생성한 결과를 어떻게 소비할 지 미리 지정했으므로 CompletableFuture<Void>를 반환한다.

가장 느린 상점에서 응답을 받아서 반환된 가격을 출력할 기회를 제공하고 싶다고 가정하면

다음과 같이 스트림의 모든 CompletableFuture<Void>를 배열로 추가하고 실행 결과를 기다려야 한다.(?? thenAccept로 동작을 전달했으니 join()을 하지 않아도 실행되어야 하지 않나?)

```java
CompletableFuture[] futures = findPricesStream("phone")
		.map(f -> f.thenAccept(System.out::println)) //Stream<CompletableFuture<Void>>
		.toArray(size -> new CompletableFuture[size]);
CompletableFuture.allOf(futures).join();
```

팩토리 메서드 allOf는 CompletableFuture 배열을 입력으로 ㅂ다아서 CompletableFuture<Void>를 반환한다.

전달된 모든 CompletableFuture가 완료되어야 completableFuture<Void>가 완료되기 때문에 allOf메서드가 반환하는 CompletableFuture에 join을 호출하면 원래 스트림의 모든 CompletableFuture의 실행 완료를 기다릴 수 있다.

하나의 작업이 끝나길 기다리는 상황에선 anyOf를 사용하면된다.

### 16.5.2 응용

p.529 ~ 530

## 16.6 로드맵

## 16.7 마치며

- 긴 동작을 실행할 댄 비동기 방식을 고려할 수 있다.
- 동기 API를 CompletableFuture로 감싸서 비동기적으로 소비할 수 있다.
- allOf, anyOf로 CompletableFuture 배열의 값이 완료되는 개수에 따라 기다릴지 말지 선택할 수 있다.
- 자바 9에선 CompletableFuture의 orTimeout, completeOnTimeout 메서드로 타임아웃 기능을 사용할 수 있다.

### 질문

- p.500 상단. 선언형으로 이용한다에서 선언형이 무엇?