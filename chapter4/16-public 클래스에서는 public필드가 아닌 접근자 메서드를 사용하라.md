## 16. public 클래스에서는 public필드가 아닌 접근자 메서드를 사용하라

ex) 인스턴스 필드를 모아놓는 일 외에는 목적이 없는 클래스

```java
class Point{
	public double x;
	public double y;
}
```

위와 같은 클래스의 경우 데이터 필드에 직접 접근가능하므로 캡슐화의 이점을 제공하지 못한다.

API수정 없이는 내부 표현을 바꿀 수 없고,
불변식을 보장할 수 없으며,
외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.

캡슐화하고 멤버에 접근할 수 있는 public 접근자를 추가한다

ex)

```java
class Point{
	private double x;
	private double y;

	public Point(double x, double y){
		this.x = x;
		this.y = y;
	}
	
	public double getX() {
		return x;
	}
	
	public double getY(){
		return y;
	}
	
	public void setX(double x){
		this.x = x;
	}
	
	public void setY(double y){
		this.y = y;
	}
}
```

패키지 바깥에서 접근할 수 있는 클래스의 경우, 위와같이 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.

반대로 public 클래스가 필드를 공개할 경우 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현 방식을 마음대로 바꿀 수 없게 된다.

하지만 package-private 클래스 또는 private 중첩 클래스라면 데이터 필드를 노출한다고 해도 문제가 없다. 그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다.

이 방식은 클래스 선언 면에서나 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다.

만약 필드가 불변이라면 public으로 노출해도 괜찮은가?
public 클래스의 필드가 불변이라면 직접 노출할 때의 단점은 조금 줄어들겠지만 여전히 좋지 않다.

API를 변경하지 않고는 표현방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다. 단 불변식은 보장할 수 있다.

ex) 불변 필드를 노출한 public 클래스

```java
public final class Time{
	private staitc final int HOURS_PER_DAY = 24;
	private static final int MINUTES_PER_HOUR = 60;
	
	public final int hour;
	public final int minute;
	
	public Time(int hour, int minute){
	...
	}
}
```

**결론**

public클래스는 절대 가변 필드를 직접 노출해서는 안된다.
불변 필드라고 하더라도 필드를 직접 노출하는 것은 좋지 않다.

다만 package-private클래스나 private중첩 클래스에서는 종종 필드를 노출하는 편이 나을 때도 있다.