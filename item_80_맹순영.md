## 아이템 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라

### 실행자

```java
ExecutorService exec = Executors.newSingleThreadExecutor();
```

클라이언트가 요청한 작업을 백그라운드 스레드에 위임해 처리하기 위해 작업 큐를 사용하는데, 위의 코드 한 줄로 생성 가능함

### 실행자의 서비스들

- 작업 넘기기
    - exec.execute(runnable);
- 종료시키기
    - exec.shutdown();
- 특정 태스크가 완료되기를 기다림

    ```java
    // 아이템 79의 코드
    ExecutorService exec = Executors.newSingleThreadExecutor();
      try {
          exec.submit(() -> s.removeObserver(this)).get();
      } catch (ExecutionException | InterruptedException ex) {
          throw new AssertionError(ex);
      } finally {
          exec.shutdown();
      }
    ```

    - 위의 get() 메서드가 완료를 기다리는 메서드임
- 태스크 모음 중 아무것 하나 혹은 모든 태스크가 완료되기를 기다림
    - invokeAny(), invokeAll();
- 실행자 서비스가 종료하기를 기다림
    - awaitTermination();
- 완료된 태스크들의 결과를 차례로 받기
    - ExecutorCompletionService 이용
- 태스크를 특정 시간에 혹은 주기적으로 실행하게 하기
    - ScheduledThreadPoolExecutor  이용

### 여러 상황들

- 작업 큐를 여러 스레드가 처리하게 하고 싶으면,  다른 종류의 실행자 서비스(스레드 풀)를 생성하면 됨
    - ThreadPoolExecutor 클래스를 직접 사용하면 거의 모든 속성을 설정할 수 있음
- 가벼운 서버에서 사용할 때는 CachedThreadPool 사용
    - 무거운 서버에서 사용하면, 요청받은 태스크들이 큐에 쌓이지 않고 바로 스레드에 위임이 돼서, 서버가 무거운 경우 CPU 이용률이 100%가 된 뒤에도 새로운 태스크가 도착하는 족족 다른 스레드를 생성하게 되는 문제가 있음
- 무거운 프로덕션 서버에서 사용할 때는 스레드 개수가 고정된 FixedThreadPool 혹은 ThreadPoolExecutor로 직접 설정하면 됨

⇒ 작업 큐와 스레드를 직접 다루는 것은 일반적으로 좋지 않지만, 실행자 프레임워크에서는 작업 단위와 실행 메커니즘이 분리됨

### 태스크 종류

- Runnable
    - 반환 값이 없음

    ```java
    public class Main {
        public static void main(String[] args) {
            // 스레드 풀 생성 (여기서는 크기가 2인 스레드 풀을 생성)
            ExecutorService executorService = Executors.newFixedThreadPool(2);
    
            // 실행할 Runnable 객체 생성
            Runnable myRunnable = new MyRunnable();
    
            // Runnable 객체를 스레드 풀에 제출
            executorService.execute(myRunnable);
    
            // 스레드 풀 종료
            executorService.shutdown();
        }
    }
    
    class MyRunnable implements Runnable {
        @Override
        public void run() {
            // 스레드가 실행할 코드를 작성
            System.out.println("Runnable is running in thread: " + Thread.currentThread().getName());
        }
    }
    ```

- Callable
    - 값을 반환하고 예외를 던질 수 있음

        ```java
        public class MyCallable implements Callable<Integer> {
            @Override
            public Integer call() throws Exception {
                // 비동기적으로 실행하고자 하는 작업을 구현
                int result = 0;
                for (int i = 1; i <= 10; i++) {
                    result += i;
                }
                return result;
            }
        
            public static void main(String[] args) throws Exception {
                // ExecutorService 생성 (스레드 풀)
                ExecutorService executorService = Executors.newFixedThreadPool(1);
        
                // Callable 작업을 스케줄링하고 Future 객체를 통해 결과를 얻음
                Future<Integer> future = executorService.submit(new MyCallable());
        
                // 작업이 완료될 때까지 기다린 후 결과를 얻어옴
                int result = future.get();
        
                // 결과 출력
                System.out.println("Result: " + result);
        
                // ExecutorService 종료
                executorService.shutdown();
            }
        }
        ```


⇒ 여기서 핵심은 실행자 프레임워크가 작업 수행을 담당해준다는 것

### 포크-조인 태스크

- 이름에서 알 수 있듯이 포크-조인 풀이라는 실행자 서비스가 실행함
    - ForkJoinTask의 인스턴스는 하위 태스크로 나뉠 수 있고, ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리함
- 높은 처리량과 낮은 지연 시간

### 스트림 (왜 책엔 없지)

```java
public class Main {
    public static void main(String[] args) {
        // 리스트 생성
        List<String> myList = Arrays.asList("apple", "orange", "banana", "grape", "melon");

        // 순차 스트림으로 처리
        myList.stream().forEach(s -> System.out.println(s));

        // 병렬 스트림으로 처리
        myList.parallelStream().forEach(s -> System.out.println(s));
    }
}
```

- 이런 식으로 parallelStream 쓰면 병렬 처리가 쉽게 됨