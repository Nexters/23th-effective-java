# 생성자에 매개변수가 많다면 빌더를 고려하라
## 점층적 생성자 패턴
필수 매개변수만 받는 생성자, 매개변수 1개.., 매개변수 2개... 등 처럼 필요한 매개변수를 인자로 하는 생성자를 점차 생성하는 방식이다.

예전에는 많이 사용한 방식이지만, 매개변수 개수가 많아지면 그 만큼 생성자도 많아지게 되고 그에 따라,

매개 변수 수, 타입 등의 고려해야할 점과 실수할 여지가 많아진다.

## 자비빈즈 패턴
매개변수가 없는 생성자로 객체를 인스턴스화 한 후, setter 메서드를 통해 매개변수 값을 지정하는 방식이다.

```java
//자비빈즈 패턴
Member member = new Member();
member.setAge(10);
member.setName("effective java");
```
점층적 생성자 패턴의 단점이 보완됬지만, 객체하나를 생성하려면 여러 메서드를 호출해야하고, 
객체가 완전히 생성되기 전까지는 일관성이 무너진 상태이다.
(여기서 일관성 무너진 상태란, 앞서 점층적 생성자 패턴에서는 생성자에서 매개변수의 유효성을 검증하면 되는 장금장치가 무너진 것.)

또한, 버그, 런타입 문제등에서  디버깅하는 것도 쉽지 않으며, 불변 클래스를 만들 수도 없다.

## 빌더 패턴
필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(혹은 정적 펙토리 메서드)를 호출해 빌더 객체를 얻은 뒤, 
빌더 객체가 제공하는 세터 메서드로 매개변수 값을 설정하는 방식.

(Lombok의 Builder 사용 예시)
```java


//빌더 패턴
static abstract class Pizza{
    public enum Topping {HAM, ONION}
    final Set<Topping> toppings;
    @Builder
    Pizza(Set<Topping> toppings) {
        this.toppings = toppings;
    }
}

final String name = "effective java";
final Integer age = 10;

BuilderMember.BuilderMemberBuilder builder = BuilderMember.builder();
builder.name(name);
builder.age(age);
BuilderMember builderMember = builder.build();

NyPizza pizza = NyPizza.builder()
        .size(NyPizza.Size.MEDIUM)
        .toppings(EnumSet.of(Pizza.Topping.HAM, Pizza.Topping.ONION))
        .build();
```
빌더의 세터 메서드는 빌더 자신을 호출하기 때문에 연쇄적으로 호출(method chaining) 할 수 있고,
무엇보다도, 사용자는 코드를 쓰기 쉬고 읽이 쉽다.
(파이썬의 선택적 매개변수를 모방한 것)

또한, **계층적으로 설계된 클래스와 함께 쓰기에도 좋다.**
```java
static class NyPizza extends Pizza{
    public enum Size {SMALL, MEDIUM}
    private final Size size;
    @Builder
    NyPizza(Set<Topping> toppings, Size size) {
        super(toppings);
        this.size = size;
    }
}
    
//빌더 패턴
final String name = "effective java";
final Integer age = 10;

BuilderMember.BuilderMemberBuilder builder = BuilderMember.builder();
builder.name(name);
builder.age(age);
BuilderMember builderMember = builder.build();

NyPizza pizza = NyPizza.builder()
        .size(NyPizza.Size.MEDIUM)
        .toppings(EnumSet.of(Pizza.Topping.HAM, Pizza.Topping.ONION))
        .build();
```
빌더 하나로 여러 객체를 순회하면서 만들 수 있다.

하지만, 객체를 생성하려면 해당 객체의 빌더부터 생성해야되는 단점이 있다. 빌더의 생성 비용은 크지 않지만 성능이 민감한 상황에서는 영향이 생길 수 있다.
그러므로, 점층적 생성자 패턴보다 매개변수가 4개 이상일 때 진정한 값어치가 있다.
(생성자나 정적 팩터리 메서드에서 처리해야할 매개변수가 많으면 빌더 패턴을 사용하자!)