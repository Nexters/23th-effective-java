# 매개변수가 유효한지 검사하라
- 메소드 내 로직이 시작되기 전에 검사해야 한다.
  - ex1. 인덱스 값은 음수가 될 수 없다.
  - ex2. 객체 참조는 null이 아니어야 한다.
- 검사를 제대로 하지 않는다면
  - 메소드가 수행되는 중간에 모호한 예외를 던진다.
  - 메소드가 수행은 되지만 잘못된 값을 던진다.
  - 메소드가 수행은 되지만 객체가 이상한 상태가 되어 알 수 없는 미래의 시점에 해당 메소드와 관련 없는 오류를 낼 수 있다.

## public protected 메소드는 예외를 문서화하자
```java
  /**
   * (현재 값 mod m) 값을 반환한다. 이 메소드는
   * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메소드와 다르다.
   * 
   * @param m 계수 (양수여야 한다.)
   * @return 현재 값 mod m
   * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
   */
  public BigInteger mod(BigInteger m) {
    if (m.signum() <= 0) {
      throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
    }

    // logic
  }
```
- m이 null이면 m.signum() 호출 이 `NullPointerException`이 발생하지만 설명에는 없다.
  - BigInteger 클래스 수준에서 기술했기 때문
  - 클래스 수준 주석은 그 클래스의 모든 public 메소드에 적용된다.

## null 검사 방법
> 자바 7에 추가된 `java.util.Objects.requireNonNull` 메소드 활용

```java
  this.strategy = Objects.requireNonNull(strategy, "예외 메시지");
```
- 더이상 null 검사를 수동으로 하지 않아도 된다.
- 예외 메시지를 원하는대로 지정할 수 있다.
- 자바 9에는 Objects에 범위 검사 기능이 추가되었다.
  - `checkFromIndexSize` `checkFromToIndex` `checkIndex`
  - 예외 메시지를 지정할 수 없음
  - 리스트와 배열 전용으로 설계됨

## assert (단언문) 사용
> public이 아닌 메소드에 사용해서 유효성을 검증한다.

```java
  private static void sort(long a[], int offset, int length) {
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;

    // logic
  }
```
- 자신이 단언한 조건이 모두 참이라고 선언하는 것
- 거짓이라면 AssertionError를 던진다.
- 런타임에 아무런 효과도, 성능 저하도 없다.
  - `-ea` `--enableassertions` 플래그를 설정하면 런타임에 영향을 준다.

## 나중에 쓰기 위해 저장하는 매개변수
```java
  // 입력 받은 int 배열의 List 뷰를 반환하는 메소드
  static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a);

    return new AbstractList<>() {
      @Override public Integer get(int i) {
        return a[i];
      }

      @Override public Integer set(int i, Integer val) {
        int oldVal = a[i];
        a[i] = val;
        return oldVal;
      }

      @Override public int size() {
        return a.length;
      }
    };
  }
```
- `Objects.requireNonNull` 메소드를 생략했다면 클라이언트가 받은 List를 사용할 때 NullPointerException이 발생할 것
  - 에러 추적이 어려워서 디버깅이 힘들어짐
- 생성자는 클래스 불변식을 어기는 객체가 만들어지지 않도록 매개변수 유효성 검사를 해야 한다.

## 예외
- 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때
- 계산 과정에서 암묵적으로 검사가 수행될 때
  - ex. Collections.sort(list)의 경우 list 안의 객체들은 모두 비교될 수 있어야 한다.
    - 정렬 과정에서 비교될 수 없는 타입이 존재한다면 ClassCastException 에러를 던진다.
  - 암묵적 유효성 검사에 너무 의존하면 실패 원자성을 해칠 수 있으니 주의하도록 한다.
- 발생한 예외와 문서에 작성된 예외가 다를 수 있다.
  - 예외 번역 (exception translate) 관용구를 사용해서 API 문서에 기재된 예외로 번역해야 한다.

## 정리
- 메소드나 생성자를 작성할 때 어떤 제약이 있을지 생각해야 한다.
- 제약들을 문서화하고, 메소드 시작 부분에서 명시적으로 검사해야 한다.
