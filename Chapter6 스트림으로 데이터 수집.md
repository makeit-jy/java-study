# Chapter 6 스트림으로 데이터 수집

**collect, Collector, Collection  차이**

- collect(): Collector를 매개변수로 하는 스트림의 최종 연산.
- Collector: collect에서 필요한 메서드를 정의해 놓은 인터페이스.
- Collectors 클래스: 다양한 기능의 Collector를 구현한 클래스를 제공.
- Collection: ex) List, Set

## 6.1 컬렉터란 무엇인가?

```java
// 그룹화한 트랜잭션을 저장할 맵을 생성.
Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>();

for(Transaction transaction : transactions) {
    Currency currency = transaction.getCurrency();  // 트랜잭션의 통화 추출.
    List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);

    // 현재 통화를 그룹화하는 맵에 항목이 없으면 항목을 만든다.
    if(transactionsForCurrency == null) {
        transactionsForCurrency = new ArrayList<>();
        transactionsByCurrencies.put(currency, transactionsForCurrency);
    }
    // 같은 통화를 가진 트랜잭션 리스트에 현재 탐색 중인 트랙잭션 추가.
    transactionsForCurrency.add(transaction);
}
```

명령형 버전

- **어떤** 방법으로 원하는 것을 얻을 수 있을지 코드로 구현해야 함.
- 문제를 해결하는 과정에서 다중 루프와 조건문을 추가하여 가독성과 유지보수성이 크게 떨어짐.

```java
Map<Currency, List<Transaction>> transactionsByCurrencies =
    transactions.stream().collect(groupingBy(Transaction::getCurrency));
```

함수형 버전

- '**무엇**'을 원하는지 직접 명시 가능.
- 필요한 컬렉터를 쉽게 추가할 수 있다. (6.3절 참조)
- 높은 수준의 조합성과 재사용성이 함수형 API의 장점.

### 6.1.1 고급 리듀싱 기능을 수행하는 컬렉터

![Chapter%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%2020961862e2ac4ec59c9d5fa5f7fcd8cc/1____.jpg](Chapter%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%2020961862e2ac4ec59c9d5fa5f7fcd8cc/1____.jpg)

[그림 1] 통화별로 트랜잭션을 그룹화하는 리듀싱 연산

- 스트림에 collect를 호출하면 스트림의 요소에 (컬렉터로 파라미터화된) 리듀싱 연산이 수행됨.
- collect에서는 리듀싱 연산을 이용해서 스트림의 각 요소를 방문하면서 컬렉터가 작업을 처리함.
- 함수를 요소로 변환할 때는 컬렉터를 적용하며 최종 결과를 저장하는 자료구조에 값을 누적함.

### 6.1.2 미리 정의된 컬렉터

**Collectors 클래스에서 제공하는 메서드의 기능**

- 스트림 요소를 하나의 값으로 리듀스하고 요약
- 요소 그룹화
- 요소 분할

## 6.2 리듀싱과 요약

- 컬렉터로 스트림의 모든 항목을 하나의 결과로 합칠 수 있다.

```java
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

![Chapter%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%2020961862e2ac4ec59c9d5fa5f7fcd8cc/2summingInt___.jpg](Chapter%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%2020961862e2ac4ec59c9d5fa5f7fcd8cc/2summingInt___.jpg)

[그림 2] summingInt 컬렉터의 누적 과정

칼로리로 매핑된 각 요리의 값을 탐색하면서 초깃값(여기서는 0)으로 설정되어 있는 누적자에 칼로리를 더한다.

- 다른 형식들도 존재
    - summingLong, summingDouble
    - averagingInt, averagingLong, averagingDouble

### 6.2.3 문자열 연결

컬렉터에 joining 팩토리 메서드를 이용하면,

스트림의 각 객체에 toString 메서드를 호출 → 추출한 모든 문자열을 하나의 문자열로 연결해서 반환.

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
```

```java
pork, beef, chicken, french fries, rice, season fruit, pizza, prawns, salmon
```

### 6.2.4 범용 리듀싱 요약 연산

collect와 reduce

collect: 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드.

reduce: 두 값을 하나로 도출하는 불변형 연산.

```java
Stream<Integer> stream = Arrays.asList(1, 2, 3, 4, 5, 6).stream();
List<Integer> numbers = stream.reduce(
                            new ArrayList<Integer>(),
                            (List<Integer> l, Integer e) -> {
                              l.add(e);
                              return l;
                            },
                            (List<Integer> l1, List<Integer> l2) -> {
                              l1.addAll(12);
                              return l1;
                            });
```

- 여러 스레드가 동시에 같은 데이터 구조체를 고치면 리스트 자체가 망가짐 → 리듀싱 연산을 병렬로 수행 불가능.
- 이 문제를 해결하려고 매번 새로운 리스트를 할당한다면 성능이 저하됨.
- 가변 컨테이너 관련 작업이면서 병렬성을 확보하려면 **collect** 메서드로 리듀싱 연산을 구현하는 것이 바람직함.

### 자신의 상황에 맞는 최적의 해법 선택

- 스트림 인터페이스에서 직접 제공하는 메서드를 이용하는 것에 비해 컬렉터를 이용하는 코드가 더 복잡하다.
- 코드가 좀 더 복잡한 대신 재사용성과 커스터마이즈 가능성을 제공하는 높은 수준의 추상화와 일반화를 얻을 수 있다.

## 6.3 그룹화

팩토리 메서드 Collectors.groupingBy를 이용해서 쉽게 그룹화 할 수 있다.

```java
Map<Dish.Type, List<Dish>> dishesByType =
    menu.stream().collect(groupingBy(Dish::getType));
```

```java
{FISH=[pawns, salmon], OTHER=[french fries, rice, season fruit, pizza], MEAT=[pork, beef, chicken]}
```

![Chapter%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%2020961862e2ac4ec59c9d5fa5f7fcd8cc/3____.jpg](Chapter%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%2020961862e2ac4ec59c9d5fa5f7fcd8cc/3____.jpg)

[그림 3] 그룹화로 스트림의 항목을 분류하는 과정

- 단순한 속성 접근자 대신 더 복잡한 분류 기준이 필요한 상황에서는 메서드 레퍼런스를 분류함수로 사용할 수 없다. (메서드 레퍼런스 대신 **람다 표현식** 사용)

```java
public enum CaloricLevel { DIET, NORMAL, FAT }

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(
  groupingBy(dish -> {
      if (dish.getCalories() <= 400) {
        return CaloricLevel.DIET;
      } else if (dish.getCalories() <= 700) {
        return CaloricLevel.NORMAL;
      } else {
        return CaloricLevel.FAT;
      }
  })
);
```

### 6.3.2 다수준 그룹화

- Collectors.groupingBy는 일반적인 분류함수와 컬렉터를 인수로 받는다.

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel =
  menu.stream().collect(
      groupingBy(Dish::getType,   // 첫 번째 수준의 분류 함수
          groupingBy(dish -> {    // 두 번째 수준의 분류 함수
              if (dish.getCalories() <= 400) {
                return CaloricLevel.DIET;
              } else if (dish.getCalories() <= 700) {
                return CaloricLevel.NORMAL;
              } else {
                return CaloricLevel.FAT;
              }
          })));
```

```java
{MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork]}, FISH={DIET=[pawns], NORMAL=[salmon]}, OTHER={DIET=[rice, season fruit], NORMAL=[french fries, pizza]}}
```

- 외부 맵은 첫 번째 수준의 분류 함수에서 분류한 키 값 'fish, meat, other'를 갖는다.
- 외부 맵의 값은 두 번째 수준의 분류 함수의 기준 'normal, diet, fat'을 키 값으로 갖는다.
- 최종적으로 두 수준의 맵은 첫 번째 키와 두 번째 키의 기준에 부합하는 요소 리스트 값으로 갖는다.

![Chapter%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%2020961862e2ac4ec59c9d5fa5f7fcd8cc/4n___n_.jpg](Chapter%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%2020961862e2ac4ec59c9d5fa5f7fcd8cc/4n___n_.jpg)

[그림 4] n수준 중첩 맵과 n차원 분류표

groupingBy의 연산을 버킷(bucket, 물건을 담을 수 있는 양동이) 개념으로 생각하면 쉽다.

- 첫 번째 groupingBy는 각 키의 버킷을 만든다.
- 그리고 준비된 각각의 버킷을 서브스트림 컬렉터로 채워가기를 반복하면서 n수준 그룹화를 달성한다.

## 6.4 분할

- 분할은 분할 함수(partitioning function)라 불리는 **프레디케이트**를 분류 함수로 사용하는 특수한 그룹화 기능.
- 불리언을 반환하므로 맵의 키 형식은 Boolean.
- 결과적으로 그룹화 맵은 최대 두 개의 그룹(true/false)으로 분류됨.

```java
Map<Boolean, List<Dish>> partitionedMenu =
  menu.stream().collect(partitioningBy(Dish::isVegetarian));  // 분할 함수
```

```java
{false=[pork, beef, chicken, prawns, salmon], true=[french fries, rice, season fruit, pizza]}
```

### 6.4.1 분할의 장점

분할 함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지한다는 것이 분할의 장점.

```java
Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType =
  menu.stream().collect(
      partitioningBy(Dish::isVegetarian,  // 분할 함수
          groupingBy(Dish::getType)));    // 두 번째 컬렉터
```

```java
{false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]}, true={OTHER=[french fries, rice, season fruit, pizza]}}
```

## 6.5 Collector 인터페이스

Collector 인터페이스는 리듀싱 연산(즉, 컬렉터)을 어떻게 구현할지 제공하는 메서드 집합으로 구성된다.

```java
public interface Collector<T, A, R> {
  Supplier<A> supplier();
  BiConsumer<A, T> accumulator();
  Function<A, R> finisher();
  BinaryOperator<A> combiner();
  Set<Characteristics> characteristics();
}
```

- T는 수집될 스트림 항목의 제네릭 형식.
- A는 누적자, 즉 수집과정에서 중간 결과를 누적하는 객체의 형식.
- R은 수집 연산 결과 객체의 형식(항상 그런것은 아니지만 대게 컬렉션 형식).

### 6.5.1 Collector 인터페이스의 메서드 살펴보기

- characteristics 메서드는 collect 메서드가 어떤 최적화(예를 들면 병렬화 같은)를 이용해서 리듀싱 연산을 수행할 것인지 결정하도록 돕는 힌트 특성 집합을 제공.
- 그 외 4개의 메서드는 collect 메서드에서 실행하는 함수를 반환.

### supplier 메서드: 새로운 결과 컨테이너 만들기

- supplier는 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수.

```java
public Supplier<List<T>> supplier() {
  return () -> new ArrayList<T>();
}
```

### accumulator 메서드: 결과 컨테이너에 요소 추가하기

- 리듀싱 연산을 수행하는 함수를 반환.
- 스트림에서 n번째 요소를 탐색 할 때 누적자(스트림의 첫 n-1개 항목을 수집한 상태)와 n 번째 요소를 함수에 적용.

```java
ublic BiConsumer<List<T>, T> accumulator() {
  return (list, item) -> list.add(item);
}
```

### finisher 메서드: 최종 변환값을 결과 컨테이너로 적용하기

- 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환하면서 누적 과정을 끝낼 때 호출할 함수를 반환.
- 누적자 객체가 이미 최종 결과인 경우 변환 과정이 필요하지 않으므로 finisher 메서드는 항등 함수를 반환.

```java
public Function<List<T>, List<T>> finisher() {
  return Function.identity();
}
```

### combiner 메서드: 두 결과 컨테이너 병합

- 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할지 정의.
- 스트림의 리듀싱을 병렬로 수행할 때 자바 7의 포크/조인 프레임워크와 7장에서 배울 Spliterator를 사용.

```java
public BinaryOperator<List<T>> combiner() {
  return (list1, list2) -> {
    list1.addAll(list2);
    return list;
  }
}
```

### Characteristics 메서드

- 컬렉터의 연산을 정의하는 Characteristics 형식의 불변 진합을 반환.
- Characteristics는 스트림을 병렬로 리듀스할 것인지 그리고 병렬로 리듀스한다면 어떤 최적화를 선택해야 할지 힌트를 제공.
    - UNORDERED : 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않는다.
    - CONCURRENT : 다중 스레드에서 accumulator 함수를 동시에 호출할 수 있으며 이 컬렉터는 스트림의 병렬 리듀싱을 수행할 수 있다.
        - UNORDERED를 함께 설정하지 않았다면 데이터소스가 정렬되어 있지 않은 상황에서만 병렬 리듀싱을 수행할 수 있다.
    - IDENTITY_FINISH : finisher 메서드가 반환하는 함수는 단순히 identity를 적용할 뿐이므로 이를 생략할 수 있다.
        - 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용할 수 있다.
        - 누적자 A를 결과 R로 안전하게 형변환 할 수 있다.

## 6.6 커스텀 컬렉터를 구현해서 성능 개선하기

커스텀 컬렉터로 n까지의 자연수를 소수와 비소수로 분할

```java
public Map<Boolean, List<Integer>> partitionPrimes(int n) {
  return IntStream.rangeClosed(2, n).boxed()
                  .collect(partitioningBy(candidate -> isPrime(candidate));
}
```

제곱근 이하로 대상(candidate)의 숫자 범위를 제한해서 isPrime 개선

```java
public boolean isPrime(int candidate) {
  int candidateRoot = (int) Math.sqrt((double) candidate);
  return IntStream.rangeClosed(2, candidateRoot)
                  .noneMatch(i -> candidate % i == 0);
}
```

Collector 인터페이스에서 요구하는 메서드 다섯 개

### 1단계: Collector 클래스 시그너처 정의

```java
public interface Collector<T, A, R>
```

- T : 스트림 요소의 형식
- A : 중간 결과를 누적하는 객체의 형식
- R : collect 연산의 최종 결과 형식

```java
public class PrimeNumbersCollector
        implements Collector<Integer,   // 스트림 요소의 형식
                            Map<Boolean, List<Integer>>,    // 누적자 형식
                            Map<Boolean, List<Integer>>>    // 수집 연산의 결과 형식
```

### 2단계: 리듀싱 연산 구현

- supplier 메서드는 누적자를 만드는 함수를 반환해야 한다.

```java
public Supplier<Map<Boolean, List<Integer>>> supplier() {
  return () -> new HashMap<Boolean, List<Integer>>() {{
    pub(true, new ArrayList<Integer>());
    put(false, new ArrayList<Integer>());
  }};
}
```

- 누적자로 사용할 맵을 만들면서 true, false 키와 빈 리스트로 초기화.
- accumulator 메서드에서 스트림의 요소를 어떻게 수집할지 결정.
    - accumulator는 최적화의 핵심.

### 3단계: 병렬 실행할 수 있는 컬렉션 만들기(가능하다면)

- 병렬 수집 과정에서 두 부분 누적자를 합칠 수 있는 메서드를 만든다.

```java
public BinaryOperator<Map<Boolean, List<Integer>>> combiner() {
  return (Map<Boolean, List<Integer>> map1,
          Map<Boolean, List<Integer>> map2) -> {
          map1.get(true).addAll(map2.get(true));
          map1.get(false).addAll(map2.get(false));
          return map1;
          }
}
```

### 4단계: finisher 메서드와 컬렉터의 characteristics 메서드

```java
public Function<Map<Boolean, List<Integer>>,
                Map<Boolean, List<Integer>>> finisher() {
  return Function.identity();
}
```

커스텀 컬렉터는 CONCURRENT도 아니고 UNORDERED도 아니지만 IDENTITY_FINISH 이므로 다음처럼 구현 가능.

```java
public Set<Characteristics> characteristics() {
  return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH));
}
```

## 6.7 요약

- collect는 스트림의 요소를 요약 결과로 누적하는 다양한 방법(컬렉터라 불리는)을 인수로 갖는 최종 연산이다.
- 스트림의 요소를 하나의 값으로 리듀스하고 요약하는 컬렉터뿐 아니라 최솟값, 최댓값, 평균값을 계산하는 컬렉터 등이 미리 정의되어 있다.
- 미리 정의된 컬렉터인 groupingBy로 스트림의 요소를 그룹화하거나, partitioningBy로 스트림의 요소를 분할할 수 있다.
- 컬렉터는 다수준의 그룹화, 분할, 리듀싱 연산에 적합하게 설계되어 있다.
- Collector 인터페이스에 정의된 메서드를 구현해서 커스텀 컬렉터를 개발할 수 있다.