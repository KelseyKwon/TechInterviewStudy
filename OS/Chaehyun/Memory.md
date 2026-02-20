## 메모리 계층 구조와 주소 바인딩

### 메모리 계층 구조

- **Register → L1/L2/L3 Cache → Main Memory → Secondary Storage**
- 목적: **비용 ↓ + 속도 ↑**
- 핵심 원리:
    - **시간 지역성 (Temporal locality)**
    - **공간 지역성 (Spatial locality)**

⇒ 가상 메모리 + 캐시 + 페이지 테이블은 모두 **지역성**을 전제로 설계됨

### 주소의 종류

- 논리주소 (Logical Address)
    - CPU가 생성하는 주소
    - 프로세스 관점에서의 주소
    - 0번지부터 시작하는 연속 공간
    - 실제 물리 위치와 무관
- 물리주소 (Physical Address)
    - 실제 메모리(RAM)에 올라가는 주소
    - 메모리 버스에 올라가는 값
    - 현대 OS에서는 CPU는 물리 주소를 직접 보지 않음

### 주소 바인딩 시점

| 시점 | 설명 | 특징 |
| --- | --- | --- |
| Compile Time | 컴파일 시 절대 주소 결정 | 재배치 불가 |
| Load Time | 로딩 시 주소 재배치 | MMU 단순 |
| **Execution Time** | **실행 중 동적 변환** | **현대 OS 표준** |

⇒ 실행 도중에도 프로세스 위치가 바뀔 수 있음 → MMU가 반드시 필요

### MMU (Memory Management Unit)

<img width="330" height="207" alt="image" src="https://github.com/user-attachments/assets/2517b0e9-abeb-434e-9ef9-acd957cc2a25" />


- CPU ↔ Memory 사이의 하드웨어
- 논리 주소를 물리 주소로 빠르게 변환해 주는 하드웨어 장치
- `Virtual Address = [ Page Number | Offset ]`
    - Page Number → Page Table Index
    - Offset → Frame 내부 위치
- `Physical Address = Frame Number × Page Size + Offset`
- 논리 주소는 프로그램 실행 시 CPU가 참조하는 주소이고, 물리 주소는 실제 RAM의 주소
    
    ⇒ 이 둘 사이의 변환은 하드웨어 장치인 MMU가 런타임에 수행하여 CPU의 오버헤드를 줄임
    

### TLB (Translation Lookaside Buffer)

- 페이지 테이블을 매번 메모리에서 찾으면 느림
- 페이징을 사용하면 메모리에 두번 접근해야 함
    - Page Table 접근 1회 + 실제 데이터 접근 1회
        
        ⇒ 속도가 2배 느려짐
        
- 페이지 테이블 캐시
- 자주 참조되는 VA → PA 매핑 저장
- TLB hit:
    - 메모리 접근 1번
- TLB miss:
    - Page Table 접근 + 메모리 접근

⇒ TLB miss 비용은 Page fault보다 훨씬 작지만, 성능에 매우 치명적

## 메모리 할당 방식

- 단편화 (Fragmentation)
    - 메모리 공간이 남는데도 활용하지 못하는 현상
    - 외부 단편화(External)
        - 할당할 총 공간은 있는데, 조각나 있어서 큰 프로세스를 못 넣음
        - 가변 분할 방식의 문제
        - 해결책
            - Compaction
            - Paging
    - 내부 단편화(Internal)
        - 프로세스가 필요한 양보다 더 많은 공간을 할당받아 낭비됨
        - 고정 분할 방식의 문제
        - 평균 낭비량
            
            Page size = P → 평균 P/2 바이트 낭비
            
- 페이징 (Paging)
    - 프로세스를 고정크기(Page)로 잘라서 메모리(Frame)에 불연속적으로 할당
    - 외부 단편화 해결, 내부 단편화 발생
- 세그먼테이션 (Segmentation)
    - 프로세스를 논리적 단위(Code, Data, Stack 등)로 잘라서 할당
    - 의미 단위 보호/공유 용이
    - 내부 단편화 해결, 외부 단편화 발생

### 계층적 페이징

- 32bit/64bit 시스템에서 단일 페이지 테이블은 크기가 너무 커서 메모리에 연속적으로 할당하기 힘듦
- 페이지 테이블 자체를 다시 페이징함 (Outer Page Table -> Inner Page Table -> Frame)
- 공간 효율성을 높이지만 메모리 접근 횟수는 늘어남(TLB 필수)

### 리눅스의 buddy system & slab allocator

- Buddy System
    - 커널 메모리 할당 시, 메모리를 2의 제곱수 크기로 쪼개며 할당하고, 해제 시 인접한 빈 블록을 합치는(Coalescing) 방식
    - 외부 단편화 완화
- Slab Allocator
    - 커널 내의 작은 객체들 (프로세스 디스크립터 등)을 위해 미리 할당된 캐시(Slab)를 사용
    - 내부 단편화 해결

## 가상 메모리와 페이지 폴트

- 물리 메모리보다 큰 프로그램을 어떻게 실행하느냐
- 가상 메모리
    - 실제 물리 메모리보다 큰 프로세스를 실행할 수 있게 해주는 기술
    - 당장 필요한 부분만 메모리에 올림 (Demand Paging)
        - Damand Paging
            - Page Tabe Entry (PTE)
                - Valid bit
                - Dirty bit
                - Refernce bit
                - Protection bits
    - 실제 RAM은 캐시처럼 사용됨
- 페이지 폴트 (Page Fault)
    
    <img width="800" height="400" alt="image" src="https://github.com/user-attachments/assets/3537ca76-0360-4c1e-ba9e-264328dc423f" />

    
    - CPU가 액세스 하려는 페이지가 현재 물리 메모리에 없을 때 발생하는 인터럽트
    1. CPU → MMU → PTE 확인
    2. Valid bit = 0
    3. **Trap → Kernel mode**
    4. OS가 접근 합법성 검사
        
        불법이면 Segmentation fault
        
    5. Disk I/O 발생
    6. Victim page 선택
    7. Dirty면 write-back
    8. Page Table / TLB 갱신
    9. **Fault 난 명령 재실행**

⇒ page fault는 exception이지 error가 아님

- Copy-on-Write (COW)
    - 프로세스 생성(`fork()`) 시, 부모의 메모리를 다 복사하면 낭비
    - 처음에는 부모/자식이 같은 무리 메모리를 공유(Read only)하다가, 누군가 쓰기(Write) 작업을 할 때 비로소 새로운 페이지를 할당하고 복사
    - 프로세스 생성 속도 최적화의 핵심 기법
- Dirty Bit
    - 페이지 폴트 시 Swap-out할 때, 무조건 디시크에 쓸 필요가 없음
    - 메모리에 올라온 후 수정된 적 있는 페이지(Dirty Bit = 1)만 디스크에 기록하고, 수정 안된 건 그냥 덮어 씌움 (I/O 비용 절감)

## 페이지 교체 알고리즘과 스레싱

- 이론적 최적
    - OPT (Belady)
    - 미래 참조 필요 → 구현 불가
- 구현 가능
    - FIFO
        - 가장 먼저 들어온 페이지 교체
        - Belady의 모순 발생 가능
    - LRU (Least Recenlty Used)
        - 가장 오랫동안 참조되지 않은 페이지를 교체
        - 시간 지역성 고려, 가장 실용적
        - 하드웨어 지원 없으면
            - Timestamp
            - Stack → 오버헤드 큼
        - 이론적으로는 좋지만 구현 비용이 비쌈
    - Clock (Second Chance)
        - LRU를 하드웨어적으로 완벽 지원하려면 모든 접근마다 타임스탬프를 찍어야 해서 오버헤드가 큼
        - 원형 큐를 사용하여 Reference Bit를 확인
        - 비트가 1이면 0으로 바꾸고 기회를 한 번 더 줌
        - 0이면 교체
        - 대부분의 상용 OS가 채택한 방식 (LRU 근사)
    - LFU / MFU
        - LFU: 참조 횟수가 가장 적은 페이지 교체
- Belady’s Anomaly
    
    <img width="960" height="720" alt="image" src="https://github.com/user-attachments/assets/b7b1e2ea-b94a-4dba-8b52-088a5a2fb3bf" />

    
    - FIFO만 발생
    - 프레임 수 ↑ → Page fault ↑ 가능
    - Stack Algorithm (LRU, OPT)는 발생하지 않음
        
        <img width="623" height="381" alt="image" src="https://github.com/user-attachments/assets/30799706-6c6f-4502-9b4e-fed14fa8f2d0" />

        
        - **3 Frames:** 페이지 폴트 10회 (Miss Ratio 83.3%)
        - **4 Frames:** 페이지 폴트 8회 (Miss Ratio 66.7%)
        - FIFO와 달리 프레임 수가 늘어날수록 페이지 폴트가 감소하는 **안정적인 성능**
        - **FIFO** 알고리즘은 특이하게도 프레임 수를 늘려줬는데(3개 -> 4개), 오히려 페이지 폴트가 더 많이 발생하는 기현상(모순)이 생길 수 있음
        - 하지만 **LRU**는 '스택 알고리즘(Stack Algorithm)'에 속하기 때문에, **프레임 수를 늘리면 페이지 폴트 수가 줄어들거나 최소한 같다는 것이 수학적으로 보장**
- 스레싱 (Thrashing)
    - 페이지 부재가 너무 빈번하게 발생하여, CPU가 프로세스 실행보다 페이지 교체(I/O)에 대부분의 시간을 쏟는 현상
    - 원인
        
        Working set > 할당된 frame
        
    - 성능 급락
    - 해결 방법: 다중 프로그래밍의 정도(MPL)를 낮추거나, 프로세스가 필요로 하는 최소한의 프레임 수를 보장해주는 워킹 셋(Working Set) 알고리즘이나 PFF(Page Fault Frequency) 알고리즘을 사용하여 조절
        - Working Set Model
            - 최근 시간 동안 참조된 페이지 집합
            - 각 프로세스의 최소 프레임 수 요구량 추정
        - PFF (Page Fault Frequency)
            - Fault 빈도 기준으로 frame 조절
            - OS가 동적으로 MPL 제어
            
            ⇒ Thrashing은 ‘알고리즘 문제’가 아니라 ‘메모리 정책 문제’
            

## Javascript의 메모리 구조

<img width="880" height="495" alt="image" src="https://github.com/user-attachments/assets/b21fb1d9-08a8-4cad-b1be-23720b761f4e" />


- Call Stack
    - 원시 타입(Primitive) 데이터와 실행 컨텍스트가 저장
    - 함수 호출 깊이 제한 → Stack Overflow
- Heap
    - 참조 타입(Object, Array) 데이터가 저장됨. 크기가 동적으로 변함
    - 실제로는 OS 가상 메모리 위에 힙 영역 할당
- Garbage Collection (GC)
    - OS가 메모리를 관리하듯, JS 엔진 (V8)은 ‘Mark and Sweep’ 알고리즘 등을 통해 도달 불가능한 객체의 메모리를 해제함
- V8 메모리 구조 (The Heap)
    - New Space: 생명주기가 짧은 객체들이 겨주
        - Minor GC
            - Cheney’s Algorithm 사용
            - 공간을 Semispace 2개 (From, To)로 나누어 살아있는 객체만 To로 복사하고 나머지는 버림
            - 속도가 매우 빠름
    - Old Space: New Space에서 2번 이상 살아남은 객체가 승격되어 이동
        - Major GC: Mark-Sweep-Compact 알고리즘 사용
            - **Mark:** Root (Global, Stack)에서 도달 가능한 객체 마킹 (Tri-color marking).
            - **Sweep:** 마킹 안 된 객체 해제.
            - **Compact:** 단편화 해결을 위해 메모리 한쪽으로 몰기 (비용이 큼, Stop-the-world 발생 최소화를 위해 Incremental Marking 사용)
- GC는 메모리 누수를 자동으로 해결하지 않음 
→ 참조가 남아있으면 누수 
→ Closure, Global 변수, Event Listener 주의
    - 의도치 않은 전역 변수: window 객체에 붙은 변수들
    - 해제되지 않은 타이머/이벤트 리스너: setInterval, addEventListener 후 remove 안했을 때
    - Closure: 불필요하게 스코프를 잡고 있는 경우
    - Detached DOM elements: DOM 트리에서는 제거되었으나 JS 변수가 참조하고 있어 메모리에 남은 DOM 노드
