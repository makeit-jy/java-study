# chapter 7 병렬 데이터 처리와 성능

- 병렬 스트림

    병렬 스트림이란 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림이다. 따라서 병렬 스트림을 이용하면 모든 멀티코어 프로세서가 각각의 청크를 처리하도록 할당할 수 있다.

    ```java
    public long iterativeSum(long n){
    	long result = 0;
    	for(long i = 1L; i<=n; i++){
    		result += 1;
    	}
    	return result;
    }
    ```

    ```java
    public long sequentialSum(long n){
    	return Stream.iterate(1L, i -> i+1)
    							 .limit(n)
    							 .reduce(0L, Long::sum);
    }
    ```

    ```java
    public long parallelSum(long n){
    	return Stream.iterate(1L, i -> i+1)
    							 .limit(n)
    							 .parallel()
    							 .reduce(0L, Long::sum);
    ```

    스트림이 여러 청크로 분할되어 있다. 따라서 리듀싱 연산을 여러 청크에 병렬로 수행할 수 있다. 마지막으로 리듀싱 연산으로 생성된 부분 결과를 다시 리듀싱 연산으로 합쳐서 전체 스트림의 리듀싱 결과를 도출한다.

- 스트림 성능 측정

    자바 마이크로벤치마크 하니스(JMH) 라는 라이브러리를 이용해 벤치마크를 구현하여 측정

    iterativeSum - 3.278

    sequentialSum - 121.842

    parallelSum - 604.059

    반복 결과로 박싱된 객체가 만들어지므로 숫자를 더하려면 언박싱을 해야 한다.

    반복 작업은 병렬로 수행할 수 있는 독립 단위로 나누기가 어렵다.(iterate는 본질적으로 순차적이다)

    병렬 프로그래밍의 효과적인 사용은 어렵다. 그래서 내부적으로 어떤 일이 일어나는지 알아야 한다.

    박싱과 언박싱의 오버헤드가 사라진다.

    쉽게 청크로 분할할 수 있는 숫자 범위를 생산한다.

    5장에서 소개했던 LongStream.rangeClosed 라는 메서드

    ```java
    public long rangedSum(){
    	return LongStream.rangeClosed(1,N)
    									 .reduce(0L, Long::sum);
    }//5.315
    ```

    ```java
    public long parallelRangedSum(){
    	return LongStream.rangeClosed(1,N)
    									 .parallel()
    									 .reduce(0L, Long::sum);
    }//2.677
    ```

    올바른 자료구조를 선택해야 하고, 코어 간의 데이터 전송 시간도 고려해야 한다.

    - 병렬 스트림을 효과적으로 사용하는 방법
        - 직접 측정하라
        - 박싱 언박싱을 주의하라
        - 요소의 순서에 의존하는 limit와 findFirst 같은 연산을 주의하라
        - 전체 파이프라인 연산 비용을 고려하라
        - 작은 데이터에서는 병렬 스트림 사용을 피해라
        - 자료구조를 확인해라
        - 최종 연산의 병합 과정 비용을 고려하라
- 포크/조인 프레임워크

    포크/조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음에 서브태스크 각각의 결과를 합쳐서 전체 결과를 만들도록 설계되었다. 포크/조인 프레임워크에서는 서브태스크를 스레드 풀의 작업자 스레드에 분산 할당하는 ExecutorService 인터페이스를 구현한다.

    ```java
    if(태스크가 충분히 작거나 더 이상 분할할 수 없으면){
    	순차적으로 태스크 계산
    }
    else{
    	태스크를 두 서브태스크로 분할
    	태스크가 다시 서브태스크로 분할되도록 이 메서드를 재귀적으로 호출함
    	모든 서브태스크의 연산이 완료될 때까지 기다림
    	각 서브태스크의 결과를 합침
    }
    ```

    분할정복 알고리즘과 비슷

    - 포크/조인 프레임워크를 제대로 사용하는 방법
        - 서브 태스크가 모두 끝난 다음에 join을 호출하라(그렇지 않으면 결과가 준비될 때까지 블록)
        - invoke 메서드 사용을 주의할 것
        - 서브태스크에 fork 메서드를 호출해서 ForkJoinPool의 일정을 조절할 수 있다.
        - 스택 트레이스가 도움이 되지 않아 디버깅이 어렵다
        - 예측보단 측정이 중요하다.

    작업 훔치기 기법을 통해 ForkJoinPool의 모든 스레드를 거의 공정하게 분할한다.

- Spliterator

    분할할 수 있는 반복자로 병렬 작업에 특화

    ```java
    public interface Spliterator<T>{//T는 탐색하는 요소의 형식
    	boolean tryAdvance(Consumer<? super T> action);
    //tryAdvance 메서드는 Spliterator의 요소를 하나씩 순차적으로 소비하면서 탐색해야 할 요소가 남아있으면 참을 반환한다.(일반적인 Iterator동작과 같다))
    	Spliterator<T> trySplit();
    //trySplit 메서드는 Spliterator의 일부 요소를 분할해서 두번째 Spliterator를 생성하는 메서드. 재귀적으로 분할한다
    	long estimateSize();
    //탐색해야 할 요소 수 정보
    	int characteristics();
    //Spliterator 자체의 특성 집합을 포함하는 int를 반환하는 추상 메서드
    }
    ```