## 아이템 39. 명명 패턴보다 애너테이션을 사용하라

### 명명 패턴

- 오타가 나면 안된다.
    - JUnit 버전 3에서 테스트 메서드 이름을 test로 시작하지 않으면 무시했음
- 올바른 프로그램 요소에서만 사용되리라는 보증이 없다.
    - 메서드가 아닌 클래스 이름을 Test로 지어도 JUnit은 관심이 없음
- 프로그램 요소를 매개변수로 전달할 방법이 딱히 없다.

### 애너테이션

```java
/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

- 애너테이션 선언에 다는 애너테이션을 **메타애너테이션**이라고 함
    - @Retention: @Test가 런타임에도 유지되어야 한다는 뜻
        - @Retention이 없다면, 테스트 도구는 @Test를 인식할 수 없음
    - @Targe(ElementType.METHOD): @Test가 메서드 선언에만 사용돼야 한다는 뜻

### 실제 적용 예

```java
// 코드 39-2 마커 애너테이션을 사용한 프로그램 예 (239쪽)
public class Sample {
    @Test
    public static void m1() { }        // 성공해야 한다.
    public static void m2() { }
    @Test public static void m3() {    // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    public static void m4() { }  // 테스트가 아니다.
    @Test public void m5() { }   // 잘못 사용한 예: 정적 메서드가 아니다.
    public static void m6() { }
    @Test public static void m7() {    // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    public static void m8() { }
}
```

- @Test 애너테이션을 아무 매개변수 없이 단순히 대상에 마킹한다해서 **마커 애너테이션**이라고 함
- m1은 성공, m3, m7은 실패, m5는 잘못 사용함
    - m5: 정적 메서드 전용 애너테이션인데 이를 무시했음

### 테스트 테스트(?)

```java
public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n",
                passed, tests - passed);
    }
```

- @Test 애너테이션은 Sample 클래스의 의미에 직접적인 영향을 주지 않음
    - 그저 @Test 애너테이션에 관심있는 프로그램에게 추가 정보를 제공할 뿐임
- 동작 방식
    1. Class.forName: Sample 클래스를 리플렉션을 통해 로드함
    2. getDeclaredMethods: Sample 클래스의 모든 메서드를 가져옴
    3. isAnnotationPresent: 현재 메서드가 @Test 애너테이션을 가지고 있는지 확인
    4. invoke(null): 현재 메서드를 호출하여 실행 (null은 정적 메서드를 호출하기 위해 사용됨)
    5. getCause: 원래 예외에 담긴 실패 정보 추출

### 특정 예외를 던져야만 성공하는 테스트

```java
/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

- Throwable을 확장한 클래스의 Class객체?
    - 모든 예외와 타입을 수용함

### 사용 방법

```java
// 코드 39-5 매개변수 하나짜리 애너테이션을 사용한 프로그램 (241쪽)
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {  // 성공해야 한다.
        int i = 0;
        i = i / i;
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {  // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }  // 실패해야 한다. (예외가 발생하지 않음)
}
```

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (InvocationTargetException wrappedEx) {
        Throwable exc = wrappedEx.getCause();
        Class<? extends Throwable> excType =
                m.getAnnotation(ExceptionTest.class).value();
        if (excType.isInstance(exc)) {
            passed++;
        } else {
            System.out.printf(
                    "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                    m, excType.getName(), exc);
        }
    } catch (Exception exc) {
        System.out.println("잘못 사용한 @ExceptionTest: " + m);
    }
}
```

- Sample1 예시와 달리 애너테이션 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인함

### 예외를 여러 개 명시하고 그 중에 하나가 발생했을 때 성공하는 테스트 - 배열 사용

```java
// 코드 39-6 배열 매개변수를 받는 애너테이션 타입 (242쪽)
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();
}
```

- 배열로 수정함

```java
// 배열 매개변수를 받는 애너테이션을 사용하는 프로그램 (242-243쪽)
public class Sample3 {
    // 코드 39-7 배열 매개변수를 받는 애너테이션을 사용하는 코드 (242-243쪽)
    @ExceptionTest({ IndexOutOfBoundsException.class,
                     NullPointerException.class })
    public static void doublyBad() {   // 성공해야 한다.
        List<String> list = new ArrayList<>();

        // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
        // NullPointerException을 던질 수 있다.
        list.addAll(5, null);
    }
}
```

- 여러 개를 중괄호로 감싸서 구분해주면 됨

### 사용 방법

```java
// 배열 매개변수를 받는 애너테이션을 처리하는 코드 (243쪽)
if (m.isAnnotationPresent(ExceptionTest.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (Throwable wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        int oldPassed = passed;
        Class<? extends Throwable>[] excTypes =
                m.getAnnotation(ExceptionTest.class).value();
        for (Class<? extends Throwable> excType : excTypes) {
            if (excType.isInstance(exc)) {
                passed++;
                break;
            }
        }
        if (passed == oldPassed)
            System.out.printf("테스트 %s 실패: %s %n", m, exc);
    }
}
```

- for 문으로 하나씩 확인하는 방식으로 구현함

### 예외를 여러 개 명시하고 그 중에 하나가 발생했을 때 성공하는 테스트 - @Repeatable 메타애너테이션 사용

@Repeatable을 단 애너테이션을 반환하는 “컨테이너 애너테이션”을 하나 더 정의하고, @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야함

```java
// 코드 39-8 반복 가능한 애너테이션 타입 (243-244쪽)
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

```java
// 반복 가능한 애너테이션의 컨테이너 애너테이션 (244쪽)
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```

- 컨테이너 애너테이션은 value 메서드를 재정의해야됨
- 컨테이너 애너테이션 타입에는 적절한 보존 정책 (@Retension)과 적용 대상 (@Target)을 명시해야 함

### 사용 방법

```java
// 코드 39-9 반복 가능 애너테이션을 두 번 단 코드 (244쪽)
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {
    List<String> list = new ArrayList<>();

    // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
    // NullPointerException을 던질 수 있다.
    list.addAll(5, null);
}
```

- 반복 가능 애너테이션을 여러 개 달면 컨테이너 애너테이션 타입이 적용됨

```java
// 코드 39-10 반복 가능 애너테이션 다루기 (244-245쪽)
if (m.isAnnotationPresent(ExceptionTest.class)
        || m.isAnnotationPresent(ExceptionTestContainer.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (Throwable wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        int oldPassed = passed;
        ExceptionTest[] excTests =
                m.getAnnotationsByType(ExceptionTest.class);
        for (ExceptionTest excTest : excTests) {
            if (excTest.value().isInstance(exc)) {
                passed++;
                break;
            }
        }
        if (passed == oldPassed)
            System.out.printf("테스트 %s 실패: %s %n", m, exc);
    }
}
```

- 위에서 @ExceptionTest를 두 개 단 것과 한 개 단 것을 공통으로 검사하고 싶다면 따로따로 확인해야 됨
    - m.isAnnotationPresent(ExceptionTest.class) || m.isAnnotationPresent(ExceptionTestContainer.class)로 각각 확인해줘야 된다는 뜻
    - 여러 개 달면 @ExceptionTestContainer가 적용되기 때문
    - 아래의 getAnnotationsByType은 해당하는 컨테이너 애너테이션까지 가져오기 때문에 상관 없음
- 가독성은 좋지만, 컨테이너 애너테이션을 추가로 선언하고 처리하는 부분에서 코드 양 늘어나고 복잡하긴 함

### 결론

- 명명 패턴 좋지 않다. 애너테이션 써라