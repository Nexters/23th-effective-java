# item40 @Override 애너테이션을 일관되게 사용하라.

### 잘못된 상황

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second){
        this.first = first;
        this.second = second;
    }
    public boolean equals(Bigram b){
        return b.first==first && b.second==second;
    }
    public int hashCode(){ //잘한 부분
        return 31 * first + second;
    }
    public static void main(String[]args){
        //Set은 중복 허용하지 않음.
        Set<Bigram> s = new HashSet<>();
        for(int i=0;i<10;i++){
            for(char ch = 'a'; ch <= 'z'; ch++){
                s.add(new Bigram(ch, ch));
            }
        }
        System.out.println(s.size()); //출력:260
    }
}
```

- Object 클래스의 equals 메서드를 재정의(오버라이딩)하지 않고 **다중 정의함**
- Object equals은 == 연산자처럼 **객체 식별성(논리적 동치성이 아닌)**을 확인

### @Override 애너테이션을 사용하자.

```java
@Override public boolean equals(Object o){
        if(!(o instanceof Bigram)){
            return false;
        }
        Bigram b = (Bigram) o;
        return b.first==first && b.second==second;
    }
```

- **컴파일 시점**에 잘못된 오류를 알려준다.

  ex) 매개변수 Bigram 사용 등

- 상위 클래스의 추상 메서드(abstract method)를 재정의 할때 굳이 사용할 필요없음. 컴파일러가 이미 알고 있다.(하지만 일관되게 다는 것이 좋음)
- 즉, 상의 클래스의 메서드를 재정의하려는 모든 메서드에 **@Override** 애너테이션을 달자!