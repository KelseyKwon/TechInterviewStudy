## Concurrency

- 병행성: 여러 개의 스레드가 동시에 실행되는 것처럼 보이는 성질
- 멀티 코어 환경에서는 실제로 동시에 실행되는 Parallelism과 구분
  - concurrency
    - 여러 작업이 겹쳐 진행되는 구조 (인터리빙)
    - 단일 코어에서도 가능 (타임슬라이스 + 컨텍스트 스위치)
  - parrellelism
    - 물리적으로 동시에 실행(멀티코어)
    - 단일 코어에서도 concurrency 문제가 생기나? => 인터리빙 때문에 race 발생
  - os에서 concurrency가 어려운 이유
    - 공유 자원: 메모리, 파일, 소켓, 커널 자료구조(runqueue 등)
    - 스레드 스케쥴링에 의해 실행 순서가 바뀜 -> 결과가 바뀜
    - 원자적으로 보이던 연산이 사실 여러 단계(load/add/store)로 쪼개짐   

### thread vs Process
- process: 독립된 메모리 공간(Code, Data, Heap, Stack)을 가짐
- thread: 프로세스 내에서 stack을 제외한 나머지 메모리 영역 공유
  - 장점: 컨텍스트 스위칭 비용 저렴. 데이터 공유 용이.
  - 단점: 동기화 문제(Race Condition) 발생. 하나의 스레드 장애가 전체 프로세스에 영향
  - 스레드 안전성을 깨는 원인
    - Atomicity 위반: 한 덩어리여야 하는데 쪼개짐
    - Ordering 위반: 한 스레드의 쓰기가 다른 스레드에 제때 보이지 않음
      ```c
      // 겉보기: 1줄
      counter = counter + 1;
      
      // 실제: (개념적으로) load → add → store
      ```
      두 스레드가 interleaving되면 lost update 발생
    - Ordering/Visibility
      - 컴파일러/CPU는 성능을 위해 재정렬 가능
      - 캐시/버퍼 때문에 내가 쓴 값이 상대에게 바로 안보일 수 있음
      - 동기화는 상호배제 + 메모리 가시성 보장까지 포함
        - mutex lock/unlock은 보통 acquire/release 메모리 의미를 제공 

### Race Condition 
- 여러 프로세스/스레드가 공유 자원에 동시에 접근하여 결과값이 접근 순서에 따라 달라지는 상태
- race condition
  - 결과가 실행 순서에 의존해서 달라질 때
  - 버그인지 아닌지는 의도/명세에 달림
    - 어떤 프로그램은 인터리빙이 달라도 결과가 같도록 설계되어 race-free
- data race
  - 두 스레드가 같은 메모리 위치에 접근하고, 최소 하나가 write, 그리고 그 접근들이 동기화되지 않을 때
  - 정의되지 않은 동작 또는 예측 불가능한 동작으로 이어짐
  - Data race ⊂ Race condition 인 경우가 많지만,
  - race condition은 있지만 data race는 없는 설계도 가능함 (락/원자변수로 동기화해서 결과만 순서에 따라 달라ㄴ지는 경우)   

### Critical Section
- 공유 자원에 접근하는 코드 영역
- 3가지 조건을 만족해야 함
  1. Mutual Exclusion (상호배제): 하나의 하나만 진입
  2. Progress (진행): 아무도 없으면 진입하려는 프로세스는 바로 진입 가능해야 함
  3. Bounded Waiting(한정대기): 무한히 기다려서는 안됨 (Starvation 방지)

### Synchronization Tools
- Mutex (Mutual Exclusion)
  - Locking 메커니즘: 자원을 점유할 때 lock()을 걸고, 반납할 때 unlock()
  - Binary Semaphore와 유사하지만, Ownership 개념이 있어 Lock을 건 스레드만 해제 가능
- Semaphore
  - Signaling 매커니즘: 정수 변수 S를 사용하여 가용한 자원의 개수를 나타냄
  - wait(): S를 감소 시킴. S < 0이면 대기
  - signal(): S를 증가시킴. 대기 중인 프로세스를 깨움
- Monitor
  - 세마포어의 복잡한 구현을 해결하기 위해 고안된 고수준 동기화 도구
  - Java의 synchornized
  - 내부적으로 Condition Variable을 사용하여 실행 순서 제어 (wait(), signal())
  - condition variable
    - 락만으로는 조건이 만족될때까지 기다리기를 표현하기 어려움
    - 큐가 비어있으면 소비자는 락을 잡은 채로 대기하면 안됨. 생산자가 못 들어옴
    - 항상 mutex와 함께, while로 조건 검사하기
      - spurious wakeup, signal 경쟁, 꺠어난 뒤  조건이 다시 깨질 수 있음  

### Locks
- 락이 보장해야 하는 것
  - Safety
    - mutual exclusion 보장
    - 보호하는 데이터 불변식 깨지지 않음
  - Liveness
    - deadlock-free
    - starvation-free
  - Performance
    - 경쟁 없을 때 빠름
    - 경쟁 있을 때도 과도한 CPU 낭비/컨텍스트스위치 최소화
- Spin vs Sleep
  - Spin
    - 컨텍스트 스위치 없이 짧게 끝날 때 유리
    - 오래 기다리면 CPU 낭비
  - Sleep
    - 오래 기다릴 때 CPU 절약
    - 대신 잠들고 깨는 비용(스케줄링/시스템콜)이 듦      

### Problems
- Producer-Consumer Problem
  - 공유 버퍼에 데이터를 넣는 생산자와 가져가는 소비자의 속도 차이 및 동기화 문제
  - 해결: Empty/Full 세마포어와 Mutex 사용
- Dining Philosophers Problem
  - 교착 상태를 설명하기 위한 대표적 모델
  - 문제: 모두가 왼쪽 포크를 집으면 아무도 오른쪽 포크를 집을 수 없어 무한 대기 발생

## Deadlock
<img width="679" height="514" alt="image" src="https://github.com/user-attachments/assets/ba6efc05-7e32-41c5-96d6-171e2213bbe7" />
스레드 1은 자원 A를 점유하고 있는 상태에서 자원 B가 필요한 상황, 스레드 2는 자원 B를 점융하고 있는 상태에서 자원 A가 필요한 상황
=> 스레드 1인 자원 B가 필요한 상황에서 자원 A을 빌려줄 수 있는 상황이 아니고, 스레드2도 마찬가지
=> 다수의 스레드가 같은 lock을 동시에, 다른 명령에 의해 획득하려 할 때 서로 불가능한 일ㅇ르 계속적으로 기다리는 ㄴ상황

- 두 개 이상의 프로세스가 서로 상대방이 보유한 자원을 기다리며 무한히 대기하는 상태
- 발생 조건
  1. Mutual Exclusion(상호배제): 자원은 한번에 한 프로세스만 사용 가능
  2. Hold and Wait(점유와 대기): 자원을 가진 상태에서 다른 자원을 기다림
  3. No Preemption(비선점): 다른 프로세스의 자원을 강제로 뺏을 수 없음
  4. Circular Wait(환형 대기): 대기 프로세스들이 사이클을 형성
- 자원 할당 그래프 (Resource Allocation Graph)
  - Vertex: 프로세스(P)와 자원(R)
  - Edge: P -> R (요청), R -> P(할당)
  - 그래프에 Cycle이 없으면 데드락이 아님. Cycle이 있을 때, 자원의 인스턴스가 하나라면 100% 데드락
- 해결 전략
  - Deadlock Prevention
    - 상호배제 부정: 모든 자원을 공유 가능하게 함
    - 저유대기 부정: 실행 전 모든 자원을 한꺼번에 요청하거나, 아무것도 안가졌을 때만 요청 가능
    - 비선점 부정: 요청한 자원을 못얻으면 가진 걸 다 내려놓음
    - 환형 대기 부정: 자원에 고유 번호를 매기고 오름차순으로만 요청
  - Deadlock Avoidance
    - 자원 용청 시 시스템이 Safe State를 유지할 수 있는지 동적으로 확인
    - Banker's Algorithm: 최악의 상황(모든 프로세스가 최대 자원 요청)을 가정해도 해결 가능한지 판단
      - 가용 자원 >= 필요량인 프로세스를 찾아 순차적으로 실행
  - Deadlock Detection & Recovery
    - 데드락 발생을 허용하되, 주기적으로 체크핳여 해결
    - Deteection: 자원할당 그래프를 통해서 사이클 탐지
    - Recovery:
      - Process Termination: 데드락 걸린 프로세스를 모두 종료하거나 하나씩 종료하며 해소 확인
      - Resource Preemption: 희생자를 선정해 자원을 강제로 뺏음
  - Deadlock Ignorance
    - Ostrich Algorithm: 데드락이 매우 드물게 발생하므로 무시
    - 현대 대부분의 OS가 채택하는 방식  
