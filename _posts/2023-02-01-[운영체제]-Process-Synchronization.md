---
title:  "[운영체제] Process Synchronization"
excerpt: "데이터 접근과 OS에서의 Race Condition, Critical Section Problem과 여러 해결법, Test_and_Set, Semaphore, Monitor, Synchronization의 고전적인 문제들 (Bounded-Buffer Problem, Readers-Writers Problem, Dining-Philosophers Problem)"

categories:
  - OS
tags:
  - [OS]
  
toc: true
toc_sticky: true
 
date: 2023-02-01T15:10:21-04:00
last_modified_at: 2023-02-01T15:10:21-04:00
---

# Process Synchronization

## 데이터의 접근과 Process Synchronization 문제

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205161414751.png" alt="image-20230205161414751" style="zoom:50%;" />

- 컴퓨터의 연산은 기본적으로 어딘가에 저장되어 있는 데이터를 읽어와 연산을 거친 후 결과값을 저장하는 방식으로 이루어진다.
  - 데이터가 저장되는 곳을 S-box, 연산이 이루어지는 곳을 E-box라고 이름 붙인다면, 각각에 해당하는 요소의 예시는 위 그림과 같다.

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205162024251.png" alt="image-20230205162024251" style="zoom:50%;" />

- **S-box**를 공유하는 **E-box**가 여럿 있는 경우, **Race Condition**의 가능성이 있다.

  **Memory**				 **CPU**		->	Multiprocessor system

  **Address Space**	 **Process**  ->	공유메모리를 사용하는 프로세스들

  ​													   커널 내부 데이터를 접근하는 루틴들 간

  ​														(ex_ 커널모드 수행 중 인터럽트로 커널모드 다른 루틴 수행시)

  - **Race condition(경쟁 상태)**

    - 여러 프로세스들이 공유 데이터(shared data)에 동시 접근(concurrent access)하는 상황에서 어떤 순서로 접근하느냐에 따라 결과값이 달라질 수 잇는 상황을 의미한다.

    - 데이터의 최종 연산 결과는 마지막에 그 데이터를 다룬 프로세스에 따라 달라진다.

      - 동시 접근은 데이터의 **불일치 문제(inconsistency)** 발생시킬 수 있다.

        -> race condition을 막기 위해서는 협력 프로세스(cooperating process) 간의 실행 순서를 정해주는 **동기화(synchronize)**가 이루어져야 한다.

      <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205164908171.png" alt="image-20230205164908171" style="zoom:50%;" />



## OS에서 race condition이 발생하는 경우

### 1. kernel 수행 중 인터럽트 발생 시 (interrupt handler VS kernel)

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205162437397.png" alt="image-20230205162437397" style="zoom:67%;" />

- 문제: 커널모드 running 중 interrupt가 발생하여 interrupt 처리루틴이 수행

  -> 양쪽 다 커널 코드이므로 kernel address space 공유

- 해결책: kernel이 수행중일 때는 인터럽트를 disable시켜, 수행이 끝난 후 interrupt 처리루틴이 수행되도록 함

### 2. Process가 system call을 하여 kernel mode로 수행 중인데 context switch가 일어나는 경우

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205162953402.png" alt="image-20230205162953402" style="zoom:67%;" />

- 해결책: 커널 모드에서 수행중일 때는 CPU를 preempt하지 않고 사용자 모드로 돌아갈 때 preempt

### 3. Multiprocessor에서 shared memory 내의 kernel data

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205163256057.png" alt="image-20230205163256057" style="zoom:67%;" />

- multiprocessor의 경우 interrupt enable/disable로 해결되지 않음
- 해결책
  - 1: 한번에 하나의 CPU만이 커널에 들어갈 수 있게 하는 방법 -> 비효율적
  - 2: 커널 내부에 있는 각 공유 데이터에 접근할 때마다 그 데이터에 대한 lock/unlock을 하는 방법



## The Critical Section Problem

- n 개의 프로세스가 공유 데이터를 동시에 사용하기를 원하는 경우

- 각 프로세스의 code segment에는 공유 데이터를 접근하는 코드인 **임계영역(critical section)**이 존재

- 문제점

  - 여러 프로세스가 동일 자원을 동시에 참조하여 값(공유하는 변수명, 파일 등)이 오염될 가능성이 있다.

  - 하나의 프로세스가 critical section에 있을 때 다른 모든 프로세스는 critical section에 들어갈 수 없어야 한다.

    <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205165227606.png" alt="image-20230205165227606" style="zoom:50%;" /> 



## Solution

- 한 프로세스가 critical section을 수행중일 때 다른 프로세스는 critical section을 수행하지 못하게 막는 여러 소프트웨어적 or 하드웨어적인 방법들

### 1. Initial Attempts to Solve Problem

- 두 개의 프로세스가 있다고 가정 P0, P1

- 모든 프로세스의 수행 속도는 0보다 크다.

- 프로세스들 간의 상대적인 수행 속도는 가정하지 않는다.

- 프로세스들의 일반적인 구조

  <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205205204742.png" alt="image-20230205205204742" style="zoom:67%;" /> 

  - entry section에서 공유 데이터 접근에 대한 lock을 걸고, exit section에서 lock을 푸는 구조가 되어야 할 것

- 프로세스들은 수행의 동기화(synchronization)를 위해 몇몇 변수를 공유할 수 있다.

  -> synchronization variable

### 2. 프로그램적 해결법의 충족 조건

- **Mutual Exclusion (상호 배제)**
  - 프로세스 Pi가 critical section 부분을 수행중이면 다른 모든 프로세스들은 그들의 critical section에 들어가면 안 된다.
- **Progress (진행)**
  - 아무도 critical section에 있지 않은 상태에서 critical section에 들어가고자 하는 프로세스가 있으면 critical section에 들어가게 해주어야 한다.
- **Bounded Waiting (유한 대기)**
  - 프로세스가 critical section에 들어가려고 요청한 후부터 그 요청이 허용될 때까지 다른 프로세스들이 critical section에 들어가는 횟수에 한계가 있어야 한다. ( = 무한정 대기해서는 안 된다.)
  - 한 프로세스만 CS에 들어가 다른 프로세스가 starvation을 겪는 것을 방지하고 모든 프로세스가 CS에 들어가기 위한 기회를 가질 수 있게 하기 위한 조건

### 3. 여러 알고리즘

#### Algorithm 1

- Synchronization variable

  - **int turn**;  => 차례를 나타내는 변수
  - initially **turn = 0**;  => Pi 는 **(turn == i)** 인 경우 critical section에 들어갈 수 있다.

- Process P0

  <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205174050786.png" alt="image-20230205174050786" style="zoom:67%;" /> 

  1. 차례가 아닐 때 while문을 돌며 대기

  2. 차례가 되면 critical section 수행 

  3. 나오면 turn을 바꿔줌

     

- 조건 충족 여부

  - Mutual Exclusion을 만족하지만 **Progress를 만족하지 못한다.**

    ex) critical section에 P0는 n (n > 1) 번, P1은 1번 들어가려고 한다면, P0은 P1이 1번 critical section에 들어간 이후부터는 순서를 바꿔주지 않기 때문에 들어갈 수 없음

#### Algorithm 2

- Synchronization variables
  - **boolean flag[2];**
  - initially **flag [모두] = false;** => no one is in CS
  - Pi는 **(flag[i] == true)** 로 세팅하여 CS에 들어가고 싶다는 의사 표시
- Process Pi

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205175344268.png" alt="image-20230205175344268" style="zoom:50%;" /> 

1. CS에 들어간다는 의사표시를 한다

2. 상대방이 flag를 세팅했다면 기다린다

3. CS를 수행한다

4. 나오면 flag를 내려놓는다(false로 바꾼다)



- 조건 충족 여부

  - Mutual Exclusion을 만족하지만 **Progress를 만족하지 못한다**.

  - flag를 내려놓는 행위는 critical section에 들어갔다 나와야 할 수 있는데, 모든 프로세스가 2행까지 수행 후 끊임없이 양보해서 아무도 들어가지 못하는 상황이 생길 수 있음

#### Algorithm 3 (Peterson's Algorithm)<sup>[1](#footnote_1)</sup>

- Algorithm 1과 2의 variable을 모두 사용

- Process Pi

  <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205180617120.png" alt="image-20230205180617120" style="zoom: 50%;" /> 

- 조건 충족 여부

  - 세 가지 충족 조건을 모두 만족한다.

    - CS에 접근하려면 자신의 turn 이어야(turn == i) 한다 => **Mutual Exclusion 만족**

    - 자신의 turn이 아니더라도 남들이 아무도 안 쓰면 (flag[j] == false) 쓸 수 있다 => **Progress 만족**

    - 다른 프로세스가 종료되면 차례를 부여받고, 남에 의해서만 다시 CS에 들어갈 수 있어서 한 프로세스가 독점하지 않는다 => **Bounded Waiting 만족**

      - Pi는 while 코드에서 수행되고 있을 동안 변수 turn의 값을 변화시키지 않는다.

        -> Pi는 오로지 Pj에 의해 turn이 i로 바뀌어야만 CS에 들어갈 수 있다.

  - 문제점: **Busy Waiting ( = spin lock)** 으로 인해 **CPU와 Memory를 쓰면서 wait** => 자원을 많이 소모한다.



- Algorithm의 entry section과 exit section이 복잡해진 이유
  - **Instruction들이 실행되는 도중에 CPU를 빼앗길 수 있다**는 가정 하에 코드를 짜기 때문에



### 4. Synchronization Hardware : Test_and_set()<sup>[2](#footnote_2)</sup>

- **하드웨어**적으로 **Test&modify**를 **atomic**하게 수행할 수 있도록 지원하는 경우 앞의 문제는 간단히 해결

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205210003047.png" alt="image-20230205210003047" style="zoom:67%;" />

- **Test_and_set()은 입력값의 상태를 그대로 반환하고, 입력값은 항상 TRUE로 갱신**하는 함수이다.
- 하드웨어적으로 구현되어 있기 때문에 하나의 명령어로 취급해도 된다.



### 5. Semaphores

- 공유 자원들을 여러 프로세스가 접근하는 것을 막기 위한 **추상 자료형**

  - **추상 자료형(Abstract Data  Type)**이란 기능의 구현 부분을 나타내지 않고 순수한 기능이 무엇인지 나열한 것

    ex) 선풍기의 사용 설명서에는 정지, 미풍, 약풍, 강풍, 회전, 타이머 등의 기능 설명과 사용 방법이 나와 있으나,

    ​		버튼을 눌렀을 때 선풍기 내부 회로에서 어떤 일이 발생하는지는 나와있지 않다.

  - 사용자는 Semaphore와 같은 추상 자료형을 사용함으로써 **기능을 일일이 구현할 필요 없이 프로그램을 간단하게 작성**할 수 있다.

- **자원의 갯수**를 나타내는 **정수값(integer, 아래 예시에서의 S)**을 가질 수 있다.

- 아래의 **두 가지 atomic 연산**에 의해서만 접근 가능

  - **P 연산**은 가용한 자원이 있는 경우 **자원을 사용**
  - **S 연산**은 사용한 **자원을 반납**

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205233253570.png" alt="image-20230205233253570" style="zoom:67%;" /> 

- **busy-wait** 문제 있음 (자원이 없다면 기다리면서 CPU 시간을 다 쓰고 반납)



#### Critical Section of n Processes

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205234648675.png" alt="image-20230205234648675" style="zoom:67%;" /> 

- 사용 가능한 자원의 수(semaphore mutex)가 양수가 아니라면 무한정 대기하므로 비효율적
- 대신 Block & Wakeup 방식의 구현 ( = sleep lock)



#### Block / Wakeup Implementation

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205235646898.png" alt="image-20230205235646898" style="zoom:67%;" /> 

1. 누군가가 공유 데이터를 이미 쓰고 있다면 프로세스를 **Block** 시켜서 CPU를 반납하게 한다.

2. 공유 데이터를 가지고 있던 프로세스가 내어놓으면 깨어나서(**Awake**) Ready Queue에 들어온다.
3. CPU를 얻고 공유 데이터를 얻는다.

- Semaphore를 다음과 같이 정의
  - value : 변수 값
  - L : 잠들어 있는 프로세스들을 연결하기 위한 queue

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230206000007661.png" alt="image-20230206000007661" style="zoom:67%;" /> 

- block과 wakeup을 다음과 같이 가정

  - **block** 

    - 커널은 block을 호출한 프로세스를 suspend 시킴
    - 이 프로세스의 PCB를 semaphore에 대한 wait queue에 넣음

  - **wakeup(P)**

    - block된 프로세스 P를 wakeup시킴
    - 이 프로세스의 PCB를 ready queue로 옮김

    <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230206000049236.png" alt="image-20230206000049236" style="zoom:67%;" /> 

- Semaphore 연산이 다음과 같이 정의됨

  ​	 <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230206001650351.png" alt="image-20230206001650351" style="zoom:67%;" /> 

  - value를 1 줄였을 때 그 값이 음수라면 자원의 여분이 없다는 것

    -> 프로세스를 대기열에 세우고 block 시킴

  ​	<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230206001708975.png" alt="image-20230206001708975" style="zoom:67%;" />  

  - value를 1 늘렸을 때 그 값이 '**0 이하**'라면 공유자원이 0인 상태에서 P 연산을 수행하여 대기열에 등록된 프로세스가 하나 이상 있다는 것

    -> 프로세스를 대기열에서 빼서 wakeup한다.

    **※ 주의** : 이때 **S.value**는 Semaphore 맨 위 예시에서의 S와 달리, 자원의 갯수를 나타내는 게 아니라 **대기열의 상황(wakeup해야 하는 프로세스가 있는지)**을 나타낸다

#### Busy-wait vs Block/wakeup 중 어느 게 더 나은가?

- 일반적으로 Block/wakeup 방식이 더 좋음
  - 공유 자원을 사용할 수 없다면, CPU를 잡고 대기하기보다는 반납하고 Block 상태로 들어가는 편이 CPU를 의미있게 사용
- 다만 Block/wakeup 역시 프로세스의 상태를 변경시켜야 한다는 점에서 overhead가 들기 때문에 Critical Section의 길이에 따라 Busy-wait도 괜찮음
  - Critical Section의 길이가 긴 경우 Block/wakeup이 적당
  - Critical Section의 길이가 매우 짧은 경우, Block/wakeup 오버헤드가 busy-wait 오버헤드보다 더 커질 수 있음 

#### Semaphore의 두 가지 종류

- **Counting semaphore**

  - 도메인이 0 이상인 임의의 정수값
  - 여분의 자원을 count
  - 주로 resource sharing에 사용

- **Binary semaphore**<sup>[3](#footnote_3)</sup>
  - 0 또는 1 값만 가질 수 있는 semaphore
  - 주로 mutual exclusion (lock / unlock)에 사용

#### Semaphore를 사용할 때 생길 수 있는 문제: Deadlock and Starvation

- **Deadlock**

  - 둘 이상의 프로세스가 서로 상대방에 의해 충족될 수 있는 event를 무한히 기다리는 현상

  - S와 Q가 1로 초기화된 semaphore라 하자.

    <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230206150835180.png" alt="image-20230206150835180" style="zoom:67%;" />  

    - P0와 P1이 자원 하나를 들고 상대방이 가진 것을 기다리며 영원히 대기 

      -> 해결책: 자원을 획득하는 순서를 동일하게 맞춘다. P1도 S를 얻고 난 후 Q를 얻게끔.

- **Starvation**

  - 특정 프로세스의 우선순위가 낮아서 원하는 자원을 계속 할당받지 못하는 상태
  - Indefinite blocking. 프로세스가 suspend된 이유에 해당하는 세마포어 큐에서 빠져나갈 수 없는 현상

### 6. Monitor

#### Semaphore의 문제점과 해결 방법

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230208160416274.png" alt="image-20230208160416274" style="zoom:67%;" /> 

- 코딩하기 힘들다.

- 정확성(correctness)의 입증이 어렵다.

- 자발적 협력(voluntary cooperation)이 필요하다.

- 한 번의 실수가 모든 시스템에 치명적인 영향을 미친다.

  

- 이러한 문제를 해결하기 위해 Monitor를 사용할 수 있다.

  <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230208190249953.png" alt="image-20230208190249953" style="zoom:50%;" /> 

  - Monitor의 내부에 공유 데이터와 공유 데이터에 접근하기 위한 procedure를 정의하고 이 procedure를 통해서만 공유 데이터에 접근할 수 있게 한다.
  - 모니터 내에서는 한 번에 하나의 프로세스만 활동할 수 있으므로, 프로그래머가 동기화 제약 조건을 명시적으로 코딩할 필요가 없다.

#### Monitor란?

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230208190356160.png" alt="image-20230208190356160" style="zoom:67%;" />  

- **Monitor**란 공유 자원(shared resource)에 접근하려는 쓰레드들이 상호 배제(mutual exclusion)를 준수하고, 어떤 조건(condition)이 true가 될 때까지 기다리게 하는 동기화 방법, 혹은 이 방법을 활용하기 위해 구현된 기능 또는 모듈이다.`
  - **monitor**는 (1) **mutex (lock)**와 (2) **1개 이상의 condition variable**로 구성되어 있다.
    - **condition variable(이하 CV)**은 쓰레드들이 특정 조건(condition)이 충족되기 전까지 기다리는 대기열(queue)이다.
      - 이러한 '조건'은 주로 공유 데이터의 상태(state)와 관련된 것이다.
      - CV은 **wait**와 **signal** 연산에 의해서만 접근할 수 있다.
        	- x라는 CV가 있을 때, x.wait()를 호출한 프로세스는 x.signal()을 호출하기 전까지 suspend된다.
        	- x.signal()은 정확하게 하나의 suspend된 프로세스를 resume한다. suspend된 프로세스가 없다면 아무 일도 일어나지 않는다.
- 일반적으로 mutex, semaphore는 low-level 또는 OS level에서 제공되는 반면, monitor는 그보다 고수준(higher-level)인 language level에서 제공된다.
  - mutex는 프로그래머로 하여금 프로그래밍 언어 차원에서 synchronization 문제를 해결하고, 정확하고 효율적인 멀티쓰레드(multithreaded) 프로그램을 작성할 수 있게 해준다.


## Synchronization의 고전적 문제들

### 1. Bounded-Buffer Problem (Producer-Consumer Problem)

#### (1) 개요

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230206164041130.png" alt="image-20230206164041130" style="zoom:67%;" /> 

- Producer는 공유 버퍼에 데이터를 만들어 집어넣고, Consumer는 데이터를 꺼내서 사용한다.

- **Shared data**

  - buffer 및 buffer의 조작 변수(empty / full buffer의 시작 위치)

- **Synchronization variables**

  - **mutual exclusion**
    - 공유 데이터의 mutual exclusion을 위해 **binary semaphore**가 필요

  - **resource count**
    - 남은 empty / full buffer의 수를 표시하기 위해 **integer semaphore**가 필요

#### (2) Semaphore를 활용한 해결책

- 구현 코드
  - **Synchronization variables**
    - semaphore full = 0, empty = n, mutex = 1;

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230206182100739.png" alt="image-20230206182100739" style="zoom:67%;" /> 

1. **P(empty / full)** : (가용한 자원이 있어서) 자원을 획득했다면

2. **P(mutex)** : buffer 전체에 lock을 걸고,

3. 공유 buffer에 데이터를 추가하거나 제거하고
4. **V(mutex)** : buffer의 lock을 해제하고
5. **V(full / empty)** : full / empty buffer의 개수를 1 증가시킨다.

#### (3) Monitor를 활용한 해결책

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230208172913987.png" alt="image-20230208172913987" style="zoom:67%;" /> 

- produce : 빈 버퍼가 없다면 CV empty에서 대기(wait)한다 -> 있다면 빈 버퍼에 x를 채운다 -> 채워진(full) 버퍼를 대기하는 프로세스를 깨운다.

  consume : 채워진 버퍼가 없다면 CV full에서 대기(wait)한다 -> 있다면 채워진 버퍼를 비우고 x에 저장한다. -> 빈(empty) 버퍼를 대기하는 프로세스를 깨운다.

- 공유 buffer가 monitor 내부에 정의되어 있어, monitor 내부 코드를 실행해야 한다

  -> monitor 내부에서는 producer든 consumer든 하나의 프로세스만 활성화되기 때문에

  -> 직접 lock을 걸거나 풀 필요가 없다

### 2. Readers-Writers Problem

#### (1) 개요

- Readers-Writers Problem은 다수의 Reader와 다수의 Writer가 공통 DB를 사용할 때 발생할 수 있는 문제이며, 프로세스 동기화로 해결할 수 있다.

  - 이때, Reader는 데이터를 읽기만 하는 프로세스, Writer는 읽고 수정하는 프로세스이다.

- Solution의 접근법

  - Reader들끼리만 임계구역에 들어간 경우에는 데이터 수정이 일어나지 않아 데이터 일관성(data consistency)이 유지된다.

    -> DB를 임계구역으로 지정하여, 한 번에 하나의 프로세스만 접근할 수 있도록 mutex 처리하는 방법은 비효율적

    -> "Reader들만 공통 DB에 동시 접근할 수 있게 하고, Writer는 Mutual exclusion을 적용"

    

  - Writer가 DB에 접근 허가를 얻지 못한 상태에서는 모든 대기중인 Reader들을 다 DB에 접근하게 해준다.

  - Writer는 대기중인 Reader가 하나도 없을 때 DB 접근이 허용된다.

  - 일단 Writer가 DB에 접근중이면 Reader들은 접근이 금지되며, Writer가 DB에서 빠져나가야만 Reader의 접근이 허용된다.

#### (2) Semaphore를 활용한 해결책

- **Shared data**

  - DB 자체
  - readcount (현재 DB에 접근중인 Reader의 수)

- **Synchronization variables**

  - **mutex** : 공유 변수 readcount를 접근하는 코드(critical section)의 mutual exclusion 보장을 위해 사용
  - **db** : Reader와 Writer가 공유 DB 자체를 올바르게 접근하게 하는 역할

- 구현 코드

  - **Shared data**

    - DB 자체;
    - int readcount = 0;

  - **Synchronization variables**

    - semaphore mutex = 1, db = 1;

    <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230206184740888.png" alt="image-20230206184740888" style="zoom:67%;" /> 

    - **Writer**

      1. **P(db)** : (누군가가 db를 사용하고 있지 않다면) lock을 건다.
      2. DB에 **write** 작업을 한다.
      3. **V(db)** : lock을 해제한다.

    - **Reader**

      1. **P(mutex)** : (공유 변수인) readcount를 변경하기 위해 lock을 건다.

      2. **readcount++**; : readcount를 1 증가시킨다.

      3. **if (readcount == 1)** : 자신이 최초의 reader 프로세스라면,

         **P(db)** : DB에 lock을 건다. => 1보다 크다면 자신 이전의 process가 lock을 걸었을 것이기 때문에 걸 필요가 없음

      4. **V(mutex)** : readcount에 대한 lock을 푼다.

      5. DB를 **read**한다.

      6. 이후, 마찬가지의 과정을 거쳐 readcount를 1 감소시키고,

         더 이상 DB에 접근하는 reader가 없다면,

         DB의 lock을 해제한다.

         

    ※ 만약 Reader가 DB를 읽고 있는 상황에서 Writer -> 다른 Reader 순서대로 도착해 DB에 접근하고자 한다면?

    ​    -> Reader만 DB에 접근할 수 있으며, Reader들이 모두 빠져나가고 나서야 Writer가 접근할 수 있다.

    ​    -> Writer의 **Starvation** 문제 발생 가능

    ​    *해결책: 큐에 우선순위를 두어 일정 시간보다 늦게 도착한 Reader는 대기시킨다.(신호등의 비유)

### 3. Dining-Philosophers Problem<sup>[4](#footnote_4)</sup>

#### (1) 개요

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230207230313930.png" alt="image-20230207230313930" style="zoom:67%;" /> 

- 철학자 다섯 명이 원형 식탁에 둘러앉아 생각을 하다가 배고프면 밥을 먹는다. 각 철학자의 자리 사이에 젓가락이 한 짝씩 놓여있고, 철학자가 식사를 하기 위해서는 자신의 양 옆에 놓인 두 짝을 모두 사용해야 한다.

#### (2) Semaphore를 활용한 해결책

##### a. 간단하지만 Deadlock 가능성이 있는 버전

- Synchronization variables

  - semaphore chopstick[5]; 

    -> 초기값은 1

- Philosopher i의 코드

  ```c
  do {
      // 왼쪽과 오른쪽 젓가락을 집어든다.
      P(chopstick[i]); 
      p(chopstick[(i + 1) % 5]);
      ...
  	eat();
      ...
  	// 식사가 끝났다면 왼쪽과 오른쪽 젓가락을 내려놓는다.
  	V(chopstick[i]);
      V(chopstick[(i + 1) % 5]);
      ...
      think();
      ...
  } while (1);
  ```

- 문제점

  - Deadlock 가능성이 있다.

    ex) 모든 철학자가 동시에 배가 고파져 왼쪽 젓가락을 집어버린 경우

- 해결 방안

  1. 4명의 철학자만이 테이블에 동시에 앉을 수 있도록 한다.

  2. 젓가락을 두 개 모두 집을 수 있을 때에만 젓가락을 집을 수 있게 한다.

  3. 짝수(홀수) 철학자는 왼쪽(오른쪽) 젓가락부터 집도록 한다.

##### b. 2번 해결 방안을 적용한 버전

- **Synchronization variables**

  - enum {thinking, hungry, eating} state[5];
  - semaphore self[5] = 0;
    - 0이라면 어떤 철학자가 젓가락 두 개를 다 집을 수 없음, 1이라면 두 개 다 잡을 수 있음을 의미
  - semaphore mutex = 1;
    - 모든 철학자의 상태는 자신뿐 아니라 양 옆에 위치한 철학자 역시 바꿀 수 있으므로 공유 변수
    - 상태값에 대한 동시접근을 막기 위해 lock을 거는 변수

- 구현 코드

  <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230207232403304.png" alt="image-20230207232403304" style="zoom:67%;" /> 

  


#### (3) Monitor를 활용한 해결책

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230209135201618.png" alt="image-20230209135201618" style="zoom: 67%;" /> 

- test() : 철학자 i의 상태가 '배고픔(hungry)'이고, 양 옆의 철학자들이 '식사중(eating)'이지 않다면, i의 상태를 '식사중'으로 변경한다. 

  그리고 i를 깨운다(이 부분은 putdown() 내부의 test()만 해당).

- pickup() : i의 상태를 '배고픔'으로 설정하고, test()를 호출한다. (i의 양 옆 철학자들 중 적어도 한 명이 '식사중'이어서) i의 상태가 '식사중'이 아니라면 대기(wait)한다.

- putdown() : i의 상태를 '생각중(thinking)'으로 설정한다. 양 옆의 철학자들이 자신이 '식사중'이어서 대기열에 들어갔다면 깨운다(signal).

### 참고

<a name="footnote_1">1</a>. Peterson's Algorithm

https://jhnyang.tistory.com/37

https://www.youtube.com/watch?v=gYCiTtgGR5Q

<a name="footnote_2">2</a>. Test_and_set()

https://charles098.tistory.com/92

<a name="footnote_3">3</a>. Semaphore와 Mutex 비교

https://worthpreading.tistory.com/90

https://www.geeksforgeeks.org/difference-between-binary-semaphore-and-mutex/

<a name="footnote_4">4</a>. Dining Philosophers problem

https://m.blog.naver.com/hirit808/221788147057

https://en.wikipedia.org/wiki/Dining_philosophers_problem

Monitor

https://en.wikipedia.org/wiki/Monitor_(synchronization)

https://velog.io/@max9106/%EC%9A%B4%EC%98%81-%EC%B2%B4%EC%A0%9C-Process-Synchronization-Monitor

