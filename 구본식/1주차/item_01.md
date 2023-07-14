# 생성자 대신 정적 팩터리 메서드를 고려하라.
클래스를 생성자를 클라이언트에게 제공할때는 public 생성자만 제공하기보다는 기본 생성자와 함께 정적 팩터리 메서드를 제공할 수 있다.
그럼 정적 팩터리 메서드가 기본 생성자에 비해 장/단점을 알아보겠다.

## 장점
### 1. 이름을 가질 수 있다.
한 클래스내에서 여러 생성자가 존재할 때, 정적 팩터리 메서드로 이를 정의하고 각각의 차이점을 들어내는 이름을 짓는것이 좋다.

```java
    BigInteger name1 = new BigInteger(2, 5, random);
    BigInteger name2 = BigInteger.probablePrime(2, random);
```
random 범위 중 값이 소수인 BigIntger 클래스를 생성하는 2개의 생성자 모습이다. 아래의 생성자가 소수를 반환한다는 의미가 명확한 것을 볼 수 있다.

### 2. 인스턴스를 새로 생성하지 않아도 된다.

불변 클래스의 경우 인스턴스를 미리 만들어 놓거나 캐싱처리한 후, 
정적 팩터리 메서드를 통해 미리 만들어 놓은 인스턴스를 반환해주므로써 호출마다 인스턴스를 매번 생성하지 않아도 된다.
인스턴스를 통제할 수 있고, 이로 인해 싱글턴, 인스턴스화 불가 등 인스턴스가 하나임을 보장할 수 있다.

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 생긴다.

이 방식은 반환할 객체 클래스를 자유롭게 선택할 수 있는 유연성을 제공한다.
인터페이스를 정적 팩터리 메서드 반환 타입으로 제공하는 인터페이스 기반 프레임워크의 핵심 기술이자, 
구현 클래스를 공개하지 않고 객체를 반환할 수 있어 API를 작게 유지할 수있는 장점이 있음.

하나의 예로, 자바 컬렉션 프레임워크는 핵심 인터페이스들의 유틸리티 구현체(수정 불가, 동기화 기능 등을 덧붙인)를 제공하는데, 
이 구현체 대부분을 인스턴스 불가 클래스인 java.util.Collections 에서 정적 팩터리 메서드를 통해 얻도록 되어있다.

```java
    public static <T> Collection<T> synchronizedCollection(Collection<T> c) {
        return new SynchronizedCollection<>(c);
    }

    public static <T> Set<T> synchronizedSet(Set<T> s) {
        return new SynchronizedSet<>(s);
}
```
java.util.Collection 클래스에 있는 자바 컬렉션 프레임워크의 인터페이스 구현체를 얻는 정적 팩터리 메서드 예시이다.

자바 8이전에는 인스턴스에 정적 메서드를 선언할 수 없어 java.util.Collection 클래스와 같이 인스턴스 불가 클래스를 동반하여 사용했지만, 
자바 8부터는 이러한 제한이 풀려 인터페이스에도 정적 메서드를 사용할 수 있다.

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

클래스의 반환 타입이 하위 타입이면 어떠한 클래스를 반환해도 무방하다.
예시로, EnumSet 클래스는 기본 생성자(public)없이 정적 팩터리 메서드만 제공하고, 원소 수의 따라 하위 클래스 중 하나의 인스턴스를 반환해준다.

```java
    /**
     * Creates an empty enum set with the specified element type.
     *
     * @param <E> The class of the elements in the set
     * @param elementType the class object of the element type for this enum
     *     set
     * @return An empty enum set of the specified type.
     * @throws NullPointerException if {@code elementType} is null
     */
    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
    }
```

### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

서비스 제공자 프레임워크(servide provider framework)를 만드는 기초가 되며, 대표적인 예시로는 JDBC 프레임워크가 있다.
서비스 제공자 프레임워크는 아래 3가지 핵심 컴포넌트로 구성된다.
구현체 동작을 정의하는 서비스 인터페이스, 구현체를 등록하는 제공자 등록 API, 서비스 인스턴스를 얻을 때 사용하는 서비스 접근 API.

즉, JDBC시에서 Connection이 서비스 인터페이스, DriverManager.registerDriver가 제공자 등록 API, DriverManager.getConnetion이 서비스 접근 API에 역할을 한다.

## 단점
### 1. 상속을 하려면 public이나 protected 생성자가 필요하니, 정적 팩터리 메서드만 제공하면 하위 클래스(상속)를 만들 수 없다.
앞서 예시를 든 자바 컬렉션 프레임워크의 유틸리티 구현체들은 정적 팩터리 메서드만 제공하므로 상속을 할 수가 없게 된다.

### 2. 정적 펙터리 메서드는 프로그래머가 찾기 어렵다.
생성자 처럼 API 설명이 확실하게 들어나지 않게 때문에, 정적 팩터리 메서드 방식의 클래스를 인스턴스화할 방법을 알아내야 된다.

# 정적 팩터리 매서드 명명 방식 예(많이 사용하는)
* from: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 메서드
* of: 여러 매개변수를 받아서 적합한 타입의 인스턴스를 반환하는 메서드
* valueOf: from과 of의 자세한 버전
* instance of getInstance: 매개변수로 명시한 인스턴스를 반환하고, 같은 인스턴스(싱글톤)을 보장하지 않음.
* create or newInstance: 매개변수로 명시한 인스턴스를 반환하고, 매번 새로운 인스턴스를 반환함을 보장.
* get + Type(반환할 객체 타입): getInstance와 같으나, 생성할 클래스가 다른 클래스인 경우.
* new + Type(반환할 객체 타입): newInstance와 같으나, 생성할 클래스가 다른 클래스인 경

