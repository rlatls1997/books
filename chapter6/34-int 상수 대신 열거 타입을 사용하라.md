## 34. int 상수 대신 열거 타입을 사용하라

정수열거패턴

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
...
public static final int ORANGE_PIPPIN = 1;
```

정수열거패턴의 단점

- 타입 안전을 보장할 방법이 없다.(오렌지를 건내야할 메서드에 사과를 보내도 어떤 경고도 발생하지 않는다)
- 표현력이 좋지 않다
- 같은 열거 그룹에 속한 모든 상수를 순회하는 방법이 마땅치 않다.
- 상수가 몇 개인지 알기 힘들다.

결론은 런타임 버그를 생성할 가능성이 높아진다.

열거타입을 사용하자

ex) 열거타입 사용

```java
public enum Apple {FUJI, PIPPIN, ...}
public enum Orange {PIPPIN.....}
```

자바의 열거 타입은 완전한 형태의 클래스이다.

자바의 열거 타입은 상수 하나당 자신의 인스턴스를 하나씩 만든다.

열거타입은 밖에서 접근가능한 생성자를 제공하지 않으므로 final이라 볼 수 있다. 인스턴스를 생성하거나 확장할 수 없으니 열거타입 인스턴스는 딱 하나씩만 존재한다

즉, 열거타입은 인스턴스 통제되고 싱글턴이다.

열거타입의 장점

- 다른 열거 타입끼리의 비교를 컴파일타임에 방지할 수 있다
- 열거타입에는 각자의 이름공간이 있어서 이름이 같은 상수도 공존할 수 있다.
- 열거타입의 toString메서드는 출력하기 적합한 문자열을 내어준다.
- 임의의 메서드나 필드를 추가할 수 있고 인터페이스를 구현하게 할 수도 있다.

ex) 데이터와 메서드를 갖는 열거타입

```java
public enum Planet {
	MERCURY(1, 1),
	VENUS(2, 2),
	...
	NEPTUNE(9, 9);
	
	private final double mass;
	private final double radius;
	private final double surfaceGravity;

	private static final double G = 6.67300E-11;
	
	Planet(double mass, double radius){
		this.mass = mass;
		this.radius = radius;
		surfaceGravity = G * mass / ...
	}

	public double getMass(){
		return mass;
	}

	...

	public double surfaceWeight(double mass){
		return mass * surfaceGravoity;
	}
}
```

열거타입을 만드는것은 어렵니 않다.

열거타입 상수 각각을 특정 데이터와 열결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.

열거타입은 근본적으로 불변이라 모든 필드는 final이어야 한다.

필드는 public이어도 되지만 private으로 두고 접근자를 제공하는 편이 좋다.

열거타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values를 제공한다. 값들은 선언된 순서로 저장된다.

열거타입값의 toString메서드는 상수 이름을 문자열로 반환하므로 출력하기에도 쉽다.

범용적으로 사용되는 열거타입은 톱레벨 클래스로 만들고 특정 톱레벨 클래스에서만 사용된다면 멤버클래스로 만들자.

**상수별 메서드 구현**

switch-case문으로 상수별동작을 명시할 수 있따

ex)

```java
public enum Operation{
	PLUS, MINUS, TIMES, ..;

	...
	public double apply(double x, double y){
		switch(this){
			case PLUS: return x+y
			...
		}
		
		throw new AssertionError();
	}
}
```

동작은 하지만 좋지 않다.

마지막 throw엔 실제론 도달할 일이 없어도 기술적으론 도달할 수 있기 때문에 생략하면 컴파일이 되지 않는다.

또한 깨지기 쉬운 코드이다. 만약 상수를 추가하면 해당 case문도 추가해야한다. 이를 깜빡하면 런타임에 오류가 발생할 것이다.

더 나은 방법으로 apply라는 추상멧드를 선언하고 각 상수에 맞게 정의할 수 있따

이를 **상수별 메서드 구현이라고 한다**

```java
public enum Operation {
	PLUS {public double apply(double x, double y){return x+y;},
	...;

	public asbstract double apply(double x, double y);
}
```

상수를 추가할 때 apply메서드를 깜빡하긴 어려울 것이고 추상메서드이므로 정의하지 않는다면 컴파일타임에 알려줄 것이다.

상수별 메서드 구현을 상수별 데이터와 결합해서도 사용할 수 있다.

**valueOf(String)**

열거타입에는 상수 이름을 입력받아서 그 이름을에 해당하는 상수를 반환해주는 valueOf(String) 메서드가 자동 생성된다.

열거타입의 toSTring 메서드를 재정의하려거든 toSTring이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString메서드(커스텀메서드)도 함께 제공하는 것을 고려해보자.

ex)

```java
private static final Map<String, Operation> stringToEnum = 
	String.of(values()).collect(
		toMap(Object::toString, e -> e));

public static Optional<Operation> fromString(String symbol){
	return Optional.ofNullable(stringToEnum.get(symbol));
}
```

Opreation 상수가 stringToEnum맵에 추가되는 시점은 열거 타입 상수 생성 후 정적 필드가 초기화될 때다.

열거타입의 정적 필드 중 열거타입의 생성자에서 접근할 수 있는 것은 상수 변수뿐이다.

상수별 메서드 구현에선 열거 타입 상수끼리 코드를 공유하기 어렵다.

값에 따라 분기하여 코드를 공유하는 열거타입을 보자.

```java
enum PayrollDay{
	MONDAY, TUESDAY, ...;

	privaet static final int MIN_PER_SHIFT = 8 * 60;

	int pay(int minutesWorked, int payRate){
		int basePay = minutesWorked * payRate;
	
		int overtimePay;
	
		switch(this{
			case SATURDAY: case SUNDAY:
				overtimePay = basePay...
				break;
			default
				...
		}
	
		return basePay + overtimePay;
}
```

간결하지만 관리면에서는 어렵다.

휴가와 같은 새로운 열거타입이 추가된다면 그 값을 처리하는 case문을 잊지 말고 넣어줘야한다.

상수별 메서드드 구현으로 위 계산을 하는 법은 두가지이다.

- 모든 상수에 잔업수당을 계산하는 메서드를 넣는다.
- 계산코드를 평일용, 주말용으로 나눠 각각을 도우미메서드로 작성한 다음 각 상수가 자신에게 필요한 메서드를 호출하게 한다.

두 방법 모두 가독성이 크게 떨어지고 오류 발생 가능성이 높아진다.

가장 깔끔한 방법은,

새로운 상수를 추가할 때 잔업수당 **전략**을 선택하도록 하는 것.

잔업수당 계산을 private 중첩 열겨ㅓㅌ타입으로 옮기고 PayRollDay열거타입의 생성자에서 적당한 private 열거타입을 선택한다.

switch문보다는 복잡하지만 더 안전하고 유연하다.

ex) 전략 열거 타입 패턴

```java
enum PayrollDay{
	MONDAY(WEEKDAY), TUESDAY(WEEKDAY), ...;

	privaet static final int MIN_PER_SHIFT = 8 * 60;

	PayrollDay(PayType payType){this.payType = payType;}

	int pay(int minutesWorked, int payRate){
		return payType.pay(minutesWorked, payRate)
	}
	
	enum PayType{
		WEEKDAY {
			int overtimePay(int minsWorked, int payRate){
				return ...
			}
		},
		WEEKEND {
			int overtimePay(int minsWorked, int payRate){
				return ...
			}
		};

		abstract int overtimePay(int minsWorked, int payRate);
		
		int pay(int minutesWorked, int payRate){
			...
		}			
	}

}
```

하지만 기존 열거 타입에 상수별 동작을 혼합해 넣을때는 switch문이 좋은 선택이 될 수 있따.

ex)

```java
public static Opertaion inverse(Operation op){
		swtich(op){
			case PLUS: return Operation.MINUS;
			case MINUS: return Operation.PLUS;
			...
		}
}
```

열거타입은 언제 사용하나?

**필요한 원소를 컴파일 타임에 다 알 수 있는 상수집합이라면 항상 열거타입을 사용하자**