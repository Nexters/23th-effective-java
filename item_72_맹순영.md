## 아이템 72. 표준 예외를 사용하라

우리가 많은 코드를 재사용하는 것처럼 예외도 재사용하는 것이 좋다! 자바 라이브러리가 충분한 예외들을 제공한다.

### 표준 예외 재사용 시 좋은 점

- API가 다른 사람이 익히고 사용하기 쉬워짐
- 반대로 내가 개발한 API를 사용한 프로그램 또한 낯선 예외를 사용하지 않게 되어 읽기 쉬움
- 예외 클래스가 적을수록 메모리 사용량도 줄고 클래스 적재 시간도 적음

### IllegalArgumentException

- 호출자가 인수로 부적절한 값을 넘길 때 사용

### IllegalStateException

- 대상 객체의 상태가 호출된 메서드를 수행하기에 적합하지 않을 때 사용
    - 제대로 초기화되지 않은 객체를 사용하려고 할 때

### NullPointerException

- IllegalArgumentException 써도 되지만, 관례상 null 관련해서는 NullPointerException 사용

### IndexOutOfBoundsException

- 마찬가지로 IllegalArgumentException 써도 되지만, 특정 허용 범위를 넘을 때는 관례상 IndexOutOfBoundsException 사용

### ConcurrentModificationException

- 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려 할 때 사용
- 동시 수정을 검출할 수 있는 안정된 방법이 없어서, 문제가 생길 가능성을 알려주는 정도로 사용

### UnsupportedOperationException

- 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때 사용
    - 대부분 자신이 정의한 메서드를 모두 지원해서 자주 쓰이지는 않음
- 보통 구현하려는 인터페이스의 메서드 일부를 구현할 수 없을 때 쓰임
    - 원소를 넣을 수만 있는 List 구현체에 remove 메서드를 호출하는 경우

<aside>
💡 이외에도 ArithmeticException, NumberFormatException 등을 재사용할 수 있음

</aside>

### Exception, RuntimeException, Throwable, Error는 직접 재사용하지 말기

- 추상 클래스라고 생각하면 됨
- 안정적인 테스트 불가

### 나만의 예외 만들기?

- 예외는 직렬화할 수 있고 직렬화는 많은 부담이 따르기 때문에, 나만의 예외를 따로 만드는 것이 좋지는 않다.

### 어떤 예외를 줘야될 지 애매한 경우..

ex) 카드 덱을 표현하는 객체와 인수로 건넨 수만큼의 카드를 뽑아 나눠주는 메서드

```java
public void drawAndDistribute(int count) {
	if (remainCards < count) {
		// 어떤 예외?	
	}
}
```

어떤 에러를 사용해야할까?

1. IllegalArgumentException - 호출자가 인수로 부적절한 값을 넘길 때 사용
    - count의 값이 너무 크기 때문에 호출..?
2. IllegalStateException - 대상 객체의 상태가 호출된 메서드를 수행하기에 적합하지 않을 때 사용
    - reaminCards의 수(객체의 상태)가 너무 적기 때문에 호출..?

해결법

- 인수의 값과 상관없이 어차피 실패한 경우라면 IllegalStateException
    - count가 1이었어도 실패하는 경우
- 그렇지 않으면 IllegalArgumentException
    - count가 1이면 성공하는데 2라서 실패하는 경우