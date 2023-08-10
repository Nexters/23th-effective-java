## 아이템 34. int 상수 대신 열거 타입을 사용하라

### 기존 방식 1 - 정수 열거 패턴

```java
public static final int APPLE_FUJI = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int APPLE_PIPPIN = 1;
```

- 타입 안전 보장 X
- 오렌지 대신 사과를 써도 컴파일러는 모름 (그냥 상수라)
- 정수 열거 패턴을 사용한 프로그램은 깨지기 쉬움
    - 단순 상수 저장 → 바뀌면 다시 컴파일 해야됨

### 기존 방식 2 - 문자열 열거 패턴

- 더 안좋음
- 상수의 의미 출력해서 좋지만, 문자열 값 하드코딩
- 오타 있으면 런타임에 오류

### 최고의 대안 Enum

```java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
public enum Orange {NAVEL, TEMPLE, BLOOD}
```

- 완전한 형태의 클래스
    - 원리(?)
        - 상수 하나당 자신의 인스턴스를 하나씩 만들어서 public static final 필드로 공개
        - 생성자 없어서 딱 하나만 존재함
- 타입 안전성 제공
    - 위 코드의 Apple 열거 타입을 매개변수로 받는 메서드 작성하면, 건네받은 참조는 위의 세 가지 값 중 하나임이 확실함 → 다른 타입의 값 넘기면 컴파일 오류
- 이름이 같은 상수도 공존할 수 있음
    - 각자의 이름공간이 있음
- 새로운 상수 추가, 순서 바꿔도 컴파일할 필요 없음
    - 필드의 이름만 공개돼서 정수 열거 패턴에서처럼 상수 값이 각인되지 않기 때문
- toString 메서드 쓸 때 좋음
- 임의의 메서드나 필드를 추가할 수 있고 인터페이스 구현도 가능
    - Object 메서드
    - Comparable
    - Serializable

### 열거 타입에 메서드나 필드 추가하기

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```

- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 **생성자와 데이터를 받아 인스턴스 필드에 저장**하면 됨
    - 이때 모든 필드는 불변이므로 final이어야 함
    - public으로 열어도 되지만, 아이템 16(public 클래스에서는 public 필드가 아닌 접근자 메서드 사용)의 원칙 지키는게 좋음

엄청 단순하다.

```java
// 어떤 객체의 지구에서의 무게를 입력받아 여덟 행성에서의 무게를 출력한다. (212쪽)
public class WeightTable {
   public static void main(String[] args) {
      double earthWeight = Double.parseDouble(args[0]);
      double mass = earthWeight / Planet.EARTH.surfaceGravity();
      for (Planet p : Planet.values())
         System.out.printf("%s에서의 무게는 %f이다.%n",
                           p, p.surfaceWeight(mass));
   }
}
```

- values() 쓸 수 있음
    - values()는 정의된 상수들의 값을 배열에 담아 반환함
    - 선언된 순서대로 저장됨
- toString은 상수 이름을 문자열로 반환해줌

### 열거 타입에서 상수 하나를 제거하면 어떻게 되나?

- 다시 컴파일하면 제거된 상수를 참조하면 컴파일 오류 발생, 컴파일하지 않으면 런타임에 오류 발생
    - 정수 열거 타입에서는 기대할 수 없는 대응

### 열거 타입 선언하는 방법

- 클래스 혹은 패키지에서만 유용하다면 private이나 package-private 메서드로 구현
- 널리 쓰이는 열거 타입은 톱레벨 클래스로 만들기
- 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만들기

### 상수마다 동작이 달라지는 경우

- switch 문 사용하기
  1. 코드가 안이쁨
  2. 상수 추가하면 case 문도 추가해줘야됨
- apply 추상 메서드 선언 - 상수별 메서드 구현

    ```java
    public enum Operation {
        PLUS {public double apply(double x, double y) { return x + y; }},
        MINUS {public double apply(double x, double y) { return x - y; }},
        TIMES {public double apply(double x, double y) { return x * y; }},
        DIVIDE {public double apply(double x, double y) { return x / y; }};
    
    		public abstract double apply(double x, double y); // 추상 메서드 선언
    }
    ```

  - apply가 추상 메서드라서 상수 선언에서 apply 재정의하지 않으면 컴파일 오류

- toString 재정의

    ```java
    public enum Operation {
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
    		Operation(String symbol) { this.symbol = symbol; }
    		@Override public String toString() { return symbol; }
    
    		public abstract double apply(double x, double y); // 추상 메서드 선언
    }
    ```

- fromString을 함께 구현하면 좋음

    ```java
    // 코드 34-7 열거 타입용 fromString 메서드 구현하기 (216쪽)
    private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(
                    toMap(Object::toString, e -> e));
    
    // 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }
    ```

  - 열거 타입 생성 후 정적 필드가 초기화될 때 Operation 상수가 stringToEnum 맵에 추가됨
  - 열거 타입의 각 상수는 public static final 필드로 선언되어 있음
    - 생성자가 실행되는 시점에서는 정적 필드들(열거 타입의 상수들)이 초기화되기 전이라 자기 자신을 추가하지 못하게 해야됨


### 열거 타입 상수끼리 코드를 공유하기

1. switch

    ```java
    enum PayrollDay {
            MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
            SATURDAY, SUNDAY;
    
            private static final int MINS_PER_SHIFT = 8 * 60;
    
            int pay(int minutesWorked, int payRate) {
                    int basePay = minutesWorked * payRate;
    
                    int overtimePay;
                    switch(this) {
                        case SATURDAY: case SUNDAY: // 주말
                            overtimePay = basePay / 2;
                            break
                        default: // 주중
                            overtimePay = minutesWorked <= MINS_PER_SHIFT ?
                                0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
            return basePay + overtimePay
    }
    ```

   - 새로운 값을 추가하려면 case 또 추가 해줘야돼서 안쓰는게 좋음
2. 잔업 수당을 계산하는 코드(공통되는 코드)를 모든 상수에 중복해서 넣기
3. 도우미 메서드로 작성

⇒ 모두 코드가 장황하고 가독성 떨어짐

그럼 어떻게 해야될까?

### 새로운 상수를 추가할 때 ‘전략’을 선택하도록 하기

- 잔업수당 계산을 private 중첩 열거 타입(PayType)으로 옮기고 PayrollDay 열거 타입의 생성자에서 이 중 적당한 것을 선택
  - Enum의 Enum 사용

```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```

```java
public static void main(String[] args) {
    for (PayrollDay day : values())
        System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
}
```

### 그래서 언제 쓰라고?

- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합인 경우
  - 태양계 행성, 한 주의 요일, 체스 말 등
- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없음

### 핵심 정리

- 열거 타입은 읽기 쉽고 안전하고 강력함
- 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작 → **명시적 생성자나 메서드**가 필요
- 하나의 메서드가 상수 별로 다르게 동작 →  switch 보다는 **상수별 메서드 구현** 사용하기
- 열거 타입 상수 일부가 같은 동작을 공유 → **전략 열거 타입 패턴** 사용하기