# Chapter 3 람다 표현식

---

### Java 8 에서 함수형 프로그래밍 패러다임의 일환으로 람다 표현식 기능이 추가되었다

---

- 함수형 프로그래밍
    - 함수형 프로그래밍이란?
        - 순수한 함수를 작성하고, 공유된 상태와 변경 가능한 데이터 및 부작용을 피하여 소프트웨어를 작성하는 프로세스이다.
    - 함수형 프로그래밍 컨셉
        1. 변경 가능한 상태를 불변상태로 만들어 Side Effect 를 제거한다
        2. 모든 것은 객체이다
        3. 코드를 간결하게 하고 가독성을 높여 구현할 로직에 집중 시킨다
        4. 동시성 작업을 보다 쉽게 안전하게 구현한다
    - 조건
        1. 순수한 함수
            - 부작용이 없는 함수로 함수의 실행이 외부의 상태를 변경시키지 않는 함수가 정의되어야 한다
        2. 익명 함수
            - 이름이 없는 함수를 정의 할 수 있어야 한다
        3. 고계 함수
            - 함수를 다루는 상위의 함수로 함수형 언어에서는 함수도 하나의 값을 취급하고 함수의 인자로 함수를 전달할 수 있는 특성이 있어야 한다
    - 장점
        1. 높은 수준의 추상화를 제공한다
        2. 함수 단위의 코드 재사용이 수월하다
        3. 불변성을 지향하기 때문에 프로그램의 동작을 예측하기 쉬워진다

---

- 람다 표현식
    - 람다란?
        - 람다 표현식(lambda expressions)은 일급객체인 메서드를 전달 할 수 있는 익명 함수를 단순화 한 것이다.
        - 람다 표현식은 파라미터, 화살표, 바디로 이루어지고 화살표를 기준으로 왼쪽에는 파라미터 리스트, 오른쪽에는 함수에 내용에 해당하는 바디가 위치한다.
        - 쉽게 말해 메서드로 전달할 수 있는 익명 함수를 단순화한 것
        - (String 형식의 파라미터 하나를 가지고 int를 반환하는 예시)

            ```java
            (String s) -> s.length()
            ```

    - 사용 이유
        1. 코드를 전달하는 과정에서 자질구레한 코드가 많이 생긴다. 람다를 이용해서 간결한 방식으로 코드를 전달할 수 있다
        2. 동작 파라미터를 이용할 때 익명 클래스 등 판에 박힌 코드를 구현할 필요가 없다
        3. 결과적으로 코드가 간결하고 유연해진다
    - 어디에 어떻게 사용하는가?
        - 함수형 인터페이스(오직 하나의 추상 메서드만 가지는 Comparator, Runnable 과 같은 것들)라는 문맥에서 람다 표현식을 사용할 수 있다
        - 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다
    - 예시
        - 다음과 같은 Apple 클래스가 있을 때

            ```java
            public Class Apple{
            	private int weight = 0;
            	private Color color;

            	public Apple(){
            		this.weight = weight;
            		this.color = color;
            	}

            	//weight와 color 에 대한 getter,setter 존재
            }
            ```

        - 1단계 코드전달

            ```java
            public class Sorting{
            	public static void main(String... args){
            		List<apple> inventory = new ArrayList<>();
            		inventory.addAll(Arrays.asList(
            			new Apple(80,Color.GREEN),
            			new Apple(155,Color.green),
            			new Apple(120,Color.RED)
            		));

            		inventory.sort(new AppleComparator());
            	}

            	static class AppleComparator implements Comparator<Apple> {
            		@Override
            		public int compare(Apple a1, Apple a2){
            			return a1.getWeight() - a2.getWeight();
            		}
            	}
            }
            ```

        - 2단계 익명 클래스 사용

            ```java
            public class Sorting{
            	public static void main(String... args){
            		List<apple> inventory = new ArrayList<>();
            		inventory.addAll(Arrays.asList(
            			new Apple(80,Color.GREEN),
            			new Apple(155,Color.green),
            			new Apple(120,Color.RED)
            		));

            		inventory.sort(new Comparator<Apple>(){
            			@Override
            			public int compare(Apple a1,Apple a2){
            				return a1.getWeight() - a2.getWeight();
            			}
            		});
            	}
            }
            ```

        - 3단계 람다 표현식 사용

            ```java
            public class Sorting{
            	public static void main(String... args){
            		List<apple> inventory = new ArrayList<>();
            		inventory.addAll(Arrays.asList(
            			new Apple(80,Color.GREEN),
            			new Apple(155,Color.green),
            			new Apple(120,Color.RED)
            		));

            		inventory.sort( (a1,a2) -> a1.getWeight() - a2.getWeight() );
            			
            	}
            }
            ```

        - 4단계 메서드 참조 사용

            ```java
            public class Sorting{
            	public static void main(String... args){
            		List<apple> inventory = new ArrayList<>();
            		inventory.addAll(Arrays.asList(
            			new Apple(80,Color.GREEN),
            			new Apple(155,Color.green),
            			new Apple(120,Color.RED)
            		));

            		inventory.sort( comparing(Apple::getWeight) );
            	}
            }
            ```

---

- 메서드 참조
    - 메서드 참조는 람다 표현식이 단 하나의 메소드만을 호출하는 경우에 해당 람담 표현식에서 불필요한 매개변수를 제거하고 사용할 수 있도록 해준다
    - :: 기호를 사용하여 표현한다
    - 예시

    ```java
    (Apple apple) -> apple.getWeight()
    Apple::getWeight

    (String s) -> System.out.println(s) (String s) -> this.isValidName(s)
    System.out::println
     this::isValidName
    ```

---

- 기타 내용
    - Comparator, Predicate, Function 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있는 다양한 디폴트 메서드를 제공한다
    - 람다와 동작 파라미터를 활용한 실행 어라운드 패턴을 람다와 활용하면 유연성과 재사용성을 추가로 얻을 수 있다
    - 기본형 특화 인터페이스와 언박싱
    - 생성자 참조
    - 유효성 검사 방법
    - 실행 어라운드 패턴(실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는 형식의 코드)은 람다와 동작 파라미터화를 사용한 실용적인 예제중에 하나이다

---

- 질문
    - @FunctionalInterface 사용에 대한 질문
    함수형 인터페이스임을 표시하는 용도로 쓰이는 어노테이션 이다
    - 박싱과 언박싱에 대한 질문
    프리미티브 타입의 값을 Wrapper 클래스로 바꾸는 것이 박싱

        ```java
        int a = 10;

        //boxing
        Integer b = new Integer(a);

        //unboxing
        int c = b.intValue();
        ```

    - 실행 어라운드 패턴 이해가 어렵다는 질문
    람다와 동작 파라미터화를 사용한 실용적인 예제중에 하나이다
