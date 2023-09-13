# Chapter 13. 디폴트 메서드
**내용**

- 디폴트 메서드란?
- 진화하는 API의 호환성 유지 방법
- 디폴트 메서드 활용 패턴

인터페이스를 구현하는 클래스는 인터페이스에서 정의한 모든 메서드 구현을 제공하거나,
슈퍼클래스의 구현을 상속받아야 한다.

만약 인터페이스에 새로운 메서드를 추가하려고 한다면 해당 인터페이스를 구현했던 모든 클래스의 구현도 고쳐야한다.

이런 문제룰 해결하기 위해 두 방법이 존재

- 인터페이스 내부에 정적메서드 사용
- 인터페이스 기본 구현을 제공할 수 있도록 디폴트메서드 사용

디폴트 메서드를 이용하면 인터페이스의 기본 구현을 그대로 상속하므로 인터페이스에 자유롭게 새로운 메서드를 추가할 수 있다.

## 13.1 변화하는 API

모양의 크기를 조절하는데에 필요한 메서드를 정의하는 인터페이스가 있다고 가정하자.

얼마 후 몇 가지 기능이 부족하여 해당 인터페이스에 기능을 추가할 경우 어떤 문제가 발생하는지 생각해본다.

### 13.1.1 API 버전 1

아래와 같이 interface와 interface를 구현한 초기 경우를 가정해보자.

모양의 크기를 조절하는 Resizable인터페이스의 초기 버전

```java
public interface Resizable extends Drawable{
	int getWidth();
	int getHeight();
	void setWidth(int width);
	void setHeight(int height);
	void setAbsoluteSize(int width, int height);
}
```

위 인터페이스를 구현한 Ellipse 클래스

```java
public class Ellipse implements Resizable{
	...
}
...
```

### 13.1.2 API 버전 2

얼마 후 Resizable을 구현하는 구현체 개선에 요청을 받는다.

따라서 다음과 같이 Resizable 인터페이스를 고치게 된다.

```java
public interface Resizable extends Drawable{
	int getWidth();
	int getHeight();
	void setWidth(int width);
	void setHeight(int height);
	void setAbsoluteSize(int width, int height);
	void setRelativeSize(int wFactor, int hFactor); //새로 추가된 메서드
}
```

이렇게 Resizable을 고치면 문제가 발생한다.

- Resizable을 구현하는 모든 클래스는 setRelativeSize 메서드를 구현해야 한다.
  인터페이스에 새로운 메서드를 추가하면 **바이너리 호환성**은 유지된다.

  그렇다 하더라도 언젠가 새로 추가된 구현되지 않은 메서드를 호출하는 경우 에러가 발생할 것이다.

- 사용자가 Ellipse를 포함하는 전체 애플리케이션을 재빌드할 때 컴파일 에러가 발생한다.

**바이너리 호환성, 소스 호환성, 동작 호환성**

- 바이너리 호환성 : 새로 추가된 메서드를 호출하지만 않으면 새로운 메서드 구현 없이도 기존 클래스 파일 구현이 잘 동작함.
- 소스 호환성 : 코드를 고쳐도 기존프로그램을 성공적으로 재컴파일 할 수 있음을 의미.
  예로 인터페이스에 메서드를 추가하면 소스호환성이 깨진다.
- 동작 호환성 : 코드를 바꾼 다음에도 같은 입력값이 주어지면 프로그램이 같은 동작을 실행함을 의미.

## 13.2 디폴트 메서드란 무엇인가?

디폴트메서드 : 인터페이스를 구현하는 클래스에서 구현하지 않아도 인터페이스 자체에서 기본으로 제공하는 메서드

default 라는 키워드로 정의할 수 있다.

```java
public interface Sized{
	int size();
	default boolean isEmpty() {
		 return size() == 0;
	}
}
```

Sized인터페이스를 구현하는 모든 클래스는 isEmpty의 구현도 상속받는다.

디폴트메서드를 사용하면 소스 호환성이 유지된다.

**추상 클래스와 인터페이스**

둘 다 추상 메서드와 바디를 포함하는 메서드를 정의할 수 있다.

차이점은,

- 클래스는 하나의 추상 클래스만 상속받을 수 있지만 인터페이스는 여러개 구현할 수 있다.
- 추상클래스는 인스턴스 변수(필드)로 공통 상태를 가질 수 있으나 인터페이스는 인스턴스 변수를 가질 수 없다.

## 13.3 디폴트 메서드 활용 패턴

디폴트 메서드를 이용하는 두 가지 방식 학습

- 선택형 메서드(Optional Method)
- 동작 다중 상속(multiple inheritance of behavior)

### 13.3.1 선택형 메서드

인터페이스에서 정의한 메서드를 클래스에서 구현하지 않고 내용을 비워놓는 경우가 있다. (Iterator를 구현하는 클래스에서 remove메서드에 빈 구현을 제공하는 예)

디폴트메서드를 이용하면 remove같은 메서드에 기본 구현을 제공할 수 있으므로 빈 구현을 제공할 필요가 없다.

ex) Java8 Iterator 인터페이스의 remove() 메서드

```java
interface Iterator<T> { 
	boolean hasNext();
	T next();
	default void remove(){
		throw new UnsupportedOperationException();
	}
}
```

이렇게 하면 기본 구현이 제공되기 때문에 구현 클래스에서 빈 remove메서드를 구현할 필요가 없다.

만약 remove()메서드를 구현해야 한다면?

default method의 구현이 필요할 땐 추상 method와 똑같이 Override하면 된다.

### 13.3.2 동작 다중 상속

디폴트메서드를 이용하여 동작 다중 상속 기능을 구현할 수 있다.

**다중 상속 형식**

ArrayList는 여러개의 인터페이스를 구현한다. ArrayList는 이 인터페이스들의 서브형식이 된다.
디폴트메서드를 사용하지 않아도 다중 상속을 활용할 수 있다.

자바8에서는 인터페이스가 구현을 포함할 수 있으므로 여러 인터페이스에서 동작을 상속받을 수 있다.

**기능이 중복되지 않는 최소의 인터페이스**

각자의 기능을 갖고 다른 인터페이스와 중복되지 않는 인터페이스를 구현할 수 있다.

**인터페이스 조합**

필요한 기능이 포함된 여러개의 인터페이스들을 구현하여 다양한 클래스를 구현할 수 있다.

모든 추상메서드들의 구현은 제공해야 하지만,
디폴트메서드의 구현은 제공할 필요가 없다.

인터페이스에 디폴트 구현을 포함시킨 것의 또 다른 장점으로
구현의 개선이 필요할 때 디폴트메서드만 수정하면 인터페이스를 구현하는 모든 클래스에서 개선된 메서드를 자동으로 상속받는다.

**옳지 못한 상속**

상속으로 코드 재사용 문제를 모두 해결할 수 있는 것은 아니다.

메서드 1개를 사용하기 위해 100개의 메서드와 필드가 정의되어 있는 클래스를 상속받는 것은 별로다.

이럴땐 델리게이션, 멤버 변수를 이용해서 클래스에서 필요한 메서드를 직접 호출하는 메서드를 작성하는 것이 좋다.

## 13.4 해석 규칙

같은 디폴트 메서드 시그니처를 포함하는 두 인터페이스를 구현하는 상황이라면?

여러 인터페이스를 구현하는 경우 같은 시그니처를 갖는 디폴트메서드를 상속받는 상황이 생길 수 있다.

### 13.4.1 알아야 할 세 가지 해결 규칙

다른 클래스나 인터페이스로부터 같은 시그니처를 갖는 메서드를 상속받을 때 다음 규칙을 따라야 한다.

1. 클래스가 항상 이긴다.클래스나 슈퍼클래스에서 정의한 메서드가 디폴트메서드보다 우선권을 갖는다.
2. 1번 규칙 이외의 상황에서는 서브인터페이스가 이긴다. B가 A를 상속받는다면 B가 A를 이긴다.
3. 이외의 경우라면 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트메서드를 오버라이드하고 호출해야 한다.

### 13.4.2 디폴트 메서드를 제공하는 서브인터페이스가 이긴다.

아래와 같이 Interface B가 A를 상속받고 class A가 B, C를 구현하는 경우

서브인터페이스 B의 디폴트메서드가 우선시된다.

- interface A

```java
public interface A {
	default void hello(){
		sout("hello A");
	}
}
```

- interface B

```java
public interface B extends A{
	default void hello(){
		sout("Hello B");
	}
}
```

- class C

```java
public class C implements B, A {
	public static void main(String... args){
		new C().hello(); // hello B
	}
}		
```

여기서 다음과 같이 interface A를 구현하는 class D가 추가되고 C가 D를 상속받는다면?

- class D 추가

```java
public class D implements A{}
public class C extends D implements B, A{
	public static void main(String... args){
		new	C().hello();
	}
}
```

클래스 D에서 hello() 메서드를 오버라이드하지 않았기 때문에 인터페이스 A의 디폴트 메서드 구현을 상속받는다.

따라서 이 경우에도 서브인터페이스인 B의 hello()가 실행된다.

만약 D에서 hello() 메서드를 구현하는 경우 슈퍼클래스인 D의 hello메서드 정의가 우선권을 가지게 된다.

- class D 에 hello 메서드 구현

```java
public class D implements A{
	void hello(){
			sout("hello D");
	}
}
public class C extends D implements B, A{
	public static void main(String... args){
		new	C().hello(); // hello D
	}
}
```

### 13.4.3 충돌 그리고 명시적인 문제 해결

아래와 같이 두 interface가 독립적인 경우

```java
public interface A {
	default void hello(){
		sout("hello A");
	}
}

public interface B {
	default void hello(){
		sout("hello B");
	}
}

public class C implements B, A {}
```

위와 같은 경우는 자바 컴파일러가 어떤 인터페이스의 메서드를 호출해야 할지 알 수 없으므로 에러가 발생한다.

**충돌 해결**

이 경우에는 어떤 인터페이스의 디폴트 메서드를 사용할 것인지 명시적으로 선택해야 한다.

```java
public class C implements B, A {
	void hello(){
		B.super.hello(); // hello B
	}
}
```

메서드 시그니처가 완전히 같지 않고 거의 비슷한 경우에도 위와 같은 컴파일 에러가 발생한다.

```java
public interface A {
	default Number getNumber(){
		return 10;
	}
}

public interface B {
	default Integer getNumber(){
		return 40;
	}
}

public class C implements B, A {
	public static void main(String... args){
		sout(new C().getNumber());
	}
}
```

### 13.4.4 다이아몬드 문제

아래와 같은 상속관계의 다이어그램은 다이아몬드를 닮았다고 해서 다이아몬드 문제라 부름

```java
public interface A{
	default void hello(){
		sout("hello A");
	}
}

public interface B extends A{}
public interface C extends A{}

public class D implements B, C {
	public static void main(String ... args) {
		new D().hello(); // hello A
	}
}
```

위의 경우 hello라는 메서드 정의가 A에만 존재하기 때문에 hello A가 출력된다.

- B에 같은 시그니처의 디폴트메서드 hello를 정의하면?
  서브 인터페이스가 먼저 선택되므로 B의 hello가 실행된다.
- A, B에 모두 같은 시그니처의 디폴트메서드 hello를 정의하면?
  충돌이 발생하므로 명시적 호출이 필요하다.
- C에 디폴트메서드가 아닌 추상메서드 hello를 추가하면?

```java
public interface C extends A{
	void hello();
}
```

C의 추상메서드 hello가 A의 디폴트메서드보다 우선권을 가지므로 D는 hello 메서드를 구현해야한다.

## 13.5 마치며

- 자바 8 인터페이스는 구현 코드를 포함하는 디폴트메서드, 정적 메서드를 정의할 수 있다.
- 디폴트메서드 덕에 API가 변경되어도 기존 버전과 호환성을 유지할 수 있다.
- 디폴트메서드를 상속하면서 생기는 충돌 문제를 해결하는 규칙이 있다.
1. 클래스나 슈퍼클래스에 정의된 메서드는 디폴트메서드보다 우선시된다.
2. 1번 규칙 이외의 상황에서는 서브인터페이스가 슈퍼인터페이스보다 우선시된다.
3. 이외 우선순위가 결정되지 않은 상황이라면 명시적으로 디폴트메서드를 오버라이딩하고 호출해야한다.

p.431쪽

인터페이스 C에 추상메서드 hello(디폴트메서드가 아닌)를 추가했을 때

> C는 A를 상속받으므로 C의 추상 메서드 hello가 A의 디폴트 메서드 hello보다 우선권을 갖는다. 따라서 컴파일 에러가 발생하며 클래스 D가 어떤 hello를 사용할지 명시적으로 선택해서 에러를 해결해야 한다.
>

⇒ 어떤 hello를 사용할지 명시하는 것이 아니라 우선권을 가지는 c의 추상메서드 hello()를 구현해서 에러를 해결해야 하는 것이 아닌가?