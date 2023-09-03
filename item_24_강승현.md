# **Summary**

---

- 정적(static) 멤버 클래스
    - 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스
        - `Calculator.Operation.PLUS`
- 비정적(non-static) 멤버 클래스
    - 바깥 클래스의 인스턴스와 암묵적으로 연결된다
    - 어댑터를 정의할 때 자주 쓰인다
    - **멤버 클래스에서 바깥 인스턴스를 참조할 필요가 없다면 무조건 정적 멤버 클래스로 만들자**
- 익명 클래스
    - 바깥 클래스의 멤버가 아니며, 쓰이는 시점과 동시에 인스턴스가 만들어진다
    - 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다
    - **자바에서 람다를 지원하기 전에** 즉석에서 작은 함수 객체나 처리 객체를 만들 때 사용했다
    - `정적 팩터리 메서드`를 만들 때 사용할 수도 있다
        - valueOf() → 객체 생성
- 지역 클래스
    - 가장 드물게 사용된다
    - 지역 변수를 선언하는 곳이면 어디든 지역 클래스를 정의해 사용할 수 있다
    - 가독성을 위해 짧게 작성해야 한다
    

# 정적/비정적 멤버 클래스

---

```java
public class OuterClass {
	
		private static int number = 10;
	
		// 정적
		static private class InnerClass {
			void doSomething() {
				System.out.println(number);
	    }
	  }

		public static void main(String[] args) {
			InnerClass innerClass = new InnerClass();
			innerClass.doSomething();	
		}
}
```

```java
public class OuterClass {
	
		private static int number = 10;
	
		void printNumber() {
			InnerClass innerClass = new InnerClass();
		}
		
		// 비정적
		private class InnerClass {
			void doSomething() {
				System.out.println(number);
				OuterClass.this.printNumber();
	    }
	  }

		public static void main(String[] args) {
			InnerClass innerClass = new OuterClass().new InnerClass();
			innerClass.doSomething();	
		}
}
```

- **Q. 비정적 멤버 클래스를 사용하면 왜 OuterClass는 GC로부터 해제되지 않을까?**
    - GC? 참조되고 있지 않은 객체를 메모리에서 해제하는 역할
    - InnerClass와 OuterClass는 `Strong Reference`로 엮이게 됨
    - 즉, InnerClass는 OuterClass를 참조하고 있는 형태임
    - **InnerClass가 해제되기 전까지 OuterClass도 메모리에 계속 상주하고 있음**

# 익명 클래스

---

```java
public class IntArrays {
		static List<Integer> intArrayAsList(int[] a) {
				
				Objects.requireNonNull(a);
				
				// 다이아몬드 연산자를 이렇게 사용하는 건 Java9부터 가능
				// 더 낮은 버전이라면 <Integer>로 써야함
				return new AbstractList<>() {
						@Override public Integer get(int i) { return a[i]; } // 오토박싱(아이템 6)
						
						@Override public Integer set(int i, Integer val) {
									int oldVal = a[i];
									a[i] = val;  // 오토언박싱
									return oldVal;  // 오토박싱
						}
						@Override public int size() {
									return a.length;
						}
				}
		}

		public static void main(String[] args) {
					int[] a = new int[10];
					for (int i = 0; i < a.length; i++)
								a[i] = i;
		}
}
```

- 클래스 이름은 없지만 정의함과 동시에 사용할 수 있음
- Java8에서 Labmda가 사용되기 전에 많이 사용되었음

# 지역 클래스

---

```java
public class MyClass {
	
	private int number = 10;
	
	void doSomething() {
			class LocalClass {
					private void printNumber() {
							System.out.println(number);
					}
			}
			
			LocalClass localClass = new LocalClass();
			localClass.printNumber();
  }

		public static void main(String[] args) {
			MyClass myClass = new MyClass();
			myClass.doSomething();
		}
}
```

- 이런 경우 거의 없다
- 메서드 자체의 코드가 길어진다

# 어탭터 패턴

---

> *147p*
> 
- 실제로 많이 쓰임
- 기존 코드를 클라이언트가 사용하는 인터페이스의 구현체로 바꿔주는 패턴
- 쉽게 말해서, 이미 제공 중인 인터페이스 vs 사용하려는 인터페이스 차이를 매꿔주는 것임