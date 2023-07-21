# 상속보다는 컴포지션을 사용하라
- 상속은 코드를 재사용하는 강력한 수단이지만 항상 최선은 아니다.
- 메소드 호출과 달리 상속은 캡슐화를 깨뜨린다.
  - 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.
    - 상위 클래스 내부구현이 달라지는 경우
    - 상위 클래스 설계자가 확장을 충분히 고려하고 문서화도 제대로 해두지 않는다면 하위 클래스는 변화에 맞춰서 수정되어야 한다.
  - 자기사용 (self use)으로 인한 오작동이 있을 수 있다.
    ```java
      public class InstrumentedHashSet<E> extends HashSet<E> {
        private int addCount = 0;

        public InstrumentedHashSet() {}

        public InstrumentedHashSet(int initCap, float loadFactor) {
          super(initCap, loadFactor);
        }
        
        @Override
        public boolean add(E e) {
          addCount++;
          return super.add(e);
        }

        @Override
        public boolean addAll(Collection<? extends E> c) {
          addCount += c.size();
          return super.addAll(c);
        }

        public int getAddCount() {
          return addCount;
        }
      }
    ```
    ```java
      InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
      s.addAll(List.of("a", "b", "c"));
    ```
    - `getAddCount()` 메소드 호출 시 6이라는 결과가 나온다.
      - HashSet의 addAll 메소드에서 각 원소를 add 메소드를 호출해서 원소를 추가한다.
        - 그 add 메소드는 하위 클래스에서 재정의한 메소드이므로 addCount에 값이 중복해서 더해진다.
        - 하위 클래스에서 addAll 메소드를 재정의하지 않으면 문제를 해결할 수 있다.
        - addAll 메소드 내에서 주어진 컬렉션을 하나씩 순회하면서 add 메소드를 호출하는 방식으로도 해결할 수 있다.
  - 상위 클래스에서 새로운 메소드가 추가된다면?
    - 하위 클래스에서 재정의하지 못한 새로운 메소드를 사용해서 허용되지 않은 원소가 추가될 수도 있다.
    - 예전부터 존재하던 Hashtable, Vector가 컬렉션 프레임워크에 포함되었을 때 관련 보안 문제들을 해결해야 하는 문제가 있었음

## 컴포지션
> 기존 클래스가 새로운 클래스의 구성요소로 쓰이는 설계

```java
  // 래퍼 클래스 - 상속 대신 컴포지션 사용
  public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
      super(s);
    }

    @Override
    public boolean add(E e) {
      addCount++;
      return super.add(e);
    }

    @Override public boolean addAll (Collection< ? extends E > c){
      addCount += c.size();
      return super.addAll(c);
    }

    public int getAddCount () {
      return addCount;
    }
  }
```
```java
  // 재사용할 수 있는 전달 클래스
  public class ForwardingSet<E> implements Set<E> {

    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s;}

    public void clear() { s.clear(); }
    public boolean contains(Object o) {return s.contains(o); }
    public boolean isEmpty() { return s.isEmpty(); }
    public int size() { return s.size(); }
    public Iterator<E> iterator() { return s. iterator(); }
    public boolean add(E e) { return s.add(e); }
    public boolean remove(Object o) {return s.remove(o); }
    public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
    public boolean removeAll(Collection<?> c) { return s.removeAll(c); }
    public boolean retainAll(Collection<?> c) { return s.retainAll(c); }
    public Object[] toArray() { return s.toArray(); }
    public <T> T[] toArray(T[] a) { return s.toArray(a); }

    @Override public boolean equals(Object o) { return s.equals(o); }
    @Override public int hashCode() {return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
  }
```
- 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다.
- 새 클래스의 인스턴스 메소드들은 기존 클래스의 대응하는 메소드를 호출해서 결과를 반환한다.
  - 이 방식을 `전달 (forwarding)`이라고 한다.
  - 새 클래스의 메소드는 `전달 메소드 (forwarding method)`라고 한다.
- 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 기존 클래스에 새로운 메소드가 추가되더라도 영향을 받지 않는다.
- 한 번만 구현해두면 어떠한 Set 구현체라도 계측할 수 있으며, 기존 생성자들과도 함께 사용할 수 있다.

### 래퍼 클래스
> 래퍼 클래스는 다른 인스턴스를 감싸고 있다는 뜻

- 다른 Set에 계측 기능을 덧씌운다는 뜻에서 데코레이션 패턴이라고 한다.
- 컴포지션과 전달의 조합은 위임 이라고 한다.
  - 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당
- 래퍼 클래스는 단점이 거의 없다.
  - 래퍼 클래스가 콜백 프레임워크와는 어울리지 않는다는점만 주의한다.

## 정리
- 클래스 B가 클래스 A와 is-a 관계일 때만 상속해야 한다.
- 그렇지 않다면 A를 private 인스턴스로 두고, A와는 다른 API를 제공한다.
  - ex. Stack은 Vector가 아니므로 확장해서는 안됐고, 컴포지션을 사용했다면 더 좋았을 것이다.
- 컴포지션을 써야할 상황에서 상속을 사용하는건 내부 구현을 불필요하게 노출하는 꼴이다.
  - API가 내부 구현에 묶이고, 그 클래스의 성능도 영원히 제한된다.
  - 클라이언트가 노출된 내부에 직접 접근할 수 있다.
- 컴포지션 대신 상속을 사용할 때
  - 확장하려는 클래스의 API에 아무런 결함이 없는지
  - 결함이 있다면 클래스의 API까지 전파돼도 괜찮은지
  - 컴포지션은 결함을 숨지는 새로운 API를 설계할 수 있다.
  - 상속은 상위 클래스의 API를 결함까지도 전달한다.
- 상속의 취약점을 피하려면 컴포지션과 전달을 사용하자
  - 레퍼 클래스로 구현할 적당한 인터페이스가 있다면 사용하자
  - 레퍼 클래스는 하위 클래스보다 견고하고 강력하다.
