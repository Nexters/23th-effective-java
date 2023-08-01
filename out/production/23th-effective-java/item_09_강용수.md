# try-finally 보다는 try-with-resources를 사용하라
- close를 통해 자원을 닫는 작업
  - InputStream, OutputStream, java.sql.Connection
  - 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능문제로 이어진다.
  - 전통적으로 자원이 제대로 닫히는 것을 보장하는 수단으로 try-finally가 쓰였다.

## try-finally
```java
  // 코드 9.1 try-finally - 더 이상 자원을 회수하는 최선의 방책이 아니다.
  static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
      return br.readLine();
    } finally {
      br.close();
    }
  }
```
```java
  // 코드 9.2 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다.
  static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
      OutputStream out = new FileOutputStream(dst);
      try {
        byte[] buf = new bute[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0) {
          out.write(buf, 0, n);
        }
      } finally {
        out.close();
      }
    } finally {
      in.close();
    }
  }
```
- try-finally 문의 결점
  - 예외는 try, finally 모두에서 발생할 수 있다.
  - finally에서 발생한 두 번째 예외가 try에서 발생한 첫 번째 예외를 집어 삼킬 수 있다.
    - 스택 추적 내역에 finally에서 발생한 예외만 표시된다. -> try에서 발생한 예외는 확인할 수 없다.

## try-with-resources
```java
  // 코드 9.3 try-with-resources - 자원을 회수하는 최선책
  static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
      return br.readLine();
    }
  }
```
```java
  // 코드 9.4 복수의 자원을 처리하는 try-with-resources - 짧고 매혹적이다.
  static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
          OutputStream out = new FileOutputStream(dst)) {
      byte[] buf = new bute[BUFFER_SIZE];
      int n;
      while((n = in.read(buf)) >= 0) {
        out.write(buf, 0, n);
      }
    }
  }
```
- 이 구조를 사용하려면 해당 자원리 AutoCloseable 인터페이스를 구현해야 한다.
  - 단순히 void를 반환하는 close 메소드 하나만 정의한 인터페이스
- 짧고 읽기 수월하고, 문제를 진단하기도 좋다.
- readLine, close에서 예외 발생 시 close에서 발생한 예외가 숨겨지고, readLine에서 발생한 예외가 기록된다.
  - 숨겨진 예외는 스택 추적 내역에 `suppressed`라는 꼬리표를 달고 출력된다.
  - 자바 7부터 도입된 Throwable의 getSuppressed 메소드를 이용해서 예외를 확인할 수 있다.
    ```java
      static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
          Throwable[] suppressedExceptions = e.getSuppressed();
          for (Throwable suppressedException : suppressedExceptions) {
              System.out.println("Suppressed Exception: " + suppressedException);
          }
          throw e;
        }
      }
    ```
