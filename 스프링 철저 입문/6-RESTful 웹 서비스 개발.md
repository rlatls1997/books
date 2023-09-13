# 4. 리소스 클래스 구현
## 6.4.2 Jackson을 이용한 포맷 제어

Jackson을 이용해 JSON 포맷을 제어하는 방법

다양한 애너테이션으로

### 스프링이 제공하는 Jackson 지원 클래스

Jackson은 com.fasterxml.jackson.databind.ObjectMapper 클래스를 사용해서 JSON과 자바 객체를 서로 변환할 수 있다.

ObjectMapper에는 기본 독장 방식을 커스터마이징하기 위한 옵션들이 있으며 스프링이 제공하는 지원클래스를 이용하면 ObjectMapper를 직접 다루는 것보다 간단하게 옵션 지정을 할 수 있다.

**스프링이 제공하는 헬퍼클래스(helper class)**

- org.springframework.http.converter.json.Jackson2ObjectMapperBuilder
- org.springframework.http.converter.json.Jackson2ObjectFactoryBean

### JSON에 들여쓰기를 설정하는 방법

ObjectMapper의 기본 동작 방식은 JSON에 들여쓰기나 개행을 포함하지 않기 때문에 기본 설정으로 만들어진 JSON 정보는 읽기 어려움

- Java Based 설정을 이용한 ObjectMapper 빈 정의 예

```java
@Bean
ObjectMapper objectMapper(){
	return Jackson2ObjectMapperBuilder.json()
					.indentOutput(true)
					.build();
```

### Date and Time API 클래스를 지원하는 방법

Java SE 8에서 추가된 Date and Time API 클래스를 지원하려면 Jackson에서 제공하는 라이브러리 jackson-datatype-jsr310 을 추가해야한다.

pom.xml

```xml
<dependency>
	<groupId>com.fasterxml.jackson.datatype</groupId>
	<artifactId>jackson-datatype-jsr310</artifactId>
</dependency>?
```

jackson-datatype-jsr310 의존 라이브러리를 추가하면 Date and Time API 클래스를 다룰 수 있다. 하지만 포맷을 지정하지 않으면 결과가 예상대로 나오지 않는다.

```json
...
	"date" : [2015, 3, 19]
...
```

날짜/시간 타입의 포맷지정은?

ObjectMapper 에서 지정해보자

- java based 방식의 ObjectMapper 빈 정의

```java
@Bean
ObjectMapper objectMapper() {
	return Jackson2ObjectMapperBuilder.json()
				.indentOutput(true)
				.dateFormat(new StdDateFormat())
				.build();
}
```

format 적용 이후에는 각 클래스에 따라 다음 결과가 나온다.

- LocalDate : yyyy-MM-dd
- LocalDateTime : yyyy-MM-dd’T’HH:mm:ss.SSS
- ZonedDateTime : yyyy-MM-dd’T’HH:mm:ss.SSS’Z’ (ex : .....+9:00)
- Localtime : HH:mm:ss.SSS