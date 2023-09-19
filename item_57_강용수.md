# 지역변수의 범위를 최소화하라
- 지역변수의 범위를 줄이는 가장 강력한 방법: 가장 처음 쓰일 때 선언하기
  - 미리 선언해두면
    ```cpp
      // 이거 전부 전역변수
      // https://github.com/emost22/cpp-boj/blob/main/bfs/13460.cpp
      queue<pair<pair<pair<int, int>, pair<int, int> >, int> > q;
      char map[10][10];
      bool visit[10][10][10][10];
      int x_direct[] = { 0,1,0,-1 }, y_direct[] = { 1,0,-1,0 };
      int N, M, dx, dy, ans = -1, rcheck, bcheck;
    ```
    - 가독성이 떨어짐
    - 실제 사용하는 시점에서 타입과 초기값이 기억나지 않을 수 있음
- 거의 모든 지역변수는 선언과 동시에 초기화해야 한다.
  - 초기화에 필요한 정보가 충분하지 않다면 선언을 미루도록 한다.
  - 예외: try-catch
    ```java
      int ret;
      try {
        ret = getResult();
      } catch (Exception e) {
        ret = -1;
      }
    ```
    - 변수를 초기화하는 표현식에서 예외를 던질 가능성이 있다면 try 블록 안에서 초기화해야 한다.
    - 변수를 try블록 밖에서도 사용해야 한다면 try블록 앞에서 선언하도록 한다.
- 메소드를 작게 유지하고 한 가지 기능에 집중하도록 한다.
  - 한 메소드에서 여러 기능을 처리한다면 로직 내에서 다른 기능을 수행하는 변수를 사용할 수 있다.

## 반복문을 이용한 초기화
```java
  for (Iterator<Element> i = c.iterator(); i.hasNext();) {
    Element e = i.next();

    // logic
  }
```
```java
  Iterator<Element> i = c.iterator();
  while (i.hasNext()) {
    func(i.next());
  }

  Iterator<Element> i2 = c2.iterator();
  while (i.hasNext()) { // 에러 발생
    func(i2.next());
  }
```
- 반복문 밖에서도 변수를 사용해야하는 상황이 아니라면 while문보다 for문을 쓰는 편이 낫다.
  - for문에서 복붙 실수 발생 시 컴파일 시 확인할 수 있다.
  - while문에서 복붙 실수 발생 시 실제 에러는 발생하지 않아 문제를 확인하기 어려울 수 있다.
  - 복붙 과정에서도 for문에서는 똑같은 변수를 여러번 선언해서 사용할 수 있다.
    - for문 내에서만 선언하여 사용한다고 가정
  - for문이 가독성이 더 좋다.

## 정리
- 지역변수는 처음에 사용될 때 선언하여 초기화한다.
- 한 메소드에는 하나의 기능만 작성한다.
- while보다는 for가 낫다.
