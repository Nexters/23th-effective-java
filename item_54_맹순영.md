## 아이템 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

```java
private final List<Cheese> cheesesInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록 반환
 *   단, 재고가 하나도 없다면 null 반환
*/
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? null
		: new ArrayList<>(cheesesInStock);
}
```

- 비어있으면 null 반환

```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
	System.out.println("야호");
```

- null을 반환하게하면 클라이언트에서 null 체크를 한 번 더 해줘야됨
    - 이를 **방어코드**라고 함

### 빈 컨테이너(컬렉션, 배열)를 할당하는 비용이 null을 반환하는 비용보다 더 크다?

그렇지 않다.

1. 성능 차이는 매우 적음
2. 빈 컬렉션과 배열은 새로 할당하지 않고도 반환할 수 있음

    ```java
    public List<Cheese> getCheeses() {
        return new ArrayList<>(cheesesInStock);
    }
    ```

    - 일반적인 경우 그냥 할당해서 반환하면 됨
    - 성능이 걱정된다면, 매번 같은 빈 **‘불변’컬렉션**을 반환하면 된다.

        ```java
        public List<Cheese> getCheeses() {
            return cheesesInStock.isEmpty() ? Collections.emptyList()
                : new ArrayList<>(cheesesInStock);
        }
        ```

        - Collections.emptyList, Collections.emptySet, Collections.emptyMap

    - 배열도 마찬가지로 그냥 할당해서 반환하면 됨

        ```java
        private Cheese[] getCheeses() {
            return cheesesInStock.toArray(new Cheese[0]);
        }
        ```

        - 여기서 Cheese[0]은 반환 타입을 알려주는 역할을 함
        - 성능이 걱정되면 길이 0인 배열을 미리 선언해두고 매번 해당 배열을 반환하면 됨 (길이 0인 배열은 모두 불변임)

            ```java
            private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
            
            public Cheese[] getCheeses() {
                return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
            }
            ```
        - 그렇다면 cheesesInStock 크기를 미리 할당해서 반환하면 성능이 좋지 않을까??
          ```java
          return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
          ```
          - 단순히 성능 개선이 목적이라면 미리 할당하는건 좋지 않음
          - 오히려 성능을 떨어뜨릴 수 있음