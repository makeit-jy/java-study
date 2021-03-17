# Chapter 10 null 대신 Optional 클래스

**NullPointerException** 익숙한 문구..!!

- **Null Pointer:** '데이터가 아직 없는 상태', 프로그램을 만들다 보면 여러 가지 이유로 어떤 메모리 공간에 아직 데이터를 저장하지 못할 때가 있습니다.
- **Null Pointer가 야기하는 대표적 문제:** '**널 포인터 참조**', 데이터가 아직 없는데 읽으려고 시도했을 때 나타나는 오류 (그냥 그 프로그램을 종료시킴.)

**Null Pointer**라는 개념은 '**토니 호어**'(Tony Hoare)가 1965년에 처음 발명했습니다. 그리고 40년이 흐른 2009년, 토니 호어는 한 강연에서 다음과 같이 말했습니다.

> (널 포인터는) 내 10억 달러짜리 실수였다. 1965년 당시, 나는 ALGOL W라는 객체 지향 언어에 쓰기 위해 포괄적인 타입 시스템을 설계하고 있었다. 내 원래 목표는 어떤 데이터를 읽든 항상 안전하도록 컴파일러가 자동으로 확인해 주는 것이었다. 그러나 나는 널 포인터를 집어넣으려는 유혹을 이길 수가 없었다. 그렇게 하는 게 훨씬 쉬웠기 때문이다. 이 결정은 셀 수도 없는 오류와 보안 버그, 시스템 다운을 낳았다. 지난 40년 동안 이러한 문제들 때문에 입은 고통과 손해는 10억 달러는 될 것이다.

## 1. 값이 없는 상황을 어떻게 처리할까?

```java
// 자동차와 자동차 보험을 갖고 있는 사람 객체를 중첩 구조로 구현.

public class Person {
	private Car car;
	public Car getCar() { return car; }
}
public class Car {
	private Insurance insurance;
	public Insurance getInsurance() { return insurance; }
}
public class Insurance {
	private String name;
	public String getName() { return name; }
}

// 메인 클래스
public String getCarInsuranceName(Person person) {
	return person.getCar().getInsurance().getName();
}

// getCarInsuranceName 메서드 실행 시 Person의 Car가 비어있다면 NullPointerException 발생.
```

### 1.1) 보수적인 자세로 NullPointerException 줄이기

- 필요한 곳에 다양한 null 확인 코드를 추가해서 null 예외 문제를 해결

```java
// null 안전 시도 1) 깊은 의심 
public String getCarInsuranceName(Person person) {
	if ( person != null ) { 
		Car car = person.getCar();
		if ( car != null ) {
			Insurance insurance = car.getInsurance();
			if ( insurance != null ) {
				return insurance.getName();
			}
		}
	}
	return "Unknown";
}
// null 확인 코드 때문에 나머지 호출 체인의 들여쓰기 수준이 증가함. -> 반복하면 코드 구조 엉망, 가독성 떨어짐.
// 이와 같은 반복 패턴 코드를 '깊은 의심(deep doubt)' 이라고 부름. 

// null 안전 시도 2) 너무 많은 출구
public String getCarInsuranceName(Person person) {
	if( person == null ) {
		return "Unknown";
	}
	Car car = person.getCar();
	if ( car == null ) {
		return "Unknown";
	}
	Insurance insurance = car.getInsurance();
	if ( insurance == null ) {
		return "Unknown";
	}
	return insurance.getName();
}
// null 확인 코드마다 출구가 생김.
// 많은 출구 때문에 유지보수가 어려워짐.
```

### 1.2) null 때문에 발생하는 문제

- 에러의 근원이다: NullPointerException은 자바에서 가장 흔히 발생하는 에러다.
- 코드를 어지럽힌다: 때로는 중첩된 null 확인 코드를 추가해야 하므로 null 때문에 가독성이 떨어진다.
- 아무 의미가 없다:  null은 아무 의미도 표현하지 않는다. 특히 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다.
- 자바 철학에 위배된다: 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 한 가지 예외가 있는데 그것이 null 포인터다.
- 형식 시스템에 구멍을 만든다: null은 무형식이며 정보를 포함하고 있지 않으므로 모든 레퍼런스 형식에 null을 할당할 수 있다. 이런 식으로 null이 할당되기 시작하면서 시스템의 다른 부분으로 null이 퍼졌을 때 애초에 null이 어떤 의미로 사용되었는지 알 수 없다.

### 1.3) 다른 언어는 null 대신 무얼 사용하나?

자바 8에서 null 대신 제공하는 것

- '선택형 값' 개념의 영향을 받아서 java.util.Optional<T> 클래스 제공

## 2. Optional 클래스 소개

- Optional : 선택형값을 캡슐화하는 클래스

    예) null 사용 시, 차를 소유하고 있지 않은 Person 클래스의 car 변수는 null을 가져야 함.

    ![Chapter%2010%20null%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20Optional%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%2003e92cd286824595a62178416264216a/1Optional_Car.jpg](Chapter%2010%20null%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20Optional%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%2003e92cd286824595a62178416264216a/1Optional_Car.jpg)

- 값이 있다면 Optional 클래스는 값을 감싸고, 값이 없으면 Optional.empty 메서드로 Optional을 반환.
- Optional.empty는 Optional의 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드
- null 레퍼런스와 Optional.empty() 차이
    - null을 참조하려 하면 NullPointerException 발생.
    - Optional.empty()는 Optional 객체이므로 이를 다양한 방식으로 활용 가능.
    - null 대신 Optional을 사용하면서 Car 형식이 Optional<Car>로 바뀜.

    ```java
    public class Person {
        private Optional<Car> car; // 사람이 차를 소유했을 수도 소유하지 않았을 수도 있으므로 Optional
        public Optional<Car> getCar() { return car; }
    }
    public class Car {
        private Optional<Insurance> insurance; // 자동차 보험에 가입 or 미가입일수 있으므로 Optional
        public Optional<Insurance> getInsurance() { return insurance; }
    }
    public class Insurance {
        private String name; // 보험회사에는 반드시 이름이 존재함.
        public String getName() { return name; }
    }
    ```

- Optional을 이용하면 값이 없는 상황이 데이터의 문제인지, 알고리즘의 버그인지 명확하게 구분 가능.
- 그러나 모든 null 레퍼런스를 Optional로 대치하는 것은 바람직하지 않다.

    Optional의 역할은 더 이해하기 쉬운 API를 설계하도록 돕는 것!

## 3. Optional 적용 패턴

### 3.1) Optional 객체 만들기 - Optional 객체를 만드는 방법

```java
// 1. 빈 Optional
Optional<Car> optCar = Optional.empty();

// 2. null이 아닌 값으로 Optional 만들기
Optional<Car> optCar = Optional.of(car); 
// car가 null이면 즉시 NullPointerException 발생
// (Optional을 사용하지 않았다면 car의 프로퍼티에 접근하려 할 때 에러 발생)

// 3. null값으로 Optional 만들기
Optional<Car> optCar = Optional.ofNullable(car);
// car가 null이면 빈 Optional 객체 반환
```

### 3.2) 맵으로 Optional의 값을 추출하고 변환하기

```java
// 이름 정보에 접근하기 전에 insurance가 null인지 확인 
String name = null;
if ( insurance != null ) { 
    name = insurance.getName();
}

// Optional의 map 메서드 지원
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
/*
 - Optional 객체를 최대 요소의 개수가 한 개 이하인 데이터 컬렉션으로 생각할 수 있다.
 - Optional이 값을 포함하면 map의 인수로 제공된 함수가 값을 바꾼다.
 - Optional이 비어있으면 아무 일도 일어나지 않는다.
*/
```

![Chapter%2010%20null%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20Optional%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%2003e92cd286824595a62178416264216a/2_Optional_map__.jpg](Chapter%2010%20null%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20Optional%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%2003e92cd286824595a62178416264216a/2_Optional_map__.jpg)

### 3.3) flatMap으로 Optional 객체 연결

```java
Optional<Person> optPerson = Optional.of(person);
Optional<String> name = optPerson
							.map(Person::getCar)
							.map(Car::getInsurance)  // 컴파일 에러!!
							.map(Insurance::getName); 
/*
 - 에러 원인 
  optPerson의 타입은 Optional<Person> -> map 메서드 호출 가능.
  getCar는 Optional<Car> 형식의 객체를 반환.
  -> map 메서드의 결과는 Optional<Optional<Car>> 형식의 중첩 Optional 객체 구조 
  -> 해결하기 위해서 flatMap 메서드 사용! (스트림의 flatMap과 유사한 기능)
     함수를 적용해서 생성된 모든 스트림을 하나의 스트림으로 병합하여 평준화시킴.
*/
```

![Chapter%2010%20null%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20Optional%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%2003e92cd286824595a62178416264216a/3Optional.jpg](Chapter%2010%20null%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20Optional%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%2003e92cd286824595a62178416264216a/3Optional.jpg)

![Chapter%2010%20null%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20Optional%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%2003e92cd286824595a62178416264216a/4_Optional_flatMap__.jpg](Chapter%2010%20null%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20Optional%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%2003e92cd286824595a62178416264216a/4_Optional_flatMap__.jpg)

- Optional로 자동차의 보험회사 이름 찾기

```java
public String getCarInsuranceName(Optional<Person> person) {
	return person.flatMap(Person::getCar)   // Optional<Person>를 Optional<Car>로 반환
							.flatMap(Car::getInsurance) // Optional<Car>를 Optional<Insurance>로 반환
							.map(Insurance::getName)    // getName은 String을 반환하므로 flatMap 필요 X
							.orElse("Unknown");         // 결과 Optional이 비어있으면 기본값 사용
}
/*
 - null을 확인하려고 조건 분기문을 추가해서 코드가 복잡해지는 현상 막을 수 있음.
 - 쉽게 이해할 수 있는 코드
 - 이 메서드가 빈 값을 받거나 빈 결과를 반환할 수 있음을 잘 문서화해서 제공하는 것과 같음. 
*/
```

- Optional을 이용한 Person/Car/Insurance 참조 체인

![Chapter%2010%20null%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20Optional%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%2003e92cd286824595a62178416264216a/5Optional__PersonCarInsurance__.jpg](Chapter%2010%20null%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20Optional%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%2003e92cd286824595a62178416264216a/5Optional__PersonCarInsurance__.jpg)

- 파이프라인 연산
    - flatMap을 이용한 평준화 (두 Optional을 합치는 기능을 수행하면서 둘 중 하나라도 null이면 빈 Optional을 생성하는 연산)
    - flatMap을 빈 Optional에 호출하면 아무 일도 일어나지 않고 그대로 반환됨.
    - 반면 Optional이 Person을 감싸고 있다면 flatMap에 전달된 Function이 Person에 적용됨.

Q. 도메인 모델에 Optional을 사용했을 때 데이터를 직렬화할 수 없는 이유

- Optional 클래스는 필드 형식으로 사용할 것을 가정하지 않았으므로 Serializable 인터페이스 구현X.
- 도메인 모델에 Optional을 사용한다면 직렬화 모델을 사용하는 도구, 프레임워크에 문제 발생.
- 그럼에도 일부 또는 전체 객체가 null일 수 있는 상황에서는 Optional 사용이 바람직함.

```java
// 직렬화 모델이 필요할 때
// Optional로 값을 반환받을 수 있는 메서드를 추가하는 방식 권장 
public class Person { 
    private Car car;
    public Optional<Car> getCarAsOptional() {
        return Optional.ofNullable(car);
    }
}
```

### 3.4) 디폴트 액션과 Optional 언랩

- Optional 인스턴스에서 값을 읽을 수 있는 다양한 인스턴스 메서드
    - get() : 값을 읽는 가장 간단한 메서드, 가장 안전하지 않은 메서드

        래핑된 값이 있으면 해당 값 반환, 없으면 NoSuchElementException 발생

        따라서 Optional에 값이 반드시 있다고 가정할 수 있는 상황이 아니라면 사용 권장 X

    - orElse() : Optional이 값을 포함하지 않을 때 디폴트값 제공 가능
    - orElseGet(Supplier<? extends T> other) : Optional에 값이 없을 때만 Supplier가 실행됨

        디폴트 메서드를 만드는 데 시간이 걸리거나 Optional이 비어있을 때만 디폴트값을 생성하고 싶다면 이를 사용해야 함

    - orElseThrow(Supplier<? extends X> exceptionSupplier) : Optional 비어있을 때 예외발생시킴

        발생시킬 예외의 종류를 선택할 수 있음

    - ifPresent(Consumer<? super T> consumer) : 값이 존재할 때 인수로 넘겨준 동작 실행 가능

        값이 없으면 아무 일도 일어나지 않음

### 3.5) 두 Optional 합치기

- 두 Optional을 인수로 받아서 Optional<Insurance>를 반환하는 null 안전버전의 메서드 구현할 때

인수로 전달한 값 중 하나라도 비어있으면 빈 Optional<Insurance>를 반환

- Optional 클래스는 Optional이 값을 포함하는지 여부를 알려주는 isPresent 메서드 제공

```java
public Insurance findCheapestInsurance(Person person, Car car) {
	return cheapestCompany;
}
	
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person,
                                                         Optional<Car> car) {
	if ( person.isPresent() && car.isPresent() ) {
		return Optional.of(findCheapestInsurance(person.get(), car.get()));
	} else {
		return Optional.empty();
	}
}
/*
 - isPresent() 메서드
  * person과 car의 시그니처만으로 둘 다 아무 값도 반환하지 않을 수 있다는 정보를 명시적으로 보여줌
  * 그러나 이 코드도 null 확인 코드와 크게 다른 점이 없다.
*/

// map과 flatMap을 이용하여 해결
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
	return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```

### 3.6) 필터로 특정값 거르기

```java
// 예제) 보험회사 이름이 'CambridgeInsurance'인지 확인하기 
// 전 
Insurance insurance = ...;
if ( insurance != null && "CambridgeInsurance".equals(insurance.getName())) {
    System.out.println("ok");
}

// 후
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance ->
        "CambridgeInsurance".equals(insurance.getName()))
    .ifPresent(x -> System.out.println("ok"));
```

```java
// 예제) 인수 person이 minAge 이상의 나이일 때만 보험회사 이름을 반환
public String getCarInsuranceName(Optional<Person> person, int minAge) {
    return person.filter(p -> p.getAge() >= minAge)
                .flatMap(Person::getCar)
                .flatMap(Car::getInsurance)
                .map(Insurance::getName)
                .orElse("Unknown");
}
```

## 4. Optional을 사용한 실용 예제

### 4.1) 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기

```java
// Map<String, Object> 형식의 맵에서 다음과 같이 key로 값에 접근할 때
Object value = map.get("key");
// key 에 해당하는 값이 없으면 null 반환
// 방법1) 기존의 if-then-else 추가
// 방법2) Optional.ofNullable 이용
Optional<Object> value = Optional.ofNullable(map.get("key"));
```

### 4.2) 예외와 Optional

- 자바 API는 어떤 이유에서 값을 제공할 수 없을 때 null을 반환하는 대신 예외를 발생시킬 때 있음

- 이에 대한 예로 문자열을 정수로 변환하는 Integer.parseInt(String)을 들 수 있다.

```java
public static Optional<Integer> stringToInt(String s) {
    try {
        return Optional.of(Integer.parseInt(s)); // 변환 가능 시 변환된 값 포함하는 Optional 반환
    } catch (NumberFormatException e) { 
        return Optional.empty(); // 그 외에는 빈 Optional 반환
    }
}
// 위와 같은 메서드를 포함하는 OptionalUtility 유틸리티 클래스 만드는 것 추천
```

- 기본형 Optional과 이를 사용하지 말아야 하는 이유
    - Optional은 기본형으로 특화된 OptionalInt, OptionalLong, OptionalDouble 등 클래스 제공
    - Optional의 최대 요소 수는 한 개 이므로 Optional에서는 기본형 특화 클래스로 성능 개선 X
    - 기본형 특화 Optional은 map, flatMap, filter 메서드 지원 X
    - 기본형 특화 Optional로 생산한 결과는 다른 일반 Optional과 혼용 X

### 4.3) 응용

```java
// 예제) 프로퍼티에서 지속시간 읽기
// 1. 명령형 코드 
public int readDuration(Properties props, String name) {
	String value = props.getProperty(name);
	if ( value != null ) {
		try {
			int i = Integer.parseInt(value);
			if ( i > 0 ) {
				return i;
			}
		} catch (NumberFormatException e) {	}
	}
	return 0;
}

// 2. Optional 사용 코드
public int readDuration(Properties props, String name) {
	return Optional.ofNullable(props.getProperty(name))
				.flatMap(OptionalUtility::stringToInt)
				.filter(i -> i > 0)
				.orElse(0);
}
class OptionalUtility {
    public static Optional<Integer> stringToInt(String s) {
	    try {
	        return Optional.of(Integer.parseInt(s)); 
	    } catch (NumberFormatException e) { 
	        return Optional.empty(); 
	    }
	}
}
```

## 5.요약

- 역사적으로 프로그래밍 언어에서는 null 레퍼런스로 값이 없음을 표현해왔다.
- 자바 8에서는 값이 있거나 없음을 표현할 수 있는 클래스 java.util.Optional<T>를 제공한다.
- 팩토리 메서드 Optional.empty, Optional.of, Optional.ofNullable 등을 이용해서 Optional 객체를 만들 수 있다.
- Optional은 스트림과 비슷한 연산을 수행하는 map, flatMap, filter 등의 메서드를 제공한다.
- Optional로 값이 없는 상황을 적절하게 처리하도록 강제할 수 있어 예상치 못한 null 예외를 방지 가능하다.
- 더 좋은 API를 설계할 수 있다. 사용자는 메서드의 시그니처만 보고도 Optional값이 사용되거나 반환되는지 예측 가능하다.