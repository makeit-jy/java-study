# Chapter15 OOP와 FP의 조화 : 자바와 스칼라 비교

- 스칼라란?
    - 스칼라는 **객체지향과 함수형 프로그래밍을 혼합한 언어**.
    - JVM에서 동작하고 스칼라 코드에서 모든 자바 라이브러리 사용 가능.
    - 스칼라에서는 모든 것이 **객체**. (자바보다 완전한 객체지향 언어)
    - 스칼라에서의 변수는 일반 선언 var과 상수 선언(자바의 final)인 val이 존재.
- Why 자바 책에서?
    - 이 책은 자바에서 **함수형 프로그래밍**을 적용하는 방법을 설명.
    - 자바와 마찬가지로 스칼라는 컬렉션을 함수형으로 처리하는 개념(스트림과 비슷한 연산), 일급 함수, 디폴트 메서드 등을 제공.
    - 스칼라는 자바에 비해 **더 다양하고 심화된 함수형 기능을 제공**.
    - 자바와 비교하며 자바의 한계와 '**함수형**'의 세계에 대해 알아보자.
- 스칼라의 자료구조

    리스트, 집합, 맵, 튜플, 스트림, 옵션 등

    - 문자열 보간법(string interpolation): 문자열 자체에 변수와 표현식을 바로 삽입하는 기능. (접두어 s를 붙인다.)

    ```scala
    // Scala
    s"Hello ${n} bottles of beer"
    ```

    ```scala
    // Java
    "Hello " + n + " bottles of beer"
    ```

    - 자료구조 예시

    ```scala
    import scala.io.Source

    class ScalaExam {
      object Beer {
        def main(arg : Array[String]) {
    	  // 명령형 스칼라
          var n : Int = 2;
          while( n <= 6) {
            println(s"Hello ${n} bottles of beer") // 문자열 보간법 
            n += 1
          }

    	  // 함수형 스칼라
          // 2 ~ 6 foreach (n)
          2 to 6 foreach { n => println(s"Hello ${n} bottles of beer")}

    	  // 컬렉션 만들기
          // Map<String, Int>
          val authorsToAge = Map("Raoul" -> 23, "Mario" -> 40, "Alan" -> 53); // ->로 키를 값에 대응시켜 맵 만들기

    	  // 리스트(단반향 연결 리스트)
          // List<String>
          val authors = List("Raoul", "InWoo", "Alan")

    	  // 집합(중복된 요소가 없는)
          // Set<Int>
          val numbers = Set(1, 1, 2, 3, 4) // 4개의 요소를 포함
          // numbers 요소와 8 요소를 포함한 새로운 Set 생성
          val newNumbers = numbers + 8

    	  // 컬렉션은 기본적으로 불변(immutable). 갱신할 때는 19장의 영속이라는 용어 스칼라에도 적용 가능. 
    	  // -> 결과적으로 암묵적인 데이터 의존성을 줄일 수 있다. -> 언제, 어디서 컬렉션(또는 다른 공유된 자료구조 등)을 갱신했는지 크게 신경 쓰지 않아도 된다.

    	  // 컬렉션 사용하기
          // filter : 길이가 10 이상인 행만 선택한다.
          // Map : 긴 행을 대문자로 변환한다.
          val fileLines = Source.fromFile("data.txt").getLines().toList
          val linesLongUpper = fileLines.filter(l => l.length() > 10)
                              .map(l => l.toUpperCase())

          // 자바의 병렬 비슷한 기능 In Scala
          val linesLongUpper2 = fileLines.par filter (_.length() > 10) map(_.toUpperCase())

    	  // 튜플
    	  val raoul = ("Raoul", "+44 887007007") // 튜플 축약어
    	  val book = (2018, "Modern Java in Action", "Manning") // (Int, String, String)형식의 튜플

    	  // 스트림
    	  // 게으른 평가 제공. 스칼라의 스트림은 이전 요소가 접근할 수 있도록 기존 계산값을 기억. 
    	  // but, 이전 요소를 기억(캐시)해야 하기 때문에 자바의 스트림에 비해 메모리 효율성이 조금 떨어짐.

      	  // 옵션
          // option == java.optional 과 같은 기능
    	  // 사람의 나이가 최소 나이보다 클 때 보험회사 이름을 반환하는 코드      
          def getCarInsuranceName(person: Option[person], minAge: Int) =
              person.filter(_.getAge() >= minAge)
              .flatMap(_.getCar)
              .flatMap(_.getInsurance)
              .map(_.getName)
              .getOrElse("Unknown")
           
        }
      }
    }
    ```

- 함수
    - 스칼라의 함수는 어떤 작업을 수행하는 일련의 명령어 그룹.
    - 명령어 그룹을 쉽게 추상화할 수 있는 것도 함수 덕분이며 동시에 함수는 함수형 프로그래밍의 중요한 기초석.

    1. **함수 형식** : 함수 형식은 3장에서 설명한 자바 함수 디스크립터의 개념을 표현하는 편의 문법. (즉, 함수형 인터페이스에 선언된 추상 메서드의 시그니처를 표현하는 개념)

        - 일급 함수
            - 스칼라의 함수는 일급값이기에 String, Integer 처럼 함수를 인수로 전달하거나, 결과로 반환하고 변수에 저장할 수 있다.
    2. **익명 함수** : 자바의 람다 표현식과 달리 비지역 변수 기록에 제한을 받지 않음.
        - 클로저
            - 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스.
            - 자바의 람다에서 사용하는 변수는 상수 처리 혹은 그에 준하여 사용해야 하지만, Scala 에서는 상관없이 사용 가능.
    3. **커링 지원** : 여러 인수를 받는 함수를 일부 인수를 받는 여러 함수로 분리하는 기법.
        - 여러 인수를 받는 함수를 인수의 일부를 받는 여러 함수로 분할할 수 있음.
- 클래스와 트레이트
    - Scala의 클래스
        - 스칼라는 완전한 객체지향 언어이므로 클래스를 만들고 객체로 인스턴스화할 수 있다. 자바와 비슷한 구조.
    - Scala의 트레이트
        - 자바의 인터페이스를 대체.
        - 필드와 디폴트 메서드를 포함할 수 있는 인터페이스.
        - 객체를 인스턴스화할 때 메서드 상속 기능을 지원.
    - 게터와 세터
        - 게터, 세터, 생성자 암시적으로 생성 → 코드 단순해짐.

- 결론
    - 자바와 스칼라는 객체지향과 함수형 프로그래밍 모두를 하나의 프로그래밍 언어로 수용한다. 두 언어 모두 JVM에서 실행되며 넓은 의미에서 상호운용성을 갖는다.
    - 스칼라는 자바처럼 리스트, 집합, 맵, 스트림, 옵션 등의 추상 컬렉션을 제공한다. 또한 튜플도 추가로 제공한다.
    - 스칼라는 자바에 비해 풍부한 함수 관련 기능을 제공한다. 스칼라는 함수 형식, 지역 변수에 접근할 수 있는 클로저, 내장 커링 형식 등을 지원한다.
    - 스칼라의 클래스는 암묵적으로 생성자, 게터, 세터를 제공한다.
    - 스칼라는 트레이트를 지원한다. 트레이트는 필드와 디폴트 메서드를 포함할 수 있는 인터페이스다.
