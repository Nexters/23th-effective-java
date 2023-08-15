# 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라
- 열거 타입은 거의 모든 상황에서 타입 안전 열거 패턴보다 우수하다.
  - 예외는 타입 안전 열거 패턴은 확장할 수 있으나 열거 타입을 그럴 수 없다.
- 확장할 수 있는 열거 타입이 연산 코드에 사용될 수 있다.
  - 연산 코드의 각 원소는 특정 기계가 수행하는 연산을 뜻함
  - API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가할 때가 있다.

## 열거 타입이 임의의 인터페이스를 구현
```java
  public interface Operation {
    double apply(double x, double y);
  }

  public enum BasicOperation implements Operation {
    PLUS("+") {
      public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
      public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
      public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
      public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
      this.symbol = symbol;
    }

    @Override
    public String toString() {
      return symbol;
    }
  }

  public enum ExtendedOperation implements Operation {
    EXP("^") {
      public double apply(double x, double y) {
        return Math.pow(x, y);
      }
    },
    REMAINDER("%") {
      public double apply(double x, double y) {
        return x % y;
      }
    };

    private final String symbol;

    ExtendedOperation(String symbol) {
      this.symbol = symbol;
    }

    @Override
    public String toString() {
      return symbol;
    }
  }
```
- 연산 코드용 인터페이스를 정의하고 열거 타입이 인터페이스를 구현하도록 한다.
- BasicOperation은 확장할 수 없지만 Operation은 인터페이스이므로 확장할 수 있고, 이 인터페이스를 타입으로 사용한다.
- 새로 작성한 연산은 Operation 인터페이스를 사용하도록 작성되어 있기만 하면 사용할 수 있다.
- 열거 타입끼리 구현을 상속할 수 없다.
  - 아무 상태에도 의존하지 않는 경우에는 디폴트 구현을 이용해서 인터페이스에 추가하는 방법이 있다.
  - 열거 타입 간의 공유하는 기능이 많다면 별도의 도우미 클래스나 정적 클래스 메소드로 분리하는 방식으로 코드 중복을 줄일 수 있다.

## 확장된 열거타입 사용
### Class 객체를 넘기는 방법
```java
  public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
  }

  private static <T extends Enum<T> & Operation> void test(
    Class<T> opEnumType, double x, double y) {
      for (Operation op: opEnumType.getEnumConstants()) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
      }
    }
```
- test 메소드에 ExtendedOperation의 class 리터럴을 넘겨서 확장된 연산들이 무엇인지 알려준다.
- `<T extends Enum<T> & Operation>` `Class<T>`: Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다.
- 열거 타입이어야 원소를 순회할 수 있고, Operation이어야 원소가 뜻하는 연산을 수행할 수 있다.

### 한정적 와일드카드 타입을 넘기는 방법
```java
  public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
  }

  private static void test(Collection<? extends Operation> opSet, double x, double y) {
    for (Operation op: opSet) {
      System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
  }
```
- 코드가 덜 복잡하고, test 메소드가 유연해진다.
- 특정 연산에서는 EnumSet과 EnumMap을 사용하지 못한다.

## 정리
- 열거 타입 자체는 확장할 수 없지만 인터페이스와 기본 열거 타입을 함께 사용해서 같은 효과를 낼 수 있다.
- 클라이언트는 인터페이스를 구현해서 자신만의 열거 타입을 만들 수 있다.
- API가 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다.
