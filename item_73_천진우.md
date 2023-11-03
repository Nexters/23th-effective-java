# 아이템73. 추상화 수준에 맞는 예외를 던져라

저수준(하위 클래스들) 에서 예외가 발생하는 경우에, 어느정도 식별 가능한 수준으로 예외를 맞춰서 던져줘야 한다.
<img width="542" alt="스크린샷 2023-11-03 오후 9 43 35" src="https://github.com/Nexters/23th-effective-java/assets/76773202/f7fe7f62-01cd-49c4-b5c0-91bf2309b7c4">

`@Repository` → @component


오! 이거 보니까 딱 생각나는게, hibernate 예외처리.

- JDBC에서 던지는 정합성 관련된 예외들이 너무 디테일함.
- → Hibernate에서 적절한 수준으로 예외를 추상화해서 예외를 줌.
    
<img width="966" alt="스크린샷 2023-11-03 오후 9 44 04" src="https://github.com/Nexters/23th-effective-java/assets/76773202/a195d2e1-2982-4646-b7e7-b89411382860">

    

JDBC에서 다음 예외를 **`DataIntegrityViolationException`** 으로 추상화 시켰습니다.

```jsx
Primary Key Violation: 주 키(primary key)로 지정된 열에 중복된 값이 삽입되는 경우, 데이터 무결성 위반으로 인한 예외가 발생할 수 있습니다.
Foreign Key Violation: 외래 키(foreign key) 관계에서 참조 무결성을 위반하는 경우, 즉, 부모 테이블에 해당 레코드가 없는데 자식 테이블에서 참조하려고 하는 경우에 발생할 수 있습니다.
Unique Constraint Violation: 고유(unique) 제약 조건을 위반하여 열에 중복된 값을 삽입하려는 경우 예외가 발생할 수 있습니다.
Check Constraint Violation: 체크 제약 조건(check constraint)을 위반하는 데이터를 삽입하거나 업데이트하는 경우 발생할 수 있습니다.
Not Null Constraint Violation: 열에 null 값을 허용하지 않는데 null 값을 삽입하려는 경우에 발생할 수 있습니다.
기타 데이터 무결성 위반 사례: 데이터베이스 테이블에 정의된 다양한 제약 조건(예: 길이 제약, 범위 제약)을 위반하는 경우에도 DataIntegrityViolationException이 발생할 수 있습니다.
```

```jsx
데이터를 삽입하거나 업데이트하려는 시도가 무결성 제약 조건을 위반할 때 발생하는 예외이다.
이것은 순수하게 관계적인 개념은 아니며 고유한 기본 키와 같은 무결성 제약 조건은 대부분의 데이터베이스 유형에서 요구된다.

DuplicateKeyException과 같은 보다 구체적인 예외에 대해 수퍼클래스 역할을 한다.

그러나 특정 예외 서브클래스에 의존하지 말고 Data IntegrityViolationException 자체를 처리하는 것이 일반적으로 권장된다.
```


Q. 그러면 만약 디테일한 예외를 보고싶은데, 추상화되면 그것이 안되지 않는가?

→ 추상화된 예외에 저수준 예외를 담아서 구현

<img width="633" alt="스크린샷 2023-11-03 오후 9 44 17" src="https://github.com/Nexters/23th-effective-java/assets/76773202/527209f4-8f60-45ae-93f6-c1ed23210986">
