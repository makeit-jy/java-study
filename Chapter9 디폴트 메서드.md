# Chapter9 디폴트 메서드

[https://programmers.co.kr/learn/courses/5/lessons/241](https://programmers.co.kr/learn/courses/5/lessons/241)

- **개요**

    **자바 8에서는 기본 구현을 포함하는 인터페이스를 정의하는 두 가지 방법 제공**

    ### 1. 인터페이스 내부에 **정적 메서드**를 사용하는 방법

    - 정적 메서드와 인터페이스

        보통 인터페이스, 인터페이스의 인스턴스를 활용할 수 있는 유틸리티 클래스를 활용한다.
        (유틸리티 클래스 : 다양한 정적 메서드를 정의하는 클래스)
        ex) Collections : Collection 객체를 활용할 수 있는 유틸리티 클래스

        자바 8에서는 인터페이스에 직접 정적 메서드를 선언할 수 있으므로 유틸리티 클래스를 없애고 직접 인터페이스 내부에 정적 메서드를 구현할 수 있다. 그럼에도 불구하고 과거 버전과 호환성을 유지할 수 있도록 자바 API에는 유틸리티 클래스가 남아있다.

    ### **2. 디폴트 메서드라는 기능을 사용,  인터페이스 기본 구현을 제공**

    - 즉 자바 8에서는 메서드 구현을 포함하는 인터페이스를 정의할 수 있다.
    - 기존의 코드 구현을 바꾸도록 강요하지 않으면서도 인터페이스를 바꿀 수 있다.

    ![Chapter9%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%2035d9daff81274d7f8cbf81eeb6e345ff/Untitled.png](Chapter9%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%2035d9daff81274d7f8cbf81eeb6e345ff/Untitled.png)

    - 순서
        1. API가 바뀌면서 어떤 문제가 생기는지
        2. 디폴트 메서드란 무엇이며 API가 바뀌면서 발생한 문제를 디폴트 메서드로 어떻게 해결할 수 있는지 설명
        3. 디폴트 메서드를 만들어 **다중 상속**을 달성하는 방법
        4. 같은 시그니처를 갖는 여러 디폴트 메서드를 상속받으면서 발생하는 모호성 문제를 자바 컴파일러가 어떻게 해결하는지 살펴본다.

- **진화하는 API가 호환성을 유지하는 방법**

    ### 9.1 변화하는 API

    ### **바이너리 호환성, 소스 호환성, 동작 호환성**

    https://blogs.oracle.com/darcy/kinds-of-compatibility:-source,-binary,-and-behavioral

    - 바이너리 호환성
    새로 추가된 메서드를 호출하지만 않으면 새로운 메서드 구현이 없이도 기존 클래스파일 구현이 잘 동작함
    - 소스 호환성
    코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일할 수 있음을 의미함, 예를 들어 인터페이스에 메서드를 추가하면 소스 호환성이 아니다. 추가한 메서드를 구현하도록 클래스를 고쳐야 하기 때문
    - 동작 호환성
    코드를 바꾼다음에도 같은 입력값이 주어지면 프로그램이 같은 동작을 실행함, 예를 들어 인터페이스에 메서드를 추가하더라도 프로그램에서 추가된 메서드를 호출할 일은 없으므로 동작 호환성은 유지 가능

    ### API가 바뀐다면?

    책의 예시 : 인기 있는 자바 그리기 라이브러리 설계자! 바로 너

    - 초기 ver1

        ```java
        public interface Resizable extends Drawable {
        	int getWidth();
        	int getHeight();
        	void setWidth(int width);
        	void setHeight(int height);
        	void setAbsoluteSize(int width, int height);
        }
        ```

    나의 라이브러리를 사용한 누군가의 클래스들 ....

    - 사용자 구현

        ```java
        public class Ellipse implements Resizable {
        	...
        }
        ```

    - 릴리즈 ver2
        - + setRelativeSize

        ```java
        public interface Resizable extends Drawable {
        	int getWidth();
        	int getHeight();
        	void setWidth(int width);
        	void setHeight(int height);
        	void setAbsoluteSize(int width, int height);
        	**void setRelativeSize(int wFactor, int hFactor);**
        }
        ```

    ### 사용자가 겪는 문제

    - **바이너리 호환성 및 동작 호환성은 유지**된다.
    (새로 추가된 메서드 호출하지 않으면 잘 동작)
    그러나 누군가 setRelativeSize를 사용하게 되면 **런타임 에러 발생**
    - **소스 호환성은 유지되지 않는다.** 
    **재빌드 시 컴파일 에러 발생, 공개된 API를 고치면 기존 버전과의 호환성 문제 발생**
    → 디폴트 메서드를 이용하여 API를 바꾸면 자동으로 기본 구현을 제공하므로 기존 코드를 고치지 않아도 된다. (이 장의 핵심)
- 디폴트 메서드란 무엇인가

    ### 9.2 디폴트 메서드란 무엇인가?

    **자바 8에서 호환성을 유지하면서 API를 바꿀 수 있도록 새로운 기능인 디폴트 메서드를 제공**

    ```java
    public interface Sized {
    	int size();
    	//디폴트 메서드
    	default boolean isEmpty() {
    		return size() == 0;
    	}
    }
    ```

    - 걱정 및 의문
        - 결국 자바도 다중 상속을 지원하는 걸까?
        (인터페이스가 구현을 가질 수 있고 
        클래스는 여러 인터페이스를 동시에 구현할 수 있으므로)
        - 인터페이스를 구현하는 클래스가 디폴트 메서드와 같은 메서드 시그니처를 정의하거나 아니면 디폴트 메서드를 오버라이드한다면 어떻게 될까?
    - 자바 8 API에서 디폴트 메서드 활용 예
        - Collection 인터페이스의 stream 메서드
        - List 인터페이스의 sort 메서드
        - Predicate의 and, Function의 andThen

    ### 추상 클래스와 자바 8의 인터페이스

    같은점

    - 둘 다 추상 메서드와 바디를 포함하는 메서드를 정의할 수 있다.

    다른점

    - 클래스는 하나의 추상 클래스만 상속받을 수 있지만 인터페이스를 여러개 구현할 수 있다.
    - 추상 클래스는 인스턴스 변수(필드)로 공통 상태를 가질 수 있다.?
    하지만 인터페이스는 인스턴스 변수를 가질 수 없다.
    - 면접 질문으로 연습했던 추상클래스와 인터페이스의 차이
        1. **상태와 행위를 담는 것의 차이**
        추상클래스는 부모클래스(슈퍼클래스)의 기능을 이용하거나 확장하기 위해서 사용
        반면 인터페이스는 동일한 동작을 약속하기 위해서 사용
        2. **다중 상속의 차이**
        자바는 다중 상속을 금지. 다중 상속의 모호성 때문.
        → 추상클래스는 단일 상속만 가능
        반면 인터페이스는 여러 개의 인터페이스를 구현할 수 있다.
        3. **외형적 차이**
        클래스는 일반 클래스와 추상 클래스로 나누어짐.
        추상 클래스는 클래스 내 '추상 메서드'가 하나 이상 포함되거나 abstract로 정의된 경우를 말함.
        반면 인터페이스는 모든 메소드가 '추상 메서드'인 경우.

- 디폴트 메서드의 활용 패턴

    ### 9.3 디폴트 메서드 활용 패턴

    1. 선택형 메서드 (Optional method)
        - 자바 8 이전 : 메서드의 내용이 비어있는 상황이 많았다.(빈 구현을 제공)
        - 자바 8 이후 : 메서드에 기본 구현을 제공하였다.
        - 예

            ```java
            interface Iterator<T>{
            	boolean hasNext();
            	T next();
            	default void remove() {
            		throw new UnsupportedOperationException();
            	}
            }
            ```

    2. 동작 다중 상속 (Multiple inheritance of behavior)
        - 디폴트 메서드를 이용하면 기존에 불가능했던 동작 다중 상속 기능도 구현 가능
        - 다중 상속 형식

            ```java
            public class ArrayList<E> extends AbstractList<E> 
            												  implements List<E>, RandomAccess, Cloneable,
            																		 Serilizable, Iterable<E>, Collection<E> {
            }
            ```

            한 개의 클래스를 상속 받고, 여섯 개의 인터페이스를 구현한다.

        - 기능이 중복되지 않는 최소의 인터페이스
            - Rotatable,  Moveable, Resizable
        - 인터페이스 조합

        ![Chapter9%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%2035d9daff81274d7f8cbf81eeb6e345ff/Untitled%201.png](Chapter9%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%2035d9daff81274d7f8cbf81eeb6e345ff/Untitled%201.png)

        ![Chapter9%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%2035d9daff81274d7f8cbf81eeb6e345ff/Untitled%202.png](Chapter9%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%2035d9daff81274d7f8cbf81eeb6e345ff/Untitled%202.png)

        - 옳지 못한 상속

            상속으로 코드 재사용 문제를 모두 해결할 수 있는 것은 아니다.
            (예 : 한 개의 메서드를 재사용하기 위해 100개의 메서드와 필드를 가진 클래스를 상속)

            델리게이션(Delegation) : 멤버 변수를 이용해서 클래스에서 필요한 메서드를 직접 호출하는 메서드를 작성하는 것이 좋다.

            final로 선언된 클래스 : 다른 클래스가 상속받지 못하게 함으로써 원래의 동작, 핵심기능을 고정시킨다.

            디폴트 메서드 또한 이러한 규칙을 적용하여 필요한 기능만 포함하도록 인터페이스를 최소한으로 유지한다면 쉽게 기능을 조립할 수 있다.

- 해결 규칙

    ### 9.4 해석 규칙

    - **알아야 할 세 가지 해결 규칙**

        다른 클래스나 인터페이스로부터 같은 시그니처를 갖는 메서드를 상속받을 때

        1. **클래스가 항상 이긴다. 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다.**
        2. **1번 규칙 이외의 상황에서는 서브인터페이스가 이긴다. 상속관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는 서브인터페이스가 이긴다!**
        3. **여전히 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야 한다.**
    - **디폴트 메서드를 제공하는 서브인터페이스가 이긴다**

        예시와 Quiz로 이해하자

    - **충돌 그리고 명시적인 문제 해결**

        ![Chapter9%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%2035d9daff81274d7f8cbf81eeb6e345ff/Untitled%203.png](Chapter9%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%2035d9daff81274d7f8cbf81eeb6e345ff/Untitled%203.png)

        자바 컴파일러는 혼란스럽다.
        "Error: class C inherits unrelated defaults for hello() from type B and A."

        - 충돌 해결 : 클래스 C에서 사용하려는 메서드를 명시적으로 선택한다.
        자바 8에서는 X.super.m(...) 형태의 새로운 문법을 제공하여 인터페이스를 호출한다.

            ```java
            public class C implements B, A {
            	void hello() {
            		B.super.hello();
            	}
            }
            ```

    - **다이아몬드 문제**

        어떻게 설명할지 고민하자

        ![Chapter9%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%2035d9daff81274d7f8cbf81eeb6e345ff/Untitled%204.png](Chapter9%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%2035d9daff81274d7f8cbf81eeb6e345ff/Untitled%204.png)

        ### 무엇이 출력될까?

        - 1번

            ```sql
            public interface A {
            	default void hello(){
            		System.out.println("Hello from A");
            	}
            }

            public interface B extend A { }
            public interface C extend A { }

            public interface D implements B, C {
            	public static void main(String... args){
            		new D().hello();//Q. 무엇이 출력될까?
            	}
            }
            ```

        - 1번 정답

            답 : 'Hello from A'

            이유 : D는 실제로 선택할 수 있는 메서드 선언은 하나뿐(A의 default 메서드 hello)

        - 2번

            ```sql
            public interface A {
            	default void hello(){
            		System.out.println("Hello from A");
            	}
            }

            public interface B extend A {
            	default void hello(){
            		System.out.println("Hello from B");
            	}
            }
            public interface C extend A { }

            public interface D implements B, C {
            	public static void main(String... args){
            		new D().hello();//Q. 무엇이 출력될까?
            	}
            }
            ```

        - 2번 정답

            답 : 충돌

            이유 : hello 메서드 정의하고 있다. 둘 중 하나의 메서드를 명시적으로 호출해야 한다.

        - 3번 : 책 오타?

            ```sql
            public interface A { }

            public interface B extend A { }
            public interface C extend A {
            	void hello();
            }

            public interface D implements B, C {
            	public static void main(String... args){
            		new D().hello();//Q. 무엇이 출력될까?
            	}
            }
            ```

        - 3번 정답

            답 : 책 오타?

### 9.5 요약

- 자바 8의 인터페이스는 구현 코드를 포함하는 디폴트 메서드, 정적 메서드를 정의할 수 있다.
- 디폴트 메서드의 정의는 default 키워드로 시작하며 일반 클래스 메서드처럼 바디를 갖는다.
- 공개된 인터페이스에 추상 메서드를 추가하면 소스 호환성이 깨진다.
- 디폴트 메서드 덕분에 라이브러리 설계자가 API를 바꿔도 기존 버전과 호환성을 유지할 수 있다.
- 선택형 메서드와 동작 다중 상속에도 디폴트 메서드를 사용할 수 있다.
- 클래스가 같은 시그니처를 갖는 여러 디폴트 메서드를 상속하면서 생기는 충돌 문제를 해결하는 규칙이 있다.
- 클래스나 슈퍼클래스에 정의된 메서드가 다른 디폴트 메서드 정의보다 우선한다. 이 외의 상황에서는 서브인터페이스에서 제공하는 디폴트 메서드가 선택된다.
- 두 메서드의 시그니처가 같고, 상속관계로도 충돌 문제를 해결할 수 없을 때는 디폴트 메서드를 사용하는 클래스에서 메서드를 오버라이드해서 어떤 디폴트 메서드를 호출할지 명시적으로 결정해야한다.