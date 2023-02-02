# 새로운 날짜와 시간 API
> 자바 1.0에서는 java.util.Date 클래스 하나로 날짜와 시간 관련 기능을 제공했지만 문제가 있다.
- 날짜를 의미하는 Date라는 클래스의 이름과 달리 특정 시점을 날짜가 아닌 밀리초 단위로 표현
- 1900년을 기준으로 하는 오프셋
- 0에서 시작하는 달 인덱스 등 모호한 설계로 유용성이 떨어짐
> 이러한 이유들로 자바 1.1에서는 Date 클래스의 여러 메소드를 deprecated시키고 java.util.Calendar 클래스를 대안으로 제공했다. 하지만 여기에도 문제가 발생했다.
- Calendar 클래스가 쉽게 에러를 일으키는 설계 문제를 갖고 있음
  - 1900년을 기준으로 한 오프셋은 없지만 달의 인덱스는 여전히 0부터 시작함
- Date, Calendar 두 가지 클래스가 등장하면서 개발자들에게 혼동을 가져옴
- DateFormat 같은 일부 기능들이 Date 클래스에만 작동함
  - DateFormat에도 문제가 있었는데 쓰레드에 안전하지 않아 두 쓰레드가 동시에 하나의 포매터로 날짜를 파싱할 때 예기치 못한 결과가 일어날 수 있었음
- Date, Calendar 모두 가변 클래스 설계여서 유지보수가 어려움
> 이러한 이유들로 많은 개발자들이 Joda-Time 같은 서드파티 날짜와 시간 라이브러리를 사용하게 되었고, 오라클은 좀 더 훌륭한 날짜와 시간 API를 제공하기로 결정해
> 결국 자바 8에서 Joda-Time의 많은 기능들을 java.time 패키지로 추가하게 되었다.

## LocalDate, LocalTime
> LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체로 특히 LocalDate 객체는 어떤 시간대 정보도 포함하지 않는다.

### LocalDate.of
> LocalDate 에서는 정적 팩토리 메소드인 of로 LocalDate 인스턴스를 만들 수 있고, LocalDate 인스턴스는 연도, 달, 요일 등을 반환하는 메소드를 제공한다.
```java
  LocalDate date = LocalDate.of(2017, 9, 21);  <- 2017-09-21
  int year = date.getYear();                   <- 2017
  Month month = date.getMonth();               <- SEPTEMBER
  int day = date.getDayOfMonth();              <- 21
  DayOfWeek dow = date.getDayOfMonth();        <- THURSDAY
  int len = date.lengthOfMonth();              <- 31 (3월의 일 수)
  boolean leap = date.isLeapYear();            <- false (윤년이 아닌)
```

### LocalDate.now
> 팩토리 메소드 now는 시스템 시계의 정보를 이용해서 현재 날짜 정보를 얻는다.
```java
  LocalDate today = LocalDate.now();
```

### TempralField, ChronoField
> get 메소드에 TemporalField를 전달해서 정보를 얻는 방법이 있다. 그러나 내장 메소드인 getYear(), getMonthValue(), getDayOfMonth() 등을 이용하는 것으로
> 가독성을 높일 수 있다
- TemporalField : 시간 관련 객체에서 어떤 필드의 값에 접근할지 정의하는 인터페이스
- ChronoField : TemporalField 인터페이스를 정의함

### LocalTime
> LocalDate와 마찬가지로 정적 메소드 of로 LocalTime 인스턴스를 만들 수 있다.
```java
  LocalTime time = LocalTime.of(13, 45, 20)    <- 13:45:20
  int hour = time.getHour();                   <- 13
  int minute = time.getMinute();               <- 45
  int second = time.getSecond();               <- 20
```
### LocalDate.parse(), LocalTime.parse()
> 날짜와 시간 문자열로 LocalDate와 LocalTime의 인스턴스를 만드는 방법도 있다.
```java
  LocalDate date = LocalDate.parse("2017-09-21");
  LocalTime time = LocalTime.parse("13:45:20");
```

## LocalDateTime
> LocalDateTime은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스로 날짜와 시간을 모두 표현할 수 있다.
- LocalDateTime.of 메소드에 날짜와 시간을 전달
- LocalDate의 atTime 메소드에 시간을 제공
- LocalTime의 atDate 메소드에 날짜를 제공
- LocalDateTime의 toLocalDate나 toLocalTime 메소드로 LocalDate나 LocalTime 인스턴스를 추출할 수 있다.

## instant 클래스
> 사람은 보통 주, 날짜, 시간, 분 등으로 날짜와 시간을 계산하지만 기계는 이와 같은 단위로 시간을 표현하기 어렵다. 그래서 java.util.Instant 클래스에서는 이와 같은
> 기계적인 관점에서 시간을 표현한다. 즉, Instant 클래스는 유닉스 에포크 시간(1970년 1월 1일 0시 0분 0초 UTC)을 기준으로 특정 지점까지의 시간을 초로 표현한다.

### Instant.ofEpochSecont
> 팩토리 메소드 ofEpochSecond에 초를 넘겨줘서 Instant 클래스 인스턴스를 만들 수 있다. Instant 클래스는 나노초의 정밀도를 제공한다. 또한 오버로드된 ofEpochSecond
> 메소드는 두 번째 인수를 이용해 나노초 단위로 시간을 보정할 수 있다.

### Instant.now
> Instant 클래스도 사람이 확인할 수 있도록 시간을 표시해주는 정적 팩토리 메소드 now를 제공한다. 하지만 Instant는 기계 전용의 유틸리티이다. 즉, Instant는 초와 나노초
> 정보를 포함한다. 따라서 Instant는 사람이 읽을 수 있는 시간 정보를 제공하지 않는다.

### Duration, Period
> Instatnt에서는 Duration과 Period 클래스를 함께 활용할 수 있다. 
- Duration 클래스는 두 개의 LocalTime, 두 개의 LocalDateTime, 두 개의 Instant로 만들 수 있다. 서로 혼합할 수 없다.
- Period 클래스는 년, 월, 일로 시간을 표현하며 두 LocalDate의 차이를 확인할 수 있다.

## 날짜 조정, 파싱, 포매팅
> withAttribute 메소드로 기존의 LocalDate를 바꾼 버전을 간단하게 만들 수 있다. 다음 코드에서는 바뀐 속성을 포함하는 새로운 객체를 반환하는 메소드이며 모든 메소드는
> 기존 객체를 바꾸지 않는다.
``` java
  LocalDate date1 = LocalDate.of(2017, 9, 21);                <- 2017-09-21
  LocalDate date2 = date1.withYear(2011);                     <- 2011-09-21
  LocalDate date3 = date2.withDayOfMonth(25);                 <- 2011-09-25
  LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2); <- 2011-02-25
```
> 마지막 with 메소드는 get 메소드와 쌍을 이룬다. 이 메소드는 Temporal 인터페이스에 정의되어 있는데 LocalDate, LocalTime, LocalDateTime, Instant처럼 특정 시간을 정의한다.
> 정확히 표현하면 get과 with 메소드로 Temporal 객체의 필드값을 읽거나 고칠 수 있다. 어떤 Temporal 객체가 지정된 필드를 지원하지 않으면 UnsupportedTemporalTypeException이
> 발생한다. Temporal 인터페이스에 정의되어 있는 plus, minus 메소드를 사용해 Temporal을 특정 시간만큼 앞뒤로 이동시킬 수도 있다.

### TemporalAdjusters 
> 때로는 다음 주 일요일, 돌아오는 평일, 어떤 달의 마지막 날 등 좀 더 복잡한 날짜 조정 기능이 필요할 것이다. 이때 오버로드된 버전의 with 메소드에 좀 더 다양한 동작을
> 수행할 수 있도록 하는 기능을 제공하는 TemporalAdjuster를 전달하는 방법으로 문제를 해결할 수 있다. 그 뿐만 아니라 필요한 기능이 정의되어 있지 않을 때는 비교적 쉽게
> 커스텀 TemporalAdjuster 구현을 만들 수 있다.

``` java
  import static java.time.temporal.TemporalAdjusters.*;
  LocalDate date1 = LocalDate.of(2014, 3, 18);                  <- 2014-03-18
  LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY));   <- 2014-03-23
  LocalDate date3 = date2.with(lastDayOfMonth());               <- 2014-03-31    
```

### DateTimeFormatter
> 날짜와 시간 관련 작업에서 포매팅과 파싱은 서로 떨어질 수 없는 관계다. DateTimeFormatter는 포매팅과 파싱 전용 패키지인 java.time.format에 추가되어 있는데 정적 팩토리
> 메소드와 상수를 이요해 손쉽게 포매터를 만들 수 있다. 기존의 java.util.dateFormat 클래스와 달리 모든 DateTimeFormatter는 쓰레드에서 안전하게 사용할 수 있다.

## 다양한 시간대와 캘린더 활용 방법
> 새로운 날짜와 시간 API의 큰 편리함 중 하나는 시간대를 간단하게 처리할 수 있다는 점인데, 기존의 java.util.TimeZone을 대체할 수 있는 java.util.ZoneId 클래스가 새롭게 
> 추가되었다. 서머타임(DST) 같은 복잡한 사항이 자동으로 처리되며, 날짜와 시간 API에서 제공하는 다른 클래스와 마찬가지로 불변 클래스이다.

### ZoneId
```java
  ZoneId romeZone = ZoneId.of("Europe/Rome");   (1)
  
  Instant instant = Instant.now();
  LocalDateTime timeFromInstant = LocalDateTime.ofInstant(instant, romeZone)
```
> (1)처럼 지역 ID로 특정 ZoneId를 구분하는데 지역 ID는 {지역}/{도시} 형식으로 이루어지며 IANA Time Zone Database에서 제공하는 지역 집합 정보를 사용한다. ZoneId 객체를
> 얻은 다음 LocalDate, LocalDateTime, Instant를 이용해 ZonedDateTime 인스턴스로 변환할 수 있다. ZonedDateTime은 지정한 시간대에 상대적인 시점을 표현한다. 그리고 (2)처럼
> ZoneId를 이용해서 LocalDateTime을 Instant로 바꾸는 방법도 있다
- ZonedDateTime = LocalDateTime + ZoneId
- LocalDateTime = LocatDate + LocalTime

## 마치며
- 자바 8 이전 버전에서 제공하는 기존의 java.util.Date 클래스와 관련 클래스에서는 여러 불일치점들과 가변성, 어설픈 오프셋, 기본값, 잘못된 이름 결정 등의 설계 결함이 존재함.
- 새로운 날짜와 시간 API에서 날짜와 시간 객체는 모두 불변이다.
- 새로운 API는 각각 사람과 기계가 편리하게 날짜와 시간 정보를 관리할 수 있도록 두 가지 표현 방식을 제공한다.
- 날짜와 시간 객체를 절대적인 방법과 상대적인 방법으로 처리할 수 있으며 기존 인스턴스를 변환하지 않도록 처리 결과로 새로운 인스턴스가 생성된다.
- TemporalAdjuster를 이용하면 단순히 값을 바꾸는 것 이상의 복잡한 동작을 수행할 수 있으며 자신만의 커스텀 날짜 변환 기능을 정의할 수 있다.
- 날짜와 시간 객체를 특정 포맷으로 출력하고 파싱하는 포매터를 정의할 수 있다. 패턴을 이용하거나 프로그램으로 포매터를 만들 수 있으며 포매터는 쓰레드 안정성을 보장한다.
