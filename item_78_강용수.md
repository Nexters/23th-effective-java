# 공유 중인 가변 데이터는 동기화해 사용하라
`synchronized`: 해당 메소드나 블록을 한번에 한 스레드씩 수행하도록 보장
- 동기화된 블록에 들어간 스레드가 있다면 작업이 완료될때까지 다른 스레드는 접근하지 못함
- 언제 확인하더라도 항상 일관된 상태를 유지할 수 있음
```java
  public class SingletonTest {
    private static volatile SingletonTest instance;
 
    private SingletonTest() {
    }
    
    public static SingletonTest getInstance() {
      if (instance == null) {
        synchronized (SingletonTest.class) {
          if (instance == null) {
            instance = new SingletonTest();
          }
        }
      }
      return instance;
    }
  }
```

## 동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.
- 자바 언어 명세는 스레드가 필드를 읽을 때 항상 수정이 완전히 반영된 값을 얻는다고 보장하지만 한 스레드가 저장한 값이 다른 스레드가 확인할 수 있다고는 보장하지 않음
- `Thread.stop()`: 스레드를 강제 종료함
  - `@Deprecated(since="1.2")` 어노테이션이 붙어있는걸 보니 사용하지 않는 것이 좋음

### 다른 스레드를 멈추는 방법
- 스레드는 자신의 boolean 필드를 폴링하면서 그 값이 true가 되면 멈춘다.
  - 해당 필드를 false로 초기화하고, 다른 스레드에서 이 스레드를 멈추려고 할 때 true로 변경한다.
  - boolean을 읽고 쓰는 작업은 원자적이라 이 필드에 접근할 때는 동기화를 제거하기도 한다.
  ```java
    public class StopThread {
      private static boolean stopRequested;

      public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
          int i = 0;
          while (!stopRequested) i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
      }
    }
  ```
  - 동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 확인하지 못할 수 있음 -> 무한루프 돌 수도 있음
  ```java
    public class StopThread {
      private static boolean stopRequested;

      private static synchronized void requestStop() {
        stopRequested = true;
      }

      private static synchronized boolean stopRequested() {
        return stopRequested;
      }

      public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
          int i = 0;
          while (!stopRequested()) i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
      }
    }
  ```
  - `requestStop()`, `stopRequested()` 메소드를 동기화함으로 해결
  - 쓰기, 읽기 메소드가 모두 동기화되어야 한다.
  ```java
    public class StopThread {
      private static volatile boolean stopRequested;

      public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
          int i = 0;
          while (!stopRequested) i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
      }
    }
  ```
  - stopRequested를 volatile으로 선언하면 동기화를 생략해도 됨
    - volatile 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽는 것을 보장한다.
    ```java
      private static volatile int nextSerialNumber = 0;

      public static int generateSerialNumber() {
        nextSerialNumber++;
      }
    ```
    - 여러개의 스레드가 동시에 접근해서 값을 증가시킬 수 있음 -> 스레드안전 X -> 동기화해야함
      - synchronized 추가 시 volatile은 필요없음
    ```java
      private static final AtomicLong nextSerialNum = new AtomicLong();

      public static long generateSerialNumber() {
        return nextSerialNum.getAndIncrement();
      }
    ```
    - `java.util.concurrent.atomic`의 AtomicLong을 사용하면 락 없이도 스레드 안전을 지킬수 있음

## 정리
- 위 문제를 피할 방법은 애초에 가변 데이터를 공유하지 않는 것
  - 가변 데이터는 단일 스레드에서만 사용하는 것
- 여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화해야 한다.
  - 데이터 공유시 공유하는 부분만 동기화해도 된다.
