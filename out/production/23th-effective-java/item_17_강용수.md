# 변경 가능성을 최소화하라
- 불변 클래스: 인스턴스 내부 값을 수정할 수 없는 클래스
- 불편 클래스 규칙
  - 객체의 상태를 변경하는 메소드를 제공하지 않는다.
  - 클래스를 확장할 수 없도록 한다.
    - 하위 클래스에서 객체의 상태를 변하게 만드는 것을 막는다.
  - 모든 필드를 final로 선언한다.
    - 설계자의 의도를 명확히 드러내는 방법
  - 모든 필드를 private로 선언한다.
    - 클라이언트가 직접 접근해서 수정하는 것을 막는다.
  - 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
    - 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 한다.
  - 
```java
  public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
      this.re = re;
      this.im = im;
    }

    public double realPart() {
      return re;
    }

    public double imaginaryPart() {
      return im;
    }

    public Complex plus(Complex c) {
      return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
      return new Complex(re - c.re, im - c.im);
    }

    ...
  }
```
- 피연산자에 함수를 적용해서 결과를 반환하지만 피연산자 자체는 그대로인 프로그래밍 패턴을 함수형 프로그래밍이라고 한다.
  - 절차적 프로그래밍은 메소드에서 피연산자인 자신을 수정해서 자신의 상태가 변한다.

## 불변 객체의 장점
- 불변 객체는 단순하다.
  - 모든 생성자가 클래스 불변식을 보장한다면 그 클래스를 사용하는 프로그래머의 노력없이도 영원히 불변이다.
- 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요 없다.
  - 여러 스레드가 동시에 사용하더라도 훼손되지 않는다.
  - 불변 객체는 안심하고 공유할 수 있다.
  - 한번 만든 인스턴스를 재활용하는 것을 권장한다.
    - 자주 쓰이는 값들을 `public static final`을 사용한 상수로 제공
- 불변 클래스는 자주 사용되는 인스턴스를 중복 생성하지 않게 해주는 정적 팩토리를 제공할 수 있다.
  - 여러 클라이언트가 인스턴스를 공유함으로 메모리 사용량과 가비지 컬렉션 비용이 줄어든다.
- 불변 객체끼리는 내부 데이터를 공유할 수 있다.
  - BigInteger는 내부에서 값의 부호와 크기를 따로 표현한다.
    ```java
      public class BigInteger extends Number implements Comparable<BigInteger> {
        final int signum;//부호
        final int[] mag;//크기(절댓값)

        public BigInteger negate() {
          return new BigInteger(this.mag, -this.signum);
        }
      }

      BigInteger x = new BigInteger("1");
      BigInteger y = x.negate(); // y: -1
    ```
    - negate: 부호만 다른 새로운 BigInteger를 생성하는 메소드
    - 새로 만들어진 BigInteger y는 원본 x가 가리키는 내부 배열을 그대로 가리킨다.
- 불변 객체는 그 자체로 실패 원자성을 제공한다.
  - 상태가 변하지 않으므로 불일치 상태에 빠질 가능성이 없다.

## 불변 객체의 단점
> 값이 다르면 반드시 독립된 객체로 만들어야 한다.<br/>
값의 가짓수가 많다면 이들을 모두 만들어야 하고, 많은 비용이 든다.

```java
  // 100만 비트짜리 BigInteger에서 비트 하나를 변경한다고 가정
  BigInteger moby = ...;
  moby = moby.flipBit(0);
```
- 하나의 비트를 변경하는데도 100만 비트짜리 BigInteger 인스턴스를 하나 더 생성한다.
  - 이 연산은 BigInteger의 크기에 비례해 시간과 공간을 잡아먹는다.
- 원하는 객체를 만들기까지 단계가 많고, 중간 단계에서 만들어진 객체들이 모두 버려진다면 성능 문제가 커진다.
- 이 문제를 대처하는 방법
  - 다단계 연산들을 예측하여 기본 기능으로 제공하는 방법
    - BigInteger는 다단계 연산 속도를 높여주는 가변 동반 클래스를 package-private으로 두고 있다.
  - 예측이 안된다면 해당 클래스를 public으로 제공한다.
    - ex. String의 가변 동반 클래스인 StringBuilder, StringBuffer

## 불변 클래스를 만드는 설계 방법
- final 클래스로 선언하는 방법
- 모든 생성자를 private 혹은 package-private으로 만들고, public 정적 팩토리를 제공하는 방법
  ```java
    public class Complex {
      private final double re;
      private final double im;

      private Complex(double re, double im) {
        this.re = re;
        this.im = im;
      }

      public static Complex valueOf(doulble re, double im) {
        return new Complex(re, im);
      }

      ...
    }
  ```
  - 바깥에서 볼 수 없는 package-private 구현 클래스를 원하는만큼 만들어서 활용할 수 있다.
  - public 이나 protected 생성자가 없으니 다른 패키지에서는 이 클래스를 확장할 수 없어서 이 불변 객체는 사실상 final이다.

## BigInteger, BigDecimal
> 신뢰할 수 없는 클라이언트로부터 BigInteger or BigDecimal을 받는다면 주의해야 한다.

- 이 값들이 불변이어야 클래스의 보안을 지킬 수 있다면 객체가 진짜 BigInteger or BigDecimal이 맞는지 확인해야 한다.
- 신뢰할 수 없는 하위 클래스의 인스턴스라고 확인된다면 인수들을 가변이라고 가정하고 방어적으로 복사해서 사용해야 한다.

## 정리
- getter가 있다고 해서 setter를 무조건 만들지는 말자
- 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
- 모든 클래스를 불변으로 만들 수는 없으니 변경할 수 있는 부분을 최소한으로 줄이자
- 다른 합당한 이유가 없다면 모든 필드는 `private final`이어야 한다.
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.
  - 확실한 이유가 없다면 생성자와 정적 팩토리 외에는 어떤 초기화 메소드도 public으로 제공해서는 안된다.
