# equals는 일반 규약을 지켜 재정의하라
## 재정의가 필요없는 경우
- 각 인스턴스가 본질적으로 고유하다.
  - 값을 표현하는 게 아닌 동작하는 개체를 표현하는 클래스가 해당 (ex. Thread)
- 인스턴스의 논리적 동치성을 검사할 일이 없다.
  - ex. java.util.regex.Pattern은 equals를 재정의해서 두 Pattern의 정규표현식을 비교
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는 경우
  - ex. AbstractSet이 구현한 equals를 사용하는 Set
- 클래스가 private이거나 package-private이고, equals 메소드를 호출할 일이 없는 경우
  - equals가 실수로라도 호출될 수 없도록 구현할 수도 있다.
    ```java
      @Override
      public boolean equals(Object o) {
        throw new AssertionError(); // 호출 금지
      }
    ```
## 재정의가 필요한 경우
> 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때

- Integer, String같은 값을 표현하는 값 클래스들이 해당
- equals가 논리적 동치성을 확인하도록 재정의하면 값 비교 뿐만 아니라 Map의 key, Set의 원소로 사용할 수 있다.
- 값 클래스라도 해도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 equals를 재정의하지 않아도 된다.
  - ex. Enum

## equals 메소드를 재정의할 때 따라야할 규약
> equals 메소드는 동치관계를 구현한다.

- 반사성 (reflexivity): null이 아닌 모든 참조 값 x에 대해 x.equals(x)는 true다.
- 대칭성 (symmetry): null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true면 y.equals(x)도 true다.
- 추이성 (transitivity): null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고, y.equals(z)도 true면, x.equals(z)도 true다.
- 일관성 (consistency): null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true거나 false다.
- null 아님: null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 false다.

### 반사성
- 객체는 자기 자신과 같아야 한다.
- 반사성을 만족하지 못하는 경우 컬렉션의 contains 메소드 호출 시 false가 나올 것이다.
  - ex. set.contains(x)

### 대칭성
```java
  public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
      this.s = Objects.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
      if (o instanceof CaseInsensitiveString) {
        return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
      }

      if (o instanceof String) {
        return s.equalsIgnoreCase((String) o);
      }

      return false;
    }
  }
```
```java
  CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
  String s = "polish";

  cis.equals(s); // true 반환
  s.equals(cis); // false 반환
```
- CaseInsensitiveString의 equals에는 String에 대한 처리를 했지만 String의 equals는 CaseInsensitiveString의 존재를 모른다.
- CaseInsensitiveString와 String을 비교하지 말고, CaseInsensitiveString끼리 비교하도록 수정하여 해결한다.
  ```java
    @Override
    public boolean equals(Object o) {
      return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }
  ```

### 추이성
```java
  public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
      this.x = x;
      this.y = y;
    }

    @Override
    public boolean equals(Object o) {
      if (!(o instanceof Point)) {
        return false;
      }

      Point p = (Point) o;
      return p.x == x && p.y == y;
    }
  }

  public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
      super(x, y);
      this.color = color;
    }

    @Override
    public boolean equals(Object o) {
      if (!(o instanceof Point)) {
        return false;
      }

      if (!(o instanceof ColorPoint)) {
        return o.equals(this);
      }

      return super.equals(o) && ((ColorPoint) o).color == color;
    }
  }
```
```java
  ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
  Point p2 = new Point(1, 2);
  ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

  p1.equals(p2); // true
  p2.equals(p3); // true
  p1.equals(p3); // false
```
- 이 방식은 무한 재귀의 가능성이 있음
  - Point의 하위 클래스 SmellPoint를 만들어서 equals를 같은 방식으로 구현 후 `colorPoint.equals(smellPoint)`를 호출하면 스택 오버플로우 에러 발생
- 구체 클래스를 확장해서 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.
```java
  // 잘못된 코드 - 리스코프 치환 원칙 위배
  @Override
  public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass()) {
      return false;
    }

    Point p = (Point) o;
    return p.x == x && p.y == y;
  }
```
- 리스코프 치환 원칙: 부모 객체를 호출하는 동작에서 자식 객체가 부모 객체를 완전히 대체할 수 있는 원칙
- `o.getClasS() != getClass()` 직접 클래스를 비교하는 로직
  - 타입이 정확이 일치해야함 -> 하위 클래스는 false

```java
  public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
      point = new Point(x, y);
      this.color = Objects.requireNonNull(color);
    }

    // 이 ColorPoint의 Point 뷰를 반환한다.
    public Point asPoint() { // view 메서드 패턴
      return point;
    }

    @Override
    public boolean equals(Object o) {
      if (!(o instanceof ColorPoint)) {
        return false;
      }

      ColorPoint cp = (ColorPoint) o;
      return cp.point.equals(point) && cp.color.equals(color);
    }
  }
```
- 상속 대신 컴포지션을 사용하는 해결방법
  - 컴포지션: 한 객체가 다른 객체를 포함하고 있는 관계
  - Point를 상속하는 대신 Point를 ColorPoint의 private 필드로 두고, ColorPoint와 같은 위치의 일반 Point를 반환하는 뷰 메소드를 public으로 추가

### 일관성
- 두 객체가 같으면 수정되지 않는 한 영원히 같아야 한다.
- 가변 객체는 비교 시점에 따라 다를 수 있지만 불변 객체는 영원히 같거나 달라야 한다.
- 클래스가 불변/가변 여부에 상관없이 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다.
  - java.net.URL의 equals는 주어진 URL과 매핑된 호스트의 IP주소를 이용해 비교한다.
  - 호스트 이름을 IP주소로 바꾸려면 네트워크를 통해야 하므로 결과가 항상 같다고 보장할 수 없다.
  - 이런 문제를 피하려면 equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다.

### null-아님
- 모든 객체가 null과 같지 않아야 한다.
```java
  // 명시적 null 검사 필요없음
  @Override
  public boolean equals(Object o) {
    if (o == null) {
      return false;
    }

    ...
  }
```
```java
  // 묵시적 null 검사 이쪽이 나음
  @Override
  public boolean equals(Object o) {
    if (!(o instanceof MyType)) {
      return false;
    }

    ...
  }
```
- `o == null`이면 false를 반환하므로 명시적으로 null 검사를 하지 않아도 된다.

### 정리 (equals 메소드 구현 방법)
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    - 자기 자신이면 true를 반환한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    - 인터페이스는 자신을 구현한 클래스끼리도 비교할 수 있도록 equals 규약을 수정하기도 한다.
      - 인터페이스를 구현한 클래스라면 equals에서 해당 인터페이스를 사용해야 한다.
      - ex. Set, List, Map, Map.Entry 등의 컬렉션 인터페이스
3. 입력을 올바른 타입으로 형변환한다.
    - 2에서 instanceof 검사를 했으므로 100% 성공하는 단계
4. 입력 객체와 자기 자신의 대응하는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.
    - 모든 필드가 일치해야 true, 아니면 false를 반환한다.

## 비교 방법
- float, double을 제외한 기본 타입 필드
  - `==` 연산자로 비교한다.
- 참조 타입 필드
  - 각각의 equals 메소드로 비교한다.
- float, double
  - Float.NaN, -0.0f or 특수한 부동소수 값을 다뤄야하기 한다.
  - `Float.compare(x, y)` `Double.compare(x, y)` 메소드로 비교한다.
- 배열 필드는 원소 각각을 위 지침대로 비교한다.
  - 배열의 모든 원소가 핵심 필드라면 Arrays.equals 메소드를 사용한다.
- null도 정상 값으로 취급하는 참조 타입 필도가 있다.
  - Objects.equals(Object, Object) 메소드로 비교해서 NullPointerException을 예방한다.
- 비교하기 복잡한 필드를 가진 클래스
  - 그 필드의 표준형을 저장한 후 표준형끼리 비교한다.
- 어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 한다.
  - 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교한다.

## 주의
- equals를 재정의할 때 hashCode도 반드시 재정의하자
- 너무 복잡하게 해결하려고 하지 말자
  - 필드들의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있다.
  - 별칭(alias)은 비교하지 않는 것이 좋음
- Object 외의 타입을 매개변수로 받는 equals 메소드는 선언하지 말자
  - ex. `public boolean equals(MyClass o)`

## AutoValue 프레임워크
- equals를 작성하고 테스트하는 작업을 대신해주는 오픈소스
- 클래스에 @AutoValue를 추가하면 메소드를 알아서 작성해준다.
