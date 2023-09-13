# Chapter 14. 자바 모듈 시스템
**내용**

- 자바의 모듈 시스템 지원 시작
- 주요 구조 : 모듈 declarations, requires, exports 지시어
- 기존 자바 아카이브(JAR)에 적용되는 자동 모듈
- 모듈화와 JDK라이브러리
- 모듈과 메이븐 빌드
- 기본적인 requires, exports 외의 모듈 지시어 간단 요약

자바 모듈 시스템의 사용처와 사용했을 때의 이점, 그리고 어떻게 사용할 수 있는지 학습

## 14.1 압력 : 소프트웨어 유추

추론하기 쉬운 소프트웨어를 만드는 데 도움을 주는 관심사 분리, 정보 은닉을 살펴본다.

### 14.1.1 관심사분리

관심사분리(SoC, Sepraration of concerns) : 컴퓨터 프로그램을고유의 기능으로 나누는 동작을 권장하는 원칙.

**관심사분리 원칙의 장점**

- 개별기능을 따로 작업할 수 있으므로 팀이 쉽게 협업할 수 있다.
- 개별 부분을 재사용하기 쉽다.
- 전체 시스템을 쉽게 유지보수할 수 있다.

### 14.1.2 정보 은닉

정보 은닉 : 세부 구현을 숨기도록 장려하는 원칙. 세부 구현을 숨김으로 프로그램의 어떤 부분이 바뀌었을 때 다른 부분까지 영향을 미칠 가능성을 줄일 수 있다.

캡슐화 : 특정 코드 조각이 애플리케이션의 다른 부분과 고립되어 있음을 의미. 캡슐화된 코드의 내부적인 변화가 의도치 않게 외부에 영향을 미칠 가능성이 줄어든다.

잘 설계된 소프트웨어를 위해서 관심사분리와 정보은닉을 따르는 것은 필수적이다.

## 14.2 자바 모듈 시스템을 설계한 이유

### 14.2.1 모듈화의 한계

자바 9 이전까지는 모듈화된 소프트웨어 프로젝트를 만드는 데에 한계가 있었다.

자바는 클래스, 패키지, JAR 세 가지 수준의 코드 그룹화를 제공한다.

클래스와 관련하여 자바는 접근 제한자와 캡슐화를 지원했으나
패키지와 JAR수준에서는 캡슐화를 거의 지원하지 않았다.

**제한된 가시성**

자바에선 한 패키지의 클래스와 인터페이스를 다른 패키지로 공개하려면 public으로 선언해야 한다.

결과적으로 public으로 선언한 클래스와 인터페이스는 특정 패키지가 아닌 모든 곳에 공개가 된다.

내부적으로 사용할 목적으로 만든 구현을 다른 프로그래머가 임시적으로 사용해서 정착해버릴 수 있으므로 기존의 애플리케이션을 망가뜨리지 않고 라이브러리 코드를 바꾸기가 어려워진다.

또한 보안적으로 코드를 임의로 조작하는 위협에 더 많이 노출될 수 있다.

**클래스 경로**

자바에선 클래스를 모두 컴파일하여 한 개의 JAR파일에 넣고 클래스 경로에 이 JAR파일을 추가해서 사용할 수 있다.

여기서의 약점은,

1. 클래스 경로에는 같은 클래스를 구분하는 버전 개념이 없기 때문에 두 가지 버전의 같은 라이브러리가 존재한다면 어떤 일이 발생할 지 예측할 수 없다.
2. 클래스 경로는 명시적인의존성을 지원하지 않는다. 각각의 JAR안에 있는 모든 클래스는 classes라는 집합으로 합쳐진다. 즉 한 JAR가 다른 JAR에 포함된 클래스 집합을 사용하라고 명시적으로 의존성을 정의하는 기능을 제공하지 않는다. 이 상황에서는 클래스 경로 때문에 어떤 일이 일어나는지 파악하기 어렵다. (빠진게 있는지, 충돌이 있는지)

   메이븐같은 빌드 도구는 이런 문제를 해결하는데에 도움을 준다.


### 14.2.2 거대한 JDK

자바 개발 킷(JDK) : 자바 프로그램을 만들고 실행하는 데 도움을 주는 도구의 집합.
자바 프로그램을 컴파일하는 javac, 자바 애플리케이션을 로드하고 실행하는 java, 런타임을 지원을 제공하는 JDK라이브러리, 컬렉션 스트림 등 포함

JDK가 커지면서 사용에 관계 없이 JDK에 포함되는 경우도 생겼다.(ex, CORBA)

이런 문제를 해결하기 위해 **컴팩트 프로파일** 기법이 제시됨.

관련 분야에 따라 세 가지 프로파일로 나뉘어 각각 다른 메모리 풋프린트(프로그램 실행 중 사용하거나 참조하는 메모리 총량)를 제공했다.

자바의 낮은 캡슐화지원때문에 JDK 라이브러리의 많은 내부 API가 공개되어서 여러 라이브러리에서 참조하게 되었고 결과적으로 호환성을 깨지 않고는 API를 바꾸기 어려워졌다.

이런 문제들때문에 JDK 자체를 모듈화할 수 있는 자바 모듈 시스템 설계의 필요성이 제기되었다.

JDK의 필요한 부분만 골라 사용하고
클래스 경로를 쉽게 유추할 수 있으며
플랫폼 진화를 위한 강력한 캡슐화를 제공할 수 있는 구조의 필요성

### 14.2.3 OSGi와 비교

p.439.  스킵

## 14.2 자바 모듈 : 큰 그림

자바8는 모듈이라는 자바 프로그램 구조 단위를 제공한다.
모듈은 module이라는 새 키워드에 이름과 바디를 추가해서 정의한다.
**모듈 디스크립터**는 module-info.java라는 파일에 저장된다.

모듈 디스크립터는

module 모듈명,
exports 패키지명,  // 한 패키지를 노출시킨다.
requires 모듈명 // 필요한 모듈을정의

으로 구성되어 있다.

## 14.4 자바 모듈 시스템으로 애플리케이션 개발하기

모듈화 애플리케이션을 구조화하고 패키지하고 실행하는 방법 학습

### 14.4.1 애플리케이션 셋업

특정 기능을 수행하는 애플리케이션을 모듈화한다고 해볼 때 다음과 같은 기능(관심사)들이 필요하다.

- 다양한 소스에서 데이터 읽기(HttpReader, FileReader …)
- 다양한 포맷으로 구성된 데이터를 파싱(Parser, JSONParser …)
- …

문제를 해결하는데에 필요한 각 기능를 세부적으로 나누어 그룹화할 수 있다.

- expenses.readers
- expenses.readers.http
- expenses.readers.file
- expenses.parsers
- …

### 14.4.2 세부적인 모듈화와 거친 모듈화

세부적인 모듈화는 14.4.1절처럼 모든 패키지가 자신의 모듈을 갖는다.
거친 모듈화는 한 모듈이 시스템의 모든 패키지를 포함한다.

세부적인 모듈화는 설계 비용이 증가하는 문제,
거친 모듈화는 모듈화의 장점을 읽는다는 문제가 있다.

실용적으로 분해하여 이해하기 쉽고 고치기 쉬운 수준으로 적절하게 모듈화하는 것이 중요

### 14.4.3 자바 모듈 시스템 기초

모듈화 애플리케이션은 module-info.java파일을 루트에 포함하고 있다.

```java
// 이름만이 정의된 비어있는 모듈 디스크립터
module expenses.application {
}
```

모듈 디스크립터는 모듈의 의존성, 그리고 어떤 기능을 외부로 노출할지를 정의한다.

**모듈화 애플리케이션 JAR패키징 및 실행**

```java
// 자바 애플리케이션을 JAR로 패키징하는 방법
javac module-info.java 
	com/example/expenses/application/ExpensesApplication.java -d target

jar cvfe expenses-application.jar
	com.example.expenses.application.ExpensesApplication -C target

// 생성된 JAR를 모듈화 애플리케이션으로 실행하는 방법
java --module-path expenses-application.jar |
	--module expenses|com.example.expenses.application.ExpensesApplication
```

실행 옵션

- —module-path : 어떤 모듈을 로드할 수 있는지 지정한다.
- —module : 실행할 메인 모듈과 클래스를 지정한다.

## 14.5 여러 모듈 활용하기

새로운 기능을 캡슐화한 expense.reader라는 새 모듈을 추가하고
expenses.application 모듈에서 expenses.readers모듈을 필요로 한다고 할 때
두 모듈이 상호작용하는 방법을 학습한다.

### 14.5.1 exports 구문

- expenses.readers의 모듈 선언

```java
module expenses.readers{
	// exports + 패키지명 
	exports com.example.expenses.readers;
	exports com.example.expenses.readers.file;
	exports com.example.expenses.readers.http;
}
```

export 구문 : exports는 다른 모듈에서 사용할 수 있도록 특정 패키지를 공개 형식으로 만든다.

기본적으로 모듈 내의 모든 것은 캡슐화된다. 모듈시스템은 화이트리스트 기법으로 다른 모듈에서 사용할 수 있는 기능이 무엇인지 명시적으로 결정해야 한다.

### 14.5.2 requires 구문

```java
module expenses.readers{
	// requires + 모듈명
	requires java.base;

	// exports + 패키지명 
	exports com.example.expenses.readers;
	exports com.example.expenses.readers.file;
	exports com.example.expenses.readers.http;
}
```

requires 구문 : requires는 의존하고 있는 모듈을 지정한다.

위 예제는 net, io, util 등의 자바 메인 패키지를 포함하는 java.base에 의존하고 있음을 나타내는데 java.base는 항상 필요한 기본 모듈이므로 명시적으로 정의하지 않아도 된다.

java.base외의 모듈을 임포트할 땐 requires를 사용한다.

자바9에서 requires와 exports 구문을 이용하여 정교하게 클래스 접근을 제어할 수 있다.

### 14.5.3 이름 정하기

오라클은 모듈의 이름을 패키지명처럼 인터넷 도메인명을 역순으로 나타내는 규칙(ex, com.iteratrlearning.training) 으로 짓기를 권장한다.

그리고 모듈명은 노출된 주요 API 패키지와 이름이 같아야 한다는 규칙도 따라야 한다.

## 14.6 컴파일과 패키징

모듈 애플리케이션을 메이븐 동의 빌드 도구를 이용해서 프로젝트를 컴파일 할 수 있다.

각 모듈에 oom.xml을 추가해야한다.
모듈은 독립적으로 컴파일되므로 자체적으로 각각이 한 개의 프로젝트이다.
전체 프로젝트 빌드를 조정할 수 있도록 모든 모듈의 부모 모듈에도 pom.xml을 추가한다.

- 전체 구조

```java
--pom.xml
--expenses.application
	--pom.xml
	--src
		--main
			--java
				--module-info.java
				--com
					--example
						--expenses
							--application
								--ExpensesApplication.java
--expenses.readers
	--pom.xml // 각 모듈에 추가
	--src
		--main
			--java
				--module-info.java
				--com
					--example
						--....
```

pom.xml을 각 모듈에 추가해야하고
모듈 디스크립터는 src/main/java 디렉터리에 위치해야한다.

올바른 모듈 소스 경로를 이용하도록 메이븐이 javac를 설정한다.

- expenses.readers의 pom.xml

```xml
<?xml version="1.0"...
<project ...>
	
	<groupId>com.example</groupdId>
	<artifactId>expenses.readers</artifactId>
	<version>1.0</version>
	<packaging>jar</packaging>

	<parent>
		<groupId>com.example</groupId>
		<artifactId>expenses</artifactId>
		<version>1.0</version>
	</parent>

</project>
```

순조롭게 빌드될 수 있도록 명시적으로 부모 모듈을 지정했다.
(부모 모듈을 지정하면 빌드가 순조로워지나??)

- expenses.application 모듈의 pom.xml

```xml
<?xml version="1.0"...
<project ...>
	
	<groupId>com.example</groupdId>
	<artifactId>expenses.application</artifactId>
	<version>1.0</version>
	<packaging>jar</packaging>

	<parent>
		<groupId>com.example</groupId>
		<artifactId>expenses</artifactId>
		<version>1.0</version>
	</parent>

	<dependencies>
		<dependency>
			<groupId>com.example</groupId>
			<artifactId>expenses.readers</artifactId>
			<version>1.0</version>
		</dependency>
	</dependencies>

</project>
```

ExpenseApplication이 필요로 하는 클래스와 인터페이스가 있는 expenses.readers를 의존성으로 추가해야 한다.

- 빌드 과정을 가이드할 전역 pom.xml

```xml
<?xml version="1.0"...
<project ...>
	
	<groupId>com.example</groupdId>
	<artifactId>expenses</artifactId>
	<version>1.0</version>
	<packaging>pom</packaging>

	<modules>
		<module>expenses.application</module>
		<module>expenses.readers</module>
	</modules>

	<build>
		<pluginManagement>
			<plugins>
				<plugin>
						<groupId>org.apache.maven.plugins</groupId>
						....

</project>
```

두 개의 자식 모듈 expenses.application, expenses.readers를 참조하도록 한 설정이다.

이제 mvn clean pakage 명령으로 프로젝트의 모듈을 JAR로 만들 수 있따.
다음과 같은 부산물이 만들어진다.

```xml
./expenses.application/target/expenses.application-1.0.jar
./expenses.readers/target/expenses.readers-1.0.jar
```

두 JAR를 모듈 경로에 포함하여 모듈 애플리케이션을 실행할 수 있다.

```xml
java --module-path \
./expenses.application/target/expenses.application-1.0.jar:\
./expenses.readers/target/expenses.readers-1.0.jar\
--module \
expenses.application/com.example.expenses.application.ExpensesApplication
```

모듈을 만들고 requires로 참조하는 방법을 학습했다.

java.base이외의 외부 모듈과 라이브러리를 참조해야 할 때는 ?

그리고 기존 라이브러리가 명시적으로 module-info.java를 사용하도록 업데이트되지 않았다면 어떤 일이 일어나는가?

## 14.7 자동 모듈

HttpReader를 구현하는 대신 아파치 프로젝트의 httpclient 라이브러리를 사용한다고 가정하보자.
이런 라이브러리의 추가는 어떻게 하는가?

requires 구문으로 expenses.reader의 module-info.java에 구문을 추가하고 mvn clean package를 실행해보자.

다음과 같이 모듈을 찾을 수 없다는 에러가 발생한다.

```java
module not fount: httpclient
```

의존성을 기술하도록 pom.xml도 갱신해야 한다.

메이븐 컴파일러 플러그인은 module-info.java를 포함하는 프로젝트를 빌드할 때 모든 의존성 모듈을 경로에 놓아 적절한 JAR를 내려받고 이들이 프로젝트에 인식되도록 한다.

다음과 같은 의존성이 필요하다.

```xml
<dependencies>
	<dependency>
		<groupId>...
		<artifactId>httpclient</artifactId>
		<version>
	...
```

이제 프로젝트가 정상적으로 빌드된다.

httpclient는 자바 모듈로 사용하려는 외부 라이브러리인데 모듈화가 되어있지 않은 라이브러리이다.
자바는 JAR를 자동 모듈이라는 형태로 적절하게 변환한다.

모듈 경로상에 있으나 module-info파일을 가지지 않은 모든 JAR는 **자동 모듈**이 된다.

자동 모듈은 암묵적으로 자신의 모든 패키지를 노출시킨다.

jar도구의 —describe-module인수를 이용해서 자동으로정해지는 이름을 바꿀 수 있다.

- httpclient라는 이름으로 정의

```xml
jar -file=./expenses.readers/target/dependency/htttpclient-4.5.3.jar \
--describe-module httpclient@4.5.3 automatic
```

- httpclient JAR모듈 경로에 추가 후 애플리케이션 실행

```xml
java --module-path \
./expenses.application/target/expenses.application-1.0.jar:\
./expenses.readers/target/expenses.readers-1.0.jar\
./expenses.readers/target/dependency/httpclient-4.5.3.jar \
--module \
expenses.application/com.example.expenses.application.ExpensesApplication
```

## 14.8 모듈 정의와 구문들

module지시어를 사용하여 모듈을 정의할 때 넣을 수 있는 구문들

### 14.8.1 requires

requires : 컴파일타임과 런타임에 한 모듈이 다른 모듈에 의존함을 정의한다.

```xml
module com.iteratrlearning.application {
	requires com.iteratrlearning.ui;
}
```

requires에 정의된 모듈에서 외부로 노출한 공개형식을 위 모듈에서 사용할 수 있다.

### 14.8.2 exports

exports : 지정한 패키지를 다른 모듈에서 이용할 수 있도록 공개 형식으로 만든다.
아무 패키지도 공개하지 않는 것이 기본 설정.

export는 패키지명을 인수로 받고 requires는 모듈명을 인수로 받는다.

```xml
module com.iteratrlearning.ui{
	requires com.iteratrlearning.core;
	exports com.iteratrlearning.ui.panels;
	exports com.iteratrlearning.ui.widgets;
}
```

requires에 정의된 모듈에서 외부로 노출한 공개형식을 위 모듈에서 사용할 수 있다.

### 14.8.3 requires transitive

다른 모듈이 제공하는 공개 형식을 한 모듈에서 사용할 수 있다고 지정할 수 있다.

```xml
module com.iteratrlearning.ui{
	requires transitive com.iteratrlearning.core;
	exports com.iteratrlearning.ui.panels;
	exports com.iteratrlearning.ui.widgets;
}

module com.iteratrlearning.application {
	requires com.iteratrlearning.ui;
}
```

결과적으로 com.iteratrlearning.application 모듈은 com.iteratrlearning.core 모듈에서 노출한 공개형식에 접근할 수 있다.

(전이성을 가진다. application 모듈에서 core모듈이 필요한 경우 application모듈에서 또 선언해줄 필요가 없어진다)

### 14.8.4 exports to

exports to : 사용자에게 공개할 기능을 제한함으로 가시성을 좀 더 정교하게 제어할 수 있다.

```xml
module com.iteratrlearning.ui{
	requires com.iteratrlearning.core;

	exports com.iteratrlearning.ui.panels;
	exports com.iteratrlearning.ui.widgets to com.iteratrlearning.ui.widgetuser;
}
```

com.iteratrlearning.ui.widgets의 접근 권한을 가진 사용자의 권한을
com.iteratrlearning.ui.widgetuser로 제한할 수 있다.

### 14.8.5 open과 opens

모듈 선언에 open한정자를 이용하면 모든 패키지를 다른 모듈에 리플렉션을 허용할 수 있다.

opens는 특정 패키지에 대해서만 리플렉션을 허용하고 싶은 경우 사용한다.

```xml
open module com.iteratrlearning.ui{

}

module com.iteratrlearning.ui{
	opens exports com.iteratrlearning.ui.panels;
}
```

자바 9 이전에는 리플렉션으로 객체의 private 상태를 확인할 수 있었기 때문에 진정한 캡슐화는 존재하지 않았다.

자바 9는 기본적으로 리플렉션을 허용하지 않고 필요하다면 open구문을 명시해야 한다.

### 14.8.6 uses와 provides

provides 구문으로 서비스 제공자를,  uses구문으로 서비스 소비자를 지정할 수 있다.

## 14.9 더 큰 예제 그리고 더 배울 수 있는 방법

The Java Module System이라는 책을 봐라.

## 14.10 마치며

- 관심사분리, 정보은닉은 추론하기 쉬운 소프트웨어를 만드는 중요한 두가지 원칙
- 자바 9 이전에는 패키지, 클래스, 인터페이스로 모듈화를 구현했으나 효과적인 캡슐화를 달성하기엔 부족했다.
- 자바 9에선 새로운 모듈 시스템을 재공한다.
- 자동 모듈은 암묵적으로 모든 패키지를 공개한다.
- 메이븐은 자바 9 모듈 시스템으로 구조화된 애플리케이션을 지원한다.

https://iyoungman.github.io/java/Java9-Module/
