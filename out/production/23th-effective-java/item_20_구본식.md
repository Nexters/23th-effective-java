# 추상 클래스보다는 인터페이스를 우선하라

자바의 다중 구현 메커니즘은 인터페이스와 추상 클래스 두가지 방식이다.
> 추상 클래스
> - 추상 클래스를 구현한 클래스는 추상 클래스의 하위 타입
> - 추상 클래스를 단일 상속만 가능
> - 필드를 가질 수 있다.
> - 인스턴스 메서드를 구현 형태로 제공 가능
> 
> 인터페이스
> - 선언한 메서드들을 모두 정의하고 일반 규약을 지킨 클래스라면 상속해도 같은 타입
> - 다중 구현이 가능
> - 필드를 가질 수 없다.
> - 인스턴스 메서드를 구현 형태로 제공 가능(자바 8-default method)

## 인터페이스 장점

### 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.

인터페이스에 새로운 메서드를 추가하고, 클래스에 implements 구문을 추가하기만 하면 된다.

반면, 기존 클래스에 **추상 클래스를 추가**하는 것은 매우 어렵다.
두 클래스가 하나의 추상 클래스를 확장하기를 원하면, 해당 추상 클래스는 두 클래스의 공통 조상이 되어야 하므로 쉽지 않다.

### 인터페이스는 믹스인(mixin) 정의에 안성맞춤이다.

믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 주된 타입 이외에 특정 선택적 행위를 제공하는 효과를 준다.

```java
interface Loggable {
    void log(String message);
}
class OrderService implements Loggable {
    private String orderNumber;
    public OrderService(String orderNumber) {
        this.orderNumber = orderNumber;
    }
    //선택적 기능을 추가해서 믹스인 정의
    @Override
    public void log(String message) {
        System.out.println("Order #" + orderNumber + " Log: " + message);
    }
}
```
즉, 대상 타입의 주된 기능에 선택적 기능을 **혼합**한다는 의미.

**추상 클래스**는 기존 클래스를 덧씌울 수 없어서 믹스인을 정의할 수 없다.

### 인터페이스로 계층 구조가 없는 타입 프레임워크를 만들 수 있다.
```java
public interface Singer(){
    void sing();
}
public interface Songwriter(){
    void compose();
}
```
- 계층적으로 표현하기 어려운 경우가 대부분
- 계층 관계를 표현하기 보다는 인터페이스로 정의하면, 특정 클래스(가수 & 작곡가)가 두 인터페이스를 모두 구현해도 문제가 되지 않는다.
- 또한, 두 인터페이스를 확장하여 새로운 인터페이스 메서드인 제 3자 인터페이스까지 정의 가능

만약에 이러한 구조를 클래스로 만들려면 가능한 조합 전부를 각각의 클래스로 정의해야하기 때문에, 고도비만 계층 구조가 만들어진다.
(공통 기능을 정의해논 타입이 없다.)
```java
class Singer{
    public void sing() {}
}
class Songwriter{
    public void compose(){}
}
class SingerAndSongwriter{
    public void sing(){}
    public void compose(){}
}
(이런 구조?)
```

## 디폴트 메소드
자바 8 부터는 인터페이스 메서드를 구현 형태로 제공 가능.

인터페이스 메서드 중 구현 방법이 명확한 것이 있다면 **디폴트 메서드**로 제공하고, 문서화를 해야한다.
```java
//사용 방식
public interface defaultInterface {
    default void print() {
        System.out.println("Hello world");
    }
}
```

언제 사용?
> 기존에 사용하던 인터페이스를 이용하여 구현 클래스를 만들어 사용하고 있을 때,
> 인터페이스 보완 과정에서 **메소드가 추가**될 경우, 기존에 구현한 클래스에 오류가 발생하고 호환성이 떨어진다.
> 
> 인터페이스 메소드 추가 과정시 문제를 보완 가능하고, **하위 호환성**이 유지 

다만, equals, hashCode 메서드들은 디폴트 메서드로 제공하면 안된다.(구현이 따로 필요한 부분)

## 추상 골격 구현(skeletal implementation)
인터페이스, 추상 클래스 장점을 모두 취하는 방식.

- 인터페이스로 타입을 정의하고(필요시 디폴트 메소드도), 골격 구현 클래스로 필요한 메서드들을 구현한다.
- 인터페이스만 단순 구현하면 인터페이스 모든 메서드를 구현해야하지만, **골격 구현 클래스**을 확장하면 인터페이스를 구현하는데 필요한 대부분의 수고를 덜어준다.
- **템플릿 메서드 패턴**이라고 함.
> **JdbcTemplate**가 템플릿 메서드 패턴 클래스를 사용한 예시라고 할 수 있다.

```java
- 골격 구현 클래스 예시
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V>{
    //반드시 재정의해야함을 표시
    @Override public V setValue(V value){
        throw new UnsupportedOperationException();
    }
    
    //기반 메서드로 사용해서 구현할 수 있는 메서드는 디폴트 메서드로 구현 및 제공
    @Override public boolean equals(Object o){
        if(o==this){
            return true;
        }
        if(!(o instanceof Map.Entry)){
            return false;
        }
        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(), getKey()) && Objects.equals(e.getValue(), getValue());
    }
    @Override public int hashCode(){
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }
    ...
}
```

- 인터페이스 중 다른 메서드들의 구현애 사용될수 있는 메서드들을 선정한다. -> **기반 메서드**
- 즉, 기반 메서드들은 골격 구현에서 추상 메서드 역할
- "equals", "hashCode" 메서드들과 같이 기반 메서드들을 이용해서 구현할 수 있는 메서드들을 **디폴트 메서드로 제공(구현)**
- 메서드 모두 기반 메서드이거나, 디폴트 메서드 라면 골격 구현 클래스의 장점이 없음.
- 골격 구현 클래스를 **상속**해서, 디폴트 메서드는 제공해주는 것을 사용하면 되고, 기반 메서드는 구현해서 사용하면된다.
- 상속 위한 설계시와 마찬가지로 인터페이스 디폴트 메소드, 골격 구현, 추상 클래스의 동작 방식을 **문서로 정리하기**

**단순 구현(인터페이스 구현)** 도 골격 구현의 작은 변종이다. -> 인터페이스의 모든 메서드를 디폴트 메서드로 만드니.

다만 추상 클래스가 아닌란 점만이 다르다.

