---
title:  "[운영체제] CPU Scheduling"
excerpt: "CPU Burst와 I/O Burst, CPU-burst Time의 분포, CPU-bound/IO-bound 프로세스, CPU Scheduler&Dispatcher, Scheduling의 성능 척도와 알고리즘, Multiple-processor Scheduling/Real-time/Thread Scheduling, Scheduling 평가"

categories:
  - OS
tags:
  - [OS]
  
toc: true
toc_sticky: true
 
date: 2023-01-26T12:50:47-04:00
last_modified_at: 2023-01-26T12:50:47-04:00
---

# CPU Scheduling

> 이 글은 이화여자대학교 반효경 교수님의 '운영체제' 강의(2014년 1학기)를 학습한 내용을 정리한 것입니다.

## CPU and I/O Bursts in Program Execution

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205111749162.png" alt="image-20230205111749162" style="zoom:67%;" />

- 프로그램의 path는 CPU만 연속적으로 쓰는 단계와 I/O를 하는 단계가 번갈아가며 실행된다.
  - CPU만 연속적으로 쓰면서 인스트럭션을 실행하는 단계를 **CPU Burst**, I/O를 실행하는 단계를 **I/O Burst**라고 한다.
  - 프로그램의 종류에 따라 CPU / I/O burst의 길이와 빈도가 다르다.
    - 사용자와 interaction을 많이 하는 프로그램일수록 I/O burst가 많이 끼어든다(작업 결과 출력 -> 사람이 반응).
    - 오래 걸리는 연산을 하는 프로그램은 CPU burst가 오래 지속된다.



## CPU-burst Time의 분포

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205004046035.png" alt="image-20230205004046035" style="zoom:67%;" />

- 여러 종류의 job(=process)이 섞여 있기 때문에 CPU 스케줄링이 필요하다.

  - Interactive job에게 적절한 response 제공 요망

    -> CPU bound job이 CPU를 오래 붙잡고 있으면 I/O bound job은 너무 오래 기다려야 하므로, CPU scheduling을 통해 I/O bound job에 CPU를 우선적으로 주는 것이 필요하다

  - CPU와 I/O 장치 등 시스템 자원을 골고루 효율적으로 사용



## 프로세스의 특성 분류

- 프로세스는 그 특성에 따라 다음의 두 가지로 나뉜다.
  - **I/O-bound process**
    - CPU를 잡고 계산하는 시간보다 I/O에 많은 시간이 필요한 job
    - I/O가 자주 끼어들어 CPU를 짧게 자주 쓴다.
    - many short CPU bursts
  - **CPU-bound process**
    - 계산 위주의 job
    - CPU를 오랫동안 지속해서 쓴다.
    - few very long CPU bursts



## CPU Scheduler & Dispatcher

- CPU Scheduler와 Dispatcher는 하드웨어도 소프트웨어도 아닌, 운영체제 안에서 CPU Scheduling / Dispatch를 하는 코드

- **CPU Scheduler**

  - Ready 상태의 프로세스 중에서 이번에 CPU를 줄 프로세스를 고른다.

- **Dispatcher**

  - CPU의 제어권을 CPU scheduler에 의해 선택된 프로세스에게 넘긴다.
  - 이 과정을 context switch(문맥 교환)라고 한다.

- CPU 스케줄링이 필요한 경우는 프로세스에게 다음과 같은 상태 변화가 있는 경우이다.

  1. Running -> Blocked (예: I/O 요청하는 시스템 콜)
  2. Running -> Ready (예: 할당시간 만료로 timer interrupt)
  3. Blocked -> Ready (예: I/O 완료 후 interrupt)
     * I/O 가 끝난 작업이 우선순위가 더 높은 경우
  4. Terminate

  * 1, 4에서의 스케줄링은 **non-preemptive ( = 비선점형, 강제로 빼앗지 않고 자진 반납)**
  * 다른 스케줄링은 전부 **preemptive( = 선점형, 강제로 빼앗음)** 



## Scheduling Criteria(= Performance Index, 성능 척도)

- **CPU utilization (이용률)**
  - keep the **CPU as busy as possible**
- **Throughput(처리량)**
  - **number of processes** that **complete** their execution per time unit
- **Turnaround time(소요시간, 평균 시간)**
  - total amount of time spent by a process in a system
  - When present in the system, a process is either waiting in the ready queue for getting the CPU or it is executing on the CPU
  - Turnaround Time = Burst time + Waiting time
  - 프로세스를 실행하는 총 시간이 아닌, CPU를 쓰고 I/O하러 나가기 까지 걸린 시간
- **Waiting time(대기 시간)**
  - amount of time a process has been **waiting in the ready queue**
  - Waiting time = Turnaround time - Burst time
- **Response time(응답 시간)**
  - amount of time after which a process **gets the CPU for the first time after entering the ready queue**
- 위 2가지는 시스템 입장, 아래 3가지는 프로그램 입장에서의 성능 척도



## Scheduling Algorithms

### FCFS (First-Come First-Served)

- 먼저 온 순서대로 처리
- 효율적이지 않음
- Non-preemptive : 앞의 프로세스가 끝나야 뒤의 프로세스가 진행
- ex) 아래 프로세스들의 Average Waiting Time을 구해보자.
  - 도착 순서가 P1 -> P2 -> P3인 경우
    - (0 + 24 + 27) / 3 = 17
  - 도착 순서가 P2 -> P3 -> P1인 경우
    - (6 + 0 + 3) / 3 = 3

| Process | Burst time |
| ------- | ---------- |
| P1      | 24         |
| P2      | 3          |
| P3      | 3          |

- **Convoy Effect (호위 효과)** : 왕 뒤에 시중들이 따라다님. CPU 사용시간이 긴 프로세스에 의해 사용시간이 짧은 프로세스들이 오래 기다리는 현상



### SJF (Shortest-Job-First)

- 각 프로세스의 다음번 CPU burst time을 가지고 스케줄링에 활용
- CPU burst time이 가장 짧은 프로세스를 제일 먼저 스케줄
- Two schemes:
  - **Non-preemptive**
    - 일단 CPU를 잡으면 이번 CPU burst가 완료될 때까지 CPU를 선점(preemption) 당하지 않음
  - **Preemptive**
    - 현재 수행중인 프로세스의 남은 burst time보다 더 짧은 CPU burst time을 가지는 새로운 프로세스가 도착하면 CPU를 빼앗김
    - 이 방법을 **Shortest-Remaining-Time-First(SRTF)**이라고도 부름
- SJF is optimal (**preemptive**인 경우)
  - 주어진 프로세스들에 대해 **minimum average waiting time**을 보장
- ex) 아래 프로세스들의 Average Waiting Time을 구해보자.
  - SJF (non-preemptive)
    - 처리 순서 : P1 -> P3 -> P2 -> P4
    - AWT: (0 + 6 + 3 + 7) / 4 = 4
  - SJF (preemptive)
    - 처리 순서 : P1 -> P2 -> P3 -> P2 -> P4 -> P1
    - AWT : (9 + 1 + 0 + 2) / 4 = 3

| Process | Arrival  Time | Burst Time |
| ------- | ------------- | ---------- |
| P1      | 0.0           | 7          |
| P2      | 2.0           | 4          |
| P3      | 4.0           | 1          |
| P4      | 5.0           | 4          |

- 크게 두 가지 문제점이 있음
  - **Starvation (기아 현상)**
    - CPU 사용시간이 긴 (우선순위가 낮은) 프로세스는 계속 실행되지 않을 수 있다.
  - CPU 사용시간을 미리 알 수 없다.
    - 다음번 CPU burst time을 알아내기 위해, 과거 CPU 사용 기록을 통해 **추정**을 해야 함
    - 추정 기법 : **exponential averaging**
      - t n : n번째 CPU burst의 실제 길이
      - τ n+1 : 다음 CPU burst의 예측치
      - α : 0 <= α <= 1
      - Define : τ n+1 = α * t n + (1 - α) * τ n



### Priority Scheduling (우선순위 스케줄링)

- Priority number: 정수, 작을수록 priority가 높다.
- **highest priority**를 가진 프로세스에게 CPU 할당
  - Preemptive OR Non-preemptive
- SJF는 일종의 priority scheduling이다.
  - SJF의 priority : predicted next CPU burst time
- 문제점
  - Starvation(기아 현상): low priority processes may never execute
- 해결책
  - Aging(노화): as time progresses increase the priority of the process



### Round Robin (RR)

- 각 프로세스는 동일한 크기의 **할당 시간(time quantum)**을 가짐

  (일반적으로 10 - 100 ms)

- 할당 시간이 지나면 프로세스는 선점(preempted)당하고 ready queue의 제일 뒤에 가서 다시 줄을 선다

- n 개의 프로세스가 ready queue에 있고 할당 시간이 q time unit인 경우, 각 프로세스는 최대 q time unit 단위로 CPU 시간의 1/n을 얻는다.

  - **어떤 프로세스도 (n - 1) * q time unit (자신을 제외한 프로세스 전체의 실행 시간)이상 기다리지 않는다.**

- CPU를 짧게 줬다 뺏기 때문에 응답시간이 짧다.

- CPU를 사용하는 시간과 대기 시간이 비례한다.

  - CPU를 길게 사용할수록 대기를 여러번 해야 하므로

- Performance

  - q가 지나치게 크면 => FCFS와 동일
  - q가 지나치게 작으면 => context switch 오버헤드가 커짐

- RR은 일반적으로 SJF보다 average turnaround time이나 average waiting time은 길지만 **response time은 더 짧다**.

- ex) 아래 프로세스들의 Average Turnaround Time을 구해보자 

  - Time Quantum = 1 인 경우
    - ATT : (15 + 9 + 3 + 17) / 4 = 11
  - Time Quantum = 5 인 경우
    - ATT :  (15 + 8 + 9 + 17) / 4 = 12.25

| Process | Burst time |
| ------- | ---------- |
| P1      | 6          |
| P2      | 3          |
| P3      | 1          |
| P4      | 7          |



### Multilevel Queue (MLQ)

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205150538685.png" alt="image-20230205150538685" style="zoom:67%;" />

- Ready queue를 우선순위에 따라 여러 개로 분할

  - **foreground (interactive)**
  - **background (batch - no human interaction)**

- 각 큐는 독립적인 스케줄링 알고리즘을 가짐

  - foreground - **RR**
    - 상호작용을 고려해 응답시간이 짧은 스케줄링 알고리즘 선택
  - background - **FCFS**
    - 응답시간보다는 context switch 오버헤드를 줄이는 알고리즘 선택

- 큐에 대한 스케줄링이 필요

  - **Fixed priority scheduling**

    - 우선순위가 높은 프로세스가 없으면 낮은 프로세스 실행
    - possibility of starvation

  - **Time slice**

    - 각 큐에 CPU time을 적절한 비율로 할당

      ex) 80% to foreground in RR, 20% to background in FCFS

- 우선순위의 차이를 극복할 수 없음



### Multilevel Feedback Queue (MLFQ)

- 프로세스가 다른 큐로 이동 가능

- 에이징(aging)을 이와 같은 방식으로 구현할 수 있다.

- Multilevel Feedback Queue  

- MLFQ의 예시

  <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205151617319.png" alt="image-20230205151617319" style="zoom:67%;" />

  - Three queues
    - Q0 (tq = 8 ms), Q1 (tq = 16 ms), Q2 (FCFS)
  - Scheduling
    - new job이 Q0로 들어감
    - CPU를 잡아서 8 ms만큼 수행되고, 끝내지 못하면 Q1으로 내려감
    - Q1에서 기다리다가 CPU를 잡아 16 ms만큼 수행되고 끝내지 못한 경우 Q2로 내려감



## Multiple-Processor Scheduling

- CPU가 여러 개인 경우 스케줄링은 더욱 복잡해짐
- Homogeneous processor인 경우
  - Queue에 한 줄로 세워서 각 프로세서가 알아서 꺼내가게 할 수 있다.
  - 반드시 특정 프로세서에서 수행되어야 하는 프로세스가 있는 경우에는 문제가 더 복잡해진다.
- Load sharing
  - 일부 프로세서에 job이 몰리지 않도록 부하를 적절히 공유하는 매커니즘 필요
  - 별개의 큐를 두는 방법 vs 공동 큐를 사용하는 방법
- Symmetric Multiprocessing (SMP)
  - 각 프로세서가 각자 알아서 스케줄링 결정
- Asymmetric Multiprocessing
  - 하나의 프로세서가 시스템 데이터의 접근과 공유를 책임지고 나머지 프로세서는 거기에 따름

## Real-time Scheduling

- Hard real-time systems
  - Hard real-time task는 데드라인을 반드시 지키도록 스케줄링해야 함
- Soft real-time computing
  - Soft real-time task는 데드라인을 보장하진 못하지만 일반 프로세스에 비해 높은 priority를 갖도록 해야 함

## Thread Scheduling

- **Local Scheduling**
  - User level thread의 경우 OS가 thread의 존재를 모르기 때문에, 사용자 수준의 thread library에 의해 어떤 thread를 스케줄할지 결정
- **Global Scheduling**
  - Kernel level thread의 경우 일반 프로세스와 마찬가지로 커널의 단기 스케줄러가 어떤 thread를 스케줄할지 결정



## Algorithm Evaluation

- Queueing models

  <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205154536376.png" alt="image-20230205154536376" style="zoom:50%;" />

  - **확률** 분포로 주어지는 **arrival rate**와 **service rate** 등을 통해 각종 **performance index** 값을 계산
  - 이론적으로는 많이 사용되나 실질적으로는 실제 시스템에서 측정하는 방식을 더 의미있게 본다.

- Implementation (구현) & Measurement (성능 측정)

  - **실제 시스템**에 알고리즘을 **구현**하여 실제 작업(**workload**)에 대해서 성능을 **측정** 비교

- Simulation (모의 실험)

  - 알고리즘을 **모의 프로그램**으로 작성 후 trace를 입력으로 하여 결과 비교

    - trace는 input data를 의미. 임의로 만들 수도 있고 실제 프로그램을 돌려서 추출할 수도 있음.

    ex) Average Waiting Time을 계산하는 모의 프로그램으로 SJF와 새로운 알고리즘의 결과값을 비교

## 참고

-Turnaround time, Waiting time, Response time

https://www.gatevidyalay.com/turn-around-time-response-time-waiting-time/