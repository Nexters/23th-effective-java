# 타입 안전 이종 컨테이너를 고려하라

### 자바의 컨테이너? ThreadLocal 이 무엇인가

자바에서 다중 스레드 환경에서 '각 스레드마다 독립적인 값 저장 및 관리'를 위한 클래스

'매개변수화되는 대상은 원소가 아닌 컨테이너 자신이다'
> 이게 뭔소린겨?

``` java
public class ThreadLocal<T> {

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue); 
        // 내부에서 사용하는 유일한 자료구조 자체가 제네릭으로 사용됨.
        //해시맵임 ! 외부에 노출시키지 않고 내부에서만 쓰기 위함. 가바지콜랙팅 최적화가 되어 있다는듯 !!!
    }

    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    public T get() {
        return get(Thread.currentThread());
    }

    private T get(Thread t) {
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            if (map == ThreadLocalMap.NOT_SUPPORTED) {
                return initialValue();
            } else {
                ThreadLocalMap.Entry e = map.getEntry(this);
                if (e != null) {
                    @SuppressWarnings("unchecked")
                    T result = (T) e.value;
                    return result;
                }
            }
        }
        return setInitialValue(t);
    }

    public void set(T value) {
        set(Thread.currentThread(), value);
    }

    private void set(Thread t, T value) {
        ThreadLocalMap map = getMap(t);
        if (map == ThreadLocalMap.NOT_SUPPORTED) {
            throw new UnsupportedOperationException();
        }
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
    }
}
```

### 타입 안전 이동 컨테이너 패턴

컨테이너 안에서 여러개의 타입을 쓰고 싶어용 > _ <

예를들면 데이터베이스! -> 하나의 테이블에 타입이 여러개

"자료를 꺼낼 때 '타입'만 적어서 내가 원하는 타입만 가져오자"


책 예제가 좀 짜증나서, 필요 없는 부분들 다 걸러버리고 새로 만듦

``` java
    public static class Favorites {
        // 클래스를 키 값으로 갖는 자료형
        // 해시맵이라 내부에서 키값 중복이 안됨 -> 하나의 클래스는 하나의 키값만 가질 수 있다.
        private Map<Class<?>, Object> favorites = new HashMap();

        public <T> void putFavorite(Class<T> type, T instance) {
            favorites.put(type, instance);
        }

        public <T> T getFavorite(Class<T> type) {
            return type.cast(favorites.get(type));
            // cast는 타입 변환을 하면서, 해당 제네릭으로 캐스팅 가능한 타입인지 검사합니다.
            // -> 타입 안전함을 보장
        }
    }

    public static void main(String [] args) {
        Favorites f = new Favorites ();

        f.putFavorite(String.class, "Java");
        f.putFavorite(Integer.class, 1000);
        f.putFavorite(Class.class, Favorites.class);

        String favoriteString = f.getFavorite(String.class);
        int favoriteInteger = f.getFavorite(Integer.class);
        Class<?> favoriteClass = f.getFavorite (Class.class);

        // 제약사항1. Map이기 때문에 로타입 쓰면 타입 안정성 깨져버림
        // f.putFavorite((Object)Integer.class, "Java");
        // putFavorite 내부에서 확인하는 로직을 추가해야합니다.

        System.out.println("=====================================");
        System.out.println("favoriteString: " + favoriteString);
        System.out.println("favoriteInteger: " + favoriteInteger);
        System.out.println("favoriteClass.getName(): " + favoriteClass.getName());
        System.out.println("=====================================");
    }
```

- 뭐여 그럼 동일한 타입을 여러번 쓰지는 못하네
- 내부적을 HashMap을 사용하고 있어서, 아래와 같이 작성하면 같은 map 에 덮어씌어짐

``` java
f.putFavorite(String.class, "1번컬럼");
f.putFavorite(String.class, "2번컬럼");
```

- Type을 클래스로 정의해서 써야할 듯!

``` java
f.putFavorite(FirstColumn.class, new FirstColumn("1번컬럼"));
f.putFavorite(SecondColumn.class, new SecondColumn("2번컬럼"));
```

### 책에 나오는 제약사항
1. 악의적으로 제네릭이 아닌 로 타입(오브젝트 포함)을 넣어버리면 타입 안정성이 박살난다
   - 비검사 경고가 뜰테니, 알아서 제거해라(?)
   - 컨테이너 내부에서 한 번 더 체크해라

2. 실체화 불가능한 객체는 사용 불가능하다.
   - 실체화? -> 타입 자체를 .class로 명시하는 것
   - String 가능 String[] 가능
   - List<String> 불가능
     - List<String>이랑 List<Integer>는 둘 다 억지로라도 클래스로 실체화하면 List.java 인데, 구분이 불가능함.


### 한정적 타입 토큰 사용해서 사용 가능한 타입 제한하기
위의 Favorite 객체는 아무 클래스나 다 받는다.

다음 두 가지 방법으로 타입을 제한해보자

1. 제네릭 한정적 와일드카드 활용
``` java
<T extends SomeParentType>
```

2. Class.asSubclass 매서드 사용
- 형변환을 해당 타입에 동적으로 제공해줌.
- 캐스팅 불가능하면 예외가 터지는데, 런타임에 알 수 있다는 단점이 있다 -> 자료형에 넣는 시점에 알 수 있다는게 장점인건가..?
``` java
annotationType.asSubclass(Annotation.class)
```
