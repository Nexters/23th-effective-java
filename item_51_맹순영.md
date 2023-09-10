## 아이템 51. 메서드 시그니처를 신중히 설계하라

### 1. 메서드 이름을 신중히 짓기

- 표준 명명 규칙(아이템 68) 따르기
- 이해할 수 있고, 일관되게 짓기
- 개발자 커뮤니티에서 널리 받아들여지는 이름 사용하기
- 긴 이름 피하기

### 2. 편의 메서드를 너무 많이 만들지 말기

- 편의 메서드
    - 말 그대로 편의를 위해 만드는 메서드
    - ex) `Collections` 안에 있는 모든 메서드(`swap`, `min`, `max`)
- 메서드가 너무 많으면 문서화, 테스트, 유지보수하기 어려움
- 클래스나 인터페이스는 자신의 각 기능을 완벽히 수행하는 메서드로 제공해야 함

### 3. 매개변수 개수 줄이기

- 4개 이하가 좋음
- 같은 타입의 매개변수가 여러 개면 더 고민해보기
    - 순서 바꿔도 컴파일되기 때문

## 매개변수 줄이는 방법 3가지

### 1. 여러 메서드로 쪼갠다.

ex) 어떤 리스트에서 주어진 원소를 전체 리스트가 아닌 지정된 범위의 부분 리스트에서 주어진 원소를 찾는 경우

1) 하나의 메서드인 경우

- 부분 리스트의 시작, 부분 리스트의 끝, 찾을 원소로 총 3개의 매개변수 필요

```java
findElementAtSubList(int fromIndexOfSubList, int toIndexOfSubList, Object element);
```

2) 두 개의 메서드인 경우

```java
List<E> subList(int fromIndex, int toIndex);

int indexOf(Object o);
```

subList로 부분 리스트로 쪼갠 뒤 indexOf로 찾으면 됨

<aside>
☝🏻 참고) *직교성이 높다?*
- 공통점이 없는 기능들이 잘 분리되어 있다.
- ex) MSA는 직교성이 높다고 볼 수 있다.
- 언제나 좋은건 아님

</aside>

### 2. 매개변수 여러 개를 묶어주는 도우미 클래스 만들기

- 일반적으로 정적 멤버 클래스(중첩 클래스)로 만들면 됨

ex) 블랙잭 구현 (rank는 카드의 숫자, suit은 카드의 무늬)

```java
dealing(String gamerName, String rank, String suit)
```

해결) rank(카드의 숫자), suit(카드의 무늬)는 항상 같이 포함되기 때문에 아래와 같이 설계하면 좋음

```java
dealing(String gamerName, Card card)

class Blackjack {
    // 도우미 클래스 (정적 멤버 클래스)
    static class Card {
        private String rank;
        private String suit;
    }
}
```

### 3. 도우미 클래스 + 빌더 패턴 활용하기

1. 모든 매개변수를 하나로 추상화한 객체 정의
2. 세터를 호출해 필요한 값 설정
3. execute 메서드를 호출해서 앞서 설정한 매개변수들의 유효성 검증
4. 해당 객체를 넘겨 원하는 계산 수행

## 매개변수 타입으로는 클래스보다는 인터페이스가 낫다

- 매개변수로 적합한 인터페이스가 있다면, 인터페이스 넘기기
    - HashMap 대신 Map 넘기기
- 특정 구현체만 사용하도록 제한하는 꼴

## boolean 보다 원소 2개짜리 열거 타입이 낫다

- 메서드 이름 상 boolean이 명확할 때는 제외

```java
// Bad
Thermometer.newInstance(true);

// Good
Thermometer.newInstance(TemperatureScale.CELSIUS);
```

- 확장할 일이 생기더라도 열거 타입에 추가만 해주면 됨