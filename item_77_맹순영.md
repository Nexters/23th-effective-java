## 아이템 77. 예외를 무시하지 말라

```java
try {
    // 뭔가 하기
} catch (SomeException e) {
		// 아무것도 안하기
}
```

- 예외로 잡는 의미가 없음

예외를 무시해도 되는 경우가 있긴 함

- FileInputStream을 닫을 때
    - 작업을 모두 끝낸 상황이라, 굳이 Exception을 처리할 필요가 없음
    - 하지만 이런 경우 주석으로 이유를 남기고 변수의 이름을 ignored로 바꾸기

        ```java
        int numColors = 4;
        try {
        		numColors = f.get(1L, TimeUnit.SECONDS);
        	} catch (TimeoutException | ExecutionException **ignored) {
        }**
        ```