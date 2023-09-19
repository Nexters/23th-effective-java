# item41 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라.

### 마커 인터페이스란?

- 일반 인터페이스와 비슷하지만 아무 메서드도 담지 않고 있는 인터페이스를 의미.
- **단순 타입 체크** 용도로 사용되며, `Serializable` `EventListener` 등이 있음.

### 예시

```java
package java.io;
public interface Serializable {
}
```

```java
public class item {
    private long id;
    private String name;
    item(long id, String name){
        this.id = id;
        this.name = name;
    }
}
```

```java
public class SerializableTest {
    public static void main(String[]args) throws IOException {
        File file = new File("output.txt");
        ObjectOutputStream ops = new ObjectOutputStream(new FileOutputStream(file));
        ops.writeObject(new item(1, "test"));
    }
}
```

- `Serializable` 를 구현한 클래스는  `ObjectOutputStream` 를 통해 직렬화가 가능.
- `item` 은 `Serializable` 를 구현하지 않았기 때문에 직렬화 불가능.

    ```java
    Exception in thread "main" java.io.NotSerializableException: chapter6.item41.item
    	at java.base/java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1196)
    	at java.base/java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:355)
    	at chapter6.item41.SerializableTest.main(SerializableTest.java:12)
    ```

    ```java
    if (obj instanceof String) {
                    writeString((String) obj, unshared);
                } else if (cl.isArray()) {
                    writeArray(obj, desc, unshared);
                } else if (obj instanceof Enum) {
                    writeEnum((Enum<?>) obj, desc, unshared);
                } else if (obj instanceof Serializable) {
                    writeOrdinaryObject(obj, desc, unshared);
                } else {
                    if (extendedDebugInfo) {
                        throw new NotSerializableException(
                            cl.getName() + "\n" + debugInfoStack.toString());
                    } else {
                        throw new NotSerializableException(cl.getName());
                    }
                }
    ```

  - 간단히 타입 체크로 사용되기 때문에, **마커 인터페이스**라고 불림.


### 마커 인터페이스 vs 마커 애너테이션

- 마커 인터페이스는 인스턴스들을 구분하는 타입으로 쓸 수 있으나, 마커 애너테이션은 그렇지 못하다.
  마커 인터페이스도 타입이기 때문에 런타임에 발견된 오류를 **컴파일** 시점에 잡을 수 있다.
  - 앞 예시의 `ObjectOutputStream.writeObject()` 메서드는 마커 인터페이스를 사용하지만, 런타임 시점에 오류를 검출하므로 마커 인터페이스의 장점을 살리지 못한 케이스
- 마커 인테페이스는 적용 대상을 정밀하게 지정할 수 있다.
  - 마킹하고 싶은 클래스에만 마커 인터페이스를 구현하며 세밀하게 지정 가능.
  - 마커 애너테이션은 클래스, 인터페이스, 열거 타입 등에 적용대상을 지정할 수 있는데, 부착 타입을 더 세밀하게는 지정 불가능.
- 마커 애너테이션은 애너테이션 시스템에선 큰 지원을 받는다. 프레임워크의 일관성을 지킬 수 있음.

### 어떻게 구분해서 사용?

- 마커 애너테이션
  - 클래스, 인터페이스 외 요소에 마킹이 필요할 시
  - 애너테이션 기반 프레임워크 사용시
- 마커 인터페이스
  - 마킹 된 객체를 매개변수로 받는 메서드를 작성해야 할 상황일 때
    컴파일타입에 오류를 잡을 수 있음.
  - 메서드 없이 타입 체크 용도가 목적일 때.