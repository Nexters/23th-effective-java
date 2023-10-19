# 가능한 한 실패 원자적으로 만들라
- 실패 원자적: 호출된 메소드가 실패하더라도 객체는 메소드 호출 전 상태를 유지해야 한다.

## 메소드를 실패 원자적으로 만드는 방법
- 불변객체로 설계
  - 생성된 시점에 고정되어 절대 변하지 않는 값
  - 메소드가 실패해도 기존 객체가 불안정한 상태가 되지 않음
- 매개변수의 유효성 검사
  - 객체의 내부 상태를 변경하기 전에 잠재적 예외의 가능성을 걸러내는 방법
    ```java
      public Object pop() {
        if (size == 0) throw new EmptyStackException();

        Object result = elements[--size];
        elements[size] = null;
        return result;
      }
    ```
    - `size == 0`이면 미리 `EmptyStackException` 에러를 던짐
      - 해당 코드가 없더라도 `elements[--size]`에서 `ArrayIndexOutOfBoundsException` 에러가 발생함 -> 추상화 수준이 상황에 어울리지 않음
- 객체의 임시 복사본에서 작업을 수행한 다음, 원래 객체와 교체한다.
  - 데이터를 임시 자료구조에 저장해서 작업하는게 더 빠를 때 사용
    - ex. 정렬을 수행하기 전에 리스트의 원소들을 배열로 옮겨담아서 정렬하는 방식
    - 정렬에 실패하더라도 원래 데이터는 변하지 않는다.
    ```java
      // List의 sort 메소드
      default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
      }
    ```
- 작업 도중 발생하는 실패를 가로채는 복구 코드를 작성해서 작업 전 상태로 되돌리는 방법
  - 내구성을 보장해야 하는 자료구조에 사용됨 (자주 쓰이지는 않음)

## 실패 원자성은 항상 따라야하는 것은 아니다
- ex. 두 스레드가 동시에 같은 객체를 수정한다면 일관성이 깨질 수 있다.
- 에러는 복구할 수 없으므로 AssertionError에 대해서는 실패 원자적으로 만들려는 시도도 할 필요 없다.
- 실패 원자성을 달성하기 위한 비용이나 복잡도가 아주 크다면 따르지 않는 것이 좋다.
- 메소드 명세에 기술한 예외라면 객체의 상태를 유지해야 한다.
  - 아니면 실패 시의 객체 상태를 API 설명에 명시해야 한다.
