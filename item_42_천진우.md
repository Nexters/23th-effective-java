# 익명 클래스보다는 람다를 사용하라

### Collections.sort 예제
익명 클래스 사용
``` java
Collections.sort(words, new Comparator<String>()){
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
}
```


람다로 변환
``` java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```


``` java
public enum Operation {
    PLUS  ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);
 
    private final String symbol;
    private final DoubleBinaryOperator op;
 
    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }
 
    @Override
    public String toString() { return symbol; }
 
    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
 
    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        for (Operation op : Operation.values())
            System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
    }
}
```

`xxBinaryOperator` -> 인자가 두개인 연산 식 xx 는 타입을 나타냄.

`doubleBinaryOperator.applyAsDouble(x, y)` -> X 와 Y 를 인자로 넘겨서 연산


### 람다의 단점.
람다는 이름이 없고 문서화도 못 한다.

그러므로 다음과 같은 경우 가독성을 해치니 쓰지 않는 것이 좋다.
- 함수명으로 설명이 필요한 코드인 경우
- 코드가 너무 긴 경우


### 람다는 자신 을 참조할 수 없다.

Q. 익명 클래스와 람다의 차이점은?

- 람다에서의 this 키워드는 바깥 인스턴스를 가리킨다.
- 익명클래스의 this 는 익명 클래스를 가리킨다.

예시 코드를 짜봤습니당

``` java
class LambdaExample {
    
    private final String value = "outer";

    public void performAction() {

        // 람다
        Runnable runnableLambda = () -> {
            final String value = "inner";
            
            System.out.println("runnableLambda: Value from outer instance: " + this.value);
            // Value from outer instance: inner
            System.out.println("runnableLambda: this.getClass() = " + this.getClass().getName());
            // this.getClass() = com.nexters.phochak.LambdaExample
        };
        new Thread(runnableLambda).start();


        //익명클래스
        Runnable runnableAnonymous = new Runnable() {
            final String value = "inner";
            
            @Override
            public void run() {
                System.out.println("runnableAnonymous: Value from outer instance: " + this.value);
                // Value from outer instance: inner
                System.out.println("runnableAnonymous: this.getClass() = " + this.getClass().getName());
                // com.nexters.phochak.LambdaExample$1
            }
        };

        new Thread(runnableAnonymous).start();
    }

    public static void main(String[] args) {
        LambdaExample example = new LambdaExample();
        example.performAction();
    }
}
```

### 직렬화 왠만하면 금지
람다 직렬화는 JVM 마다 다르다.

그렇지만 방법을 찾아보았다.

https://stackoverflow.com/questions/22807912/how-to-serialize-a-lambda

``` java
Runnable r = (Runnable & Serializable)() -> System.out.println("Serializable!");
```

????????????????????????????????????????

람다 식을 Serializable로 mutliple 캐스팅..


### 그리고 면접에서 나왔던 캡쳐링 람다 관련

https://netal.tistory.com/entry/JAVA-%EB%9E%8C%EB%8B%A4lambda-%EC%9D%98-%EC%9E%91%EB%8F%99-%EB%B0%A9%EC%8B%9D%EA%B3%BC-%EC%A7%80%EC%97%AD%EB%B3%80%EC%88%98-%EC%82%AC%EC%9A%A9
