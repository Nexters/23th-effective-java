# 아이템3: private 생성자나 열거 타입으로 싱글턴임을 보증하라

싱글톤: 하나의 인스턴스만 생성해서 재사용하는 방식

장점: 객체 생성 비용 및 메모리 비용 절약

단점:
- 테스트하기 어려움
- 동시성 문제 발생

### 생성 방법 1. public static final

``` java
public class SingletonV1 {
    public static final SingletonV1 instance = new SingletonV1;

    private SingletonV1() {
        // private 생성자
    }
}
```
이 방식의 장점: 싱글톤임이 비교적 명확함.
-> 음 근데 `getInstance` 는 사실상 합의된(?) 매서드 명이라서 괜찮지 않을까.

### 생성 방법 2. getInstance()

``` java
public class SingletonV2 {
    private static SingletonV2 instance;

    private SingletonV2() {
        // private 생성자
    }

    public static SingletonV2 getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
이 방식의 장점: 매서드 변경 없이 싱글턴이 아니게 변경 가능하다.
> 즉 getInstace 마다 그냥 New 를 해준다는 소리인데, 사실상 암묵적인 매서드 네이밍을 어기기 때문에 안좋은 것 같습니다.

그리고 `제네릭 싱글턴 팩터리`로 만들 수 있다고 합니다.(Item30)

### 위의 두 방식은 별도의 직렬화를 제공해야한다.

일반적으로 직렬화는 Serializable 키워드를 붙여서 사용하지만, Serializable 는 직렬화를 통해 다른 인스턴스가 반환한다.
-> 싱글톤이 아니게 된다.

코드 출처: https://madplay.github.io/post/what-is-readresolve-method-and-writereplace-method
``` java
MySingleton instance = MySingleton.getINSTANCE();
SerializationTester serializationTester = new SerializationTester();
byte[] serializedData = serializationTester.serialize(instance);
MySingleton result = (MySingleton)serializationTester.deserialize(serializedData);
System.out.println("instance == result : " + (instance == result));
System.out.println("instance.equals(result) : " + (instance.equals(result)));
```
결과
```
instance == result : false
instance.equals(result) : false
```

<br/>

그래서 다음과 같은 방식으로 readResolve 매서드를 구현하여 항상 같은 인스턴스임을 보장해줘야한다.
``` java
import java.io.Serializable;

public class SingletonV1 implements Serializable {
    public static final SingletonV1 instance = new SingletonV1();

    private SingletonV1() {
        // private 생성자
    }

    // 직렬화된 객체를 역직렬화할 때 호출되는 메서드
    protected Object readResolve() {
        return instance;
    }
}
```

item 89에서 세부적으로 나오겠지만, 싱글톤 직렬화를 위해서는 다음 방식을 권장한다.

### 생성 방법 3. enum 으로 선언하기

``` java
public enum Singleton {
    INSTANCE;

    // 싱글톤의 기능들...
    
    public void doSomething() {
        // 싱글톤의 동작 코드...
    }
}
```

``` java
public class Main {
    public static void main(String[] args) {
        Singleton singleton1 = Singleton.INSTANCE;
        Singleton singleton2 = Singleton.INSTANCE;

        // 동일한 인스턴스인지 비교
        System.out.println(singleton1 == singleton2);  // 출력: true
    }
}
```

이게 무슨 소리인가 했었는데, 애초에 enum은 싱글톤임을 보장합니다.

이 경우에 직렬화나 리플랙션에서의 싱글톤 공격도 막을 수 있습니다.

직렬화도 별도의 처리 없이 가능!!!

가장 안전하고 확실한 방법이지만, class extends 가 안됨.


### 근데 왜 싱글톤이 테스트하기 힘들다고 하더라?

싱글톤 객체를 사용하기 위해 클래스 매서드를 사용한다.

-> 해당 '클.래.스'에 직접적으로 의존한다.

클래스 자체는 `Mocking` 이 안됨.

-> 실제 객체를 생성해야한다!

``` java
public class SomeOtherClass {
    public void doSomething() {
        Singleton singleton = Singleton.getInstance();
    }
}
```
