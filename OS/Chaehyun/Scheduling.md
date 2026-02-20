## 1. Basic Concepts

- CPU 스케쥴링은
    - 멀티프로그래밍의 기초
        - 동시성의 목적: CPU가 굉장히 빨라서 놀고 있음 → 컨텍스트 스위치를 해서 나눠서 시간을 분할해서 CPU 자원을 끼워넣어도 아무 문제 없음 → CPU 활용도를 높임
- CPU burst: CPU를 실제로 사용하는 시간 (running)
- I/O burst: I/O 요청 후 대기하는 시간 (waiting)
- I/O bound가 CPU bound보다 훨씬 개수가 많음
    
    ⇒ CPU 스케쥴링을 하면 더 효율이 올라감
- I/O-bound 프로세스가 많을수록 CPU는 자주 idle 상태가 됨

   → 짧은 CPU burst를 빠르게 섞어주는 스케줄링이 중요    
    <img width="1060" height="636" alt="image" src="https://github.com/user-attachments/assets/61ce4812-cb49-478f-9889-d68e9965ce0a" />

    
- CPU 스케쥴러
    - ready 상태에 있는 프로세스들 중 cpu를 할당해줄 프로세스를 선택
- Preemptive vs Non-preemtive
    - 선점형 vs 비선점형
    - 선점형: 강제로 쫓아낼 수 있음
        - 스케쥴러가 프로세스를 쫓아냄
    - 비선점형: 쫓아낼 수 없음
        - 프로세스가 자발적으로 나오기 전까지는 할당 ㄴㄴ
- dispatcher
    - CPU’s core의 control을 넘겨줌
    - context switch를 해주는 모듈
    - user mode 변경

## 2. Scheduling Criteria

- CPU utilization
- 처리량 throughput
    - 단위 시간당 완료된 프로세스 수
- turnaround time
    - 제출하고 종료되는 시간까지의 시간
- **waiting time**
    - ready 큐에서 대기하면서 보내는 시간
- response time
    - 제출 → 첫 CPU 할당까지의 시간

## 3. Scheduling Algorithms

- FCFS
- SJF
    - SRTF (Shrotest Remaining Time First)
- RR ⇒ 현대에서는 다 씀
- Priority-based
- MLQ

### FCFS

- CPU를 먼저 요청한 프로세스 할당
- FIFO 큐
- 평균 대기 시간은 일반적으로 최소가 아니며, 프로세스들의 cpu burst time(cpu 사용 시간) 편차가 클 경우 대기 시간이 상당히 달라질 수 있음
    
    ⇒ convoy effect
    
- non-preemptive

### SJF

- Shortest-Job-First: shortest-next-CPU-burst-first scheduling
- 긴 작업 앞에 짧은 작업을 먼저 처리하도록 순서를 바꾸면
    
    짧은 작업이 보는 이득(대기 시간 감소)가 긴작업이 보는 손해(대기 시간 증가)보다 훨씬 큼
    
    ⇒ 전체적인 대기 시간이 줄어들음
    
- 실제로 구현할 수 있는가
    - 미리 알 방법이 없음
    - 과거의 기록을 보고 근사치를 예측
- next CPU burst를 예측하는 방법
    - 지수 평활법 τₙ₊₁ = α · tₙ + (1 − α) · τₙ
        - 다음 예측 값 = (실제 수행 시간) * (가중치) + (1-가중치) * 이전 예측값
        - 가중치가 1인 경우
            - 최근 데이터에 민감하게 반응하지만 변동 심함
        - 가중치가 0인 경우
            - 변화에 둔감하고 보수적
        - 가중치가 0.5인 경우
            - 최근 경향, 과거도 반영하는 중도
        - “지수”평활법인 이유: 옛날 기록일수록 영향력이 지수적으로 줄어들어서 0에 가까워짐
- preemptive / non-preemptive
    - 실행 중인 프로세스가 있을 때 더 짧은 작업이 들어오면
        - 바꾼다 → Preemptive SJF (SRTF)
        - 안바꾼다 → non-preemptive SJF

### SRTF

- preemptive SJF

### RR

- time-sharing-system을 위해 설계
- preemptive FCFS
- time quantum
- RR의 평균 대기 시간은 종종 길다
    - 공정성을 최우선으로 하기 떄문에 짧은 작업도 여러번 끊겨서 기다리게됨
    - 모든프로세스에게 시간을 조금씩 나눠줌
        - 짧은 작업도 긴 작업 뒤에 섞여서 여러번 큐를 돌며 기다리게 됨
- 타이머 인터럽트 발생 → 현재 프로세스 강제 중단 → ready queue 맨 뒤로 이동
- ready queue는 circular queue로 동작하고, CPU 스케줄러는 ready queue를 돌면서 한 번에 한 프로세스에게 한 번의 time slice동안 cpu를 할당
- 보통 time slice의 크기가 context switch 시간에 비해 더 커야 함
    
    but, time slice의 크기가 너무 크게되면 fcfs 방식이 되므로, CPUburst의 80%는 시간 할당량보다 짧아야함
    
- time quantum에 따라 RR의 성능이 크게 달라짐
<img width="970" height="788" alt="image" src="https://github.com/user-attachments/assets/4ce9f485-1cbb-4448-bef5-c67d62258401" />



### Priority-base Scheduling

- CPU는 가장 높은 우선순위를 가진 프로세스에게 할당
- 우선순위가 같은 프로세스들은 FCFS 순서로 스케줄
- 우선순위는 내부적인 요인과 외부적인 요인으로 결정
    - 내부: 시간제한, 메모리 요구 비율
    - 외부: 프로세스의 중요성, 비용
- 선점형, 비선점형
- 문제점: indefinite blocking, starvation
    - 낮은 우선순위 프로세스들이 CPU를 무한히 대기 가능
        
        ⇒ 해결방법: aging: 오랫동안 시스템에서 대기하는 프로세스들의 우선순위를 점진적으로 높임
- Priority inversion이란?
    - 낮은 priority 프로세스가 자원을 점유
    - 높은 priority 프로세스가 그 자원을 기다림
    - 중간 priority가 계속 CPU 차지
  -> Priority inheritance로 해결      

### Multilevel Queue Scheduling

- ready queue를 다수의 별도의 queue로 분류
- 각 queue들은 자신의 스케줄링 알고리즘을 가질 수 있음
    - foreground는 RR로 background는 fcfs에 의해 스케줄 될 수 있음

## Multilevel Feedback Queue Scheduling

- 다단계 큐 스케줄링 알고리즘은 일반적으로 프로세스들이 시스템 진입시에 영구적으로 하나의 큐에 할당
- 다단계 피드백 큐 스케줄링 알고리즘은 프로세스가 큐들 사이를 이동하는것을 허용함
