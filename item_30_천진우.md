# 이왕이면 제네릭 매서드로 만들라

### 선 요약
타입에 대한 다형성을 제공하는 유연한 매서드를 만들기 위해서, 제네릭을 활용할 수 있다!

### 유연한 파라미터를 제공하는 Bad Case

**1. 로 타입 사용 사용하지 않기 (item 26 참고!)**

책에서 나온 예시입니다.
로 타입을 활용했기 때문에, 실행은 가능하지만 warning 발생!

``` java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

예를들어, s1에는 Set<Double> 을 넣고, s2에는 Set<Integer> 를 넣었을 때, 장애가 발생할 수 있다 !

**2. Object 사용하는 경우**

마찬가지로 타입에 대한 검사가 불가능하기 때문에 안전하지 않고, 직접 타입 검사 로직을 넣어주는 방식을 사용해야합니다.

``` java
public static Set<Object> union(Set<Object> s1, Set<Object> s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

**3. 오버로딩**

오버로딩도 매소드에서 다형성을 제공하는 방식 중 하나.

타입에 대해서는 안전함! 그렇지만 타입에 대한 사용이 제한적이고, 내부 구현이 같아도 일일이 구현에 대해서 신경써야한다.

``` java
// 정수형 매개변수를 받는 오버로딩된 메서드
public void printNumber(int num) {
    System.out.println("Integer: " + num);
}

// 실수형 매개변수를 받는 오버로딩된 메서드
public void printNumber(double num) {
    System.out.println("Double: " + num);
}

// 문자열 매개변수를 받는 오버로딩된 메서드
public void printNumber(String str) {
    System.out.println("String: " + str);
}
```

### 그래서 제네릭 매서드를 사용하자!
- 타입에 대해서 유연함 (다형성)
- 타입에 대해서 안전함

``` java
public static <E> Set<E> union (Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

생각보다 햇갈렸던 것
- \<E>: 나는 E 라는 타입을 사용할 것이다 !
- \Set<E>: 리턴값은 이거다!


제네릭 매서드에서 여러개의 타입을 사용할 수도 있다.

```
// 두 종류의 제네릭 타입을 사용하는 메서드 정의
public <T, U> void printPair(T first, U second) {
    System.out.println("First: " + first);
    System.out.println("Second: " + second);
}
```

### 제네릭 싱글턴 패턴

UnaryOperator로 살펴보자.
- 함수형 인터페이스.
- 값을 넣으면 그대로 반환
``` java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {

    static <T> UnaryOperator<T> identity() {
        return t -> t;
    }
}
```

어디에다가 쓰는걸까
``` java
void multiply() {
    UnaryOperator<Integer> unaryOperator = i -> i * 2;

    assertThat(unaryOperator.apply(1)).isEqualTo(2);  // 1*2 = 2
    assertThat(unaryOperator.apply(2)).isEqualTo(4);  // 2*2 = 4
    assertThat(unaryOperator.apply(3)).isEqualTo(6);  // 3*2 = 6
    assertThat(unaryOperator.apply(4)).isEqualTo(8);  // 4*2 = 8
}
```

Stream.iterate 에서도 사용하고 있다!
두번째 파리미터!
``` java
// 1~10 출력
void iterate() {
    Stream.iterate(1, n->n + 1)
        .limit(10)
        .forEach(System.out::println);
}
```

### 재귀적 타입 한정
제네릭으로 사용할 수 있는 타입을 '한정'시킨다

책 예제
``` java
public static <E extends Comparable<E>> E max(Collection<E> c);
```

그냥 만들어본 예제
``` java
Class Bird extends Flyable {
}

Class BehaivorUtil {
    public static <E extends Flyable> void 철새_그룹_이동_명령(List<E> birds);
}

```
