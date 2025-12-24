# 프로세스 개요

## 프로세스

> 실행 중인 프로그램
> 
- **foreground process**
    - 사용자가 보는 앞에서 실행되는 프로세스
- **background process**
    - 사용자가 보지 못하는 뒤편에서 실행되는 프로세스
    - `daemon` - 유닉스 / `service` - 윈도우
        - 사용자와 상호작용 X 정해진 일만 수행

## PCB : Process Control Block

> 모든 프로세스는 실행을 위해 CPU를 필요로 하지만, CPU 자원은 한정되어 있다.
> 
- 그래서 프로세스가 돌아가며, ***한정된 시간 만큼만 CPU을 이용***할 수 있도록, `OS`는 `PCB`(Process Control Block)을 사용한다!
- PCB는 **커널 영역에 생성**되며, OS는 PCB로 **프로세스를 식별** & **처리**하는데 필요한 정보를 판단한다.
- PCB의 생명주기 = 프로세스의 생명주기

<img width="276" height="428" alt="스크린샷 2025-12-24 오전 9 55 55" src="https://github.com/user-attachments/assets/82eb1ed6-290e-4b35-8876-2f0a1022b12c" />


- **Process State** : `new`, `ready`, `running`, `waiting`, `terminated` 상태 중 하나에 해당됨
- **PID** : 프로세스를 식별하는 고유 번호
- **Program Counter** : 메모리의 다음 명령어 주소를 저장함
- **CPU registers** : `IR`(Instruction Register), `DR`(Data Register), `PC`(Program Counter)와 같은 저장공간이 포함됨
- **CPU-scheduling information** : 프로세스 실행 순서를 정하는 정보
- **Memory-management information** : 프로세스가 저장될 메모리에 관한 정보
- **Accounting Information** : 프로세스의 실행, 시간 제한, 실행 ID 등에 사용되는 CPU양의 정보
- **I/O status information** : 어떤 입출력 장치가 할당됨? 어떤 파일을 엶?

## Context Switching

> 하나의 프로세스 수행을 다시 시작하기 위해 기억해야 하는 정보들이다!
> 

`Context`는 마지막에 명령을 수행했던 명령어 위치이고, `Context`는 `PCB`에 표시된다.

<img width="450" height="362" alt="스크린샷 2025-12-24 오전 9 56 07" src="https://github.com/user-attachments/assets/165b651a-c905-4c15-a4cb-31dc228b9b9c" />

- CPU core를 **다른 프로세스에게 양도**
- 현재 **프로세스의 상태를 저장**
- Context Switch 하게 되면 → 다른 Process Context를 복원

**Pros :** 이를 통해 여러 프로세스들이 끊김 없이 빠르게 번갈아 가며 실행된다!

**Cons :** 자주 일어나면 동시에 실행되는 것처럼 보이지만, `overhead`가 발생할 수 있다!

## 프로세스의 메모리 영역

> ***커널 영역에 PCB*** 생성 → ***사용자 영역에 프로세스***들이 배치됨.
> 

사용자 영역은 크게 다음과 같이 구성되어 있다.

<img width="223" height="189" alt="스크린샷 2025-12-24 오전 9 56 34" src="https://github.com/user-attachments/assets/2b6c27cf-d529-453c-a7a7-1e9a6da1257f" />

1. **Text Section (정적 할당 영역!)**

실행 가능한 코드를 저장하는 공간

`read-only` 공간, 데이터가 아닌 CPU가 실행할 명령어가 담겨 있다!

**2. Data Section (정적 할당 영역!)**

전역 변수를 저장하는 공간

**3. Heap Section (동적 할당 영역!)**

프로그램 실행동안 동적으로 할당되는 변수가 저장되는 공간

해당 메모리 공간이 반환해야 한다 

→ 반환하면? 더 이상 메모리 공간이 사용되지 않는 것! 

→ 만약에 반환이 안되면, `memory leak`이라는 문제가 발생한다.

*낮은 주소에서 높은 주소로 할당*된다.

**4. Stack Section (동적 할당 영역!)**

함수가 실행되는 동안 지역변수가 저장되는 임시 공간, 대표적으로 함수 매개변수, 리턴 주소, 지역 변수 등이 포함됨

*높은 주소에서 낮은 주소로 할당*된다.

# 프로세스 상태와 계층 구조

## 프로세스 상태

<img width="582" height="224" alt="스크린샷 2025-12-24 오전 9 57 00" src="https://github.com/user-attachments/assets/d014b1f7-d8e0-47c5-9d4e-f8104400364b" />


> 여러 프로세스들이 빠르게 번갈아 가며 실행되는데, 이 과정에서 하나의 프로세스는 여러 상태를 거치며 실행된다.
> 

프로세스가 가질 수 있는 상태는 다음과 같다. 

- **New** : 프로세스가 생성된 상태
- **Ready** : 프로세스가 프로세서에 의해 실행되기를 기다리는 상태(언제라도 실행 가능) → **CPU를 할당받기 위해 대기 중**
    - 준비상태의 `process`가 시작되면 → `Running`이 된다!
- **Running** : 프로세스가 수행되는 상태
- **Waiting** : 프로세스 이벤트가 발생되어 입/출력 완료를 기다리는 상태
    - 왜 기다리는 상태도 추가가 되냐?
        - → 입출력 작업은 CPU에 비해 속도가 느리므로, 입출력 작업을 요청한 프로세스는
        - → 입출력 장치가 입출력을 끝낼 때까지 기다려야 한다!
- **Terminated** : 프로세스 실행 종료 상태

## 프로세스 계층 구조

프로세스는 실행 도중 시스템 호출을 통해 다른 프로세스를 생성할 수 있다.

1. `parent process` = 새 프로세스를 생성한 프로세스 
2. `child process` = `parent process`에 의해 생성된 프로세스 

**다른 프로세스 = 다른 `PID`를 가진다!** 

(일부 운영체제는 자식 프로세스의 `PCB`에 부모 프로세스의 `PID`인 `PPID`가 기록되기도 한다)

***for example,*** 

컴퓨터를 키고 → 로그인 창을 통해 성공적으로 로그인해서 → bash shell로 → vim이라는 문서 편집기를 실행하면,

`최초의 프로세스 - 로그인 프로세스 - bash 프로세스 - vim 프로세스` 순으로 프로세스가 만들어진다.

> **최초의 프로세스 → 항상 PID가 1번!**
1. unix OS : `init`
2. linux OS : `systemd`
3. macOS : `launchd`
> 

## 프로세스 생성 기법

### `fork()`

> 자신의 복사본을 자식 프로세스로 생성한다.
> 

생성된 **자식 프로세스에는 → 부모 프로세스의 자원들 (메모리 내용, 열린 파일의 목록) 등이 상속**된다. 하지만 다른 프로세스이기 때문에 **`PID` or 저장된 메모리 위치가 다르다!**

### `exec()`

> 새로운 프로그램 내용으로 전환되어 실행하는 시스템 호출이다.
> 

즉, `fork`을 통해 **복사본**이 만들어지면 → 자식 프로세스는 `exec` **시스템 호출을 통해 새로운 프로그램으로 전환**된다.

`exec()`을 호출하면 → 코드 영역(text section)과 데이터 영역의 내용이 실행할 프로그램의 내용으로 바뀌고, 나머지 영역은 초기화된다!

<aside>
💡

**프로세스 실행에 대한 두 가지 가능성**

1. 부모 프로세스와 자식 프로세스가 동시에 실행될 수 있고
    
    ```jsx
    pid = fork();
    
    if (pid == 0) {
        // 자식 프로세스
        exec(...);   // 또는 그냥 계속 실행
    } else {
        // 부모 프로세스
        // wait 안 함 → 부모도 계속 실행
    }
    ```
    
2. 부모 프로세스가 자식 프로세스가 종료될 때까지 기다릴 수 있다.
    
    ```jsx
    pid = fork();
    
    if (pid == 0) {
        exec(...);   // 자식은 다른 프로그램 실행
    } else {
        wait(NULL);  // 부모는 자식 종료까지 block
    }
    ```
    
</aside>

<aside>
💡

**프로세스 주소 공간에 대한 두 가지 가능성**

1. 자식 프로세스가 부모 프로세스와 동일한 프로그램 수행 가능
    1. `fork()` 만 실행!
2. 자식 프로세스가 부모 프로세스와는 다르게 새로운 프로그램 수행 가능 
    
    ```jsx
    pid = fork();
    
    if (pid == 0) {
        exec("/bin/ls", ...);
    }
    ```
    
</aside>

# Thread

> `프로세스란`? → 실행되는 프로그램!
`스레드란`? → 프로세스를 구성하는 실행의 흐름 단위!
**하나의 프로세스는 → 여러 개의 스레드**를 가질 수 있다.
스레드를 이용하면 → **하나의 프로세스에서 → 여러 부분을 동시에 실행**할 수 있다.
> 

## Process & Thread

> 기존에는 → 하나의 프로세스가 → 한번에 하나의 일만 처리했다 ⇒ **단일 스레드 프로세스**

하지만 하나의 프로세스가 → 여러 일을 동시에 처리할 수 있다! (= 여러 명령어를 동시에 실행할 수 있다) ⇒ **멀티 스레드 프로세스!**
> 

즉, 스레드는 프로세스 실행에 필요한 **최소한의 정보 (Program Counter을 포함한 Register, Stack) 만을 유지**한 채 **프로세스 자원을 공유**하며 실행된다!

## MultiProcess & MultiThread

> 여러 프로세스를 동시에 실행 : `MultiProcess`
여러 스레드로 프로세스를 동시에 실행 : `MultiThread`
> 
- 멀티 프로세스는 여러 프로세스를 동시에 실행하며, **프로세스들끼지 독립적**이기 때문에 → **하나의 프로세스에 문제가 생겼다고 해서, 다른 프로세스에 지장이 가지 않는다.**
- 하지만 멀티 스레드는 여러 스레드로 프로세스를 동시에 실행하기 때문에, **하나의 스레드에 문제가 생기면 → 프로세스 전체에 문제가 생길 수 있다!**

# IPC - Inter-Process Communication

> 프로세스들은 독립적으로 실행되지만, 통신을 통해 **상호협력적으로** 실행 가능하다!
> 

프로세스들은 IPC를 통해 데이터를 공유할 수 있고, 한 프로세스에서 다른 프로세스로 데이터를 전송하거나 수신할 수 있다.

## IPC의 주요 모델

<img width="254" height="134" alt="스크린샷 2025-12-24 오전 9 57 31" src="https://github.com/user-attachments/assets/a66ccf6f-0b22-424d-8edb-e245aeb2038b" />

- `shared memory`
    - 메모리 위에 프로세스들간에 데이터를 공유할 수 있는 공유 메모리 영역을 사용하는 방법
- `message passing`
    - `message queue`를 놓고
    - 프로세스들이 전송할 데이터를 `message queue`에 전송
    - 프로세스들이 수신할 데이터를 `message queue`를 통해 받는다.

### Shared Memory

**같은 메모리를 직접 공유**

<img width="623" height="376" alt="image" src="https://github.com/user-attachments/assets/41812f0d-5fcd-4ec6-bfaa-0dde89e9b81e" />

생산자 - 소비자 문제가 발생함!

> Multi-Processing or Multi-threading 환경에서 발생하는 고전적인 동기화 문제 중 하나이다.
> 
> - 다수의 프로세스가 → 공유 자원에 접근할 때 발생.
- Role
    - 생산자 : 데이터 생성 & 공유 자원에 추가
    - 소비자 : 공유 자원에 추가된 데이터를 꺼내서 사용
    - 버퍼 : 공유 자원이고, 생산자가 만든 데이터를 임시로 보관.
- 수행 과정
    - 생산자 : 데이터를 생산해서 → 버퍼에 넣는다.
    - 소비자 : 데이터를 꺼내서 → 사용한다.
    - 생산자는 버퍼가 가득차면 → 대기한다. = `overflow`
    - 소비자는 버퍼가 비어있으면 → 대기한다. = `underflow`

***속도는 빠르지만, 동기화 문제에 대한 해결이 필요! → ex) mutex, semaphore, monitor***

### Message Passing

**메세지를 주고받음**

<img width="1701" height="1618" alt="image" src="https://github.com/user-attachments/assets/5ba45602-95ca-41c1-94c4-9a2f4ed18739" />

- 개념
    - 프로세스간에 데이터 전송 및 읽기와 같은 통신을 하기 위한 방법
- 방법
    - 메시지 큐
    - 파이프
    - 소켓
    - RPC (Remote Procedure Call)
- API
    - `send(message, destination) or send(message)`
    - `receive(message, host) or receive(message)`

***속도는 상대적으로 느리지만, 동기화를 OS가 보장해준다! ex) message queue, pipe, socket, RPC***

### Pipes

**단방향 또는 양방향 채널을 생성해 2개 이상의 프로세스들이 서로 통신할 수 있도록 하는 IPC 기술의 한 종류.**

<img width="867" height="801" alt="image" src="https://github.com/user-attachments/assets/4ddc45e5-7366-4121-a569-41030a6dac85" />

- `System Call`을 사용하여 구현이 가능.
- 장점
    - 단순하고 효율적 (데이터를 전송할 때 최소한의 오버헤드)
    - 신뢰 (에러 탐색 가능, 데이터 전달 보장 가능)
    - 유연 (단방향 가능, 양방향 가능)
- 단점
    - 용량이 제한됨 → 한 번에 전송할 수 있는 데이터의 개수가 제한됨.
    - 단방향성 → **단방향 파이프에서는 하나의 프로세스만이 데이터를 전송**할 수 있다.
    - 동기화 → **양방향에서는 데이터 전송 순서가 동기화**되어야 한다.
    - 제한된 확장성 → 파이프는 **동일 컴퓨터 내의 소수 프로세스 간의 통신으로 제한**됨 → **분산시스템에서는 단점!**
    - 같은 컴퓨터에서 간단 & 효율 통신에서는 적합, 하지만 대규모 분산 시스템이나 양방향 통신에서는 적합하지 않는다.
- 구현 시 고려할 점
    - 단방향 or 양방향?
    - 양방향이면 → 반이중? or 전이중?
    - 통신 프로세스 사이에서 관계가 존재해야 하는가?
    - 파이프가 네트워크를 통해 통신하는가?
- 종류
    - **익명 파이프 (unamed pipe)**
    <img width="512" height="184" alt="스크린샷 2025-12-24 오전 10 00 26" src="https://github.com/user-attachments/assets/cc72da62-7ff3-40cd-a5b6-c26729aec04b" />
    
    - only 단방향으로, 부모 → 자식 프로세스 간 통신한다.
    - FIFO 방식으로 임시 공간인 파이프를 기반으로 데이터를 주고 받고, 단방향 방식의 읽기 전용, 쓰기 전용 파이프를 만들어서 작동한다. 
    - **명명 파이프 (named pipe)**
        - 파이프 서버와 하나 이상의 파이프 클라이언트 간의 통신을 위한 명명된 단방향 또는 양방향 파이프를 말한다.
