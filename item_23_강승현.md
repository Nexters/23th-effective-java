# Summary

---

- 태그 달린 클래스의 단점
    - 태그 클래스?
        
        하나의 클래스가 두 개 이상의 역할을 수행하도록 설계된 클래스, **다형성 X**
        
    - 쓸데없는 코드가 많다
        - 열거 타입 선언
        - 태그 필드
        - switch 문
    - (여러 구현이 한 클래스에 혼합되어 있어서) **가독성이 나쁘다**
    - 메모리도 많이 사용한다
        - 사각형만 필요한데 원의 메모리도 필요하기 때문
    - 필드를 final로 선언하려면 불필요한 필드까지 초기화해야 한다
    - 인스턴스 타입만으로는 현재 나타내는 의미를 알 길이 없다
- 클래스 계층 구조(서브타이핑 subtyping)로 바꾸면 모든 것을 해결할 수 있다

# To-Be

---

```java

// 코드 23-1 태그 달린 클래스 - 클래스 계층구조보다 훨씬 나쁘다! (142-143쪽)
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}

```

- **Q. 위 코드에서 다른 도형을 추가한다고 가정해보자 어떻게 구현할 것인가?**
    - 삼각형이 추가되면 enum에 하나 추가됨
    - length뿐 아니라 height도 필요하게 됨 (밑변 * 높이 / 2 )
    - 삼각형용 생성자가 하나 더 필요함
    - area에 switch문 추가하고 다른 연산 로직이 들어감

# As-Is

---

```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}
```

- `JsonSubType`