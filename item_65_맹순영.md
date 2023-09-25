## 아이템 65. 리플렉션보다는 인터페이스를 사용하라

### 리플렉션 기능

- 프로그램에서 **임의의 클래스**에 접근할 수 있음
- Class 객체가 주어지면 해당 클래스의 생성자, 메서드, 필드에 해당하는 Construct, Method, Field 인스턴스를 가져올 수 있음

    ```java
    import java.lang.reflect.*;
    
    public class ReflectionExample {
        public static void main(String[] args) {
            try {
                // 클래스 이름을 사용하여 Class 객체 얻기
                Class<?> myClass = Class.forName("com.example.MyClass");
    
                // 클래스의 생성자 정보 얻기
                Constructor<?>[] constructors = myClass.getDeclaredConstructors();
                for (Constructor<?> constructor : constructors) {
                    System.out.println("Constructor: " + constructor);
                }
    
                // 클래스의 메서드 정보 얻기
                Method[] methods = myClass.getDeclaredMethods();
                for (Method method : methods) {
                    System.out.println("Method: " + method);
                }
    
                // 클래스의 필드 정보 얻기
                Field[] fields = myClass.getDeclaredFields();
                for (Field field : fields) {
                    System.out.println("Field: " + field);
                }
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
        }
    }
    ```

- ex) Method.invoke
    - 어떤 클래스의 어떤 객체가 가진 어떤 메서드라도 호출할 수 있게 해줌

        ```java
        import java.lang.reflect.Method;
        
        public class ReflectionMethodInvokeExample {
            public static void main(String[] args) {
                try {
                    // 클래스 이름을 사용하여 Class 객체 얻기
                    Class<?> myClass = Class.forName("com.example.MyClass");
        
                    // 인스턴스 생성 (기본 생성자를 사용한 예제)
                    Object myInstance = myClass.newInstance();
        
                    // 호출할 메서드 이름과 메서드 매개변수 타입을 지정
                    String methodName = "myMethod";
                    Class<?>[] parameterTypes = new Class[]{int.class, String.class};
        
                    // 호출할 메서드 가져오기
                    Method method = myClass.getDeclaredMethod(methodName, parameterTypes);
        
                    // 메서드 호출하기
                    Object result = method.invoke(myInstance, 42, "Hello, World!");
        
                    // 호출 결과 출력
                    System.out.println("Method return value: " + result);
        		        } catch (ClassNotFoundException | NoSuchMethodException | InstantiationException | IllegalAccessException e) {
        		            e.printStackTrace();
        		        } catch (Exception e) {
        		            e.printStackTrace();
        		        }
            }
        }
        
        class MyClass {
            public int myMethod(int number, String text) {
                System.out.println("myMethod called with number: " + number + " and text: " + text);
                return number * 2;
            }
        }
        ```


### 리플렉션 단점

- 컴파일타임의 타입 검사, 예외 검사가 의미가 없어짐
    - 존재하지 않거나 접근할 수 없는 메서드를 호출할 수 있기 때문
- 코드가 지저분해짐
- 성능이 떨어짐 (일반 메서드 호출보다 훨씬 느림)

### 그럼 어떻게 써야하나?

- 컴파일 타임에 이용할 수 없는 클래스를 사용해야만 하는 프로그램이라면, 적절한 인터페이스 or 상위 클래스를 이용할 수 있을 것임
    - 이런 경우라면 인스턴스 생성에만 리플렉션을 쓰고, 이 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하면 됨

    ```java
    public class ReflectiveInstantiation {
        // 코드 65-1 리플렉션으로 생성하고 인터페이스로 참조해 활용한다. (372-373쪽)
        public static void main(String[] args) {
            // 클래스 이름을 Class 객체로 변환
            Class<? extends Set<String>> cl = null;
            try {
                cl = (Class<? extends Set<String>>)  // 비검사 형변환!
                        Class.forName(args[0]);
            } catch (ClassNotFoundException e) {
                fatalError("클래스를 찾을 수 없습니다.");
            }
    
            // 생성자를 얻는다.
            Constructor<? extends Set<String>> cons = null;
            try {
                cons = cl.getDeclaredConstructor();
            } catch (NoSuchMethodException e) {
                fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
            }
    
            // 집합의 인스턴스를 만든다.
            Set<String> s = null;
            try {
                s = cons.newInstance();
            } catch (IllegalAccessException e) {
                fatalError("생성자에 접근할 수 없습니다.");
            } catch (InstantiationException e) {
                fatalError("클래스를 인스턴스화할 수 없습니다.");
            } catch (InvocationTargetException e) {
                fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
            } catch (ClassCastException e) {
                fatalError("Set을 구현하지 않은 클래스입니다.");
            }
    
            // 생성한 집합을 사용한다.
            s.addAll(Arrays.asList(args).subList(1, args.length));
            System.out.println(s);
        }
    
        private static void fatalError(String msg) {
            System.err.println(msg);
            System.exit(1);
        }
    }
    ```

  ### 예시에서 찾을 수 있는 단점

    - 런타임에 예외가 많이 발생할 수 있다.
        - 리플랙션이 아니라면 컴파일 시점에 잡을 수 있음
    - 코드가 길다

### 그럼 리플렉션은 어디에 쓰일까?

- 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때
- 버전이 여러 개 존재하는 외부 패키지 다룰 때