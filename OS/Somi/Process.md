# 프로세스

# 1. 프로세스

: 쉽게 말해 ‘ 실행 중인 프로그램 ‘

디스크에 있는 프로그램이 메모리에 로드 되면 → 프로세스가 된다

하나의 프로그램 → 여러개의 프로세스

### 프로세스 문맥 ( Process Context )

: 프로세스 특정 시점의 상태를 표현

### 하드웨어 문맥 ( Hardware Context )

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FberUN5%2FbtrejfCRS7h%2FAAAAAAAAAAAAAAAAAAAAAACBR3DMWsEqQX_fobr1ZGVplsExYAWK4VQNLb_Bawev%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1767193199%26allow_ip%3D%26allow_referer%3D%26signature%3DnNKIQHzHhl8m8KCwrz9fQjGaYvY%253D">

- 프로그램 카운터 ( Program Counter )
- 각종 레지스터

→ 프로세스가 어디까지 실행되었는지 알 수 있음

- 프로세스의 주소공간
- PCB ( Process Control Block ) : 프로세스 관련 커널 자료구조
- 프로세스 커널 스택 ( Process Kernel Stack ) : 프로세스 관련 커널 자료구조

# 2. Process State

- New : 프로세스 생성된 상태
- Ready :  프로세스가 CPU에 할당되기를 기다리는 상태
- Running : 프로세스 할당 후 CPU를 잡고 명령 수행 중인 상태
- Waiting : 프로세스가 어떠한 이벤트가 발생하기를 기다리는 상태
    - CPU 할당해도 당장 명령 수행 불가 상태
- Terminated : 프로세스 실행 마친 후 프로세스 제거 전 상태

# 3. Process Control Block ( PCB )

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FczZPFr%2Fbtrec4iShbB%2FAAAAAAAAAAAAAAAAAAAAAOU8qZzjaK1peSfIe-Ps6An8yD1XDilyU7CVPmlMvI2P%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1767193199%26allow_ip%3D%26allow_referer%3D%26signature%3DgorxU8Cf8r0MRTFqX90vwDjMp50%253D">

: 운영체제가 각 프로세스를 관리하기 위해 프로세스별로 보유하고 있는 자신의 정보 묶음

== 운영체제가 프로세스를 표현한 것이다

== 프로세스 스케줄링을 위해 프로세스에 관한 모든 정보를 가지고 있는 DB

- 프로세스가 생성될 때마다 고유의 PCB 생성 → 프로세스 완료 시 제거

1. 운영체제가 관리상 사용하는 정보
    - Process state, Process ID
    - Scheduling information : 프로세스의 중요도, 스케줄링 파라미터 정보
    - Priority : 프로세스의 우선순위
2. CPU 수행 관련 하드웨어 값
    - Program Counter : 해당 프로세스가 이어서 실행해야 할 명령의 주소를 가리키는 포인터
    - Register : 프로세스가 인터럽트 이후 올바르게 작업을 이어가기 위해 참조하는 CPU 레지스터 값
3. 메모리 관련
    - Code, Data, Stack의 위치 정보, base/limit 레지스터 값
4. 파일 관련
    - Open file descriptors : 열린 파일 목록

# 4. Context Switch

: 프로세스가 실행되다가 → 인터럽트 발생 → CPU를 다른 프로세스로 넘겨주는 과정

CPU 입장에서 Context == PCB 이므로 **PCB 정보가 바뀌는 것이 Context Switch**

👉🏻 인터럽트가 발생한다고 반드시 Context Switch가 아님. 다른 프로세스에 프로세서가 넘어가야 Context Switch임

💡 ‼️ CPU를 다른 프로세스로 옮기기 위해 이전의 프로세스의 상태를 PCB에 보관하고 새로운 프로세스의 보관된 상태를 복구하는 작업

### 언제 발생하는가?

1. 멀티태스킹 ( Multitasking )
    
    : 실행 가능한 프로세스들이 운영체제의 스케줄러에 의해 번갈아가며 수행
    
    → 번갈아가면서 프로세스가 CPU 할당받을 때 → Context Switching ! 
    
    ( 사용자가 체감하기 힘들다 )
    
2. 인터럽트 핸들링 ( Interrupt handling )
    
    : 인터럽트 발생시 → Context Switching ! 
    
    ---
    
    ⁉️ 인터럽트란?
    
    컴퓨터 시스템에서 예외 상황이 발생했을 때 CPU에게 알려 처리할 수 있게 하는 것
    
    ---
    
3. 사용자와 커널 모드 전환 ( User and kernel mode switching )
    
    : Context Switch가 필수는 아니지만 운영체제에 따라 발생할 수도 있음
    

### 과정

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fc3wWnK%2Fbtq2NcrH2wm%2FAAAAAAAAAAAAAAAAAAAAAPdctWo6g8ZU8mTcUX7Oxg5EmEembAgcA3GI6tQgcV5b%2Fimg.jpg%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1767193199%26allow_ip%3D%26allow_referer%3D%26signature%3DgoHmv5UYhjmYZbjdu%252BElQECXi5A%253D">
1. 요청 발생 : 인터럽트에 의한 요청 발생
2. PCB에 저장 : 운영체제가 현재 실행중인 프로세스의 정보를 PCB에 저장
3. CPU 할당 : 운영체제가 다음 프로세스의 정보를 PCB에서 가져와 CPU를 할당

---

⁉️ Overhead : Context Switching하는 동안 CPU가 아무일도 하지않는 시간이 발생함

→ 잦아지면 성능이 떨어질 수 있음

---

# 5. Process Scheduling

‘ 멀티 프로그래밍 ‘ : CPU를 최대한 사용하기 위해 몇몇 프로세스를 항상 실행

‘ 시간 공유 ‘ : 프로세스 간에 CPU를 빠르게 전환, 각 프로그램 실행되는 동안 사용자가 서로 상호작용할 수 있도록

**👉🏻 이러한 목적을 달성하기 위한 것이 프로세스 스케줄링**

### **Scheduling Queue**

: 운영체제가 여러 프로세스들을 효율적으로 관리하기 위해 사용하는 대기 리스트 

→ 고객들이 목적에 따라 다른 창구 줄에 서 있는 것처럼, 프로세스들도 자신의 상태에 맞는 큐에 들어감

- Job Queue
    - 프로세스가 시스템에 들어오면, Job Queue에 놓이게 되며 Job Queue에는 시스템의 모든 프로세스가 존재함
- Ready Queue
    - 스케쥴링 큐는 메인메모리에 존재
    - Ready 상태에서 실행을 대기하는 Process들이 Ready Queue에 존재
- Device Queue
    - 특정 입출력 장치를 대기하는 프로세스들의 리스트

### Schedulers

: 프로세스에 CPU와 같은 자원을 할당하는 역할 담당

- 장기 스케줄러 (Long-Term Scheduler)
    - Job Queue에서 어떤 프로세스를 메모리로 가져와 Ready Queue에 넣을지 결정
    - 즉, 시스템에 제출된 작업 중 어떤 것을 실행할지 선택한다.
    
    💡 시스템의 다중 프로그래밍 정도를 제어하는 것 !!
    
    메모리에 너무 많은 프로세스가 올라오면 시스템 성능이 저하되기 때문에 이를 적절히 조절해야함
    
- 단기 스케줄러 (Short-Term Scheduler)
    - Ready Queue에 있는 프로세스들 중 하나를 선택하여 CPU를 할당
    
    💡 CPU의 효율성을 극대화한다. 매우 자주 실행되므로, 이 스케줄러의 속도가 곧 시스템의 반응 속도와 직결
    
- 중기 스케줄러 (Medium-Term Scheduler)
    - 메모리에 있는 프로세스 중 일부를 보조 저장 장치로 잠시 내보냈다가, 다시 메모리로 가져오는 역할
    
    💡 시스템에 너무 많은 프로세스가 있어 메모리 공간이 부족할 때, 프로세스를 일시적으로 메모리에서 제거하여 시스템의 혼잡도를 완화
    

# 6. 스레드

: 프로세스내에서 실행되는 흐름의 단위

하나의 프로그램은 하나 이상의 프로세스를 가지고 있고, 하나의 프로세스는 반드시 하나 이상의 스레드를 갖는다.

<img src="https://velog.velcdn.com/cloudflare/aeong98/ce1b2913-bef7-46ff-9444-00dbd22489ef/image.png">

- 스레드는 프로세스 내에서 각각 Stack만 따로 할당, 다른 영역은 공유
- 프로세스는 다른 프로세스의 메모리에 직접 접근할 수 없음

---

⁉️ 이런 방식으로 메모리 공유하는 이유?

✅ 스레드는 흐름 단위 ! 이 말이 무슨 뜻이냐?

운영체제는 이렇게 작은 단위까지 직접 작업하지 않음

운영체제 관점에서는 프로세스가 최소 작업 단위, 스레드는 그 하위의 흐름 단위인 것

즉, 같은 프로세스 소속의 스레드끼리 메모리를 공유 안 할 수가 없는 것 !

---

### **멀티 프로세스 ( = 멀티 태스킹 )**

: 하나의 운영체제 안에서 여러 프로세스가 실행되는 것

**특징**

- 안정성이 좋음
- 여러 개의 자식 프로세스 중 하나에 문제가 발생해도, 다른 자식 프로세스에 영향이 확산되지 않음
- 프로세스 간 통신을 하기 위해서는 IPC를 통해야 한다.
- 메모리 사용량이 많음
- 스케줄링에 따른 Context Switch이 많아지고, 성능 저하의 우려가 있다.

---

### IPC 란 ? ( Inter Process Communication )

: 프로세스 간 정보 공유

각 프로세스들은 별도의 공간에서 실행

한 프로세스에서 다른 프로세스의 메모리영역에 접근 불가능

→ IPC를 이용하면 가능해짐

<img src="https://velog.velcdn.com/cloudflare/aeong98/4f520ab9-6459-4a9b-b18f-293fe42d8487/image.png">
1. Message Passing 모델 : 메시지 전달을 위해 커널을 들림
    
  → user level, kernel level을 계속 넘나들고 system call이 계속 호출 → 오버헤드
    
2. Shared Memory 모델 : 프로세스 사이에 공유 공간이 있음
    
  → system call이 필요하지 않음 → 속도가 빠르지만 공유 공간에 제한이 존재함
    

---

### 멀티 스레드

: 하나의 프로세스가 여러 작업을 여러 스레드를 사용해 동시에 처리하는 것

**특징**

- 장점
    - Context-Switching 할 때 공유하고 있는 메모리만큼 메모리 자원을 아낄 수 있음
    - 스레드는 프로세스 내의 Stack 영역을 제외한 모든 메모리를 공유하기 때문에 통신 부담이 적어서 응답 시간이 빠르다.
- 단점
    - 스레드 하나가 프로세스 내 자원을 망쳐버린다면 모든 프로세스가 종료될 수 있다.
    - 자원을 공유하기 때문에 필연적으로 동기화 문제가 발생할 수 밖에 없다. 교착상태가 발생하지 않도록 주의해야 한다.

# ⁉️ 예상 질문 모음

> **✅ 기술 면접 답변틀**
> 
> 1. 기술의 정의 
> 2. 어디에서 사용하는지
> 3. 어떻게 사용할 수 있는지

<aside>
💡

 **Core란 무엇인가요?**

**Process란 무엇이며 어떤 메모리 구조로 되어있나요?**

**프로세스는 운영체제에 어떻게 저장되나요?**

**프로세스 동기화란 무엇이며 관련된 문제 상황에는 무엇이 있을까요?**

**임계 구역 문제를 해결하기 위한 조건은 무엇인가요?**

**임계 구역 문제를 해결하기 위한 기법에는 무엇이 있나요?**

**Thread란 무엇인지 설명해주세요**

**스택을 스레드마다 독립적으로 할당하는 이유는 무엇일까요?**

**PC Register를 스레드마다 독립적으로 할당하는 이유**

**Multi-Thread Programming은 무엇인가요?**

**멀티 쓰레드의 동시성과 병렬성을 설명해주세요.**

**멀티 쓰레드 환경에서의 주의사항을 설명해주세요.**

**Thread-Safe는 무엇을 의미하나요?**

**Process와 Thread의 차이를 설명해주세요.**

**Multi-Process와 Multi-Thread의 차이를 설명해주세요,**

**Multi-Process, Multi-Thread는 언제 사용할까요?**

**PCB가 무엇인가요??**

**PCB는 메모리에 어떤 형식으로 저장되나요?**

 **Context Switching 컨텍스트 스위칭에 대해 설명해주세요.**

</aside>
