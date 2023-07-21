# 클래스와 맴버의 접근 권한을 최소화하라.

### 접근권한
- public
- default: 동일 패키지 = package private
- protected: 동일 패키지 & 하위 클래스
- private: 동일 클래스

### 접근제어자의 목적: 캡슐화 / 데이터 은닉화
- 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 잘 숨겼는가?
- 잘 설계된 컴포넌트: 구현과 API의 분리

여기에서 API는 어떤 의미로 쓰인 것 일까?
- 컴포넌트(객체) 사이에 연결되는 포인트

### 캡슐화 / 데이터 은닉화의 이점
- 컴포넌트 병렬 개발 분업 가능
- 컴포넌트 복잡도 낮춤
- 컴포넌트 재사용
- 추후 완성된 시스템을 프로파일링해 최적화 도모
  - >정보 은닉 자체가 성능을 높어주지는 않지만, 성능 최적화에 도움을 준다. 완성된 시스템을 프로파일링해 최적화할 컴포넌트를 정한 다음(아이템 67), 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화할 수 있기 때문이다.
  - 프로파일링? 소프트웨어나 시스템의 실행 중에 발생하는 성능과 관련된 정보를 수집하고 분석하는 작업
  - 즉 외부에 노출된 부분들만 집중해서 분석하고 빠른 최적화 가능.

### 대 원칙: 일단 숨겨라
가능한 최대한 낮은 접근권한을 부여할 것
패키지 외부에서 접근할 일이 없다면, package private 으로 설정할 것

### 잠깐 package private 을 살펴봅시다

계층 구분 없이 무조건 동일한 패키지 내에서만 접근이 가능합니다.

몰라서 찾아봤습니다.

GPT가 상위 패키지의 클래스에서 하위 패키지 클래스에 접근이 가능하다고 했음

-> 실제 테스트 해보니까 안됨 ㅡㅡ

### 테스트를 위해 접근제어자를 풀지 마라

http://shoulditestprivatemethods.com

> 이규원님 TDD 강의: 공개된 인터페이스를 통해서 사용자가 사용을 하게 되는데, 테스트도 이와같다. 사용자는 알 수 없는 숨겨진 private를 억지로 테스트하려면 내용이 노출 될 수밖에 없다. 테스트가 숨겨진 내부에 강하게 결합하게 된다.

ㄷㄷ 테스트에 결합이 된다라..

그리고 시간이 없다고 합니다 ^^..

또한 차라리 상위 api 태스트에서 해당 private 매서드의 동작에 대해서 테스트 하는 방법이 좋을 것 같네요.

### public 클래스의 인스턴스 필드는 되도록 publicol 아니어야 한다
private 으로 두고, getter setter 등의 접근 제어 매서드 사용

그러나 추상 개념을 완성하는 데 꼭 필요한 구성요소로써의 상수라면 public static final 필드로 공개해도 좋다.
> 반복되는 개념에 대한 얘기일까.

단 불변 타입 / 객체를 참조할 것!


### 배열을 외부에 공유하는 방법?

unmodifiableList: 불변 객체를 return. 추가 삭제 수정 불가능

``` java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES = 
    Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

앞에 나왔던 clone 으로 방어적 복사
``` java
private static final Thing [] PRIVATE_ VALUES = { ... };
public static final Thing [] values () {
    return PRIVATE_VALUES.clone();
}
```

### 모듈 시스템 접근제어

멀티모듈 프로젝트에서 특정 패키지를 공유 가능!

예시
``` gradle
dependencies {
    api project(':다른_모듈_이름')
}
```

우리에에게 익숙한 Java 기본 제공 라이브러리는 JDK에서 제공하는데, 물론 JDK도 내부적으로 사용되는 구현 패키지들이 있으나 외부에 공개하지 않았기 때문에 안보이는 것.