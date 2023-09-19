# item50 적시에 방어적 복사본을 만들라.

클래스가 **가변 필드**를 가진 경우 외부에서 내부를 수정할 수 있기 때문에 주의해야 된다.

```java
public class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦습니다");
        }
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }
}
```

## 첫번째 공격

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부 수정!
```

`Date` 가 가변이므로 외부에서 내부의 값을 변경할 수 있다.

(Java 8 이상에서는 Date 대신 Instant, LocalDateTime, ZonedDateTime 등 불변 값들을 사용해서 해결 가능)

### 해결법(defensive copy-방어적 복사)

```java
public Period(Date start, Date end) {
	this.start = new Date(start.getTime());
  this.end = new Date(end.getTime());

  if (start.compareTo(end) > 0) {...}
}
```

- 생성자에 받은 가변 매개변수를 **방어적 복사**를 통해 수정되는 것을 막을 수 있다.
- 방어적 복사본을 만들고, **복사본의 유효성 검사**를 진행하자.
  → **멀티 스레드 환경**에서 먼저 유효성 검사를 하게 되면, 복사본을 만드는 찰나에 원본 객체가 수정될 위험이 있음.
- 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들때,
  **clone**을 사용하지 말자.

## 두번째 공격

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78); // p의 내부 수정!
```

접근자 메서드가 내부의 가변 정보를 그대로 들어내면 외부에서 값을 변경할 수 있다.

### 해결법(defensive copy-방어적 복사)

```java
public Date start() {
   return new Date(start.getTime());
}
public Date end() {
   return new Date(end.getTime());
}
```

- 가변 필드의 복사본을 반환.
- 생성자와 달리 접근자 메서드에서는 방어적 복사에 **clone**를 사용해도 된다.

참고로, 길이가 1이상인 배열은 반드시 가변이므로, 내부에서 사용하는 배열을 반환할 때는 항상 **방어적 복사** or **불변 뷰**을 반환해야된다.

방어적 복사 비용이 크거나 클라이언트가 해당 요소를 잘못 수정할 일이 없음을 신뢰한다면 방어적 복사 대신, 수정에 따른 책임을 클라이언트에 있음을 문서에 명시하자.