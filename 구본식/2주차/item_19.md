# 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

## 상속을 위한 설계 방법
### 1. 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는 지 문서로 남겨야 한다.
공개된 메서드(public)에서 클래스 자신의 또 다른 메서드를 호출 할 수 있고,

호출 하려는 메서드가 **재정의 가능 메서드**라면 **API 설명, 호출 순서, 호출 결과가 처리에 어떤 영향**을 주는지를 문서로 남겨야 한다.

```java
ex) java.util.AbstactCollection implementation Requirements
/**
 * {@inheritDoc}
 *
 * @implSpec
 * This implementation iterates over the collection looking for the
 * specified element.  If it finds the element, it removes the element
 * from the collection using the iterator's remove method.
 *
 * <p>Note that this implementation throws an
 * {@code UnsupportedOperationException} if the iterator returned by this
 * collection's iterator method does not implement the {@code remove}
 * method and this collection contains the specified object.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 */
public abstract Iterator<E> iterator();

public boolean remove(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext()) {
                if (it.next()==null) {
                    it.remove();
                    return true;
                }
            }
        } else {
            while (it.hasNext()) {
                if (o.equals(it.next())) {
                    it.remove();
                    return true;
                }
            }
        }
        return false;
 }
```
remove 메서드 동작내에 **재정의 가능한 iterator**을 사용하게 되고,
재정의한 iterator remove 메서드가 해당 remove 메서드에 영향을 주게된다.
(remove 메서드를 구현하지 않으면, UnsupportedOperationException 예외 발생)

=> 내부 구현 방식과 재정의 가능 메서드를 호출할 때 생기는 상황을 문서화 해야한다.

### 2. 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅을 잘 선별하여 protected 메서드 형태로 제공해야 할 수 도 있다.

```java
ex) java.util.AbstactList implementation Requirements
public void clear() {
        removeRange(0, size());
}

/**
 * Removes from this list all of the elements whose index is between
 * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.
 * Shifts any succeeding elements to the left (reduces their index).
 * This call shortens the list by {@code (toIndex - fromIndex)} elements.
 * (If {@code toIndex==fromIndex}, this operation has no effect.)
 *
 * <p>This method is called by the {@code clear} operation on this list
 * and its subLists.  Overriding this method to take advantage of
 * the internals of the list implementation can <i>substantially</i>
 * improve the performance of the {@code clear} operation on this list
 * and its subLists.
 *
 * @implSpec
 * This implementation gets a list iterator positioned before
 * {@code fromIndex}, and repeatedly calls {@code ListIterator.next}
 * followed by {@code ListIterator.remove} until the entire range has
 * been removed.  <b>Note: if {@code ListIterator.remove} requires linear
 * time, this implementation requires quadratic time.</b>
 *
 * @param fromIndex index of first element to be removed
 * @param toIndex index after last element to be removed
 */
protected void removeRange(int fromIndex, int toIndex) {
        ListIterator<E> it = listIterator(fromIndex);
        for (int i=0, n=toIndex-fromIndex; i<n; i++) {
            it.next();
            it.remove();
        }
}
```

클래스의 내부 메서드 중 clear 동작 과정에서 **removeRange**를 호출하는 것을 볼 수 있고, implementation Requirements 설명에도 나와있다.

API 를 사용하는 클라이언트가 removeRange 메서드에 관심이 없어라도,
removeRange를 protected 메서드로 제공하므로써 **clear** 메서드를 성능 향상시킬 수 있는 방법을 제공해줌.

이가 없었다면, clear 메서드 매커니즘을 밑 바닥부터 구현해야 했을 것이다.

protected 메서드를 선정하는 것에는 명확한 정답이 없다.

하위 클래스를 생성해 테스트 해보고, 하위 클래스 작성 간 확연히 빈자리가 느껴진다면 protected로 만드는 것이 좋고,
하위 클래스간 protected 멤버가 사용되지 않으면 사실 private이 가능성이 크다.

### 3. 상속용 클래스의 생성자는 직접적이든 간접적이든 재정의 가능 메서드를 호출하면 안된다.

상위 클래스 생성자가 하위 클래스 생성자 보다 먼저 실행되므로, **하위 클래스에서 재정의 한 메서드**가 **하위 클래스의 생성자보다 먼저 호출**된다.

```java

public class Super {
    public Super(){
        overrideMe();
    }
    public void overrideMe(){
    }
}
public class Sub extends Super{
    private final Instant instant;
    Sub(){
        instant = Instant.now();
    }
    @Override public void overrideMe(){
        System.out.println(instant);
    }
    public static void main(String[] args){
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```
상위 클래스 생성자가 하위 클래스 생성자의 인스턴스(Instance)가 초기화되지 전에 overrideMe 메서드가 호출되어 NULL이 출력된다.

-> 재정의 가능한 public 메서드 보다는, private ,final, static 메서드를 사용해 재정의 불가 메서드로 지정하자.

## 상속용으로 설계되지 않은 클래스는 상속을 금해라
구체 클래스의 내부를 변경했을 때, 이를 확장한 클래스가 정상적으로 동작하지 못할 수 있다.
그러므로, 상속용으로 설계되지도 않았고 문서화 되지 않은 일반적인 구체 클래스는 상속을 막아야 한다.

핵심 기능을 정의한 인터페이스가 존재하고, 클래스가 이 인터페이스를 구현한 방식이라면 상속을 금지해도 개발에 불편함이 없다.
- 방법
  - 클래스를 final로 선언하자.
  - 모든 생성자를 private이나 package-private으로 선언하고, public 정적 팩터리 메서드를 제공하자.

하지만, 지금까지 동기화, 기능 제약, 통지, 계측 등을 위해 구체 클래스를 상속해서 사용하는 경우가 많았기에 상속을 금하면 사용하기에 불편해진다.

이럴 경우에는,  클래스 내부에서 **재정의 메서드를 사용하지 못하게**하고, 문서화해라.

-> 상속에 위험을 벗어날 수 있고, 재정의해도 다른 메서드들에 영향을 주지 않을 수 있다.
