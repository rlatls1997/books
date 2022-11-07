## 37. ordinal인덱싱 대신 EnumMap을 사용하라

종종 배열이나 리스트에서 원소를 꺼낼 때 ordinal메서드로 인덱스를 얻는 코드가 있다

ex)

```java
class Plant{
	enum LifeCycle {ANNUAL, PERENNITAL, BIENNIAL}

	final String name;
	final LifeCycle lifeCycle;
	
	Plant(String name, LifeCycle lifeCycle){
		this.name = name;
		this.lifeCycle = lifeCycle;
	}

	@Override
	public String toSTring(){
		return name;
	}
}
```

Plant배열을 만들고 ordinal 메서드로 배열을 순회하게 할 수 있따.

```java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for(int i = 0; i<plantsByLifeCycle.length; i++){
	plantsByLifeCycle[i] = new HashSet<>();
}

// garden의 Plant들을 생애주기별로 집합에 넣는..
for(Plant p : garden){
	plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
}

//결과 출력
for(int i = 0; i< plantsByLifeCycle.length; i++){
	sout("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

위 코드는 동작은 하지만 문자가 많다.

- 배열은 제네릭과 호환이 되지 않으므로 비검사 형변환을 수행햐야 하고 깔끔하게 컴파일되지 않는다.
- 배열은 각 인덱스의 의미를 모르기에 출력 결과에 직접 레이블을 달아야 한다.
- 정확한 정수값을 사용한다는 것을 직접 보증해야한다. 정수는 열거타입과 달리 타입안전하지 않고, 잘못된 값을 사용하면 잘못된 동작을 수행하거나 ArrayINdexOutOfBoundException을 던질것임.

해결책으로 EnumMap을 사용할 수 있다.

EnumMap은 열거타입을 키로 사용하도록 설계한 Map구현체이다.

ex) EnumMap 사용

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
for(Plant.LifeCycle lc : Plant.LifeCycle.values()){
	plantsByLifeCycle.put(lc, new HashSet<>());
}

for(Plant p : garden){
	plantsByLifeCycle.get(p.lifeCycle).add(p);
}

sout(plantsByLifeCycle);
```

EnumMap을 사용하여 더 짧고 명료하고 안전하고 성능도 비슷한 코드를 짤 수 있다.

안전하지 않은 형변환은 사용하지 않고

맵의 키인 열거타입이 출력용 문자열을 제공하므로 출력결과에 직접 레이블을 달 일도 없다.

그리고 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 닫힌다.

EnumMap의 성능이 ordinal을 사용한 배열과 비견되는 이유는 EnumMAp내부에서 배열을 사용하기 때문이다.

EnumMap의 생성자가 받는 키 타입의 Class객체는 한정적 타입 토큰으로 런타임 제네릭 타입 정보를 제공한다.

스트림을 사용하면 코드를 더 줄일 수 있다.

ex) 스트림 사용 코드 - EnumMap을 사용하지 않는다.

```java
sout(Arrays.stream(garden)
	.collect(groupingBy(p -> p.lifeCycle));
```

위 코드는 간결하지만 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에
EnumMap을 써서 얻는 공간, 성능의 이점이 사라진다.

이를 해결하기 위해 매개변수 3개짜리 groupingBy메서드를 사용하여 매개변수에 원하는 맵 구현체를 명시할 수 있따.

ex) 스트림 사용 코드 - EnumMap을 이용해 데이터와 열거 타입 매핑

```java
sout(Arrays.tream(garden)
	.collect(groupingBy(p -> p.lifeCycle,
		() -> new EnumMap<>(LifeCycle.class). toSet())));
```

**스트림을 사용했을때와 EnumMap만 사용했을때의 차이점**

EnumMap만 사용하면 항상 식물의 생애주기당 하나씩의 중첩 맵을 만든다

스트림은 해당생애주기에 속하는 식물이 있을 때만 만든다.

두 열거타입값을 매핑하기 위해 ordinal() 메서드를 사용하는 배열이 있다.

예로, 두 가지 상태를 전이-매핑하도록 구현한 프로그램을 보자

ex) 배열의 인덱스에 ordinal()메서드 사용 - 나쁜예

```java
public enum Phase{
	SOLID, LIQUID, GAS;
	
	public enum Transition {
		MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
	
		// 각 Phase에서 Phase로의 전이 정보를 담고있는 이차원배열
		private static final Transition[][] TRANSITIONS = {
			{ null, MELT, SUBLIME },
			{ FREEZE, null, BOIL },
			{ DEPOSIT, CONDENSE, null }
		};

		// 한 상태에서 다른 상태로의 전이
		public static Transition from(Phase from, Phase to){
			return TRANSITIONS[from.ordinal()][to.rodinal()];
		}
```

위 코드가 구린 이유는

Phase나 Phase.Transition열거타입을 수정하면서 TRANSITIONS 배열도 함께 수정해야한다는 점이다. 만약 수정을 까먹으면 런타임 오류가 나거나 잘못된 동작을 하게될 수 있따.

또한 새 Phase마다 새로 만들어줘야할 전이정보의 가짓수는 Phase의 개수만큼 점점 커져가기 때문에 코드변경이 많아서 수정이 어렵다.

EnumSet을 사용할 수 있따.

ex) 중첩 EnumMap으로 데이터, 열거타입 쌍 연결

```java
public enum Phase {
	SOLID, LIQUID, GAS;

	public enum Transition {
		MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID), BOIL(LIQUID, GAS),
		CONDENSE(GAS, LIQUID), SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

		private final Phase from;
		private final Phase to;
	
		Transition(Phase from, Phase to){
			this.from = from;
			this.to = to;
		}
		
		//전이 정보를 담은 맵
		private static final Map<Phase, Map<Phase, Transition>> m = 
			Stream.of(values())
						.collect(groupingBy(t -> t.from,
																() -> new EnumMap<>(Phase.class),
																toMap(t -> t.to,
																			t -> t,
																			(x, y) -> y,
																			() -> new EnumMap<>(Phase.class))));
	
		public static Transition from(Phase from , Phase to){
			return m.get(from).get(to);
		}
	}
}
```

```java
{
SOLID={LIQUID=MELT, GAS=SUBLIME}, 
LIQUID={SOLID=FREEZE, GAS=BOIL}, 
GAS={SOLID=DEPOSIT, LIQUID=CONDENSE}
}
```

중첩 EnumMap을 사용할 수 있다.

중첩 맵 m은 이전상태에서 이후상태로의 전이정보에 대한 EnumMap을 value로 갖는다.

새로운 상태 PLASMA를 추가한다고 가정하면,
그리고 기체에서 플라즈마로 변하는 전이인 IONIZE, 플라즈마에서 기체로 변하는 전이인 DEIONIZE 가 추가된다고 가정해보자.

만약 배열로 만들었던 코드라면 Phase에 상수 1개, Transition에 2개를 추가하고 3x3이었던 TRANSITIONS배열을 4x4인 이차원 배열로 수정해야한다. 이를 수정하는 과정은 런타임에서 문제를 일으키는 코드를 야기할 수 있다.

반면 EnumMap버전에서는 상태목록에 PLASMA를 추가하고 전이목록에 IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS)만 추가하면 된다.

새 상수를 추가하면서 버그를 만들 가능성이 줄어들어 명확하고 안전하게 유지보수할 수 있다.

**결론**

배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 좋지 않다. EnumMap을 사용하자.

다차원 관계라면 중첩된 EnumMap을 사용하자.