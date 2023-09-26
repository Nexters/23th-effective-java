# 문자열 연결은 느리니 주의하라
```java
  public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++) {
      result += lineForItem(i);
    }

    return result;
  }

  public String statement2() {
    StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
    for (int i = 0; i < numItems(); i++) {
      b.append(lineForItem(i));
    }

    return b.toString();
  }
```
- String의 `+` 연산으로 문자열을 연결하는 시간은 $O(n^2)$이다.
  - 문자열은 불변이므로 문자열을 복사해야한다.
- StringBuilder의 append 연산으로 문자열을 연결하는 시간은 $O(N)$이다.

```java
  for (int i = 0; i < str.length(); i++) {}
  
  vs

  int len = str.length();
  for (int i = 0; i < len; i++) {}
```
- str.length() 연산에도 O(N)의 시간이 걸린다.
