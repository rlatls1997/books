## 10. equals는 일반 규약을 지켜 재정의하라.

만약 equals() 메서드를 구현하지 않는다면 그 클래스의 인스턴스는 오직 자기 자신과만 같게 된다.

따라서 다음의 상황에 해당한다면 재정의하지 않는 것이 최선이다.

### equals를 재정의하지 않는 것이 좋은 경우

- **각 인스턴스가 본질적으로 고유한 경우**
  값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 여기에 해당한다. (ex, Thread)
- **인스턴스의 논리적 동치성(logical equality)을 검사할 일이 없을 경우**
  Pattern클래스를 예로 들면, equals를 재정의해서 두 Pattern의 인스턴스가 같은 정규표현식을 나타내는지 검사를 할 수 있다. 이런 경우를 논리적 동치성 검사라고 한다.

  만약 논리적 동치성 검사가 필요하지 않은 경우라면 기본 equals를 사용하자.

- **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는 경우**
  예로, 대부분의 Set구현체는 AbstractSet이 구현한 equals를 받아쓴다. (List → AbstractList, Map → AbstractMap).
- **클래스가 private이거나 package-private이고 equals메서드를 호출할 일이 없는 경우**
  이 경우에는 실수로라도 equals가 호출되는 것을 막고 싶다면 equals() 메서드를 오버라이딩하고 항상 throw를 던지도록 하자.

### equals를 재정의해야 하는 경우

객체 식별성(물리적으로 두 객체가 같은가)이 아닌 논리적 동치성을 확인해야 하는데 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않은 경우.

보통 값을 표현하는 값 클래스가 여기에 포함된다.

값 클래스이더라도 값이 같은 인스턴스가 둘 이상 만들어지지 않음이 보장되는 인스턴스 통제 클래스라면 equals를 재정의하지 않아도 된다. Enum도 여기에 해당한다.
이런 클래스에서는 객체 식별성과 논리적 동치성이 동일한 의미를 갖기 때문이다.

### equals메서드를 재정의할 때 따라야하는 일반 규약

- 반사성 : null이 아닌 모든 참조값 x에 대해 x.equals(x)는 true다.
- 대칭성 : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면 y.equals(x)도 true다.
- 추이성 : null이 아닌 모든 참조값 x, y, z에 대해 x.equals(y)가 true, y.equals(z)가 true 이면 x.equals(z)도 true 이다
- 일관성 : null이 아닌 모든 참조값 x, y에 대해 x.equals(y)를 반복해서 호출해도 항상 true이거나 항상 false이다.
- null-아님 : null이 아닌 모든 참조값  x에 대해 x.equals(null)은 false다.

위 규약을 어긴다면,

프로그램이 이상하게 동작할 수 있고 원인이 되는 코드를 찾기도 어려워진다.

- **동치관계를 만족시키기 위한 요건 분석**

**반사성**

객체는 자기 자신과 같아야 한다는 뜻.

이 요건이 어겨진 클래스의 인스턴스를 컬렉션에 넣고 contains() 메서드를 호출하면 false가 반환될 것이다.

**대칭성**

두 객체는 서로에 대한 동치 여부에 똑같이 답해야 함을 의미한다.

ex) 대칭성 위배 클래스

```java
public final class CaseInsensitiveString {
	private final String s;
	
	public CaseInsensitiveString(String s){
		this.s = Objects.requireNonNull(s);
	}
	
	@Override
	public boolean equals(Object o) {
		if(o instnace of CaseInsensitiveString){
			return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
		}
		if( o instnace of String){
			return s.equalsIgnoreCase((String o);
		}
		return false;
	}
}
```

위 코드에서 CaseInsensitiveString 인스턴스와 String 인스턴스의 대칭성은 위배된다.

ex)

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";

cis.equals(s); // true
s.equals(cis); // false
```

위와 같이 equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할 지 알 수 없다.

위와 같은 경우의 해결책으로, String 과의 연동을 포기해야한다.

ex) 해결책

```java
	@Override
	public boolean equals(Object o) {
		return o instanceof CaseInsensitiveString
							&& ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
	}
```

**추이성**

첫 번째 객체와 두 번째 객체가 같고 두 번째 객체와 세 번째 객체가 같다면
첫 번째 객체와 세 번째 객체도 같아야함을 의미한다.

x, y 좌표를 비교하는 Point클래스가 있다고 가정하자

```java
public class Point{
	private final int x;
	private final int y;
	
	// 생성자함수

	@Override
	public boolean equals(Object o){
		if(!(o instanceof Point)){
			return false;
		}
		Point p = (Point)o;
		return p.x == x && p.y == y;
	}
}
```

이 클래스를 확장한 클래스가 있다고 가정하자

```java
public class ColorPoint extends Point{
	private final Color color;
	
	public ColorPoint(int x, int y, Color color){
		super(x, y);
		this.color = color;
	}
	...
}
```

equals를 구현한다고 할 때 다음과 같이 구현하는 것은 대칭성에 위배된다.

ex) 대칭성 위배된 ColorPoint의 equals 오버라이딩 코드

```java
@Override 
public boolean equals(Object o){
	if(!(o instanceof ColorPoint)){
		return false;
	}
	return super.equals(o) && ((ColorPoint) o).color == color;
}
```

이유는 Point와 ColorPoint 인스턴스를 비교했을 때

Point의 equals는 color를 무시하기 때문에 true를 반환할 것이고

ColorPoint의 equals는 ColorPoint 인스턴스가 아니기 때문에 false를 반환할 것이다.

다음과 같이 Point일 때와 ColorPoint일 때의 케이스로 나눌 수 있으나 추이성 위배가 발생한다.

ex) 추이성 위배된 ColorPoint의 equals()

```java
@Override 
public boolean equals(Object o){
	if(!(o instanceof Point)){
		return false;
	}
	
	// o가 일반 Point인 경우 색상을 무시하고 비교
	if(!(o instanceof Point)){
		return o.equals(this);
	}

	// o가 ColorPoint인 경우 색상까지 비교
	return super.equals(o) && ((ColorPoint) o).color == color;
}
```

하지만 이는 다음의 경우 추이성에 위배된다.

```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```

p1.equals(p2) 는 true,

p2.equals(p3)는 true 이지만

p1.equals(p3)는 false 이므로 추이성에 위배된다.

또한 이런 방법은 무한재귀에 빠질 위험이 있다.

결론은,

**구체 클래스를 확장하여 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.**

(객체 지향적 추상화의 이점을 포기하지 않는 한..)

이 말은 instanceof 검사 대신 getClass 검사로 바꾸면 구체 클래스를 상속할 수 있는 것 같다.

(instanceof는 구현 클래스도 부모클래스의 instance로 취급,
getClass는 그 클래스에 해당하는 인스턴스만 취급)

ex) Point클래스의 equals 재정의 → 리스코프 치환 원칙 위배

```java
@Override 
public boolean equals(object o){
	if(o == null || o.getClass() != getClass()){
		return false;
	}
	Point p = (Point) o;
		
	return p.x == x && p.y == y;
}
```

위 equals코드는 같은 구현 클래스의 객체와 비교할때만 true를 반환한다.

하지만 위의 경우 Point의 하위 클래스는 정의상 Point이므로 어디서든 Point로써 활용될 수 있어야 하지만 이 방식에서는 그렇지 못하다.

예로,

반지름이 1인 원 안에 점이 속하는지 판별하기 위해 해당하는 점들을 Set에 넣어두고 contains() 로 판별하는 메서드가 있다고 가정하자.

이 메서드의 인수로 Point의 하위클래스인 ColorPoint를 전달했을 경우,
Point클래스의 equals() 메서드의 정의가 위와 같다면 판별메서드는 점 x, y가 원에 포함되었더라도 false를 반환할 것이다.

이유는 Set을 포함하여 대부분의 컬렉션은 contains() 작업을 할 때 equals메서드를 사용하기 때문이다.

구체클래스의 하위 클래스에서 값을 추가할 방법은 없지만 우회할 수 있다.

Point를 상속하는 대신 Point를 ColorPoint의 private필드로 두고,

ColorPoint와 같은 위치의 일반 Point를 반환하는 뷰(view) 메서드를 public으로 추가한다.

ex) equals 규약 지키며 값 추가하기

```java
public class ColorPoint{
	private final Point point;
	private final Color color;
	
	public ColorPoint(int x, int y, Color color){
		point = new Point(x, y);
		this.color = objects.requireNonNull(color);
	}
	
	// ColorPoint의 Point를 반환하는 뷰메서드
	public Point asPoint() {
		return point;
	}
	
	@Override 
	public boolean equals(Object o){
		if(!(o instanceof ColorPoint)){
			return false;
		}
		
		ColorPoint cp = (ColorPoint) o;
		return cp.point.equals(point) && cp.color.equals(color);
	}
}
```

**일관성**

두 객체가 같다면 앞으로도 영원히 같아야 함을 의미한다.

가변객체는 비교 시점에 따라 서로 다를 수도 있지만 불변객체는 한번 다르면 끝까지 달라야 한다.

일관성을 지키기 위해 클래스가 불변이든 가변이단 **equals 판단에 신뢰할 수 없는 자원이 끼어들어선 안된다**.

예를 들어서 java.net.URL의 equqals는 주어진 URL과 매핑된 호스트의 IP 주소를 이용해 비교한다. 만약 호스트 이름을 IP주소로 바꾸려면 네트워크를 통해야 하는데 그 결과가 항상 같다고 보장할 수 없다.

이런 문제를 피하기 위해 equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다.

**null 아님**

모든 객체는 null과 같지 않아야 한다.
equals 메서드에서는 null을 검사하기 보다는 instance 타입을 검사하는 편이 낫다. 어차피 동치성을 검사하기 위한 형변환을 할 때 올바른 타입인지 검사를 해야하기 때문이다.

```java
// 구린 코드
@Override
public boolean equals(Object o){
	if( o == null)
		return false;
	...
}

// 나은 코드
@Override
public boolean equals(Object o){
	if(!(o instanceof MyType)){
		return false;
	}
	MyYupe my = (MyType) o;
}
```

타입확인을 위해 사용하는 instanceof는 피연산자가 null이면 false를 반환하기 때문에 굳이 null체크를 명시적으로 하지않아도 된다.

### 양질의 equals 메서드 구현하기

1. == 연산으로 입력이 자기 자신의 참조인지 확인한다.
   비교 작업이 복잡한 상황일 경우 성능 최적화에 좋다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.

double, float를 제외한 기본타입 필드는==연산자로 비교하고

참조타입 필드는 equals로,

double, float는 정적메서드 Float.compare(…), Double.compare(…)로 비교한다.

성능을 위해서 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하자.

### equals 메서드 구현 시 주의하상

1. equals를 재정의할 땐 hashCode도 반드시 재정의하자
2. 너무 복잡하게 해결하려하지 말자.
3. Object외의 타입을 매개변수로 받는 equals메서드는 선언하지 말자.
   자칫 오버로드된 메서드가 호출될 수 있고 잘못된 정보를 줄 수 있다..
   @Override애너테이션을 일관되게 사용하자.