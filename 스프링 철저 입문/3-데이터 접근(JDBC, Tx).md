# 3.3 트랜잭션 관리
애너테이션을 활용해서 트랜잭션을 관리하는 선언적(declarative)방법 설명.

소스코드 안에 직접 commit이나 rollback메서드로 트랜잭션을 관리하는 프로그램적인 방법 설명

트랜잭션의 경계와 동작 방식을 결정 짓는 트랜잭션 격리 수준과 전파방식 학습.

## 1. 트랜잭션 관리자

관계형 DBMS의 데이터에 접근할 때는 트랜잭션의 경계가 어디까지이고 어떻게 관리되는지를 이해해야 한다. 하지만 트랜잭션을 염두에 두고 코드를 구현하는 것은 어렵다. 스프링 프레임워크에서는 이런 트랜잭션 처리를 비교적 쉽게 구현하도록 도와주는 기능이 있다. 트랜잭션 관리를 위한 코드를 비즈니스 로직에서 분리하기 위한 구조나 다른 트랜잭션을 투명하게 처리할 수 있게 하는 API등을 들 수 있다.

스프링 트랜잭션 처리의 중심이 되는 인터페이스는 PlatformTransactionManager이다. 트랜잭션 처리에 필요한 API를 제공한다.

스프링 프레임워크는 다양한 환경과 제품에 대응하는 PlatformTranascationManager의 구현 클래스를 제공한다.

- DataSourceTransactionManager : JDBC 및 마이베티스 등의 JDBC 기반 라이브러리로 데이터베이스에 접근하는 경우에 이용한다.
- HibernateTransactionManager : 하이버네이트를 이용해 데이터베이스에 접근하는 경우에 이용한다.
- JpaTransactionManager : JPA로 데이터베이스에 접근하는 경우에 이용한다.
- JtaTransactionManager : JTA에서 트랜잭션을 관리하는 경우에 이용한다.
- WebLogicJtaTransactionManager : 애플리케이션 서버인 웹 로직(WebLogic)의 JTA에서 트랜잭션을 관리하는 경우에 이용한다.
- WebSphereUowTransctionManager : 애플리케이션 서버인 웹스피어(WebSphere)의 JTA에서 트랜잭션을 관리하는 경우에 이용한다.

### 트랜잭션 관리자 정의

스프링 프레임워크의 트랜잭션관리자를 사용할 때는 다음의 두 가지 작업을 해야한다.

1. PlatformTransactionManager의 빈을 정의한다.
2. 트랜잭션을 관리해야 하는 메서드를 저정의한다.

### 로컬 트랜잭션을 이용하는 경우

**기본적인 설정 방법**

로컬 트랜잭션을 사용하는 경우 JDBC API를 호출하고 트랜잭션 제어를 수행하는 DataSourceTransactionManager를 사용한다. 로컬 트랜잭션은 단일 데이터 저장소에 대한 트랜잭션으로 일반적으로 자주 사용되는 트랜잭션이다.

단일 뎅터 저장소에 대한 여러 조작을 하나의 논리적 단위로 처리하고 싶을 때 사용한다. 이 경우에는 PlatformTransactionManager의 구현 클래스로 DataSourceTransactionManager를 사용한다. 이 때 PlatformTransactionManager의 빈 ID는 ‘transactionManager’로 사용하는 것이 좋다. 왜냐하면 스프링 프레임워크에서 기본적으로 트랜잭션 관리자의 빈 ID를 ‘transactionManager’로 가정하고 있기 때문이다.

XML 기반 설정 방식

```xml
<bean id="transactionManager"

	// PlatformTransactionManager로 DataSourceTransactionManager 구현 클래스를 지정한다
	class="org.springframework.jdbc.datasource.DatasurceTransactionManager">

	// dataSource 프러퍼티에 설정 완료된 데이터소스의 빈을 지정한다.
	<property name="dataSource" ref="dataSource" />
</bean>

// 애너테이션 트랜잭션 제어 활성화를 위해 추가한다 (@Transactional)
<tx:annotation-driven />
```

Java 기반 설정 방식

```java
@Bean(name = "transactionManager")
	public DataSourceTransactionManager dataSourceTransactionManager(DataSource dataSource) {
		return new DataSourceTransactionManager(dataSource);
	}
```

**’transactionManager’가 아닌 다른 빈 ID를 사용할 경우**

다른 빈 ID를 사용할 경우 <tx:annotation-driven>요소의 transaction-manager 속성에도 같은 ID를 설정해야 한다.

```xml
<bean id="txManager"
	class="org.springframework.jdbc.datasource.DatasurceTransactionManager">
	<property name="dataSource" ref="dataSource" />
</bean>

// 빈 id를 지정해준다.
<tx:annotation-driven transaction-manager="txManager"/>
```

### 글로벌 트랜잭션을 이용하는 경우

글로벌 트랜잭션은 여러 데이터 저장소에 걸쳐서 적용되는 트랜잭션이다. 각 데이터베이스에서 각각의 조작을 수행하고 그 조작들을 하나의 트랜잭션으로 묶어 모두 성공하거나 모두 실패한 것으로 처리해야 하는 경우에는 로컬 트랜잭션으로는 불가능하다.

글로벌 트랜잭션은 JTA(Java Transaction API)라는 Java EE사양으로 표준화되어있고 애플리케이션 서버가 JTA의 구현 클래스를 제공한다.

JTA를 스프링 프레임워크에서 사용하려면 PlatformTransactionManager의 구현 클래스로 JtaTransactionManager를 사용하면 된다.

```xml
<tx:jta-transaction-manager />
```

## 2. 선언적 트랜잭션

선언적 트랜잭션은 미리 선언된 룰에 따라 트랜잭션을 제어하는 방법이다. 선언적 트랜잭션으로 정해진 룰을 준수함으로써 트랜잭션의 시작과 커밋, 롤백 등의 일반적인 처리를 비즈니스 ㄹ직 안에 기술할 필요가 없다.

### @Transactional을 이용한 선언적 트랜잭션

@Transactional 애너테이션을 빈의 public 메서드에 추가하는 것으로 대상 메서드의 시작 종료에 맞춰 트랜잭션을 시작, 커밋할 수 있다. 스프링 프레임워크의 기본 상태에서는 메서드 안의 처리에서 데이터 접근 예외와 같은 비검사 예외(unchecked exception)가 발생해서 메서드 안에 처리가 중단될 때 트랜잭션이 자동으로 롤백된다.

### 트랜잭션 제어에 필요한 정보

- value
  여러 트랜잭션 관리자를 이용하는 경우 이용하는 트랜잭션 관리자의 qualifier를 지정한다. 기본 트랜잭션 관리자를 이용하는 경우에는 생략할 수 있다.
- transactionManager
  value의 별칭 (spring framework 4.2 버전부터 추가)
- propagration
  트랜잭션의 전파 방식을 지정한다.
- isolation
  트랜잭션의 격리 수준을 지저앟ㄴ다.
- timeout
  트랜잭션의 제한 시간을 지정한다. 기본값은 -1
- readOnly
  트랜잭션의 읽기 전용 플래그를 지정한다. 기본값은 false
- rollbackFor
  이 속성에 지정한 예외가 발생하면 트랜잭션을 롤백시킨다. 예외 클래스명을 여러 개 나열할 수 있으며 ,로 구분한다. 따로 지정해주지 않으면 비검사예외가 발생할 때 트랜잭션이 롤백된다.
- rollbackForClassName
  여기에 지정한 예외가 발생하면 트랜잭션을 롤백시킨다. 예외 이름을 여러개 나열가능
- noRollbackFor
  여기에 지정한 예외가 발생하더라도 트랜잭션을 롤백하지 않음. 예외 클래스명 여러개 나열 가능
- noRollbackForClassName
  여기에 지정한 예외가 발생하더라도 트랜잭션을 롤백하지 않음. 예외 이름을 여러개 나열 가능

### 기본 사용법

**트랜잭션 관리 대상에 해당하는 메서드 구현**

@Transactional 애너테이션은 클래스와 메서드에 부여할 수 있다. 차이는 애너테이션이 적용되는 범위이다.

트랜잭션 적용 우선순위 : 메서드 레벨 > 클래스 레벨

## 3. 명시적 트랜잭션

명시적 트랜잭션은 커밋이나 롤백과 같은 트랜잭션 처리를 코드에 직접 명시적으로 기술하는 방법이다. 메서드 단위보다도 더 작은 단위로 트랜잭션을 제어하고 싶거나 선언적 트랜잭션으로는 표현하기 어려운 섬세한 트랜잭션 제어가 필요할 때 이 방법을 사용한다.

명시적 트랜잭션을 이용하는 방법으로 PlatformTransactionManager와 TransactionTemplate을 사용하는 두 가지 방법을 제공한다.

### PlatformTransactionManager를 이용한 명시적 트랜잭션 제어

TransactionDefinition 및 TransactionStatus를 이용해서 트랜잭션의 시작고 ㅏ커밋, 그리고 롤백을 명시적으로 처리하게 된다.

ex)

```java
@Service
public class RoolServiceImpl implements RoomService {

	// PlatformTransactionManager 의존성 주입
	@Autowired
	PlatformTransactionManager txManager;

	...

	@Override
	public void insertRoom(Room room){
		// 이 객체를 이용해서 트랜잭션을 설정한다.
		DefaultTransactionDefinition def=  new DefaultTransactionDefinition();

		// 트랜잭션에 이름을 설정한다.
		def.setName("InsertRoomWithEqupmentTx)

		// 읽기, 쓰기 속성, 전파 방식 등을 설정한다.
		def.setReadOnly(false);
		def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

		/* DefaultTransactionDefinition 객체를 인수로 TransactionManager의 getTransaction
		메서드를 실행한다. 이 이후의 처리가 트랜잭션 범위가 된다. 
		getTransaction 메서드의 반환값 TransactionStatus를 이용해서 트랜잭션 커밋이나 롤백을 한다.*/
		Transaction status = tx.Manager.getTransaction(def);

		try {
			... 트랜잭션에 포함할 로직들
		} catch(Exception e){
			// 트랜잭션을 롤백한다.
			txManager.rollback(status);
		}
		// 트랜잭션을 커밋한다.
		txManager.commit(status);
```

PlatformTransactionManager 빈 정의는 특별한 점 없다.

### TransactionTemplate을 활용한 명시적 트랜잭션 제어

PlatformTransactionManager보다 더 구조적으로 트랜잭션 제어를 할 수 있다.

ex)

```java
@Service
public class RoolServiceImpl implements RoomService {

	// TransactionTamplete 의존성 주입
	@Autowired
	TransactionTamplete transactionTamplete;

	...
	
	@Overried
	public void insertRoom(final Room room){
		/* 갱신 처리와 같이 반환값을 필로 하지 않느 경우에는 TransactionTemplate의 execute메서드를 실행한다.
		execute메서드 인수로 TransactionCallbeckWithoutResult 객체를 지정한다
		조회와 같이 반환값이 있는 메서드라면 TransactionCallback 객체를 인수로 하고 결과를 반환값으로 돌려준다.
		*/
		transactionTemplate.execute(new TransactionCallbackWithoutResult() {
			// doInTransaction.. 메서드를 구현하고 하나의 트랜잭션을 이 메서드 안에 넣는다.
			@Overried
			protected void doInTransactionWithoutResult(TransactionStatus status){
				...
			}
		}

		// doInTransaction.. 메서드 종료 시 트랜잭션이 자동으로 커밋 또는 롤백된다.
		/* 예외를 던지지 않고 트랜잭션을 롤백하고 싶다면 doInTrans... 메서드 인수로 전달되는
		TransactionStatus 객체의 setRollbackOnly메서드를 실행한다.
		*/
```

**빈 정의**

```java
@Configuration
public class AppConfig{

	@Bean
	public TransactionTemplate transactionTemplate(
									PlatformTransactionmanager transactionManager) {
	TransactionTemplate transactioTemplate = new TransactionTemplate(transactionManager);
	
	transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
	transactionTamplete.setTimeout(30);

	return transactiontemplate;
	}

```

## 4. 트랜잭션 격리 수준과 전파 방식

### 트랜잭션 격리 수준

@Transaction 애너테이션 사용 시 isolation 속성으로 지정,
TransactionDefinition, TransactionTemplate 사용시 setIsolationLevel 메서드로 지정

- DEFAULT
  사용하는 데이터베이스의 기본 격리 수준을 이용한다.
- READ_UNCOMMITTED
  더티 리드(Dirty Read), 반복되지 않은 읽기(Unrepeatable Read), 팬텀 읽기(Phantom Read)가 발생한다. 커밋되지 않은 변경 데이터를 다른 트랜잭션에서 참조하는 것을 허용한다. 즉 변경데이터가 롤백된 경우는 무효한 데이터를 조회하게 된다.
- READ_COMMITTED
  더티리드를 방지하지만 반복되지 않은 읽기, 팬텀 읽기는 발생한다.
  커밋되지 않은 변경 데이터를 다른 트랜잭션에서 참조하는 것을 금지한다.
- REPEATABLE_READ
  더티리드, 반복되지 않은 읽기를 방지하미나 팬텀 읽기는 발생한다.
- SERIALIZABLE
  더티 리드, 반복되지 않은 읽기, 팬텀 읽기를 방지한다.

‘

### 트랜잭션 전파 방식

트랜잭션 경계에서 트랜잭션에 참여하는 방법을 결정하는 것.
새로운 트랜잭션을 시작, 이미 시작된 트랜잭션에 참여 등 선택지가 준비되어 있다.

**스프링 프레임워크에서 이용 가능한 트랜잭션 전파 방식**

기본값은 REQUIRED이다.

@Transactional의 propagation 속성,
TransactionalDefinition과 TransactionTemplate의 setPropagationBehavior 메서드에서 지정할 수 있다.

트랜잭션 전파 방식

- REQUIRED
  이미 만들어진 트랜잭션이 존재한다면 해당 트랜잭션 관리 범위 안에 함께 들어간다. 이미 만들어진 트랜잭션이 존재하지 않는다면 새로운 트랜잭션을 만든다.
- REQUIRES_NEW
  이밎 만들어진 트랜잭션 범위안에 들어가지 않고 반드시 새로운 트랜잭션을 만든다. 이미 만들어진 트랜잭션이 아직 종료되지 않았다면 새로운 트랜잭션은 보류상태가 되어 이전 트랜잭션이 끝나는 것을 기다려야 한다.
- MANDATORY
  이미 만들어진 트랜잭션 범위 안에 들어가야 한다. 만약 기존에 만들어진 트랜잭션이 없다면 예외가 발생한다.
- NEVER
  트랜잭션 관리를 하지 않는다. 만약 이미 만들어진 트랜잭션이 있다면 예외가 발생한다.
- NOT_SUPPORTED
  트랜잭션을 관리하지 않는다. 만약 이미 만들어진 트랜잭션이 있다면 이전 트랜잭션이 끝나는 것을 기다려야 한다.
- SUPPORTS
  이미 만들어진 트랜잭션이 있다면 그 범위 안에 들어가고, 만약 트랜잭션이 없다면 트랜잭션 관리를 하지 않는다.
- NESTED
  REQUIRED와 마찬가지로 현재 트랜잭션이 존재하지 않으면 새로운 트랜잭션을 만들고 이미 존재하는 경우에는 이미 만들어진 것을 계속 이용한다.

  하지만 NESTED가 적용된 구간은 중첩된 트랜잭션처럼 취급한다. NESTED구간에서 롤백이 발생한 경우 NESTED구간 안의 처리내용은 모두 롤백되지만 NESTED구간 밖에서 실행된 처리된 내용은 롤백되지 않는다.

  단 부모 트랜잭션에서 롤백되면 NESTED구간의 트랜잭션은 모두 롤백된다.


**트랜잭션 전파 방식을 의식할 필요가 있는 예**

처리내용 로그를 데이터베이스에 저장해야 하는 예를 들어보자. 만약 비즈니스 로직이 실패했을 경우 업무 데이터는 롤백, 로그 데이터는 커밋을 하고 싶을 때 트랜잭션 경계가 겹치게 된다.

만약 기본 전파 방식 REQUIRED를 사용한다면 로그 데이터도 업무 데이터와 함께 롤백될 수 있다. 이러한 경우는 로그 전파방식을 REQUIRES_NEW로 변경하면 된다.