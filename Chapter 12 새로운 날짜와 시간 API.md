# Chapter 12 새로운 날짜와 시간 API

- 이 장의 내용
    - 자바 8에서 새로운 날짜와 시간 라이브러리를 제공하는 이유
    - 사람이나 기계가 이해할 수 있는 날짜와 시간 표현 방법
    - 시간의 양 정의하기
    - 날짜 조작, 포매팅, 파싱
    - 시간대와 캘린더 다루기
- 개요
    - 자바 1.0 : [java.util.Date](http://java.util.Date) 클래스
    특정 시점을 날짜가 아닌 밀리초 단위로 표현(Date라고 왜 썼음)
    1900년을 기준으로 하는 *오프셋*, 0에서 시작하는 달 인덱스 등 모호한 설계

        ```sql
        Date date = new Date(114, 2, 18);
        // Tue Mar 18 00:00:00 CET 2014
        // CET : Central Europe Time, JVM 기본시간대인 CET, 중앙 유럽 시간을 사용,
        // 그렇다고 자체적으로 시간대 정보를 알고 있는 것도 아님
        ```

    - 자바 1.1 : Date 클래스의 여러 메서드를 컷! java.util.Calender 클래스를 대안으로 제시
    ~~1900년 기준으로 하는 오프셋~~, 0에서 시작하는 달의 인덱스, DateFormat 같은 일부 기능 X

        산넘어 산 : DateFormat도 문제, 스레드에 안전하지 않다.

    - 자바 8 : java.time 패키지로 추가

        ### 순서

        - 사람과 기기에서 사용할 수 있는 날짜와 시간을 생성하는 기본적인 방법
        - 날짜 시간 객체를 조작, 파싱, 출력
        - 다양한 시간대와 대안 캘린더 등 새로운 날짜와 시간 API를 사용하는 방법
- 사람과 기기에서 사용할 수 있는 날짜와 시간을 생성하는 기본적인 방법

    ### java.time 패키지

    - LocalDate, LocalTime, LocalDateTime, Instant, Duration, Period 등 새로운 클래스 제공

    ### LocalDate : 시간을 제외한 날짜
    LocalTime : 시간을 표현

    - LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체이다.

    ```java
    LocalDate date = LocalDate.of(2014, 3, 18); //2014-03-18
    int year = date.getYear();                  //2014
    Month month = date.getMonth();              //MARCH
    int day = date.getDayOfMonth();             //18
    DayOfWeek dayOfWeek = date.getDayOfWeek();  //TUESDAY
    int lengthOfMonth = date.lengthOfMonth();   //31
    boolean isLeapYear = date.isLeapYear();     //false

    //TemporalField를 이용해서 LocalDate 값 읽기
    int year2 = date.get(ChronoField.YEAR);           //2014 
    int month2 = date.get(ChronoField.MONTH_OF_YEAR); //3
    int day2 = date.get(ChronoField.DAY_OF_MONTH);    //18
    ```

    - 13:45:20 같은 시간은 LocalTime 클래스로 표현할 수 있다.

    ```java
    LocalTime time = LocalTime.of(13, 45, 20);  //13:45:20
    int hour = time.getHour();                  //13
    int minute = time.getMinute();              //45
    int second = time.getSecond();              //20

    LocalDate date = LocalDate.parse("2014-03-50");  //2014-03-18
    LocalTime time = LocalTime.parse("13:45:20");    //13:45:20
    ```

    - parse 메서드에 DateTimeFormatter를 전달할 수도 있다. 날짜, 시간 객체의 형식을 지정한다.
    - 문자열을 LocalDate나 LocalTime으로 파싱할 수 없을 때 parse 메서드는 DateTimeParseException(RuntimeException을 상속받은 예외)를 일으킨다.

    ### LocalDateTime : 날짜와 시간 조합, 직접 만들기도 가능

    - LocalDateTime은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스

    ```java
    //2014-03-18T13:45:20
    LocalDateTime dt1 = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45, 20);
    LocalDateTime dt2 = LocalDateTime.of(date, time);

    //LocalDate, LocalTime -> LocalDateTime 만들기
    LocalDateTime dt3 = date.atTime(13, 45, 20);
    LocalDateTime dt4 = date.atTime(time);
    LocalDateTime dt5 = time.atDate(date);

    //LocalDateTime -> LocalDate, LocalTime 추출
    LocalDate date1 = dt1.toLocalDate();
    LocalTime time1 = dt1.toLocalTime();
    ```

    ### Instant : 기계의 날짜와 시간

    - 기계의 관점에서는 연속된 시간에서 특정지점을 하나의 큰 수로 표현하는 것이 가장 자연스러운 시간 표현 방법
    - 유닉스 에포크 시간(Unix epoch time, 1970.01.01 00:00:00 UTC)을 기준으로 특정지점까지의 시간을 초로 표현

    ```java
    Instant instant1 = Instant.ofEpochSecond(3);
    Instant instant2 = Instant.ofEpochSecond(3, 0);
    Instant instant3 = Instant.ofEpochSecond(2,1000000000);

    //기계 전용 유틸리티이기에 사람이 읽을 수 있는 시간 정보를 제공하지 않아 예외를 일으킨다.
    //UnsupportedTemporalTypeException
    int dayOfMonth = Instant.now().get(ChronoField.DAY_OF_MONTH);
        
    //Instant에서는 Duration과 Period 클래스를 함께 활용할 수 있다.
    ```

    ### Duration : 두 시간 객체 사이의 지속시간
    Period : 년, 월, 일로 시간을 표현할 때 사용

    ```java
    LocalTime time1 = LocalTime(13, 00, 00);
    LocalTime time2 = LocalTime(14, 00, 00);
    Duration d1 = Duration.between(time1, time2);
        
    LocalDateTime dateTime1 = LocalDateTime.of(2014, Month.MARCH, 18, 13, 00, 00);
    LocalDateTime dateTime2 = LocalDateTime.of(2015, Month.MARCH, 18, 13, 00, 00);
    Duration d2 = Duration.between(dateTime1, dateTime2);

    Instant instant1 = Instant.ofEpochSecond(3);
    Instant instant2 = Instant.ofEpochSecond(4, 0);
    Duration d3 = Duration.between(instant1, instant2);
    ```

    - LocalDateTime : 사람, Instant : 기계 → 혼합 X
    - Duration 클래스는 초와 나노초로 시간 단위를 표현하므로 between메서드에 LocalDate를 전달할 수 없다!
    - 년, 월, 일로 시간을 표현할 때는 Period 클래스를 사용한다.

    ```java
    Period tenDays = Period.between(LocalDate.of(2014, 3, 8), LocalDate.of(2014, 3, 18));

    //Duration과 Period 클래스는 자신의 인스턴스를 마들 수 있도록 다양한 팩토리 메서드를 제공한다.
    Duration threeMinutes1 = Duration.ofMinutes(3);
    Duration threeMinutes2 = Duration.of(3, ChronoUnit.MINUTES);

    Period tenDays2 = Period.ofDays(10);
    Period threeWeeks = Period.ofWeeks(3);
    Period twoYearsSixMonthsOneDay = Period.of(2, 6, 1);
    ```

    ![Chapter%2012%20%E1%84%89%E1%85%A2%E1%84%85%E1%85%A9%E1%84%8B%E1%85%AE%E1%86%AB%20%E1%84%82%E1%85%A1%E1%86%AF%E1%84%8D%E1%85%A1%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B5%E1%84%80%E1%85%A1%E1%86%AB%20API%2018c80ab079b2433ea362c9471050fd13/Untitled.png](Chapter%2012%20%E1%84%89%E1%85%A2%E1%84%85%E1%85%A9%E1%84%8B%E1%85%AE%E1%86%AB%20%E1%84%82%E1%85%A1%E1%86%AF%E1%84%8D%E1%85%A1%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B5%E1%84%80%E1%85%A1%E1%86%AB%20API%2018c80ab079b2433ea362c9471050fd13/Untitled.png)

    - 현재까지 살펴본 모든 클래스는 불변이다?
    ? 무슨 의미이지?
        - 함수형 프로그래밍, 스레드 안정성과 도메인 모델의 일관성을 유지하는 데 좋은 특징
        - 하지만 새로운 날짜와 시간 API에서는 변경된 객체 버전을 만들 수 있는 메서드를 제공
        → 날짜 조정, 파싱, 포매팅으로
- 날짜 조정, 파싱, 포매팅

    ### withAttribute 메서드

    ```java
    //절대적인 방식으로 LocalDate의 속성 바꾸기
    LocalDate date1 = LocalDate.of(2014,3,18);   //2014-03-18
    LocalDate date2 = date1.withYear(2011);      //2011-03-18로 변경
    LocalDate date3 = date2.withDayOfMonth(25);  //2011-03-25로 변경
    LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 9) //2011-09-25로 변경

    //상대적인 방식으로 LocalDate의 속성 바꾸기
    LocalDate date1 = LocalDate.of(2014,3,18);   //2014-03-18
    LocalDate date2 = date1.plusWeeks(1);        //2014-03-25로 변경
    LocalDate date3 = date2.minusYears(3);       //2011-03-25로 변경
    LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS);  //2011-09-25로 변경
    ```

    - date4처럼 속성을 바꾸는 방법을 사용하면(TemporalField를 갖는 메서드를 사용하면) 더 범용적으로 메서드를 활용할 수 있다.
        - with 메서드와 get 메서드는 쌍을 이룬다.
        - Temporal 인터페이스 참고
            - 특정 시간을 정의
            - get과 with 메서드로 Temporal 객체의 필드값을 읽거나 고칠 수 있다.

    ![Chapter%2012%20%E1%84%89%E1%85%A2%E1%84%85%E1%85%A9%E1%84%8B%E1%85%AE%E1%86%AB%20%E1%84%82%E1%85%A1%E1%86%AF%E1%84%8D%E1%85%A1%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B5%E1%84%80%E1%85%A1%E1%86%AB%20API%2018c80ab079b2433ea362c9471050fd13/Untitled%201.png](Chapter%2012%20%E1%84%89%E1%85%A2%E1%84%85%E1%85%A9%E1%84%8B%E1%85%AE%E1%86%AB%20%E1%84%82%E1%85%A1%E1%86%AF%E1%84%8D%E1%85%A1%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B5%E1%84%80%E1%85%A1%E1%86%AB%20API%2018c80ab079b2433ea362c9471050fd13/Untitled%201.png)

    ### Quiz 12-1

    ```java
    //date의 변수값은? -> 객체 불변
    LocalDate date = LocalDate.of(2021, 03, 18);        //주소1
    date = date.with(ChronoField.MONTH_OF_YEAR, 4);     //주소2
    date = date.plusYear(2).minusDays(10).plusWeeks(1); //주소3
    //2023-04-15
    date.withYear(2011);

    //정답 : 2023-04-15
    ```

    ### TemporalAdjusters 사용

    - 복잡한 날짜 조정 기능
    - TemporalAdjuster는 인터페이스, 
    TemporalAdjusters는 여러 TemporalAdjuster를 반환하는 정적 팩토리 메서드를 포함하는 클래스

    ```java
    //미리 정의된 TemporalAdjusters의 기능 활용
    import static java.time.temporal.TemporalAdjusters.*;
    LocalDate date1 = LocalDate.of(2014, 3, 18);  //2014-03-18
    LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY));  //2014-03-23
    LocalDate date3 = date2.with(lastDayOfMonth());              //2014-03-31
    ```

    ![Chapter%2012%20%E1%84%89%E1%85%A2%E1%84%85%E1%85%A9%E1%84%8B%E1%85%AE%E1%86%AB%20%E1%84%82%E1%85%A1%E1%86%AF%E1%84%8D%E1%85%A1%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B5%E1%84%80%E1%85%A1%E1%86%AB%20API%2018c80ab079b2433ea362c9471050fd13/Untitled%202.png](Chapter%2012%20%E1%84%89%E1%85%A2%E1%84%85%E1%85%A9%E1%84%8B%E1%85%AE%E1%86%AB%20%E1%84%82%E1%85%A1%E1%86%AF%E1%84%8D%E1%85%A1%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B5%E1%84%80%E1%85%A1%E1%86%AB%20API%2018c80ab079b2433ea362c9471050fd13/Untitled%202.png)

    - 더 필요한 기능이 필요할 때는 커스텀 TemporalAdjuster 구현

    ```java
    @FunctionalInterface
    public interface TemporalAdjuster {
    	Temporal adjustInto(Temporal temporal);
    }

    //Temporal 객체를 어떻게 다른 Temporal 객초로 변환할지 정의
    ```

    ### Quiz 12-2

    ```java
    //TemporalAdjuster 인터페이스를 구현하는 NextWorkingDay 클래스를 구현하시오
    //이 클래스는 하루씩 다음 날로 바꾸는데 이때 토요일과 일요일은 건너뛴다.
    date = date.with(new NextWorkingDay());
    //만일 이동할 날짜가 평일이 아니라면, 즉 토요일이나 일요일이라면 월요일로 이동한다.
    public class NextWorkingDay implements TemporalAdjuster {
    	@Override
    	public Temporal adjustInto(Temporal temporal){
    		//현재 날짜 읽기
    		DayOfWeek dayOfWeek = DayOfweek.of(temporal.get(ChronoField.DAY_OF_WEEK));
    		int dayToAdd = 1;
    		//금요일이라면 토,일 건너뛰어야하므로
        if(dayOfWeek == DayOfWeek.FRIDAY) dayToAdd = 3;
        else if(dayOfWeek = DayOfWeek.SATURDAY) dayToAdd = 2;
        return temporal.plus(dayToAdd, ChronoUnit.DAYS);
      }
    }
    ```

    ### 날짜와 시간 객체 출력과 파싱

    - 포매팅과 파싱, java.time.format 추가됨
    - DateTimeFormatter, 정적 팩토리 메서드와 상수를 이용해서 쉽게 가능

    ```java
    //포매팅
    LocalDate date = LocalDate.of(2021, 03, 14);
    String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); //20210314
    String s1 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); //2021-03-14

    //파싱
    LocalDate date1 = LocalDate.parse("20140318", DateTimeFormatter.BASIC_ISO_DATE);
    LocalDate date2 = LocalDate.parse("2014-03-18", DateTimeFormatter.ISO_LOCAL_DATE);

    //특정 패턴
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
    LocalDate date1 = LocalDate.of(2021, 3, 14);    //2021-03-14
    String formattedDate = date1.format(formatter); //"14/03/2021"
    LocalDate date2 = LocalDate.parse(formattedDate, formatter); //2021-03-14

    //지역화된 포맷도 가능(예시 : 이탈리아~)
    ```

- 다양한 시간대와 캘린더 활용 방법
    - 시간대에 대한 간단한 처리!
    - 기존 : java.util.TimeZone
    - 신규 : java.time.ZoneId 클래스
        - 서머타임 자동 처리
        - 불변 클래스
        - 지역 ID로 특정 ZoneId를 구분한다. {지역}/{도시}

    ```java
    ZoneId romeZone = ZoneId.of("Europe/Rome");

    //특정 시점에 시간대 적용
    LocalDate date = LocalDate.of(2021, Month.MARCH, 14);
    ZoneDateTime zdt1 = date.atStartOfDay(romeZone);

    LocalDateTime dateTime = LocalDateTime.of(2021, Month.MARCH, 18, 00, 00);
    ZoneDateTime zdt2 = dateTime.atZone(romeZone);

    Instant instant = Instant.now();
    ZonedDateTime zdt3 = instant.atZone(romeZone);

    2021-03-14
    2021-03-14T00:00+01:00[Europe/Rome]
    2021-03-18T00:00
    2021-03-18T00:00+01:00[Europe/Rome]
    2021-03-13T16:47:33.602Z
    2021-03-13T17:47:33.602+01:00[Europe/Rome]
    ```

    ![Chapter%2012%20%E1%84%89%E1%85%A2%E1%84%85%E1%85%A9%E1%84%8B%E1%85%AE%E1%86%AB%20%E1%84%82%E1%85%A1%E1%86%AF%E1%84%8D%E1%85%A1%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B5%E1%84%80%E1%85%A1%E1%86%AB%20API%2018c80ab079b2433ea362c9471050fd13/Untitled%203.png](Chapter%2012%20%E1%84%89%E1%85%A2%E1%84%85%E1%85%A9%E1%84%8B%E1%85%AE%E1%86%AB%20%E1%84%82%E1%85%A1%E1%86%AF%E1%84%8D%E1%85%A1%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B5%E1%84%80%E1%85%A1%E1%86%AB%20API%2018c80ab079b2433ea362c9471050fd13/Untitled%203.png)

    - UTC/GMT 기준의 고정 오프셋
        - UTC 협정 세계시
        - GMT 그리니치 표준시
    - 대안 캘린더(ThaiBuddhistDate, MinguoDate, JapaneseDate, HijrahDate)

### 요약

- 자바 8 이전 버전에서 제공하는 기존의 [java.util.Date](http://java.util.Date) 클래스의 관련 클래스에서는 여러 불일치점들과 가변성, 어설픈 오프셀, 기본값, 잘못된 이름 결정 등의 설계 결함이 존재했다.
- 새로운 날짜와 시간 API에서 날짜와 시간 객체는 모두 불변이다.
- 새로운 API는 각각 사람과 기계가 편리하게 날짜와 시간 정보를 관리할 수 있도록 두 가지 표현 방식을 제공한다.
- 날짜와 시간 객체를 절대적인 방법과 상대적인 방법으로 처리할 수 있으며, 기존 인스턴스를 변환하지  않도록 처리 결과로 새로운 인스턴스가 생성된다.
- TemporalAdjuster를 이용하면 단순히 값을 바꾸는 것 이상의 복잡한 동작을 수행할 수 있으며 자신만의 커스텀 날짜 변환 기능을 정의할 수 있다.
- 날짜와 시간 객체를 특정 포맷으로 출력하고 파싱하는 포매터를 정의할 수 있다. 패턴을 이용하거나 프로그램으로 포매터를 만들 수 있으며 포매터는 스레드 안정성을 보장한다.
- 특정 지역/장소에 상대적인 시간대 또는 UTC/GMT 기준의 오프셋을 이용해서 시간대를 정의할 수 있으며 이 시간대를 날짜와 시간 객체에 적용해서 지역화할 수 있다.
- ISO-8601 표준 시스템을 준수하지 않는 캘린더 시스템도 사용할 수 있다.
- 진우

    [과거 자바 date객체의 (나쁜) 특징]
    1. Not thread safe − java.util.Date is not thread safe, thus developers have to deal with concurrency issue while using date. The new date-time API is immutable and does not have setter methods.

    2. Poor design − Default Date starts from 1900, month starts from 1, and day starts from 0, so no uniformity. The old API had less direct methods for date operations. The new API provides numerous utility methods for such operations.

    3. Difficult time zone handling − Developers had to write a lot of code to deal with timezone issues. The new API has been developed keeping domain-specific design in mind.

- 동진

    불변 객체
    [https://velog.io/@conatuseus/Java-Immutable-Object불변객체](https://velog.io/@conatuseus/Java-Immutable-Object%EB%B6%88%EB%B3%80%EA%B0%9D%EC%B2%B4)