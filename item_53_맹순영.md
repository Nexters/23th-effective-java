## 아이템 53. 가변인수는 신중히 사용하라

### 가변인수란?

*매개변수로 들어오는 값의 개수와 상관없이 **동적으로 인수를 받아** 기능하도록 해주는 것*

가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다. 아래와 같이 동작

1. 인수의 개수와 길이가 같은 배열 생성
2. 인수들을 이 배열에 저장하여 가변인수 메서드에 건네줌

### 인수가 1개 이상이어야 할 경우 (최솟값 찾기) - 잘못된 방식

```java
// 코드 53-2 인수가 1개 이상이어야 하는 가변인수 메서드 - 잘못 구현한 예! (320쪽)
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}
```

- **런타임**에 배열의 길이로 length를 알 수 있음
- 컴파일 타임에 알 수 없다는 것이 포인트
- args 유효성 검사를 명시적으로 해야됨
- min의 초깃값을 Integer.MAX_VALUE로 초기화해야됨

### 인수가 1개 이상이어야 할 경우 (최솟값 찾기) - 올바른 방식

```java
// 코드 53-3 인수가 1개 이상이어야 할 때 가변인수를 제대로 사용하는 방법 (321쪽)
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min;
}
```

- firstArg와 나머지 Args를 따로 받음

### 가변 인수의 단점을 해결하는 패턴

- 가변인수 메서드는 **호출될 때마다 배열을 새로 하나 할당**하고 초기화함
    - ex) 메서드 호출의 95%가 인수가 3개 이하인 경우

    ```java
    public void foo() {}
    public void foo(int a1) {}
    public void foo(int a1, int a2) {}
    public void foo(int a1, int a2, int a3) {}
    public void foo(int a1, int a2, int a3, int... rest) {}
    ```

    - 5개의 foo를 다중정의
    - 3개까지는 일반 메서드가 담당을 하고 4개부터는 가변 인수 메서드 사용
    - 5%만 배열을 할당함!