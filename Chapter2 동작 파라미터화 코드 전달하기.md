# Chapter2 동작 파라미터화 코드 전달하기

---

**이 장의 내용**
- 변화하는 요구사항에 대응
- 동작 파라미터화
- 익명 클래스
- 람다 표현식 미리보기
- 실전 예제 : Comparator, Runnable, GUI

---

소비자의 요구사항은 항상 바뀐다. 대응 방식은

1. 엔지니어링적 비용을 최소화하면서 
2. 구현은 쉽고
3. 유지보수가 쉬워야 한다. 

동작 파라미터화가 하나의 방법!

- **동작 파라미터화(behavior parameterization):** 어떻게 실행할 것인지 아직 결정하지 않은 코드 블록. 이 코드 블록은 나중에 프로그램에서 호출.

    ex) 나중에 실행될 메서드의 인수로 코드 블록을 전달. → 코드 블록에 따라 메서드의 동작이 파라미터화됨.

## 2.1 변화하는 요구사항에 대응하기

```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory) {
		if ("green".equals(apple.getColor()) {
				result.add(apple);
		}
	}
	return result;
}
```

녹색 사과만 필터링하는 코드를 만들었는데 빨간 사과도 필터링하고 싶다면?

이러한 변화에 대응하기 위해 **'비슷한 코드를 구현한 다음에 추상화하라.'**

예를 들어, 색이나 무게를 파라미터화하여 보다 유연한 코드를 만들 수 있다.

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory) {
		if (apple.getWeight() > weight) {
				result.add(apple);
		}
	}
	return result;
}
```

사과의 색 필터링 코드와 코드 중복 발생. → **소프트웨어 공학의 DRY(don't repeat yourself) 원칙에 벗어남.**

색, 무게 등의 속성을 메서드 인수로 추가하고 boolean flag 등을 세운 코드 또한 명확하지 않을 뿐더러, 요구사항이 바뀌었을 때 유연하게 대응할 수 없다.

## 2.2 동작 파라미터화

- **프레디케이트(predicate)**: boolean을 리턴하는 함수형 인터페이스.
- **전략 디자인 패턴(strategy design pattern)**: 각 알고리즘(전략이라 불리는)을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법.

    ex) 알고리즘 패밀리: ApplePredicate (인터페이스)

    	전략: AppleGreenColorPredicate, AppleHeavyWeightPredicate 

- **동작 파라미터화**: 메서드가 다양한 동작(또는 전략)을 받아서 내부적으로 다양한 동작 수행 가능.
- 동작 파라미터화의 **강점**: 컬렉션 탐색 로직과 각 항목에 적용할 동작 분리 가능.
- 유연한 API를 만들려면 동작 파라미터화라는 도구가 반드시 필요하다.

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory) {
		if (p.test(apple)) {   // 프레디케이트 객체로 사과 검사 조건을 캡슐화함.
				result.add(apple);
		}
	}
	return result;
}
```

```java
public class AppleRedAndHeavyPredicate implements ApplePredicate {
	public boolean test(Apple apple) {
		return "red".equals(apple.getColor()) && apple.getWeight() > 150;	
	}	
}

List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
```

## 2.3 복잡한 과정 간소화

현재 filterApples 메서드로 새로운 동작을 전달하려면 ApplePredicate 인터페이스를 구현하는 여러 클래스를 정의한 다음에 인스턴스화해야 한다. 이는 상당히 번거로운 작업이며 시간 낭비다!

### 2.3.2 익명 클래스 사용

**익명 클래스(anonymous class)** 

- 자바의 지역 클래스(local class; 블록 내부에 선언된 클래스)와 비슷한 개념.
- 말 그대로 이름이 없는 클래스.
- 클래스의 선언과 인스턴스화를 동시에 수행 가능.
- 즉석에서 필요한 구현을 만들어서 사용 가능.
- 익명 클래스를 이용하면 코드의 양을 줄일 수 있음.

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
	public boolean test(Apple a) {
		return "red".equals(a.getColor());
	}	
});
```

```java
button.setOnAction(new EventHandler<ActionEvent>() {
	public void handle(ActionEvent event) {
		System.out.println("Wooo a click!");
	}
});
```

### 2.3.3 람다 표현식 사용

```java
List<Apple> result = 
	filterApples(inventory, (Apple apple) -> "red".equals(apple.getColor)));
```

람다 표현식을 통해 간결해졌다. 

- 동작 파라미터화와 값 파라미터화 비교

    값 파라미터화는 뻣뻣하고 장황한 반면에 동작 파라미터화는 유연한 것이 장점이다. 

    동작 파라미터화에서 클래스, 익명 클래스, 람다 순서로 더욱 간결해진다.

### 2.3.4 리스트 형식으로 추상화

현재 filterApples는 Apple과 관련한 동작만 수행한다.

리스트 형식으로 추상화하면 Apple 이외의 다양한 물건에서 필터링할 수 있다.

```java
public interface Predicate<T> {
	boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
	List<T> result = new ArrayList<>();
	for(T e: list) {
		if(p.test(e)) {
			result.add(e);
		}
	}
	return result;
}
```

## 2.4 실전 예제

동작 파라미터화 패턴은 동작을 (한 조각의 코드로) 캡슐화한 다음에 메서드로 전달해서 메서드의 동작을 파라미터화한다 (ex.사과의 다양한 프레디케이트).

### 2.4.1 Comparator로 정렬하기

```java
// java.util.Comparator
public interface Comparator<T> {
	public int compare(T o1, T o2);
}

inventory.sort(new Comparator<Apple>() {
	public int compare(Apple a1, Apple a2) {
		return a1.getWeight().compareTo(a2.getWeight());
	}
});
```

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

### 2.4.2 Runnable로 코드 블록 실행하기

각각의 스레드는 각기 다른 코드를 실행할 수 있다.  

자바에서는 Runnable 인터페이스를 이용해서 실행할 코드 블록을 지정할 수 있다.

```java
// java.lang.Runnable
public interface Runnable {
	public void run();
}

Thread t = new Thread(new Runnable() {
	public void run() {
		System.out.println("Hello world");
	}
});
```

```java
Thread t = new Thread(() -> System.out.println("Hello world"));
```

### 2.4.3 GUI 이벤트 처리하기

GUI 프로그래밍에서 마우스 클릭이나 이동 등의 이벤트에 모두 반응할 수 있어야 하기 때문에 변화에 대응 가능한 유연한 코드가 필요하다.

```java
Button button = new Button("Send");
button.setOnAction(new EventHandler<ActionEvent>() {
	public void handle(ActionEvent event) {
		label.setText("Sent!");
	}
});
```

```java
button.setOnAction((ActionEvent event) -> label.setText("Sent!"));
```

## 2.5 요약

- 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다.
- 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있으며 나중에 엔지니어링 비용을 줄일 수 있다.
- 코드 전달 기법을 이용하면 동작을 메서드의 인수로 전달할 수 있다. 하지만 자바 8 이전에는 코드를 지저분하게 구현해야 했다. 익명 클래스로도 어느정도 코드를 깔끔하게 만들 수 있지만 자바 8에서는 인터페이스를 상속받아 여러 클래스를 구현해야 하는 수고를 없앨 수 있는 방법을 제공한다.
- 자바 API의 많은 메서드는 정렬, 스레드, GUI 처리 등을 포함한 다양한 동작으로 파라미터화할 수 있다.

## 2020.12.27 줌 스터디로 논의했던 내용 추가

- 현업에서 사용 여부

    과거에 만든 것 아닌 이상, 최소 자바 1.8, 1.11 정도로 구현. 

    코드 리뷰 시, Stream API의 filter 메소드로 변환해서 사용하는 것을 추천한다. Stream 많이 쓴다. 

- Runnable

    불필요한 override 없이, Runnable 스레드에서 실행되어야 하는 코드만 작성해주면 된다.

- 동적 파라미터화에 대한 이해
    - 인터페이스나 다른 여러가지 방법을 사용해서 런타임 중에 파라미터가 바뀌더라도 동작하는 것? ok
    - 변화에 유연하게 대응하기 위함.
    - 함수로 넘길 수 있게 됨.
    - 동적 파라미터화가 람다로 가는 단계라고 보기 보다는,

        동적 파라미터화가 더 추상적인 개념이고 그 안에 람다가 있다고 보는 게 맞다.

        동적 파라미터화를 하기 위한 방법 중 하나가 람다!

        동작을 캡슐화한 다음에 메서드로 전달해서 메서드의 동작을 파라미터화한다 (ex. predicate).

- predicate

    java.util.function 패키지에 정의되어 있다. since 1.8. 함수형 인터페이스.

- @functionalInterface
    - since 1.8. 인터페이스 중에서 정의를 강제로 해야하는 메서드가 하나인 경우에 붙여줄 수 있는데, 이럴 경우 람다를 사용할 수 있다.
    - 개발의 편의를 위해 붙여주는 어노테이션. 컴파일러가 람다의 타입(단 하나의 추상 메서드를 가지고 있는지)을 추론할 수 있도록 정보를 제공하는 역할.
    - functionalInterface는 오직 하나의 추상 메서드를 가진 인터페이스.
    - 람다 표현식은 @FunctionalInterface의 구현체로 생성되는 오브젝트. 인터페이스의 구현체라고 볼 수 있다.
