# public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

기본적으로 필드는 private 으로 두고, 접근자 매서드를 사용할 것.

>package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.

public 클래스가 아닌 내부에서 사용되는 클래스라면 안써도 괜찮거나, 오히려 필드 노출이 나을 것 같다는 저자분의 의견.

``` java
public class TopTestClass {
    
    private int test1;
    private int test2;

    public int getTest1() {
        return test1;
    }

    public void setTest1(int test1) {
        this.test1 = test1;
    }

    public int getTest2() {
        return test2;
    }

    public void setTest2(int test2) {
        this.test2 = test2;
    }
}
```
