## 36. 비트 필드 대신 EnumSet을 사용하라

예전엔 열거한 값들이 주로 단독이 아닌 집합으로 사용될 경우,

각 상수에 서로 다른 2의 거듭제곱값을 할당한 정수 열거 패턴을 사용해왔다

ex) 비트 필드 열거 상수

```java
public class Text{
		public static final int STYLE_BOLD = 1 << 0;
		public static final int STYLE_ITALIC = 1 << 1;
		public static final int STYLE_UNDERLINE = 1 << 2;
		public static final int STYLE_STRIKETHROUGH = 1 << 3;

		// 매개변수 styles는 0개 이상의 STYPE_.. 상수를 비트별 OR한 값
		public void applyStyles(int styles){...}
	}
```

비트별 OR연산을 사용해서 여러 상수를 하나의 집합으로 모을 수 있으며

(OR연산으로 조합의 경우의 수를 나타낼 수 있음. 예를 들어서 13이면 STYLE_BOLD, STYLE_UNDERLINE, STYLE_STRIKETHROUGH의 집합임을 알 수 있다.)

이렇게 만들어진 집합을 비트필드라고 한다.

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

비트필드를 사용하면 비트별 연산을 사용해서 합집합, 교집합 등 집합연산을 효율적으로 수행할 수 있다.

**단점**

- 정수 열거 상수의 단점을 그대로 지닌다.
- 비트 필드값이 그대로 출력되면 단순한 정수 열거 상수를 출력할때보다 해석하기가 더 어렵다.
- 비트필드 하나에 녹아있는 모든 원소를 순회하기가 까다롭다.
- 최대 몇 비트가 필요한지를 API작성시 미리 예측하여 적절한 타입을 선택해야한다.

**대안책**

비트필드보다 더 나은 대안으로 java.util.EnumSet이 있다.

Set인터페이스를 완벽구현하며, 타입 안전하고 어떤 Set 구현체와도 함께 사용할 수 있다.

EnumSet의 내부는 비트벡터로 구현되었다.

비트벡터 : 중복되지 않는 정수 집합을 비트로 나타내는 방법

따라서 원소가 총 64개 이하라면(대부분의 경우) EnumSet전체를 long변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.

removeAll, retainAll 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다.

난해한 작업들은 EnumSet이 다 처리해주기때문에 비트를 직접 다룰 때 겪는 오류들에서 해방된다.

ex) EnumSet - 비트필드를 대체하는 기법

```java
public class Text{
	public enum Style{BOLD,ITALIC, UNDERLINE, STRIKETHROUGH}
	
	public void applyStyles(Set<Style> styles){...}
}
```

```java
text.applyStyles(EnumSet.of(STyle.BOLD, Style.ITALIC));
```

applyStyle메서드 매개변수는 EnumSet이 아닌 Set이다.

항상 EnumSet이 건네질것이라 짐작되어도 인터페이스로 받는 것이 일반적으로 좋다.

다른 Set구현체를 넘겨도 처리할 수 있기 때문이다.

**결론**

열거할 수 있는 타입을 모아서 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다.

EnumSet을 사용하자.(유일한 단점은 불변 EnumSet을 만들 수 없다는 점)