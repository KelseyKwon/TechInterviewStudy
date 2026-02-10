OS가 프로세스에게 공정하고 합리적으로 CPU 자원을 배분하는 것.

## 프로세스 우선순위

프로세스마다 우선순위가 다르다. ex) I/O 프로세스는 가장 우선순위가 높다. 

### 프로세스의 종류

1. **I/O bound process**
    1. `입출력 작업`이 많은 프로세스 (비디오 재생, 디스크 백업 작업)
    2. 실행 상태보다 입출력을 위한 대기 상태에 더 많이 머무른다. 
2. **CPU bound process**
    1. `CPU 작업`이 많은 프로세스 (수학 연산, 컴파일, 그래픽 처리)
    2. 대기 상태보다 실행 상태에 더 많이 머무른다. 

<aside>


**CPU 버스트** : `CPU`를 이용하는 작업

**I/O 버스트** : `입출력 장치`를 기다리는 작업. 

⇒ OS는 **CPU 버스트와 입출력 버스트를 반복**하며 실행된다. 

</aside>

`CPU 집중 프로세스`와 `입출력 집중 프로세스`가 동시에 CPU 자원을 요구하면 → 입출력 프로세스를 빨리 실행 시켜서 입출력 장치를 끊임없이 작동하는 것이 좋다!

⇒ 이렇게 모든 프로세스가 CPU를 차례대로 사용하는 것보다, 각각의 상황에 맞게 CPU를 분배하는 것이 좋다!

⇒ 이를 위해 OS는 프로세스의 PCB마다 우선순위를 부여한다

## Scheduling Queue

> OS가 매번 모든 PCB를 검사해서 먼저 자원을 이용할 프로세스를 결정하는 것은 비효율적이다.
> 

왜냐? 

1. CPU를 원하는 프로세스는 한 두개가 아니며, 언제든 새로운 프로세스가 생길 수 있다.
2. 메모리에 적재되고 싶어하는 프로세스, 특정 입출력장치와 보조기억장치를 사용하길 원하는 프로세스도 여러 개가 있을 수 있다.

⇒ 따라서 OS는 ***CPU를 사용하고 싶은 프로세스 → 메모리에 적재되고 싶은 프로세스 → 특정 입출력장치 사용하고 싶은 프로세스*** 순으로 줄 세움

⇒ 이것을 `Scheduling Queue`로 구현하고 관리한다!

### Queue의 종류
<img width="543" height="358" alt="스크린샷 2026-01-07 오후 1 02 08" src="https://github.com/user-attachments/assets/6a53a874-9d93-4de1-af4c-c3e2f15af6cc" />

1. **Ready queue → CPU를 이용하기 위해 기다리는 줄**
2. **Waiting Queue → 입출력장치 이용하기 위해 기다리는 줄**

## Preemptive / Non-Preemptive
<img width="556" height="279" alt="스크린샷 2026-01-07 오후 1 02 43" src="https://github.com/user-attachments/assets/0654214e-13cb-49b5-bb47-da57470bb28e" />

### Preemptive Scheduling

> 하나의 프로세스가 다른 프로세스가 사용 중인 CPU의 자원을 요청할 때, OS로부터 자원을 강제로 뺴앗아 다른 프로세스에 할당할 수 있는 스케줄링 방식
> 
- 하나의 프로세스가 자원 독점 X
    - **Pros :** 한 프로세스의 자원 독점 막고 프로세스들에 자원 골고루 배분하지만,
    - **Cons :** Context Switching 과정에서 Overhead가 발생할 수 있다.

### Non-Preemptive Scheduling

> 하나의 프로세스가 다른 프로세스가 사용 중인 CPU의 자원을 요청할 때, 그 프로세스가 종료(terminated) or 대기(waiting)에 접어들기 전까지 다른 프로세스가 끼어들 수 없는 스케줄링 방식
> 
- 하나의 프로세스가 자원 독점 O
    - **Pros :** Context Switching 과정에서 Overhead가 적지만,
    - **Cons :** 모든 프로세스들이 골고루 자원을 사용할 수 없다.

# CPU Scheduling Algorithm

## FCFS(First Come First Served Scheduling)

> **Non-Preemptive :** Ready Queue에 삽입된 순서대로 프로세스들을 처리
> 

즉, CPU가 먼저 요청한 프로세스부터 CPU를 할당한다. 

- **Pros : 가장 공정해보이지만,**
- **Cons : 프로세스들이 기다리는 시간이 매우 길어질 수 있다.**

`convoy effect` : CPU 버스트가 높은 프로세스가 `Ready Queue` 앞에 오면 그 뒤에 CPU 버스트가 더 낮은 프로세스가 기다려야 한다. 

ex) A (17ms) → B (5ms) → C (2ms) → C는 2ms의 실행 시간때문에 22ms의 대기 시간을 거쳐야 한다. 

## SJF(Shortest Job First Scheduling)

> **Non-Preemptive or Preemptive :** convoy effect를 방지하기 위해 나타남 → ***최단 시간 프로세스***부터 처리!
> 
- Preemptive로 구현한 것이 SRT!

## RRB(Round Robin Scheduling)

> FCFS + time slice (=각 프로세스가 CPU를 사용할 수 있는 정해진 시간)
> 
- `time slice`가 많으면 : FCFS와 비슷해짐 → `convoy effect`가 생길 수 있다!
- `time slice`가 적으면 : `Context Switching`에 발생하는 비용이 크다 → CPU에 처리되는 일보다 프로세스를 전환하는데 더 많은 비용이 든다!

## SRT(Shortest Remaining Time)

> `SJF` + `RRB`
> 

⇒ 즉, 프로세스들은 정해진 `time slice`만큼 `CPU`를 사용하지만, `CPU`를 사용할 다음 프로세스로는 ***남아있는 작업 시간이 가장 적은*** 프로세스가 선택된다. 

## Priority Scheduling

> 프로세스마다 우선순위를 부여하여 → 가장 높은 우선순위의 프로세스부터!
> 
- `SJF` → 작업 시간이 짧은 프로세스에 높은 우선순위를 부여
- `SRT` → 남은 시간이 짧은 프로세스에 높은 우선순위 부여

**Prob :** `starvation` 현상 : 우선순위가 낮은 프로세스가 우선순위가 높은 프로세스들에 의해 계속 연기되는 현상

**Sol :** `aging` → 오랫동안 기다린 프로세스의 우선순위를 높이기!

## Multilevel Queue Scheduling
<img width="534" height="362" alt="스크린샷 2026-01-07 오후 1 03 02" src="https://github.com/user-attachments/assets/d5ea6157-c097-49fa-9a07-c1ca6e6f3390" />

> **Preemptive :** 우선순위 별로 ready queue를 여러 개 사용
> 

즉, `queue`에 우선순위를 매겨서, 가장 높은 우선순위 큐에 있는 프로세스 먼저 처리한다.

- **Pros :**  프로세스 유형별로 우선순위 구분하여 실행 가능, 큐별로 타임 슬라이스 여러 개 지정하여 다른 스케줄링 알고리즘 사용
- **Cons :** 프로세스들이 큐 사이 이동을 못한다 → 이것을 해결한 것이 `Multilevel Feedback Queue Scheduling`!

아래 2개로 분할한다.

1. `foreground queue` → cpu burst가 짧은 queue!
    1. `Round Robin`
2. `background queue` → batch 등 긴 시간을 필요로 하는 작업!
    1. `FCFS`

## Multilevel Feedback Queue Scheduling

> Multilevel Queue의 기아 현상을 해결했다!
> 

어떻게? queue사이에 프로세스를 이동함으로써!

- aging 기법을 활용해서, CPU 집중 프로세스 (priority queue에서 오래 기다리는 프로세스)를 점점 아래로 내리고, 새로 준비 상태가 된 프로세스를 위로 올린다.
