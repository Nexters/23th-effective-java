### 431p ‘상태 의존석 수정’

concurrent 컬랙션에서 동시성을 무력화 하지 못하므로, 여러 매서드를 원자적으로 묶어 호출하는게 불가능하다

→ 상태 의존석 수정 매서드 제공

ex) `putIfAbsent` → 1. key값에 해당하는 값이 있나? 2. 없으면 넣어주기  두 개의 작업이 통합됨.

즉 자료형 특성상, 위의 과정을 나눠서 호출할 수 없는데, 내부적으로 구현된 매서드를 제공

### wait과 notify가 무엇인가?

- 동시성 제어를 위한 기법.
- object에 선언되어 있음
- 호출하는 스레드가 `synchronized` 블록 내에서 호출되어야 함: 고유한 락을 보유하고 있어야 함.
- wait: 갖고 있던 고유 락을 해제 → 스레드 sleep
- noitfy: sleep 상태 스레드 중 임의로 호출

```jsx
producer-consumer 문제

1) 마켓에 물건이 가득 찬 경우

마켓에 물건이 가득 찼을 경우에는 생산자(producer)가 더이상 생산하지 않고 기다리게 한다. - wait()
소비자(consumer)가 물건을 소비한 후에 생산자(producer)에게 알려준다. - notify() or notifyAll()
 

2) 마트에 물건이 없을 경우

마켓에 물건이 없을 경우에는 소비자(consumer)가 더이상 소비하지 못하고 기다리게 한다. - wait()
생산자(producer)가 물건을 생산한 후에 소비자(consumer)에게 알려준다. - notify() or notifyAll()

https://kadosholy.tistory.com/124
```

### 참고 - 스레드 라이프 사이클
<img width="777" alt="스크린샷 2023-11-03 오후 9 46 00" src="https://github.com/Nexters/23th-effective-java/assets/76773202/f71c6265-65c8-416a-80f0-af2b33dcfde4">

### wait 사용시 주의사항

- 반드시 대시 반복문을 사용.
    - wait 이후에도 응답 불가 상태가 있을 수 있음.
    - 상태체크를 통해 더블체크
        
        ```java
        synchronized (obi) {
        	while (<조건이 충족되지 않았다> ) {
        		obj.wait(); // (락을 놓고, 깨어나면 다시 잡는다. )
        	}
        }
        ```
        

### notify 사용시 주의사항

- 일반 notify 말고, notifyAll을 사용하자.
- notify는 랜덤하게 하나의 스레드만을 깨우는데, 가용 범위 스레드가 깨어남을 보장하지 못함!
- 스레드를 최대로 활용하지 못해서 성능 문제가 있을 수 있음.

### 컬렉션에서의 Product-consumer 해결

일부 컬랙션은 자업이 성공적으로 완료가 될 때까지 기다려준다!

Queue를 확장한 BlockingQueue .

- 큐가 비어있으면 새로운 원소가 추가될 때까지 기다린다.
- Thread Pool에서도 이와 같은 방식 사용!

### 동기화 장치들

- CountDownLatch : 특정 스레드가 다른 스레드가 끝날때까지 기다림
- Semaphore
- Phaser
