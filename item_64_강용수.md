# 객체는 인터페이스를 사용해 참조하라
- 적합한 인터페이스가 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 전부 인터페이스 타입으로 선언하도록 한다.
  ```java
    Set<Son> sonSet = new LinkedHashSet<>();

    LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
  ```
  - 인터페이스 타입으로 사용하면 코드가 유연해짐
- 만약 인터페이스의 일반 규약 이외의 특별한 기능에 의존하고 있다면 새로운 클래스도 이 기능을 제공해야 한다.
  - LinkedHashSet은 순서를 보장한다. HashSet으로 변경한다면 순서를 보장하지 않는 문제가 발생한다.
- 적합한 인터페이스가 없다면 당연히 클래스를 참조해야 한다.
  - 값 클래스
    - ex. String, Integer
    - 클래스를 변경하는 일이 없음
    - 매개변수, 변수, 필드, 반환 타입으로 사용해도 무방
  - 클래스 기반으로 작성된 프레임워크가 제공하는 객체들
    - 특정 구현 클래스보다는 기반 클래스를 사용하는 것이 좋음
  - 인터페이스에 없는 특별한 메소드를 제공하는 클래스
    - ex. PriorityQueue는 Queue 인터페이스에 없는 comparator 메소드를 제공한다.
    - 해당 메소드가 필요할 때 사용하도록 한다.

## 정리
- 가능하면 인터페이스를 사용하도록 한다.
- 적합한 인터페이스가 없다면 클래스의 계층 구조 중 필요한 기능을 만족하는 상위 클래스를 사용하도록 한다.
