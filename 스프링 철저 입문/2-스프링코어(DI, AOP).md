# DI

**의존성 주입이 필요하게 된 이유**

애플리케이션에서 하나의 처리를 수행하기 위해 여러 개의 컴포넌트를 조합해서 구현하는 경우는 일반적이다. 필요한 여러 개의 컴포넌트들을 통합할 때 의존성 주입이라는 방식이 효율적임

예를 들어서 회원가입 기능을 생각해보면 회원가입을 위해

- 비밀번호를 암호화해주는 PasswordEncoder 클래스
- 데이터베이스와 연결되어 사용자 정보를 관리해주는 UserRepository 클래스
- 회원가입 기능을 갖고 있는 UserService 클래스

이렇게 3가지의 클래스가 있다고 가정하면

UserService 클래스에서는 회원가입 기능 개발을 위해 PasswordEncoder 클래스와 UserRepository 클래스를 new 키워드를 통해 생성하고 갖고 있어야 한다. 이렇게 생성되어 저장된 구현클래스들은 교체하기가 어렵고 클래스 간의 관계에서 결합도를 높인다.

ex)

```java
public UserServiceImpl(javax.sql.DataSource dataSource){
	this.userReposptory = new JdbcUserRepository(dataSource);
	this.passwordEncoder = new BcryptPasswordEncoder();
}
```

---

이렇게 높아진 결합도를 낮추기 위해

위와 같이 생성자 안에서 구현클래스를 직접 생성하는 대신

생성자 인수로 받아서 할당할 수 있다.

```java
public UserServiceImpl(UserRepository userRepository, PasswordEncoder passwordEncoder){
	this.userReposptory = userRepository;
	this.passwordEncoder = passwordEncoder;
}
```

이렇게하면 UserServiceImpl의 소스코드 안에서 UserRepository와 PasswordEncoder의 구현클래스 정보가 제거되어 UserServiceImpl의 외부에서 UserRepository와 PasswordEncoder의 구현체를  쉽게 변경할 수 있게 된다.

UserServcice를 사용하는 예시는 다음과 같다

```java
//의존되는 class가 UserService 외부로 나와서 클래스간의 결합도가 낮아졌다.
UserRepository userRepository = new JdbcUserRepository(dataSource);
PasswordEncoder passwordEncoder = new BCrypyPasswordEncoder();

UserService userService = new UserServiceImpl(userRepository, passwordEncoder);
```

---

하지만 위의 방식도 각 컴포넌트는 개발자가 직접 생성한 뒤 생성자에 주입을 해주어야 하기 때문에 변경이 발생하는 경우 재작업은 피할 수 없다.

위의 예시처럼 어떤 클래스가 필요로 하는 컴포넌트를 외부에서 생성한 후 내부에서 사용이 가능하도록 만들어주는 과정을 의존성 주입(DI) 이라고 한다.

이러한 의존성 주입을 자동으로 처리하는 기반을 DI 컨테이너라고 한다.

---

스프링 프레임워크가 제공하는 주요 기능이 DI 컨테이너 기능이다.

DI컨테이너에 미래에 어떤 클래스에서 의존하여 쓸 클래스들을 알려주고 의존관계를 정의해주면
UserServiceImpl이 생성될 때 해당 의존성 클래스들이 자동으로 생성되어 주입된다.

UserService를 사용하고 싶은 어플리케이션은 DI 컨테이너에서 UserService를 꺼내오기만 하면 되고 이때 UserRepository와 PasswordEncoder가 UserService에 조합된 상태로 꺼내어진다.

ex) DI 컨테이너에서 UserService 꺼내기

```java
ApplicaitonContext = ...;
UserService userService = context.getBean(UserService.class);
```

DI 컨테이너를 통해 얻는 장점

- 각 컴포넌트의 인스턴스를 생성하고 통합관리하여 컴포넌트 간의 의존성 해결
- 상황에 따라 싱글턴 객체, 또는 프로토타입 객체를 사용할 수 있도록 인스턴스의 스코프(scope)관리를 DI컨테이너가 대신한다.
- AOP 기능도 DI 컨테이너가 대신 해준다.

---

# DI 개요

DI는 의존성 주입이라고도 하며 IoC라고 하는 소프트웨어 디자인 패턴 중 하나이다.

IoC는 인스턴스를 제어하는 주도권이 역전된다는 의미로 사용된다.
인스턴스의 생성과 의존관계의 연결 처리를 해당 소스코드가 아닌 DI 컨터이너에서 대신 해주기 때문에 제어가 역전되었다고 한다.

DI컨테이너를 사용하면 인스턴스를 직접 생성해서 쓰는 방법 대신에
DI컨테이너가 만들어주는 인스턴스를 가져오는 방법을 사용할 수 있다.
이때 취득한 인스턴스가 의존하는 또 다른 인스턴스 역시 DI 컨테이너에서 관리되기 때문에
연쇄적으로 의존성 주입이 발생하여 연관된 클래스의 인스턴스 모두를 사용할 수 있게 된다.

**DI컨테이너에서 인스턴스를 관리하는 방식의 장점**

- 인스턴스 스코프를 제어할 수 있다. (싱글톤 or 프로토타입)
- 인스턴스의 생명주기를 제어할 수 있다.
- AOP방식으로 공통 기능을 집어넣을 수 있다.
- 의존하는 컴포넌트간의 결합도를 낮춘다.

스프링 공식문서에서는 DI컨테이너를 IoC컨테이너라고 칭함

---
# ApplicationContext와 빈 정의

스프링 프레임워크에서는 ApplicationContext가 DI컨테이너의 역할을 한다.

ApplicationContext를 통해 Bean을 찾고(getBean) 받을 수 있다.

아래의 AppConfig클래스는 DI컨테이너에서 설정파일 역할을 한다.
아래와 같이 Java Configuration Class로 설정하는 방식을 자바 기반 설정 방식 이라고 함

```java
@Configuration
public class AppConfig{
	@Bean
	UserRepository userRepository(){
		return new UserRepositoryImpl();
	}
	....
}
```

@Bean 애너테이션을 사용해서 DI 컨테이너에 컴포넌트를 등록하면

등록된 빈을 ApplicationContext인스턴스를 통해 가져올 수 있다.

스프링 프레임워크에서는

DI컨테이너에 등록하는 컴포넌트를 빈 이라고 하고

빈에 대한 설정정보를 빈 정의(Bean Definition)라고 한다.

DI컨테이너에서 빈을 찾아오는 행위는 룩업(lookup) 이라고 한다.

### 빈 설정방법

- 자바 기반 설정 방식
  자바 클래스에 @Configuration 애너테이션을, 메서드에 @Bean 애너테이션을 사용해서 빈을 정의하는 방법
- XML기반 설정 방식
  XML파일을 사용하는 방법. <bean< 요소의 class속성에 FQCN(Fully-Qualified Class Name, 패키지에 클래스명까지 붙여 쓴 클래스 이름)을 기술하면 빈이 정의됨
- 애너테이션 기반 설정방식
  @Component같은 마커 애너테이션(Market Annocation)이 부여된 클래스를 탐색해서(Component Scan) DI컨테이너에 빈을 자동으로 등록하는 방법

ex)

```java
//자바기반 설정방식
ApplicationContext context = new AnnocationConfigApplicationContext(AppConfig.class);

//애너테이션 기반 설정방식. 지정 패키지 이하의 경로에서 컴포넌트 스캔
ApplicationContext context = new AnnocationConfigApplicationContext("com.example.app");

//XML기반 설정 방식. 경로에 접두어가 생략된 경우 클래스패스 안에서 상대경로로 설정파일 탐색
ApplicationContext context = new ClassPathXmlApplicationContext("META-INT/spring/applicationContext.xml");

//XML기반 설정 방식. 경로에 접두어가 생략된 경우 JVM 작업 디렉터리 안에서 상대경로로 설정파일 탐색
ApplicationContext context = new ClassPathXmlApplicationContext("./spring/applicationContext.xml");
```

---

# 빈 설정

### 자바 기반 빈 설정 방식

자바 코드로 빈을 설정한다

```java
,,,

//Configuration 애너테이션으로 설정 클래스임을 선언한다. 설정클래스는 여러개를 정의할 수 있다.
@Configuration
public class AppConfig{

	//메서드에 Bean 애너테이션을 부여해서 빈을 정의한다.
	//메서드 이름이 빈의 이름이 되고 그 빈의 인스턴스가 반환값이 된다.
	//빈 이름을 다르게 명시하고 싶으면 name 속성 사용. ex) @Bean(name="userRepo")
	@Bean
	UserRepository userRepository(){
		return new UserRepsoitoryImpl();
	}
	...

	@Bean
	UserService userService(){
		//다른 컴포넌트를 참조해야할때는 해당 컴포넌트의 메소드를 호출한다.
		return new UserServiceImpl(userRepository());
	}

}
```

아래와 같이 메서드의 매개변수를 추가하는 ㅂ아법으로도 다른 컴포넌트의 의존성을 주입할 수 있다. 당연히 인수로 전달될 인스턴스에 대한 빈은 별도로 정의되어 있어야 한다.

```java
@Bean
UserService userService(UserRepository userRepository)
	return new UserServiceImpl(userService);
}
```

### XML기반 빈 설정 방식

XML 파일을 이용해서 빈을 설정한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--beans요소 안에 빈 정의를 여러개 한다.-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
				 http://www.springframework.org/schema/beans/spring-beans.xsd 
	       http://www.springframework.org/schema/context 
				 http://www.springframework.org/schema/context/spring-context.xsd">

	<!--id값이 빈의 이름이 되고 class가 해당 빈의 구현클래스이다. class는 FQCN으로 작성-->
	<bean id="userRepository" class="com.example.demo.UserRepositoryImpl"/>
	<bean id="userRepository" class="com.example.demo.UserRepositoryImpl">
		<!--생성자ㄹ를 활용한 의존성 주입. ref속성에 주입할 빈의 이름을 기재한다.-->
		<constructor-arg ref="userRepository"/>
	</bean>
</beans>
```

### 애너테이션 기반 빈 설정 방식

빈을 정의하는 애너테이션을 빈의 클래스에 부여하는 방식을 사용
해당 애너테이션이 붙은 클래스를 탐색하여 DI컨테이너에 자동으로 등록한다.
이 탐색과정을 **컴포넌트 스캔(Component Scan)**이라고 한다.

또한 의존성주입도 명시적으로 설정하지 않고
애너테이션이 붙어있으면 DI컨테이너가 자동으로 필요로 하는 의존컴포넌트를 주입하게 한다.
이러한 주입 과정을 오토와이어링(Auto Wiring)이라한다.

ex)

UserRepository.java

```java
...
@Component
public class UserRepositoryImpl implements UserRepository {
	...
}
```

위와 같이 @Component 애너테이션을 붙여서 컴포넌트 스캔이 되도록 한다.

UserServiceImpl.java

```java
@Component
public class UserServiceImpl implements UserService {
	@Autowired
	public UserServiceImpl(UserRepository userRepository){
		..
	}
}
```

생성자에 @Autowired 애너테이션을 부여해서 오토와이어링이 되도록 한다.
오토와이어링을 사용하면 기본적으로 주입 대상과 같은 타입의 빈을 DI 컨테이너에서 찾아온 뒤 와이어링 대상에 주입하게 된다.

컴포넌트 스캔을 수행할 때 스캔할 범위를 지정해야 한다.

설정 방식으로는 자바기반, 또는 XML기반 설정 방식을 사용할 수 있다.

자바 기반 컴포넌트 스캔 범위 설정

```java
@Configuration
@ComponentScan("com.example.demo")
public class AppConfig{
	...
}
```

애너테이션의 value속성이나 basePackages 속성에 컴포넌트를 스캔할 패키지를 지정한다.
속성이 생략될 경우 설정 파일이 위치한 패키지 하위의 모든 패키지를 탐색한다.
값을 지정하면 지정된 패키지 하위 모든패키지에서 스캔 대상을 탐색한다.

XML기반 설정방식

```xml
<beans
	...
	>
	<context:component-scan base-package="com.example.demo"/>
</beans>
```

컴포넌트 스캔 대상 클래스는
클래스 이름에 카멜케이스를 적용한 형태로 DI 컨테이너에 등록된다.

빈의 이름을 명시적으로 정하고 싶다면
애너테이션 안에 이름을 명시해준다.

ex) @Component(”myUserService”)
---
# 의존성 주입

세 가지의 의존성 주입 방법이 존재

- 설정자 기반 의존성 주입 방식(setter-based DI)
- 생성자 기반 의존성 주입 방식(constructor-based DI)
- 필드 기반 의존성 주입 방식(field-based DI);

### 설정자 기반 의존성 주입 방식

설정자 메서드의 인수를 통해 의존성을 주입하는 방식

설정자 메서드가 만들어져 있어야 사용할 수 있다.

ex)

```java
public class UserServiceImpl implements UserService{
	private UserRepository userRepository;
	...
	public void setUserRepository(UserRepository userRepository){
		this.userRepository = userRepository;
	}
	...
}
```

위와 같이 setter메서드로 의존성 주입을 할 수 있다.

- 세터 인젝션을 자바 기반 설정방식으로 표현

```java
@Bean
UserService userService(){
	UserServiceImpl userService = new UserServiceImpl();
	userService.setUserRepository(userRepository());
	return userService;
}
```

UserRepository를 빈으로 등록하는 userRepository() 메소드를 호출하여 세터 인젝션의 매개변수로 전달한다.

함수 매개변수를 사용해서 주입할수도 있다.

```java
@Bean
UserService userService(UserRepository userRepository){
	UserServiceImpl userService = new UserServiceImpl();
	userService.setUserRepository(userRepository);
	return userService;
}

```

자바 기반 설정 방식으로 세터 인젝션을 하는 경우
프로그램에서 인스턴스를 직접 생성하는 코드처럼 보이기 때문에 빈을 정의한 설정인지 체감이 안될수도 있다.

### 생성자 기반 의존성 주입

생성자의 인수를 사용해서 의존성을 주입하는 방식

컨스트럭터 인젝션을 사용하면 필드를 final로 선언해서 생성 후 변경되지 않게 만들 수 있다. 이렇게 필드를 변경하지 못하도록 제한을 거는 것은 오직 컨스트럭터 인젝션에서만 할 수 있다.

### 필드 기반 의존성 주입

DI컨테이너의 힘을 빌려 의존성을 주입하는 방식

필드인젝션을 할 때는 의존성을 주입하고 싶은 필드에 @Autowired 애너테이션을 달아주면 된다. 애너테이션 하나만 달아주면 되기 때문에 간결하다.

```java
@Component
public class UserServiceImpl implements UserService{
	@Autowired 
	UserRepository userRepository;
}
```

필드 인젝션을 사용할 때에는
반드시 DI컨테이너를 사용한다는 것을 전제로 둬야 한다.

DI컨테이너 없이 사용되는 독립형 라이브러리로 사용될 소스코드에서 필드 인젝션을 사용하고 있다면 잘못된 것이다.

---

# 오토와이어링

명시적으로 빈을 정의하지 않고도 DI 컨테이너에 빈을 자동으로 주입하는 방식

오토와이어링에는 두 가지 방식이 있다.

- 타입을 사용한 오토와이어링(autowiring by type)
- 이름을 사용한 오토와이어링(autowiring by name)

### 1. 타입을 사용한 오토와이어링

타입으로 오토와이어링을할 때는 기본적으로 의존성 주입이 반드시 성공한다고 가정한다.
따라서 주입할 타입에 해당하는 빈을 DI컨테이너에서 찾지 못한다면
springframework.beans.factory.NoSuchBeanDefinitionException 예외가 발생한다.

이러한 필수 조건을 완화하고 싶다면 다음과 같이 @Autowired애너테이션의 required 속성을 false로 지정하면 된다. 타입의 빈을 찾지 못하더라도 예외가 발생하지 않고 의존성 주입은 실패했기 때문에 해당 필드에는 null값이 들어간다.

ex)

```java
@Component
public class UserServiceImpl implements UserService{
	@Autowired(required = false)
	UserRepository userRepository;
}
```

스프링 프레임워크 4부터는 필수조건을 완화할 때
required = false를 사용하는 대신에 Optional을 사용할수도 있다.

타입으로 오토와이어링을 할 때 DI 컨테이너에 같은 타입의 빈이 여러 개 발견된다면
그 중 어느것을 사용해야 할지 알 수 없다.

이런 경우에는 NoUniqueBeanDefinitionException 예외가 발생한다.

만약 같은 타입의 빈이 여러 개 정의된 경우에는 @Qualifier 애너테이션을 추가하면서
빈 이름을 지정하면 같은 타입의 빈 중에서 원하는 빈만 선택할 수 있다.

ex)

```java
@Configuration
@ComponentScan
public class AppConfig{
	@Bean
	PasswordEncoder sha256PasswordEncoder(){
		return new Sha256PasswordEncoder();
	}

	@Bean 
	PasswordEncoder bcryptPasswordEncoder(){
		return new BCryptPasswordEncoder();
	}
}
```

위와 같이 PasswordEncoder 를 구현한 클래스가 두 개 있다고 해보자
이 두 클래스는 같은 인터페이스를 구현하고 있기 때문에 @Autowired로는 빈을 구분하지 못한다.

그래서 빈의 이름을 추가로 명시할 필요가 있다.
해당 빈을 주입하는 부분에서 @Qualifier 애너테이션을 추가하고 이름을 명시해주면 된다.

- @Qualifier를 사용해서 빈 이름 명시

```java
@Component
public class UserServiceImpl implements UserService{
	@Autowired
	@Qualifier("sha2256PasswordEncoder")
	PasswordEndoer passwordEncoder;
}
```

위와 같이 이름을 명시하는 방법을 사용할 수도 있고

@Primary 어노테이션을 사용할 수도 있다.
@Primary어노테이션은 우선적으로 선택된 빈을 지정할 수 있다.

- @Primary를 사용해서 기본 빈을 지정

```java
@Configuration
@ComponentScan
public class AppConfig{
	@Bean
	PasswordEncoder sha256PasswordEncoder(){
		return new Sha256PasswordEncoder();
	}

	@Bean 
	@Primary
	PasswordEncoder bcryptPasswordEncoder(){
		return new BCryptPasswordEncoder();
	}
}
```

다만 @Qualifier 로 수식하는 빈의 이름에 구현 클래스의 이름이 포함된다거나 구현과 관련된 정보가 포함되어 있다면 그 빈의 명명방법이 바람직하다고 볼 수는 없다.

왜냐하면 결합도를 낮추기 위해 DI컨테이너를 사용해서 DI를 하는데 빈을 사용할 때 특정 구현체가 사용될 것으로 의식한 이름을 지정해버리면 DI를 사용하는 의미가 없어진다.

이런 경우에는 빈의 이름으로 구현체의 이름을 사용하는 대신 역할이나 사용 목적, 혹은 용도를 이름으로 쓰는 것이 좋다.

앞의 예시에서 PasswordEncoder의 구현체를 나눈 목적이 SHA-256을 가벼운 암호화를 위해서라고 한다면 빈의 이름을 요구사항의 취지와 목적에 맞춰서 “lightweight”라고 지을 수 있다.

ex) SHA-256으로 구현한 PasswordEncoder의 이름을 용도에 맞게 지정한 예시

```java
@Configuration
@ComponentScan
public class AppConfig{
	@Bean(name = "lightweight")
	PasswordEncoder sha256PasswordEncoder(){
		return new Sha256PasswordEncoder();
	}

	@Bean 
	@Primary
	PasswordEncoder bcryptPasswordEncoder(){
		return new BCryptPasswordEncoder();
	}
}
```

```java
@Autowired
@Qualifier("lightweight")
PasswordEncoder passwordEncoder;
```

빈의 역할을 위와 같이 문자열 형태로 나타낼 수도 있고

타입(애너테이션)으로 표현할 수도 있다.

ex) @Lightweight 애너테이션을 직접 만들고 @Qualifier역할을 하도록 만든 예시

- @Lightweight 애너테이션 구현 예

```java
@Traget({ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Qualifier
public @interface Lightweight{
}
```

- @Lightweight 애너테이션을 활용하여빈 정의

```java
@Configuration
@ComponentScan
public class AppConfig{
	@Bean
	@Lightweight
	PasswordEncoder sha256PasswordEncoder(){
		return new Sha256PasswordEncoder();
	}

	@Bean 
	@Primary
	PasswordEncoder bcryptPasswordEncoder(){
		return new BCryptPasswordEncoder();
	}
}
```

- 필드 인젝션에서 활용

```java
@Autowired
@Lightweight
PasswordEncoder passwordEncoder;
```

### 2. 이름으로 오토와이어링 하기

빈의 이름이 필드명이나 프로퍼티 명과 일치할 경우에 빈 이름으로 인젝션이 가능

@Resource 애너테이션을 활용한다.

ex)

```java
@Component
public class UserServiceImpl implements UserSerivce{
	@Resource(name = "sha256PasswordEncoder")
	PasswordEncoder passwordEncoder;
}
```

name속성을 생략할 경우에

필드 인젝션을 하는 경우 필드 이름과 같은 이름의 빈이 선택되고

세터 인젝션을 하는 경우에는 프로퍼티 이름과 같은 이름의 빈이 선택된다.

ex)

- 필드 인젝션을 하는 예(필드 이름과 빈 이름이 일치)

```java
@Component
public class UserServiceImpl implements UserService{
	@Resource
	PasswordEncoder sha256PasswordEncoder;
}
```

- 세터 인젝션을 하는 예(프로퍼티 이름과 일치)

```java
@Component
public class UserServiceImpl implements UserService{
	private PasswordEncoder passwordEncoder;
	
	@Resource
	public void setSha256PasswordEncoder(PasswordEncoder passwordEncoder){
		this.passwordEncoder = passwordEncoder;
	}
}
```

위의 어느 경우에도 해당되지 않으면 타입으로 오토와이어링을 시도한다.

컨스트럭터 인젝션에서는 @Resource 애너테이션을 사용하지 못한다.

### 3. 컬렉션이나 맵 타입으로 오토와이어링

단 하나의 빈만 가져오는 방법 외에도 같은 인터페이스를 구현한 빈을 컬렉션(Collection)이나 맵(Map) 타입에 담아서 가져올 수도 있다.

ex)

- IF인터페이스를 구현한 빈을 여러개 정의한 예

```java
public interface IF<T> {
}
@Component
public class IntIF1 implements IF<Integer>{
}
@Component
public class StringIF1 implements IF<String>{
}
```

- IF 인터페이스를 구현한 빈을 모두 가져오기

```java
// List로 가져오기
@Autowired
List<IF> ifList;

// Map
@Autowired
Map<String, IF> ifMap;
```

List의 경우
IntIF1, StringIF1 빈이 리스트 형태로 주입된다.

Map의 경우

<빈 이름, 빈> 쌍의 형태로 주입된다.
---
컴포넌트 스캔은 클래스 로더(Class Loader)를 스캔하면서 특정 클래스를 찾은 다음 DI컨테이너에 등록하는 방법을 말한다.

### 기본 설정 스캔 대상 애너테이션

- @Component
- @Controller
- @Service
- @Repository
- @Configuration
- @RestController
- @ControllerAdvice
- @ManagedBean
- @Named

스캔을 할 때 클래서로더에서 위와 같은 애너테이션이 붙은 클래스를 찾아야 한다.

만약 스캔 범위가 넓은 경우 탐색 범위가 넓고 처리하는 시간도 오래걸리게 되는데 이 시간은 스프링 프레임워크를 사용한 애플리케이션의 기동 시간을 느리게 만드는 원인이 된다.

```java
// 컴포넌트 스캔 범위가 넓은 경우
@ComponentScan(basePackages = "com.example")

// 적절한 스캔 범위
@ComponentScan(basePackages = "com.example.demo")
```

대표적인 스캔 대상 애너테이션

- @Controller
  MVC패턴에서 컨트롤러 역할을 하는 컴포넌트에 붙이는 애너테이션
  실제 비즈니스 로직은 @Service 컴포넌트에 위임
- @Service
  비즈니스 로직을 처리하는 컴포넌트에 붙이는 애너테이션
  영속적인 데이터 처리는 @Repository 컴포넌트에 위임
- @Repository
  영속적인 데이터 처리를 수행하는 컴포넌트에 붙이는 애너테이션
- @Component
  위 세 경우에 해당하지 않는 컴포넌트(유틸 클래스나 기타 지원 클래스 등)

### 필터를 적용한 컴포넌트 스캔

스캔 대상 외에도 다른 컴포넌트를 더 포함하고 싶은 경우 스캔범위를 커스터마이징 할 수 있다.

스프링프레임워크에서는 다음과 같은 필터를 제공한다.

- 애너테이션을 활용한 필터(ANNOTAION)
- 할당 가능한 타입을 활용한 필터(ASSIGNABLE_TYPE)
- 정규 표현식 패턴을 활용한 필터(REGEX)
- AspectJ 패턴을 활용한 필터(ASPECTJ)

필터를 추가할때는 includeFilters 속성에 나열하면 된다.

ex)

- 할당 가능한 타입으로 필터링

```java
@ComponentScan(basePackeages = "com.example.demo" includeFilters ={
	@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {DomainService.class})
})
```

- 정규표현식 패턴으로 필터링

```java
@ComponentScan(basePackeages = "com.example.demo" includeFilters ={
	@ComponentScan.Filter(type = FilterType.REGEX,
		 pattern = { ".+DomainService$" })
})
```

- 애너테이션을 활용한 필터링

```java
@ComponentScan(basePackeages = "com.example.demo" includeFilters ={
	@ComponentScan.Filter(type = FilterType.ANNOTAION, value = Controller.class)
})
```

기본 스캔 대상을 제외하고 필터로만 컴포넌트를 스캔할수도 있다.

ex)

- 기본 스캔 대상을 제외하고 필터로만 스캔

```java
@ComponentScan(basePackeages = "com.example.demo",
	useDefaultFilters = false, 
	includeFilters ={
		@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {DomainService.class})
})
```

기본 스캔 대상에 필터를 적용해서 특정 컴포넌트를 추가하는 것과는 반대로 특정 컴포넌트를 스캔 대상에서 빼고 싶을 수도 있다. 이 경우에는 excludeFilters 속성을 활용한다.

ex)

- 정규표현식 패턴을 필터로 활용하면서 @Exclude 애너테이션이 붙은 컴포넌트를 걸러내고 싶을 때

```java
@ComponentScan(basePackeages = "com.example.demo",
	useDefaultFilters = false, 
	includeFilters = {
		@ComponentScan.Filter(type = FilterType.REGEX, 
		pattern = { ".+DomainService$" }) },
	excludeFilters = {
		@ComponentScan.Filter(type)
		pattern = { Exclude.class }) }
)
```