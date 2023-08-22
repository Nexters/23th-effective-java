# item37.ordinal 인덱싱 대신 EnumMap을 사용하라

## ordinal 인덱싱 사용 예(일차원)

```java
public class Plant {
    enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}
    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle){
        this.name = name;
        this.lifeCycle = lifeCycle;
    }
    @Override
    public String toString() {
        return name;
    }
}
```

```java
Set<Plant>[] plantByLifeCycle = 
	(Set<Plant>[])new Set[Plant.LifeCycle.values().length];

for(int i=0;i<plantByLifeCycle.length;i++){
	plantByLifeCycle[i] = new HashSet<>();
}

for(Plant plant: garden){
	plantByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
}

for(int i=0;i<plantByLifeCycle.length;i++){
	System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantByLifeCycle[i]);
}
```

**무엇이 문제일까?**

- 배열과 제네릭은 호환되지 않음. → 비검사 형변환 수행 필요(경고 발생)
- 배열 인덱스의 정확한 정수값을 사용한다는 것을 보증해야함.(`plant.lifeCycle.ordinal()`)
  → ArrayIndexOutOfBoundsException 발생 할 수 있음.

## EnumMap으로 해결

```java
EnumMap<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle 
	= new EnumMap<>(Plant.LifeCycle.class);

for(Plant.LifeCycle lc : Plant.LifeCycle.values()){
	plantsByLifeCycle.put(lc, new HashSet<>());
}
for(Plant plant: garden){
	plantsByLifeCycle.get(plant.lifeCycle).add(plant);
}
System.out.println(plantsByLifeCycle);
```
- 배열 인덱스 계산 과정이 없으므로 오류에 안전
- EnumMap의 키 타입의 객체는 **한정적 타입**으로 **제네릭 타입 정보**를 사용하기 때문에, 안전한 형변환 사용
- EnumMap 내부 구현에 **배열**을 사용하기 때문에, 배열 성능도 얻을 수 있음.

## 스트림으로 리팩토링

```java
Map<Plant.LifeCycle, List<Plant>> result = garden.stream()
                .collect(groupingBy(p -> p.lifeCycle));

System.out.println(result);
```
- 코드를 줄일 수 있음.
- 해당 코드는 EnumMap이 아닌 고유한 맵 구현체를 사용하므로, EumMap의 성능과 공간 이점이 사라짐

    ```java
    EnumMap<Plant.LifeCycle, Set<Plant>> collect1 = garden.stream()
                    .collect(groupingBy(p -> p.lifeCycle,
                            () -> new EnumMap<>(Plant.LifeCycle.class), toSet()));
    ```
    - 원하는 맵 구현체를 지정 가능(상황에 따라 최적화 가능)


> EnumMap 사용시 value의 존재여부와 상관없이 맵을 생성
스트림 사용시 value가 존재할 때 해당 맵을 생성
>

> **EnumMap의 장점(HashMap,TreeMap과 비교하여)**
>
> - enum은 단일 객체임을 보장하기 때문에, 해싱 작업이 따로 필요없어 탐색 성능이 좋다.
    > (출동 가능성이 없다)
> - TreeMap와 같이 정렬을 위한 순서를 기억한다. enum은 순서가 정해져 있기 때문에 정렬되더라도 입력시 성능이 좋다.

## ordinal 인덱싱 사용 예(다차원)

```java
public class Plant {
    enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}
    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle){
        this.name = name;
        this.lifeCycle = lifeCycle;
    }
    @Override
    public String toString() {
        return name;
    }
}
```

```java
public enum Phase_Ordinal {
    SOLID, LIQUID, GAS;

    public enum Transition{
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };
        public static Transition from(Phase_Ordinal from , Phase_Ordinal to){
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

- Phase, Phase.Transition 열거 타입 수정 시 TRANSITIONS 배열 수정 필요
  → ArrayIndexOutOfBoundsException, NullPointerException 발생 할 수 있음.
- 열거 타입 추가 시 TRANSITIONS 배열 확장 필요.

## EnumMap으로 해결

```java
public enum Phase_EnumMap {
    SOLID, LIQUID, GAS    ;

    public enum Transition{
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID) ;

        private final Phase_EnumMap from;
        private final Phase_EnumMap to;

        Transition(Phase_EnumMap from, Phase_EnumMap to){
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase_EnumMap, Map<Phase_EnumMap, Transition>>
        m = Stream.of(values()).collect(groupingBy(
								t -> t.from, () -> new EnumMap<>(Phase_EnumMap.class),
                toMap(t -> t.to, t -> t,
                        (x,y) -> y, () -> new EnumMap<>(Phase_EnumMap.class))));

        public static Transition from(Phase_EnumMap from, Phase_EnumMap to){
            return m.get(from).get(to);
        }
    }
}
```

- 첫 번째 수집기(Collector)는 이전 상태 기준으로 묶고, 두 번째 수집기는 이후 상태와 전의의 매핑 EnumMap 생성.
- 열거 타입 추가 시, 별다른 코드 변경 없이 Phase, Transition에 열거 타입을 추가하면 됨.