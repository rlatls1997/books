# Chapter 12. 새로운 날짜와 시간 API
**내용**

- 자바 8의 새로운 날짜, 시간 라이브러리
- 날짜 조작, 포맷팅, 파싱,
- 시간대, 캘린더 다루기

자바 1.0 에선 [java.util.Date](http://java.util.Date) 클래스로 날짜, 시간 관련 기능을 제공했으나 유연성이 떨어졌음

자바 1.1 에서 java.util.Calendar클래스로 대체하려했으나 month가 0부터 시작하고 쉽게 에러를 일으키는 등 문제가 많았음. 클래스가 늘어나면서 혼란만 가중됨

언어의 종류와 독립적으로 날짜 및 시간의 형식을 조절하고 파싱할 때 사용하는 DateFormat은 스레드에 안전하지 않다. 두 스레드가 동시에 하나의 포매터로 날짜를 파싱할 때 문제가 생길 수 있음

Date, Calenar는 가변클래스라서 유지보수가 어렵다.

자바 8엔 부실한 날짜, 시간 라이브러리 문제를 해결하기 위해 새 기능이 java.time에 추가됨.

## 12.1 LocalDate, LocalTime, Instant, Duration, Period 클래스

### 12.1.1 LocalDate와 Localtime 사용

LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변객체.
어떠한 시간대 정보도 포함하지 않는다.

- LocalDate 만들고 값 읽기

```java
LocalDate date = LocalDate.of(2017,9,21); //2017-09-21
LocalDate today = LocalDate.now() // 현재 날짜 정보

int year = date.getYear(); //2017;
Month month = date.getMonth(); //SEPTEMBER, enum 클래스
int day = date.getDayOfMonth(); //21
DayOfWeek dow = date.getDayOfWeek(); //THURSDAY, enum
int len = date.lengthOfMonth(); // 31(9월의 일 수)
boolean leap = date.isLeapYear(); //false(윤년이 아님)
```

- LocalDate get() 메서드

get() 메서드에 TemporalField를 전달해서 정보를 얻는 방법도 있음.

TemporalField는 시간 관련 객체에서 어떤 필드의 값에 접근할지 정의하는 인터페이스.
ChronoField는 TemporalField를 구현한 enum클래스

```java
int year = date.get(ChronoField.YEAR);
int month = date.get(ChronoField.MONTH_OF_YEAR);
int day = date.get(ChronoField.DAY_OF_MONTH);
```

getYear(), getMonthValue(), getDayOfMonth() 내장 메서드 등을 이용하여 가독성을 높일 수 있음.

LocalTime은 13:42:20 같은 시간을 표현하는 클래스

- LocaTime 만들고 값 읽기

```java
LocalTime time = LocalTime.of(13, 45, 20); //13:45:20
int hour = time.getHour();
int minute = time.getMinute();
int seconde = time.getSeconde();
```

parse() 메서드를 활용하야 날짜, 시간 문자열로도 LocalDate, LocalTime인스턴스를 만들 수 있음.

```java
LocalDate date - LocalDate.parse("2017-09-21");
LocalTime time = LocalTime.parse("13:45:20");
```

parse() 메서드에 DateTimeFormatter를 전달할 수도 있음.

DateTimeFormatter는 java.util.DateFormat클래스를 대체하는 클래스

파싱할 수 없을 땐 DateTimeParseException 발생

### 12.1.2 날짜와 시간 조합

LocalDateTime은 LocalDate, LocalTime을 쌍으로 갖는 복합 클래스.
날짜, 시간 모두 표현 가능하다.

- LocalDateTime 생성 및 날짜, 시간 조합 방법

```java
//2017-09-21T13:45:20
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(13, 45, 20); //LocalDate에 time정보를 제공하여 LocalDateTime 생성
LocalDateTime dt4 = date.atTime(time); 
LocalDateTime dt5 = time.atDate(date); //LocalTime에 date정보를 제공하여 LocalDateTime 생성
```

LocalDateTime에서 LocalDate, LocalTime 인스턴스를 추출할수도 있음

```java
LocalDate date = dt.toLocalDate();
LocalTime time = dt.toLocalTime();
```

### 12.1.3 Instant 클래스 : 기계의 날짜와 시간

java.time.Instant

Instant 클래스 : 유닉스 에포크 시간(Unix epoch time)인 1970년 1월 1일 0시 0분 0초 UTC를 기준으로 특정 지점까지의 시간을 초로 표현. 기계적인 관점에서의 시간 표현 방법

팩토리 메서드 ofEpochSecond에 초를 넘겨줘서 Instant클래스 인스턴스를 만들 수 있음.

Instant클래스는 나노초(10억분의 1초)의 정밀도 제공.

오버로드된 인자가 2개인 ofEpochSecond() 메스드의 두번째 인수를 이용하여 나노초 단위로 시간을 보정할 수 있음. 두 번째 인수로 전달된 나노초단위의 수를 합산하여 시간 계산

ex)

```java
Instant.ofEpochSecond(4); // 1970-01-01T00:00:04Z
Instant.ofEpochSecond(3, 1_000_000_000); // 1970-01-01T00:00:04Z
Instant.ofEpochSecond(3, 1_000_000_001); // 1970-01-01T00:00:04.000000001Zㅑ
```

Instant클래스도 LocalDate 등의 클래스와 마찬가지로 사람이 확인할 수 있도록 시간을 표시해주는 now() 정적 팩토리 메서드를 제공한다.

하지만 Instant는 기계 특화 유틸리티이기 때문에 초와 나노초 정보를 포함하며 사람이 읽을 수 있는 시간 정보를 제공하지 않는다.

따라서 다음과 같이 날짜 정보 등을 읽으려고 하면 예외가 발생한다.

```java
int day = Instant.now().get(ChronoField.DAY_OF_MONTH);
// java.time.temporal.UnsupportedTemporalTypeException : ...
```

Instant에서는 Duration과 Period클래스를 함께 활용할 수 있다.

### 12.1.4 Duration과 Period 정의

Temporal인터페이스는 특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의한다.

Duration클래스는 두 시간 객체 사이의 초, 나노초와 관련된 클래스.
한 쌍의 LocalTime, LocalDateTime, 또는 한 쌍의 Instant로 Duration을 생성 가능

```java
Duration d1 = Duration.between(time1, time2);
```

Period클래스는 LocalDate객체간의 날짜와 관련된 클래스

```java
Period d1 = Period.between(date1, date2);
```

두 시간객체를 사용하지 않고도 인스턴스 생성 가능

```java
Duration d1 = Duration.ofMinutes(3);
Duration d2 = Duration.of(3, ChronoUnit.MINUTES);

Period p1 = Period.ofDays(10);
Period p2 = Period.ofWeeks(3);
Period p3 = Period.of(2, 6, 1);
```

**간격을 표현하는 날짜와 시간 클래스 Duration, Period의 공통 메서드**

- 정적메서드

between : 두 시간 사이의 간격을 생성

from : 시간 단위로 간격 생성

of : 주어진 구성 요소에서 간격 인스턴스를 생성

parse : 문자열을 파싱해서 간격 인스턴스 생성

- 정적메서드X

addTo : 현재값의 복사본을 생성한 다음에 지정된 Temporal객체에 추가

get : 현재 간격 정보값을 읽음

isNegative : 간격이 음수인지 확인

isZero : 간격이 0인지 확인

minus : 현재값에서 주어진 시간을 뺀 복사본 생성

multipliedBy : 현재값에서 주어진 값을 곱한 복사본 생성

negated : 주어진 값의 부호를 반전한 복사본을 생성

plus : 현재값에 주어진 시간을 더한 복사본을 생성

subtractFrom : 지정된 Temporal 객체에서 간격을 뺌

불변객체는 스레드 안전성과 도메인 모델의 일관성을 유지하는데에 좋음.

하지만 기존 객체 인스턴스에서 변경이 필요한 경우도 필요할 수 있음 ⇒ 12.2절

## 12.2 날짜 조정, 파싱, 포매팅

withAttribute메서드로 기존의 LocalDate를 바꾼 버전을 간단하게 만들 수 있음.

모든 메서드는 새로운 객체를 반환하며 기존 객체를 변경하지 않음

```java
LocalDate date = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date1 = date.withYear(2011) // 2011-09-21
LocalDate date2 = date1.withDayOfMonth(25) // 2011-09-25
LocalDate date3 = date2.with(ChronoField.MONTH_OF_YEAR, 2); // 2011-02-25
```

상대적인 방식으로 LocalDate속성 바꾸기

```java
LocalDate date = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date1 = date.plusWeeks(1) // 2017-09-28
LocalDate date2 = date1.minusYears(6) // 2011-09-28
LocalDate date3 = date2.plus(6, ChronoUnit.MONTHS); // 2012-03-28
```

메서드 인수에 숫자와 TemporalUnit 활용 가능

ChronoUnit 열겨형은 TemporalUnit 인터페이스를 쉽게 활용할 수 있는 구현 제공

LocalDate, localTime, LocalDateTime, Instant 등 날짜와 시간을 표현하는 모든 클래스는 위와 같이(plus, with, …) 서로 비슷한 메서드 제공

### 12.2.1 TemporalAdjusters 사용하기

TemporalAdjuster는 날짜 조정에 대해 더 다양한 동작을 수행할 수 있도록 하는 기능을 제공.

오버로드된 with메서드에 전달하여 사용.

```java
import static java.time.temporal.TemporalAdjusters.*;
LocalDate date1 = LocalDate.of(2014, 3, 18);
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY)); //2014-03-23
LocalDate date3 = date2.with(lastDayOfMonth()); //2014-03-31
```

**TemporalAdjusters 클래스의 팩토리 메서드**

현재 날짜 이후 지정한 요일이 가장 처음 나타나는 날짜, 다음달의 첫 번째 날짜 등을 반환하는 TemporalAdjuster를 반환하는 팩토리 메서드들

https://docs.oracle.com/javase/8/docs/api/java/time/temporal/TemporalAdjusters.html

필요한 기능이 미리 정의되어 있지 않은 경우 커스텀 TemporalAdujuster를 구현할 수 있다.

### 12.2.2 날짜와 시간 객체 출력과 파싱

날짜와 시간 관련 작업에서 포맷팅, 파싱을 위해 java.time.format 패키지가 추가되었다.

이 패키지의 DateTimeFormatter클래스의 정적 팩토리 메서드, 상수를 이용하여 쉽게 포매터를 만들 수 있다.

DateTimeFormatter를 이용해서 날짜나 시간을 특정 형식의 문자열로 만들 수 있다.

ex)

```java
LocalDate date = LocalDate.of(2014, 3, 19);
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20140318;
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); // 2014-03-18;
```

parse() 메서드를 이용하면 문자열을 날짜 객체로 만들 수 있다.

```java
LocalDate date 1 = LocalDate.parse("20140318", DateTimeFormatter.BASIC_ISO_DATE)
LocalDate date 1 = LocalDate.parse("2014-03-18", DateTimeFormatter.ISO_LOCAL_DATE)
```

기존 java.util.DateFormat클래스와 달리 DateTimeFormatter는 스레드 안전하게 사용할 수 있다.

또한 특정 패턴으로 포매터를 만들 수 있음

ex)

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate date = LocalDate.of(2014, 3, 18);
String formattedDate = date1.format(formatter); // 18/03/2014
LocalDate date2 = LocalDate.parse(formattedDate, formatter);
```

ofPattern 메서드는 Locale로 포매터를 만들 수 있는 오버로드된 메서드도 제공한다.

```java
DateTimeFormatter italFormatter 
	= DateTimeFormatter.ofPattern("d. MMMM yyyy", Locale.ITALIAN);
```

DateTimeFormatterBuilder 클래스로 복합적인 포매터를 정의해서 세부적으로 포매터를 제어할 수 있다.

ex)

```java
DateTimeFormatter italianFormatter = new DateTimeFormatterBuilder()
	.appendText(ChronoField.DAY_OF_MONTH)
	.appendLiteral(". ")
	.appendText(ChronoField.MONTH_OF_YEAR)
	.appendLiteral(" ")
	.appendText(ChronoField.YEAR)
	.parseCaseInsensitive()
	.toFormatter(Locale.ITALIAN);
```

## 12.3 다양한 시간대와 캘린더 활용 방법

기존의 java.util.TimeZone을 대체할 수 있는 java.time.ZoneId클래스가 등장

ZoneId클래스를 이용하면 서머타임(DST, Daylight Saving Time)같은 복잡한 사항이 자동으로 처리된다.

ZoneId 또한 불변클래스이다.

### 12.3.1 시간대 사용하기

표준 시간이 같은 지역을 묶어서 **시간대(time zone)** 규칙 집합을 정의한다.

ZoneId의 getRules()를 이용해서 해당 시간대의 규정을 획득할 수 있다.

다음처럼 지역 ID로 특정 ZoneId를 구분한다.

```java
ZoneId = romeZone = ZoneId.of("Europe/Rome");
```

지역 Id는 “{지역}/{도시}” 형식이며 IANA Time Zone Datebase에서 제공하는 지역 집합 정보를 사용.

다음처럼 toZoneId 메서드로 기존 TimeZone을 이용해서도 ZoneId를 얻을 수 있다.

```java
ZoneId zoneId = TimeZone.getDefault().toZoneId();
```

ZoneId객체를 얻은 다음에는 LocalDate, LocalDateTime, Instant를 이용해서 ZonedDateTime 인스턴스로 변환할 수 있다.

ZonedDateTime은 지정한 시간대에 상대적인 시점을 표시한다.

ex)

```java
ZoneId romeZone = ZoneId.of("Europe/Rome");

LocalDate date = LocalDate.of(2014, Month.MARCH, 18);
ZonedDateTime zdt1 = date.atStartOfDay(romeZone); //2014-03-18T00:00+01:00[Europe/Rome]

LocalDateTime dateTime = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
ZonedDateTime zdt2 = dateTime.atZone(romeZone); //2014-03-18T13:45+01:00[Europe/Rome]

Instant instant = Instant.now();
ZonedDateTime zdt3 = instant.atZone(romeZone); //2022-06-30T17:23:56.107+02:00[Europe/Rome]
```

### 12.3.2 UTC/Greenwich 기준의 고정 오프셋

UTC(협정 세계시)/GMT(그리니치 표준시)를 기준으로 시간대 표현하기.

ZoneId의 서브클래스인 ZoneOffset클래스로 런던의 그리니치 0도 자오선과 시간값의 차이를 표현할 수 있음.

ex) 5시간 느린 뉴욕

```java
ZoneOffset newYorkOffset = ZoneOffset.of("-05:00");
```

하지만 ZoneOffset으로는 서머타임을 제대로 처리할 수 없어서 권장되지 않는다.

ZoneOffset은 ZoneId이므로 ZoneId를 사용하여 날짜, 시간 객체를 ZonedDateTime 인스턴스로 변환하는 것처럼 사용할 수 있다.

또는 UTC/GMT와 오프셋으로 날짜와 시간을 표현하는 OffsetDateTime을 만드는 방법도 존재.

```java
ZoneOffset newYorkOffset = ZoneOffset.of("-05:00");
LocalDateTime dateTime = LocalDateTime.of(2014, Monthl.MARCH, 18, 13, 45);

OffsetDateTime daetTimeInNewYork = OffsetDateTime.of(date, newYorkOffset);
```

### 12.3.3 대안 캘린더 시스템 사용하기

ISO-8601 캘린더 시스템은 전세계에 통용됨.

자바8에서는 추가로 4개의 캘래스가 캘린더 시스템을 제공
(ThaiBuddhistDate, JapaneseDate, MinguoDate, HijrahDate)

위 4개의 클래스와 LocalDate 클래스는 ChronoLocalDate 인터페이스를 구현

ChronoLocalDate 인터페이스 : 임의의 연대기에서 특정 날짜를 표현할 수 있는 기능을 제공

LocalDate를 이용하여 위 4개 클래스의 인스턴스 생성 가능

```java
LocalDate date = ...
JapaneseDate japaneseDate = JapaneseDate.from(date);
```

특정 Locale과 Locale에 대한 날짜 인스턴스로 캘린더 시스템을 만들 수 있음

Chronology는 캘린더 시스템을 의미하며 ofLocal을 이용해서 인스턴스 획득 가능

```java
ChronologyjapaneseChronology = Chronology.ofLocale(Locale.JAPAN);
ChronoLocalDate now = japaneseChronology.dateNow();
```

## 12.4 마치며

- 새로운 날짜와 시간 API에서 날짜와 시간 객체는 모두 불변 객체
- TemporalAdjuster를 이용하여 날짜 관련 복잡한 동작을 수행할 수 있고 동작을 커스텀화할 수 있다.
- 날짜와 시간 객체를 특정 포맷으로 출력하고 파싱하는 포매터를 정의할 수 있다.
