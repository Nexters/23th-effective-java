# item46 스트림에는 부작용 없는 함수를 사용하라.

## 스트림 연산에 건네는 함수 객체는 모두 부작용이 없어야된다.

- 스트림의 핵심은 계산을 일련의 변환으로 재구성 하는 것.
- 각 변환 단계(중간, 종단 단계든)는 **순수 함수**여야 한다.
  - 다른 가변 상태 참조 x
  - 다른 상태를 변경하지 x

```java
List<String> words = List.of("one", "one", "two", "three", "three");

//잘못된 스트림 사용!
Map<String, Long> freq = new HashMap<>();
words.stream()
	.forEach(word -> {
		freq.merge(word.toLowerCase(), 1L, Long::sum);
});
```

- 사실상 반복적 코드와 다를게 없음.
- 읽기 어렵고, 유지보수에도 좋지 않음.
- 종단 연산인 forEach의 람다에서 다른 상태(freq)를 변경함.
  → 순수 함수 특성 위반.

```java
//스트림 제대로 사용
Map<String, Long> freq1 = new HashMap<>();
freq1 = words.stream()
	.collect(groupingBy(String::toUpperCase, counting()));
```

- 순수 함수 특성을 지킴.
- forEach 종단 연산은 **스트림 계산 결과**를 보고할 때만 사용.
- 예외적으로, 기존 컬렉션에 추가하는 용도로 사용은 가능.(지양)

## 스트림의 수집기(Collectors)

- 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있는 방법.
- toList(), toSet(), toCollection(collectionFactory), toMap 등 존재.
- 컬렉션의 구현체는 Collectors 클래스 내부에 사전에 정의되어 있음.
- 39개의 메서드가 존재.
  - toList()
    - 모든 Stream의 요소를 List 인스턴스로 수집
    - 모든 Stream의 요소를 List 인스턴스로 수집
  - toSet()
    - 모든 Stream의 요소를 Set 인스턴스로 수집
    - 중복 x
  - toCollection()
    - List, Set 등의 특정 컬렉션 타입을 지정 가능.
  - toMap()
    - 모든 Stream 요소를 key, value로 매핑된 Map 인스턴스로 수집
  - groupingBy()
    - 제공된 그룹화 기준으로 그룹화 한 맵 인스턴로 수집
  - minBy(), maxBy()
    - 제공된 Comparator 기반의 비교를 통해 최소, 최대 값 반환.
  - counting
    - 스트림 요소 수를 반환, 다운 스트림 수집기 전용.
  - joining
    - 스트림 요소를 단일 문자로 연결하는 수집기
  - partitioningBy, summing, averaging, summarizing, reducing, filtering, mapping 등 존재.

```java
public class Stream_Collectors {
    public static void main(String[] args){
        List<String> words = List.of("one", "one", "two", "three", "three");

        //Collectors-ToList
        List<String> topTwo = freq1.keySet().stream()
                .sorted(comparing(freq1::get).reversed())
                .limit(2)
                .collect(toList());
        System.out.println(topTwo.toString());

        //Collectors-ToSet
        Set<String> resultSet = words.stream().collect(toSet());
        System.out.println(resultSet);

        //Collectors-toCollection
        LinkedList<String> resultToCollection = words.stream().collect(toCollection(LinkedList::new));
        System.out.println(resultToCollection);

        //Collectors-toMap(기본, 인수 2개)
        List<String> nWords = List.of("one", "two", "three");
        Map<String, Integer> resultToMap2 = nWords.stream().collect(toMap(w -> w, String::length));
        System.out.println(resultToMap2);
//        Map<String, Integer> resultToMap2 = words.stream().collect(toMap(w -> w, String::length));
//        System.out.println(resultToMap2); //예외 발생 "같은 키 사용"

        //Collectors-ToMap(인수 3개, BinaryOperator)
        //같은 키 사용 해결법
        Map<String, Integer> resultToMap3 = words.stream().collect(toMap(w -> w, String::length, (w1, w2) -> w1 + w2));
        System.out.println(resultToMap3);

        //Collectors-ToMap(인수 4개, 맵 팩터리)
        TreeMap<String, Integer> resultToMap4 = words.stream().collect(toMap(w -> w, String::length, (w1, w2) -> w1 + w2, TreeMap::new));
        System.out.println(resultToMap4);

        //groupingBy 인수 1개
				//맵키로 사용될 분류함수
        Map<String, List<String>> resultGroupingBy1 = words.stream().collect(groupingBy(w -> w));
        System.out.println(resultGroupingBy1);

        //groupingBy 인수 2개(downStream: 리스트 외 다른 값으로 맵 생성)
        Map<String, Long> resultGroupingBy2 = words.stream().collect(groupingBy(String::toUpperCase, counting()));
        System.out.println(resultGroupingBy2);
        Map<String, Set<String>> resultGroupingBy2_set = words.stream().collect(groupingBy(String::toUpperCase, toSet()));
        System.out.println(resultGroupingBy2_set);

        //groupingBy 인수 3개(맵 팩터리)
        //"점진적 인수 목록 패턴"에 어긋
        TreeMap<String, Set<String>> resultGroupingBy3 = words.stream().collect(groupingBy(String::toUpperCase, TreeMap::new, toSet()));
        System.out.println(resultGroupingBy3);

        //joining
        //List<Character> 도 가능
        String joining = words.stream().collect(joining());
        System.out.println(joining);
        String joining1 = words.stream().collect(joining(","));
        System.out.println(joining1);
        String joining3 = words.stream().collect(joining(",", "[", "]"));
        System.out.println(joining3);
    }
}
```