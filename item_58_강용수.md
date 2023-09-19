# 전통적인 for문보다는 for-each문을 사용하라
```java
  // 컬렉션 순회
  for (Iterator<Element> i = c.iterator(); i.hasNext();) {}

  // 배열 순회
  for (int i = 0; i < a.length; i++) {}
```
- while보다는 낫지만 가장 좋은 방법은 아님
  - 반복자와 인덱스 변수는 사용하지 않을 수도 있고, 코드가 지저분해짐
  - 배열/컬렉션에 따라 코드 형태가 달라짐

```java
  for (Element e : elements) {}
```
- 컬렉션, 배열 모두 같은 형태로 처리할 수 있음

## 중첩 반복문
```java
  for (Iterator<Suit> i = suits.iterator(); i.hasNext();)
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext();)
      deck.add(new Card(i.next(), j.next()));
```
- 문제점: i.next()가 너무 많이 호출됨
  - NoSuchElementException 에러 발생

```java
  for (Suit suit: suits)
    for (Rank rank: ranks)
      deck.add(new Card(suit, rank));
```
- 위 문제를 해결하면서 코드도 간결해짐

## for-each를 사용할 수 없는 상황
- 파괴적인 필터링 (destructive filtering)
  - remove 메소드를 호출해서 원소를 제거하는 경우
  - java 8부터는 Collection의 removeIf 메소드를 사용해서 컬렉션을 순회해서 제거할 필요가 없어짐
- 변형 (transforming)
  - 리스트나 배열을 순회하면서 그 원소의 값을 변경해야 한다면 인덱스를 활용해야 한다.
- 병렬 반복 (parallel iteration)
  - 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스를 사용해서 제어해야 한다.

## 정리
- 가능하면 for-each를 사용하도록 하자
