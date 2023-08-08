# 인터페이스는 구현하는 쪽을 생각해 설계하라
## 디폴트 메소드
- 자바 8부터 추가
- 인터페이스에서 기본적인 구현을 제공
- 구현 클래스에서 별도로 구현하지 않아도 됨
- 라이브러리에도 추가되어 있음

```java
  // 자바8의 Collection 인터페이스에 추가된 디폴트 메소드
  default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean result = false;
    for (Iterator<E> it = iterator(); it.hasNext();) {
      if (filter.test(it.next())) {
        it.remove();
        result = true;
      }
    }
    return result;
  }
```
- 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메소드를 작성하는 것은 어렵다.
  - `org.apache.commons.collections4.collection.SynchronizedCollection`
    - 락 객체를 사용하여 동기화하고, 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스
      - 락 객체: 멀티스레드 환경에서 공유된 데이터에 대한 접근을 동기화하는데 사용하는 객체
    - 현재는 removeIf 메소드를 재정의 하지 않고 있다.
      - removeIf 메소드는 락 객체를 사용하지 않고, 내부적으로 컬렉션을 수정하는 로직
      - 멀티스레드 환경에서 동시에 접근한다면 `ConcurrentModificationException`와 같은 에러가 발생할 수 있음
    - 인터페이스의 디폴트 메소드를 재정의하고, 다른 메소드에서는 디폴트 메소드를 호출하기 전에 필요한 작업을 수행하도록 했다.
      - ex. `SynchronizedCollection`이 반환하는 package-private 클래스들은 removeIf를 재정의하고, 이를 호출하는 다른 메소드들은 디폴트 구현을 호출하기 전에 동기화
      - 일부 기능들은 여전히 수정되고 있지 않음

## 정리
- 디폴트 메소드는 컴파일에 성공하더라도 기존 구현체에 런타임 오류를 일으킬 수 있다.
  - 기존 인터페이스에 디폴트 메소드를 추가하는 경우는 되도록 없어야 한다.
- 인터페이스 설계 시 주의해야 한다.
  - 기존 인터페이스에 디폴트 메소드를 추가한다면 어떤 위험이 있을지 모름
- 새로운 인터페이스라면 릴리스 전에 반드시 테스트를 거쳐야 한다.
  - 다른 방식으로 세 가지 이상 구현하는 것이 좋다.
  - 인터페이스를 릴리스한 후에도 결함이 있을 수 있다.
