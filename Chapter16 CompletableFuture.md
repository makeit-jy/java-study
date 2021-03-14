# chapter 16 CompletableFuture

- Future 의 단순 활용
    - Future는 비동기 계산을 모델링하는데 이용할 수 있으며, 계산이 끝났을 때 결과에 접근할 수 있는 참조를 제공하는 인터페이스이다.

    ```java
    ExecutorService executor = Excutors.newCachedThreadPool();
    Future<Double> future = executor.submit(new callable<Double>() {
    	public Double call(){
    		return doSomeLongComputation();
    	}
    });
    doSomethingElse();
    try{
     Double result = future.get(1,TimeUnit.SECONDS);
    } catch (ExcutionException ee){
    	// 계산 중 예외 발생
    } catch (InterruptedException ie){
    	// 현재 스레드에서 대기 중 인터럽트 발생
    } catch (TimeoutException te){
    	// Future가 완료되기 전에 타임아웃 발생
    }
    ```

    - 스레드 풀에 태스크를 제출하려면 ExecutorService를 만들어야 한다.
    - Callable을 ExecutorService로 로 제출한다.
    - 시간이 오래 걸리는 작업은 다른 스레드에서 비동기적으로 실행한다.
    - 비동기 작업을 수행하는 동안 다른 작업을 한다.
    - 비동기 작업의 결과를 가져온다. 결과가 준비되어 잇지 않으면 호출 스레드가 블록된다. 하지만 최대 1초까지만 기다린다.

    ```java
    그러나 여러 Future의 결과가 있을 때 이들의 의존성을 표현하기가 어렵다!
    ```

    - 이런것들이 어렵다!
        - 두 개의 비동기 계산 결과를 하나로 합친다.
        - Future 집합이 실행하는 모든 태스크의 완료를 기다린다.
        - Future 집합에서 가장 빨리 완료되는 태스크를 기다렸다가 결과를 얻는다.
        - 프로그램적으로 Future를 완료시킨다.
        - Future 완료 동작에 반응한다.
- CompletableFuture
    - 람다 표현식과 파이프라이닝을 활용한다. 따라서 Future와 CompletableFuture의 관계를 Collection과 Stream의 관계에 비유할 수 있다.
    - 동기 메서드를 비동기 메서드로 변환(이름)

    ```java
    public class Shop{
    	public double getPrice(String product){
    	}
    }

    public Future<Double> getPriceAsync(String product){
    	CompletableFuture<Double> futurePrice = new CompletableFuture<>();//계산 결과를 포함할 CF를 생성한다.
    	new Tread( () -> {
    			double price = caculatePrice(product);//다른 스레드에서 비동기적으로 계산을 수행한다.
    			futurePrice.complete(price);//계산이 완료되면 Future에 값을 설정한다.
    	}).start();
    	return futurePrice;//계산 결과가 완료되길 기다리지 않고 Future를 반환한다.
    }
    ```

    - 팩토리 메서드 supplyAsync로 CompletableFuture 만들기

    ```java
    public Future<Double> getPriceAsync(String product){
    	return CompletableFuture.supplyAsync(()->calculatePrice(product));
    }
    ```

    - 타임아웃을 통한 예외처리

- 비블록 코드 만들기
    - 병렬 처리 스트림을 이용해서 순차 계산을 병렬로 처리해서 성능을 개선할 수 있다.4→1
    - CompletableFuture로 비동기 호출 구현하기

    ```java
    List<CompletableFuture<Stirng>> priceFutures =
    	shops.stream()
    	.map(shop->CompletableFuture.supplyAsync(
    		()->String.format("%s price is %.2f",shop.getName(),shop.getPrice(product))))
    	.collect(toList());
    ```

    - 더 확장성이 좋은 해결 방법 → 커스텀 Executor 사용하기.5
