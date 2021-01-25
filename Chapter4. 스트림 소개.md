## 4.1 스트림이란 무엇인가?

일단 데이터 컬렉션 반복을 멋지게 처리하는 기능, 병렬 처리 가능하게 끔

```java
List<String> lowCaloricDishesName =
							menu.stream() // menu.paralleStream()
									.filter(d -> d.getCal()<400) // 400이하 선택
									.sorted(comparing(Dish::getCal)) // 칼로리 정렬
									.map(Dish::getName) // 요리명 추출
									.collect(toList()); // 모든 요리명 리스트에
```

- 선언형: if... 구현할 필요없이 '저칼로리의 요리만 선택하라' 동작 지정.
- 파이프라인: 복잡한 처리 파이프라인 가능. filter 결과는 sorted로, ..., ...

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a2dced8-997a-4c4c-bb72-07d926e32d41/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a2dced8-997a-4c4c-bb72-07d926e32d41/Untitled.png)

## 4.2 스트림 시작하기

**스트림**: 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소

- 연속된 요소: 연속된 값 집합의 인터페이스 제공
- 소스: 제공 소스로부터 데이터 소비, 리스트로 스트림 → 스트림 요소도 리스트 요소와 같은 순서
- 데이터 처리 연산:  filter, map, reduce, find, match, sort로 데이터 조작, 순차 or 병렬
- **파이프라이닝:** 스트림 연산끼리 연결하여 파이프라인 구성, 게으름과 쇼트서킷 최적화 얻음(5장)
- **내부 반복**
- filter: d→d.getCal()>300
- map: d→d.getName(), 요리명 추출
- limit: 스트림 크기 축소
- collect: 스트림 형식 변환

## 4.3 스트림과 컬렉션

*스트림은 게으르게 만든 컬렉션*

사용자가 데이터를 요청할 때만 값 계산

### 4.3.1 딱 한 번만 탐색할 수 있다!

탐색된 스트림의 요소는 소비된다.

```jsx
List<String> title = Arrays.asList("Java8", "In", "Action");
Stream<String> s = title.stream();
s.forEach(System.out::println);
s.forEach(System.out::println); // 스트림이 소비or닫힘
```

### 4.3.2 외부 반복과 내부 반복

컬렉션(외부 반복), 스트림(내부 반복)

```jsx
List<String> names = new ArrayList<>();
for(Dish d: menu){
		names.add(d.getName());
}

List<String> names = menu.stream().
												 .map(Dish::getName)
												 .collect(toList());
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/85ce870c-7144-4c69-9415-6eefe38373ba/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/85ce870c-7144-4c69-9415-6eefe38373ba/Untitled.png)

## 4.4 스트림 연산

- 중간 연산: filter, map, limit
- 최종 연산: collect

### 4.4.1 중간 연산

output이 스트림

### 4.4.2 최종 연산

output이 스트림 이외의 결과

### 4.4.3 스트림 이용하기

---

## 4.5요약

- 스트림은 소스에서 추출된 연속 요소로, 데이터 처리 연산을 지원한다.
- 스트림은 내부 반복을 지원한다. 내부 반복은 filter, map, sorted 등의 연산으로 반복을 추상화한다.
- 스트림에는 중간 연산과 최종 연산이 있다.
- filter와 map처럼 스트림을 반환하면서 다른 연산과 연결될 수 있는 연산을 중간 연산이라고 한다. 중간 연산을 이용해서 파이프라인을 구성할 수 있지만 중간 연산으로는 어떤 결과도 생성할 수 없다.
- forEach나 count처럼 스트림 파이프라인을 처리해서 스트림이 아닌 결과를 반환하는 연산을 최종 연산이라고 한다.
- 스트림의 요소는 요청할 때만 계산된다.

---

## 연습

출처: [https://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/](https://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/)

베이스 코드

```java
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

class Person{
    String name;
    int age;
    
    Person(String name, int age){
        this.name = name;
        this.age = age;
    }
    @Override
    public String toString(){
        return name;
    }
}
class Main {  
  public static void main(String args[]) { 
    // System.out.println("Hello, world!"); 
    List<Person> persons =
        Arrays.asList(
            new Person("Max", 18),
            new Person("Peter", 23),
            new Person("Pamela", 23),
            new Person("David", 12));  

		List<Person> filtered = 
		...
		...
  } 
}
```

1. 이름이 P 로 시작하는 사람만 filter하기

```java
List<Person> filtered = 
        persons
            .s
						.f
						.c
    System.out.println(filtered);
```

1. 정답

```java
// 1. 이름이 P 로 시작하는 사람만 출력하기
    List<Person> filtered = 
        persons
            .stream()
            .filter(p->p.name.startsWith("P"))
            .collect(Collectors.toList());
    System.out.println(filtered);
```

2. 같은 나이의 사람으로 그룹화하기 [힌트 → Collectors.groupingBy() ]

```java
Map<Integer, List<Person>> personsByAge = persons
        .stream()
        .collect(        그룹핑바이                );
		personsByAge
        .forEach((   ,  ) -> System.out.format("age %s: %s\n", age, p));
```

2. 정답

```java
// 같은 나이의 사람으로 그룹화하기
    Map<Integer, List<Person>> personsByAge = persons
        .stream()
        .collect(Collectors.groupingBy(p->p.age));
		personsByAge
        .forEach((age, p) -> System.out.format("age %s: %s\n", age, p));
    
```

3. 18세 이상인 사람은?

```java
String phrase = persons
        .
				.
				.
        .collect(Collectors.joining(" and ", "In Germany ", " are of legal age."));
        
    System.out.println(phrase);
```

```java
// 독일에서 성인은?    
    String phrase = persons
        .stream()
        .filter(p-> p.age >= 18)
        .map(p->p.name)
        .collect(Collectors.joining(" and ", "In Germany ", " are of legal age."));
        
    System.out.println(phrase);
```

4. Extra Problem

```java
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.stream.Collectors;

class NewBie {
    private String name;
    private String department;
    private int id;

    public NewBie(String name, String department, int id) {
        this.name = name;
        this.department = department;
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public int getId() {
        return id;
    }

    public String getDepartment() {
        return department;
    }

    @Override
    public String toString() {
        return String.format("name : %s - dep : %s - id : %d", getName(), getDepartment(), 
                getId());
    }
}

public class Main {

    public static void main(String[] args) {

        // Q1 : 4개의 단어로부터 문자들을 추출하고, 이 문자들이 고유하게 정렬된 리스트로 만들기
        String[] strArr = {"Hello", "World", "Home", "Study"};

        // TODO : 옆에 반환되는 자료구조를 써봅시다
        List<String> result = Arrays.stream(strArr)         // stream<String>
                .map(word -> word.split(""))            // stream<String[]>
                .flatMap(Arrays::stream)        // stream<String>
                .map(String::toLowerCase)
                .distinct()     // stream<String>
                .sorted()
                .collect(Collectors.toList());
        // output : d e h l m o r s t u w y
        System.out.println(result);

        // Q2, Q3, Q4 : 주어진 문제 풀기
        List<NewBie> newBies = Arrays.asList(
                new NewBie("jinwoo", "finding", 123),
                new NewBie("hyukjin", "finding", 153),
                new NewBie("jiwon-kang", "finding", 142),
                new NewBie("jiwon-kwon", "user-account", 125),
                new NewBie("dongmin", "SO", 221),
                new NewBie("kyubum", "rm", 266),
                new NewBie("juwon", "pdp", 177),
                new NewBie("hyejin", "escrow", 193)
        );

        int ans2 = 0; // Q2 TODO : 사번 160이상인 사람들의 사번의 합
        ans2 = newBies
            .stream()
            .filter(n->n.getId()>=160)
            .map(n->n.getId())
            .reduce(0, Integer::sum);
        System.out.println("Q2: " + ans2);

        String ans3 = "";       // Q3 TODO : 가장 높은 사번의 사람 이름 출력
        ans3 = newBies
            .stream()
            .max(Comparator.comparing(n->n.getId()))
            .map(n->n.getName())
            .get();
            // .orElse(Integer.MIN_VALUE);
        System.out.println("Q3: " + ans3);

        
        long ans4 = 0;      // Q4 TODO : 부서가 finding이 아닌 사람들 카운트
        ans4 = newBies
            .stream()
            .filter(n-> "finding".equals(n.getDepartment()))
            .count();
        System.out.println("Q4: " + ans4);
    }
}
```
