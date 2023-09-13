# Chapter 11. null 대신 Optional 클래스
**내용**

- null 참조의 문제점과 null을 멀리해야하는 이유
- null대신 Optional
- Optional활용
- Optional에 저장된 값 확인하는 방법

NullPointerException 발생 가능성을 쉽게 처리하기 위해

## 11.1 값이 없는 상황을 어떻게 처리할까?

```java
public String getCarInsuranceName(Person person){
	return person.getCar().getInsurance().getName();
}
```

위 코드에서 person이 null이라면 또는 getCar(), getInsurance()의 반환값이 null이라면 NPE가 발생

### 11.1.1 보수적인 자세로 NullPointerException 줄이기

다음과 같이 null확인코드를 추가하여 NPE를 해결할 수 있다.

```java
public String etCarInsuranceName(Person person){
	if(person != null){
		Car car = person.getCar();
		if(car != null){
			...
		}
	}
	
	return "Unknown";
}
```

위 코드는

- 들여쓰기 수준이 증가하고 가독성이 나빠진다.

아래처럼 null값이 확인될 때마다 바로 값을 반환하여 null을 참조하지 않도록 할 수 있다.

```java
public String etCarInsuranceName(Person person){
	if(person != null){
		return "Unknonw"
	}
	Car car = person.getCar();
	if(car == null){
			return "Unknown";
	}
Insurance insurance = car.getInsurance();
	....
	
	return insurance.getName();
}
```

위 코드는

- 메서드에 많은 출구가 생긴다. ⇒ 유지보수가 어려워진다.
- null값 체크를 빠뜨리는 실수를 할 여지가 있다.

### 11.1.2 null때문에 발생하는 문제

- 에러의 근원 : NPE는 매우 자주 발생하는 에러
- 코드를 어지럽힘 : 중첩된 null 확인코드는 코드 가독성을 떨어뜨린다.
- 의미가 없음 : null은 아무 의미도 표현하지 않음. 특히 정적 언어에서 값이 없음을 표현하기위해 null을 사용하는 것은 적절치 않음.
- 자바 철학에 위배됨 : 자바는 개발자로부터 모든 포인터를 숨겼으나 예외가 있다. ⇒ null 포인터
- 형식 시스템에 구멍을 만듦 : null은 무형식이기에 모든 참조 형식에 할당될 수 있고 null이란 값이 퍼져나갔을 때 애초에 null이 어떤 의미로 사용되었는지 알 수 없음.

### 11.1.3 다른 언어는 null대신 무얼 사용하나?

**그루비**

안전 내비게이션 연산자(safe navigation operator)인 `?.`을 도입해서 null문제를 해결

```java
def carInsuranceName = persoon?.car?.insurance?.name
```

안전 네이게이션 연산자를 이용하여 NPE걱정 없이 객체에 접근할 수 있다.

호출 체인에 null값이 있다면 결과로 null이 반환된다.

**하스켈**

선택형값(optional value)을 저장할 수 있는 Maybe라는 형식 제공

Maybe는 주어진 형식의 값을 갖거나 or 아무 값도 갖지 않을 수 있음. ⇒ 자연스레 null참조 개념이 사라짐

**스칼라**

스칼라 또한 T형식의 값을 갖거나 아무 값도 갖지 않을 수 있는 Optional[T]라는 구조 제공

Optional 형식에서 값의 존재 여부를 명시적으로 확인해야하기 때문에 NPE 관련 문제의 가능성이 줄어든다.

**자바 8**

선택형값 개념의 영향을 받은 java.util.Optional<T> 클래스 제공

## 11.2 Optional 클래스 소개

Optional은 선택형값을 캡슐화하는 클래스

값이 있으면 Optional클래스는 값을 감싼다.

값이 없으면 Optional.empty메서드로 Optional을 반환한다.

Optional.empty는 Optional의 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드이다.

null참조와의 차이는

null을 참조하면 NPE가 발생하지만,

Optional.empty()는 Optional객체이므로 다양한 방식으로 활용 가능

Person의 멤버인 car를 Optional<Car>형식으로 변경하여 값이 없을 수 있음을 명시적으로 나타낼 수 있다.

```java
public class Persion { 
	private Optional<Car> car;
	public Optional<Car> getCar(){
		return car; 
	}
}

public class Car { 
	private Optional<Insurance> insurance;
	public Optional<Insurance> getInsurance(){
		return insurance;
	}
}
...		
```

Optionl<Car>형식과 다르게 Car 형식을 사용했을 때는 Car에 null참조가 할당될 수 있는데 이 값이 올바른 값인지, 잘못된 값인지 판단할 정보가 없다.

Optional클래스를 사용하여 모델의 의미가 더 명확해졌다.
(멤버가 null값이 될 수 있다는 것을 명시적으로 나타내어 설명한다)

모든 null참조를 Optional로 대치하는 것은 옳지 않다.
적절한 곳에 Optional클래스를 사용하여 선택형 값인지 여부를 구별할 수 있다.

## 11.3 Optional 적용 패턴

Optional을 활용하고 Optional로 감싼 값을 사용하기

### 11.3.1 Optional 객체 만들기

Optional을 사용하기 위해 Optional객체를 만들어야한다.

다양한 방법으로 생성 가능

**빈 Optional**

Optional.empty로 빈 Optional객체 생성 가능

```java
Optional<Car> optCar = Optional.empty();
```

**null이 아닌 값으로 Optional 만들기**

정적 팩토리 메서드 Optional.of로 null이 아닌 값을 포함하는 Optional 생성 가능

```java
Optional<Car> optCar = Optional.of(car);
```

이 경우 car가 null이라면 즉시 NPE 발생.

**null값으로 Optional만들기**

정적 팩토리 메서드 Optional.ofNullable로 null값을 저장할 수 있는 Optional 생성 가능

```java
Optional<Car> optCar = Optional.ofNullable(car);
```

car가 null이라면 빈 Optional 객체 반환

### 11.3.2 맵으로 Optional의 값을 추출하고 변환하기

ex) 보험회사의 이름을 추출한다고 가정할 때

아래와 같이 이름 정보에 접근하기 전 insurance가 null인지 확인해야 한다.

```java
String name = null;
if(insurance != null){
	name = insurance.getName();
}
```

이런 유형에 Optoonal의 map 메서드를 사용할 수 있다.

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

Optional이 값을 포함한다면 map의 인수로 제공된 함수가 값을 바꾼다.

Optional이 비어있으면 아무 일도 일어나지 않는다.

### 11.3.3 faltMap으로 Optional객체 연결

```java
	public String getCarInsuranceName(Person person){
		return person.getCar().getInsurance().getName();
}	
```

위 코드처럼 Optional 을 반환하는 메서드들이 연결되어 있을 때

map을 사용하여 다음과 같이 재구현할 수 있을 것 같다.

```java
Optional<Person> optPerson = Optional.of(person);
Optional<String> name = optPerson.map(Person::getCar)
																.map(Car::getInsurance)
																.map(Insurance::getName);
```

하지만 위 코드는 컴파일되지 않는다.

optPerson 은 Optional<Person>이고 map을 호출할 수 있다.

하지만 getCar()는 Optional<Car>형식의 객체를 반환하고 결과는 Optional<Optional<Car>> 이다.

따라서 다음 map함수에서 getInsurance 메서드를 사용할 수 없다.

flatMap 메서드를 사용하면 Optional을 포함하는 중첩된 Optional객체가 아닌

평탄화된 하나의 객체를 포함하는 Optional객체를 반환한다.

따라서 flatMap을 사용하여 다음과 같이 구현할 수 있다.

```java
public String getCarInsuranceName(Optional<Person> person){
	return person.flatMap(Person::getCar)
							.flatMap(Car::getInsurance)
							.map(Insurance::getName)
							.orElse("Unknown");
}
```

**도메인 모델에 Optional을 사용했을 때 데이터를 직렬화할 수 없는 이유**

Optional의 용도는 선택형 반환값을 지원하기 위함이다.

Optional클래스는 필드 형식으로 사용할 것을 가정하지 않았기때문에 Serializable인터페이스를 구현하지 않는다. 따라서 도메인 모델에 Optional을 사용한다면 직렬화모델을 사용하는 도구나 프레임워크에 문제가 생길 수 있다.

이럼에도 Optional을 사용한 도메인 모델 구성은 필요할 수 있다. 특히 객체가 null일 수 있는 상황에선.

따라서 직렬화 모델이 필요하다면 Optional로 값을 반환받을 수 있는 메서드를 추가하는 방식이 권장된다.

```java
public class Person { 
	private Car car;
	public Optional<Car> getCarAsOptional()){
		return Optional.ofNullable(car);
	}
}
```

### 11.3.4 Optional 스트림 조작

자바 9에선 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream() 메서드를 추가했다. Optional 스트림을 값을 가진 스트림으로 변환할 때 유용하게 활용 가능.

ex)

```java
public Set<String> getCarInsuranceNames(List<Person> persons){
	return persons.stream()
								.map(Person::getCar)
								.map(optCar -> optCar.flatMap(Car::getInsurance)) // Optional<Car> => Optional<Insurance>
								.map(optIns -> optIns.map(Insurance::getName)) // Optional<Insurance> => Optional<String>
								.flatMap(Optional::stream) //Stream<Optional<String> => Stream<String>
								.collect(toSet());
```

Optional 덕에 null 걱정없이 안전하게 연산을 처리할 수 있다.

마지먹 결과를 얻으려면 빈 Optional을 제거하고 값을 언랩해야한다.

다음처럼 filter, map을순서적으로 이용해서 결과를 얻을 수도 있다.

```java
Stream<Optional<String> stream = ...
Set<String> result = stream.filter(Optional::isPresent)
													.map(Optional::get)
													.collect(toSet());
```

첫 번째 예제에서처럼 Optional클래스의 stream()메서드를 이용하면 한 번의 연산으로 위와 같은 결과를 얻을 수 있다. stream()메서드는 각 Optional이 비어있는지 아닌지에 따라서 Optional을 0개 이상의 항목을 포함하는 스트림으로 변환해준다.

stream()메서드를 사용하여 값을 포함하는 Optional을 언랩하고 비어있는 Optional은 건너뛸 수 있다.

### 11.3.5 디폴트 액션과 Optional 언랩

빈 Optional인 상황에서 기본값을 반환하도록 하는 orElse.

이외에도 Optional인스턴스에 포함된 값을 읽는 다양한 방법이 존재.

- get() : 값을 읽는 안전하지 않은 메서드. 값이 없으면 NoSuchElementException이 발생하기 때문에 값이 확실히 있는 경우에만 사용한다. Optional을 사용하는 관점에서는 사용할 이유가 없는 메서드.
- orElse(T other) : Optional이 값을 포함하지 않을 때 기본값을 제공할 수 있음.
- orElseGet(Supplier<? extends T> other) : orElse메서드에 대응하는 게으른 버전의 메서드. 디폴트 메서드를 만드는데 시간이 걸리거나(효율성 등) Optional이 비어있을 때만 기본값을 생성하고 싶을 때 사용.
- orElseThrow(Supplier<? extends X> exceptionSupplier) : Optional이 비어있을 때 인수로 전달한 예외를 발생시킬 수 있다.
- ifPresent(Consumer<? super T> consumer) : 값이 존재할 때 인수로 넘겨준 동작 실행. 값이 없으면 아무 동작 안함.
- (자바 9에 추가됨) ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction) : ifPresent 메서드에 추가로 Optional이 비어있을 경우 실행할 수 있는 Runnable을 인수로 받음.

### 11.3.6 두 Optional 합치기

ex) Person과 Car 정보를 이용하여 가장 저렴한 보험료를 제공하는 보험회사를 찾는 서비스가 있다고 가정.

```java
public Insurance findCheapestInsurance(Person person, Car car){
	// 모든 결과 데이터 비교
	return cheapestCompany;
}
```

다음으로 두 Optional을 인수로 받아서 Optional<Insurance>를 반환하는 null에 안전한 버전의 메서드를 구현해야 한다고 가정한다.

하나라도 빈 값이 있으면 빈 Optional<Insurance>를 반환한다.
isPresent를 이용하여 다음과 같이 구현할 수 있다.

```java
public Optional<Insurance> nullSafeFindChapeastInsurance(
	Optional<Person> person, Optional<Car> car) {
		if(person.isPresent() && car.isPersent()){
			return Optional.of(findCheapestInsurance(person.get(), car.get()));
		} else{
			return Optional.empty();
		}
}
```

장점은

person과 car의 시그니처만으로 둘 다 아무 값도 반환하지 않을 수 있다는 것을 명시적으로 보여줄 수 있다.

하지만 코드 자체는 null확인 코드와 크게 다른점이 없다.

**Optional 언랩하지 않고 두 Optional 합치기**

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(
	Optional<Person> person, Optional<Car> car){
		return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```

Optional<Person>인 person의 flatMap 메서드에서 Optional이 비어있다면 인수로 전달한 람다 표현식은 실행되지 않고 빈 Optional을 반환한다.

반면 person이 값을 갖고 있으면 flatMap에 인수로 전달된 람다가 실행된다.

그리고 Optional<Car>인 car의 map 메서드에서도 마찬가지로 car가 값을 가지고있지 않다면 빈 Optional을 반환하고 값이 있다면 findCheapestInsurance 메서드를 호출하므로 두 값이 모두 존재할 때만 메서드를 실행할 수 있다.

### 11.3.7 필터로 특정값 거르기

Optional 없이 객체의 메서드를 호출해서 어떤 프로퍼티를 확인해야한다면

해당 객체가 null인지 여부를 확인한 후에 메서드를 호출해야 한다.

eX)

```java
Insurance insruance = ...;

if(insurance != null && "CambridgeInsurance".equals(insurance.getName())){
	...
}
```

Optional객체와 Optional객체의 filter메서드를 사용하여 다음처럼 재구현할 수 있다.

```java
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance ->
									"CambridgeInsurance".equans(insurance.getName()))
						.ifPresnet(x -> ...);
```

filter메서드는 프레디케이트를 인수로 받는다.

Optional객체가 값을 가지고 프레디케이트가 true이면 값을 반환하고
그렇지 않다면 빈 Optional객체를 반환한다.

## 11.4 Optional을 사용한 실용 예제

### 11.4.1 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기

예로 Map의 get메서드는 값을 찾지 못했을 때 null을 반환한다.

get메서드의 시그니처를 고칠 순 없어도 반환값을 Optional로 감싸서 Optional객체로 처리할 수 있다.

```java
Optional<Object> value = Optional.ofNullable(map.get("key"));
```

### 11.4.2 예외와 Optional 클래스

예로 Interger.parseInt(String str) 의 경우, 변환이 불가능할때 NumberFormatException을 발생시키는데

정수로 변환할 수 없는 문자열 문제를 빈 Optional을 반환하도록 하여 유틸 메서드를 구현할 수 있다.

```java
public static Optional<Integer> stringToInt(String s){
	try{
		return Optional.of(Integer.parseInt(s));
	}catch(NumberFormatException e)_{
		return Optional.empty();
	}
}
```

유틸 메서드를 통해 기존처럼 거추장한 try/catch 로직을 사용할 필요가 없다.

### 11.4.3 기본형 Optional을 사용하지 말아야 하는 이유

기본형 특화인 OptionalInt, OptionalLong.. 등의 클래스도 존재한다.

위 예시에서도 Optional<Integer> 대신 OptionalInt를 반환할 수도 있음

스트림처럼 값이 많을때는 기본형 메서드로 성능향상이 가능하지만 Optional은 값이 하나이므로 기본형을 쓴다 해도 성능개선을 하기 힘들다.

또한 기본형 특화 Optional은 map, flatMap, filter 등 메서드도 지원하지 않아서 기본형 특화 메서드는 권장되지않음.

### 11.4.4 응용

386p ~ 387p 읽어보기

## 11.5 마치며

- 자바 8에선 값이 있거나 없음을 표현할 수 있는 Optional<T> 클래스를 제공
- 팩토리메서드로 값이 있거나 없는 Optional객체 생성 가능
- Optional을 활용하여 더 좋은 API 설계 가능. 사용자는 메서드 시그니처만으로 선택적인 값이 사용되거나 반환되는지 예측할 수 있다.