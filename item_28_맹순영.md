## 아이템 28. 배열보다는 리스트를 사용하라

### 배열과 제네릭의 차이

### 1. 배열은 공변, 제네릭은 불공변

- 공변: Sub가 Super의 하위 타입이면 Sub[]는 Super[]의 하위 타입 (String[] 는 Object[] 의 하위 타입)
- 불공변: List<Type1>은 List<Type2>의 하위도 상위도 아님
  - List<String>은 List<Object>의 **하위 타입이 아님**

### 배열

```java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없음"; // 런타임 시 ArrayStoreException 발생
```

- Long은 Object의 하위 타입이라 컴파일 시 오류가 발생하지 않음

### 제네릭

```java
List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입
ol.add("타입이 달라 넣을 수 없음");
```

- ArrayList<Long>은 List<Object>의 하위가 아니므로 컴파일 시 오류

### 2. 배열의 실체화

- 배열은 **런타임에도** 자신이 담기로 한 원소의 **타입을 인지하고 확인**
    - 그래서 위의 코드에서 Long 배열에 String을 넣으려하면 Exception 발생함
- 제네릭은 **소거**(erasure) 때문에 **원소 타입을 컴파일타임에만 검사**함, 런타임에는 알 수 조차 없음

> 실체화라는 것은 런타임 시에도 타입을 인지할 수 있다는 것이라고 생각하면 될 듯하다
>

### 3. 추가적인 배열의 문제점

- 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없음
    - 제네릭 타입: new List<E>[]
    - 매개변수화 타입: new List<String>[]
    - 타입 매개변수: new E[]
        - 대신에 Object[] 로 생성하고 형변환하여 사용해야됨(by Gpt)
- 허용하지 않는 이유는 모두 다 타입 안전하지 않기 때문임
    - 형변환 코드에서 런타임 시 Exception 발생 가능

```java
List<String>[] stringLists = new List<String>[1]; // (1)
List<Integer> intList = List.of(42); // (2)
Object[] objects = stringLists; // (3)
objects[0] = intList; // (4)
String s = stringLists[0].get(0); // (5)
```

- (1)이 허용된다면(제네릭 배열 생성이 가능하다면),
- (3)에서 List<String>은 Object의 하위에 있기 때문에 가능함
- (4)에서 타입 소거 때문에 (2)에서 만든 intList를 objects[0]에 넣을 수 있음
    - 런타임 시 intList는 단순 List로 인식되고, List<Integer>[]는 List[]로 인식되기 때문
- List<String>만 담겠다고 선언한 stringLists가 List<Integer>를 담는 사고가 발생
- 이를 해결하려면 (1)에서 컴파일 오류를 내야됨!

### 추가적인 불편함

- 제네릭에서 자신의 원소 타입을 담은 배열을 반환하는게 보통은 불가능함

    ```java
    public <T> T[] createArray(int size) {
        return new T[size]; // 컴파일 오류 발생
    }
    ```

- 제네릭 타입과 가변 인수 메서드를 함께 쓰면 경고 메시지 받음
    - 가변인수 메서드: 매개변수 개수가 동적임
    - 가변 인수 메서드 호출할 때마다 가변인수 매개변수를 담을 배열이 하나 만들어짐 → 실체화 불가 타입이면 경고(제네릭 사용) → @SafeVarargs로 해결

## 그러니까 배열 대신 리스트 써라

- 코드가 좀 복잡해지고, 성능이 살짝 나빠질 순 있음

### 배열을 사용한 경우

```java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

- choose 메서드를 호출할 때마다 반환된 Object를 형변환해야 함
- 런타임 시 형변환 오류 위험

(참고) toArray 메서드

```java
public static void main(String[] args) {
    Collection<String> collection = new ArrayList<>();
    collection.add("Hello");
    collection.add("World");

    // toArray() 메서드 사용
    Object[] array1 = collection.toArray();
    System.out.println(array1[0]); // 출력: Hello

    // toArray(T[] a) 메서드 사용
    String[] array2 = collection.toArray(new String[0]);
    System.out.println(array2[1]); // 출력: World
}
```

- new String[0]을 넘겨주는 이유는 크기가 자동으로 조정되기 때문임

### 제네릭으로 만들기 - 1

```java
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

- 위의 코드는 컴파일시 오류발생함
    - 위의 toArray()를 보면 Object[]를 반환함
    - 그러므로 choices.toArray(); 에서 Object 배열을 반환하기 때문에 T 배열로 형변환 해야됨 → (T[]) choices.toArray()
- 하지만 T[]로 형변환 해도 경고가 뜸
    - T가 무슨 타입인지 알 수 없기 때문에 런타임 시 안전을 보장할 수 없다.
    - 동작은 하지만 안전하지 않다.
    - 주석을 남기고 애너테이션을 달아 경고를 숨겨도 되지만 원인 제거가 더 중요!

### 제네릭과 리스트를 사용하기

```java
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

- 코드가 좀 늘고, 조금 느리지만 런타임 시 오류 발생할 일이 없음!

### 핵심 정리

- 배열은 공변, 제네릭은 불공변
- 배열은 런타임에 타입 안전하지만, 컴파일 타입에 그렇지 못함
- 제네릭은 컴파일 타임에 타입 안전함
- 둘을 섞어 쓰다가 컴파일 오류나 경고 생기면 배열을 리스트로 대체하자