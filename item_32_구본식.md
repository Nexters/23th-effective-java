# item32.제네릭과 가변인수를 함께 쓸 때는 신중하라.

## 제네릭과 varargs를 혼용하면 타입 안전성이 깨진다!

```java
//예시1
static void dangerous(List<String>... stringLists){
        List<Integer> intList = List.of(42);
        Object[] objects = stringLists;
        objects[0] = intList;
        String s = stringLists[0].get(0);
}
```

**무엇이 문제일까??**

- objects 배열은 공변이기 때문에 List[] → Object[]가 참조 가능하다.
  > 가변인수 메서드를 사용하면 가변인수를 담기 위한 배열을 자동으로 생성한다.(varargs)
  여기서, objects[] 을 varargs라 생각하면 된다.

- objects[0] = intList로 초기화
  실제론 List<String> 타입이지만, 제네릭은 **소거 방식**을 사용하므로,
  런타임에는 List<Integer> 인스턴스의 타입이 **단순 List**로 인식되어 할당이 가능하다.
  ⇒ **힙 오염** 발생
- stringLists[0].get(0) 호출 시 컴파일러가 내부적으로 **형변환 → ClassCastException** 발생
  매개변수화 타입: String , 실제 참조 타입:Integer

런타임에 **타입 안정성**을 지켜주고자 하는 제네릭 타입 시스템에 어긋.

제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.

> 보통 varargs 형태로 받아와서 그대로 사용할텐데, 실제 이런 경우는 흔하지 않을 거 같다.
>

## @SafeVarargs

- 자바 7전에는 제네릭 가변인수 메서드의 작성자가 호출자 쪽에서 발생하는 경고에 대해서 해줄 수 있는 일이 없었다.
- 사용자는 경고를 그냥 두거나, @SuppressWarnings(”unchecked”) 애너테이션으로 숨겨야 했음.
- **@SafeVarargs** 은 메서드가 타입 안전함을 보장하는 장치.
  단, 메서스 타입이 안전함을 보장할 때 사용.

### 어떤 경우가 안전할까?

- varargs 매개변수 배열에 아무것도 저장하지 않고(덮어쓰지도 않고, 예시1),
  그 배열의 참조를 밖으로 직접 노출시키지 않는다.(예시2)
- varargs 목적에 맞게, 순수하게 인수를 전달하는 역할만 수행한다.

### 제네릭 매개변수 배열의 참조를 노출하는 것은 안전하지 않다.

```java
//예시2
static <T> T[] toArray(T... args){
        return args;
    }

    static <T> T[] pickTwo(T a, T b ,T c){
        switch (ThreadLocalRandom.current().nextInt(3)){
            case 0:
                return toArray(a, b);
            case 1:
                return toArray(a, c);
            case 2:
                return toArray(b,c);
        }
        throw new AssertionError();
    }

    public static void main(String[]args){
        String[] attributes = pickTwo("좋은", "빠른", "저렴한");
        
    }
```

**무엇이 문제일까??**

`String[] attributes = pickTwo()` 메서드에서 **형변환시** 문제가 발생한다.(ClassCastException)

`toArray()` 메서드는 자신의 **varargs 배열**을 그대로 반환한다.(즉, **Object[] 배열**)
String[]로 현변환하는 코드를 컴파일러가 자동으로 생성하고,
Object[]은 String[]의 **하위 타입이 아니므로**, 형변환에 실패한다.

단 아래와 같은 예외에서는 안전하게 사용해도 된다.

- `@SafeVarargs` 로 제대로 애노테이트된 다른 varargs 메서드에 넘기는 것.
- 배열 내용의 일부 함수를 호출만 하는(varargs를 받지 않는) 일반 메서드에 넘기는 것.
    - 예시

        ```java
        public void printNumbers(int... numbers) {
            for (int number : numbers) {
                System.out.print(number + " ");
            }
        }
        public void processArray(int[] arr) {
            printNumbers(arr); // 배열을 varargs 메서드에 전달
        }
        ```



## 제네릭 varargs 매개변수를 안전하게 사용하는 메서드

```java
//@SafeVarargs 애노테이션을 사용하는 방식
@SafeVarargs
    static <T> List<T> flatten(List<? extends T>...lists){
        List<T> result = new ArrayList<>();
        for(List<? extends T> list : lists){
            result.addAll(list);
        }
        return result;
    }
```

varargs 배열을 **직접 노출시키지 않고**, (제네릭)**한정적 와일드 카드 타입을** 사용했기 때문에,
형변환 예외가 발생할 일이 없다.

안전한 varargs 메서드일 경우 `@SafeVarargs` 로 경고를 없애는 것이 좋다.

## 제네릭 varargs 매개변수를 List로 대체해라.

```java
//실제는 배열인 varargs 매개변수를 List 매개변수로 바꾸는 방식
static <T> List<T> flatten_2(List<List<? extends T>> lists){
        List<T> result = new ArrayList<>();
        for(List<? extends T>list : lists){
            result.addAll(list);
        }
        return result;
    }
```

`List.of` 를 활용해서 해당 메서드에 임의 개수의 인수를 넘기면 된다.

컴파일러가 타입 안전성을 검증하므로, 안전한 방법.

직접 `@SafeVarargs` 애노테이션을 달지 않아도 되고, 안전성을 판단할 걱정도 없다.

단점으로는, 클라이언트 코드가 지저분해지며 속도가 조금 느려질 수 있다.

→ List로 변환 작업이 필요하므로.

## 정리

- varargs 매개변수는 단순히 파라미터를 받아와 T 타입의 객체를 제공하는 용도로만 사용하자.
- 제네릭과 함께 사용 시, varargs에 아무런 데이터를 저장하지 말자.
- varargs 배열을 외부에 노출시키지 않고, 왠만하면 List(컬렉션)으로 담자.