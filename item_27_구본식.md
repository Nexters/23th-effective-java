# 비검사 경고를 제거하라

## 비검사 경고란(Unchecked Warnings)?

---
* 자바 컴파일러가 **타입 안전성(type-safe)** 을 확인할 때, **정보가 불충분**할때 발생하는 경고이다.
* 비검사 형변환 경고, 비검사 메서드 호출 경고, 비검사 매개변수화 가변인수 타입 경고, 비검사 변환 경고 등 **제네릭** 사용시 마주칠 수 있는 컴파일 경고이다.
* 컴파일 경고가 발생해도 컴파일이 정상적으로 실행되고, 동작 가능하다.
* 컴파일 경고는 문제가 발생할 수 있고, 권장 방식을 알려준다.
* `javac` 명령어 인수에 `-Xlint:unchecked` 옵션 추가해서 사용

### 예시
```java
Set<String> words = new HashSet();
```
```java
.\Unchecked_Warnings.java:9: warning: [unchecked] unchecked conversion
        Set<String> words = new HashSet();
                            ^
  required: Set<String>
  found:    HashSet
```
인스턴스로 `raw type`를 사용했을 때, 컴파일러가 **타입을 정확히 추론**하지 못하기 때문에 발생하는 **비검사 경고**이다.

가능한 비검사 경고를 모두 제거해야한다. 모두 제거한다면 **타입 안정성**이 높아지고, `ClassCastException`발생 여지를 없애준다. 

## 비검사 `Unchecked`

---
### 비검사 변환 경고 `Unchecked conversion`
```java
@Test
void uncheckedConversion(){
    Map<String,String> map = getMap();
}
private Map getMap(){
    return new HashMap<String, String>();
}
```
```java
.\Unchecked_WarningsTest.java:11: warning: [unchecked] unchecked conversion
        Map<String,String> map = new HashMap();
                                 ^
  required: Map<String,String>
  found:    HashMap
```
`Map<String,String>` 타입이 할당 되어야 하지만 인스턴스 타입을 `raw type`인 `Map`를 사용해서 발생하는 경고.

### 비검사 형변환 경고 `Unchecked cast`
```java
@Test
void uncheckedCast(){
    Map<String,String> map = (Map<String, String>) getMap();
}
```
```java
.\Unchecked_WarningsTest.java:16: warning: [unchecked] unchecked cast
        Map<String,String> map = (Map<String, String>) new HashMap();
                                                       ^
  required: Map<String,String>
  found:    HashMap
```
`raw type`인 Map을 강제 형변환를 하는 상황이다.
`raw type`를 type checking 없이 casting(형변환)할 때 발생하는 경고.

### 비검사 메서드 호출 경고 `Unchecked call`
```java
@Test
void uncheckedCall(){
    Set words = new HashSet();
    words.add("hello");
    words.add(1);
}
```
```java
.\Unchecked_WarningsTest.java:22: warning: [unchecked] unchecked call to add(E) as a member of the raw type Se
t
        words.add("hello");
                 ^
  where E is a type-variable:
    E extends Object declared in interface Set
```
Set이 `raw type`을 선언되었다.
잘못된 타입이 들어갔을 때 런타입 시점에 타입 관련 오류가 발생할 수 있는 경고를 컴파일러가 알려준다.
> 이 같이 `raw type`이 허용되는 이유는 java 제네릭 등장 이전 버전들의 호환성을 허용하기 위해서.

### 비검사 매개변수화 가변인수 타입 경고
```java
@Test
    void uncheckedParameterizedVarargType(){
        List<String> list = createList("apple", "banana");
}
```
```java
.\Unchecked_WarningsTest.java:29: warning: [unchecked] Possible heap pollution from parameterized vararg type 
T
    private <T> List<T> createList(T... e){
                                        ^
  where T is a type-variable:
    T extends Object declared in method <T>createList(T...)
```
가변인수에 제네릭을 함께 사용했을 때 발생하는 경고이다.

가변인수는 내부적으로 배열을 생성하고, 이때 자바는 제네릭 배열의 생성을 허용하지 않으므로 비검사 경고가 발생한다.

> ####  Heap Pollution(힙 오염) 이란?
>  매개변수 유형이 **다른 타입** 을 참조할 경우 발생하는 문제이며, 컴파일 중에 정상적으로 처리되며 경고가 발생하지 않고, 런타입 시점에 힘 오염으로 `ClassCastException`이 발생한다. 
> 
> 즉, 제네릭 타입에서 변수화된 타입과 실제 타입이 다를때를 의미한다.
>

## 비검사 경고 해결 방법

---
위의 상황의 대부분의 원인은 `raw type`를 사용하는 것이다. 

따라서 `raw type` 대신 **제네릭 타입**을 명시하거나, 제네릭 타입 추론의 `<>(다이아몬드 연산자)`를 사용해서 해결한다.

비검사 매개변수화 가변인수 타입 경고는, 제네릭과 가변이수를 사용하지 않도록 주의하면 해결이 가능하다.

### @SuppressWarnings("unchecked")
경고를 제거할 수는 없지만 **타입이 안전(type safe)** 하다고 확실할 수 있다면 사용하는 애너테이션이다.
```java
private <T> T[] toArray(T[] a){
    if(a.length < size){
        @SuppressWarnings("unchecked")
        T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    return null;
}
```
* 생성한 배열과 매개변수로 받은 배열의 타입이 모두 `T[]`로 같으므로, 올바른 형변환이다.
* 경고를 제거할수는 없지만, 타입 변환이 안전하다고 확실할 수 있으므로 `@SuppressWarnings("unchecked")`를 사용해 경고를 숨긴다.
* **타입 안정성**을 충분히 검증 후에 사용해야한다.
* 가능한 범위를 좁혀 사용한다. 
  * 변수 선언, 짧은 메서드, 생성자 등
* 붙힌 이유에 대해서 항상 주석을 남긴다.

