---
title:  "[운영체제] Memory Management"
excerpt: "Demand Paging, Page Fault, Page Replacement Algorithm(Optimal, FIFO, LRU, LFU)"

categories:
  - OS
tags:
  - [OS]
  
toc: true
toc_sticky: true
 
date: 2023-02-15T17:54:17-04:00
last_modified_at: 2023-02-15T17:54:17-04:00
---

# Virtual Memory

- 이번 챕터에서는 메모리 관리 기법 중 Paging 기법을 사용하는 것으로 가정

## Demand Paging

- 프로세스에서 사용할 메모리 페이지를 프로세스 생성 시에 모두 할당하지 않고, 필요(demand)할 때마다 동적으로 할당하는 기법

  - I/O 양의 감소 : disk로부터 필요한 page만을 읽어오기 때문
  - Memory 사용량 감소
  - 빠른 응답 시간
  - 더 많은 사용자(프로그램) 수용

- Valid / Invalid bit의 사용

  - Invalid의 의미

    - 사용되지 않는 주소 영역인 경우 (아래 그림의 6, 7)
    - 페이지가 물리적 메모리에 없는 경우 (아래 그림의 1, 3, 4)

  - 처음에는 모든 page entry가 invalid로 초기화

  - address translation 시에 invalid bit이 set되어 있으면

    -> "**page fault**" 발생

    -> CPU가 운영체제로 넘어감

![image-20230312001918564](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230312001918564.png) 



## Page Fault

- Page Fault란<sup>[1](#footnote_1)</sup>

  - 프로그램이 자신의 주소공간에는 존재하지만 시스템의 메모리에는 현재 없는 데이터나 코드에 접근 시도하였을 경우 발생하는 현상
  - Page Fault가 발생하면 운영체제는 그 데이터를 메모리로 가져와서 마치 Page Fault가 전혀 발생하지 않은 것처럼 프로그램이 계속적으로 동작하게 해줌

- 과정

  <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230312214841968.png" alt="image-20230312214841968" style="zoom: 80%;" /> 

  1. invalid page를 접근하면 MMU가 trap을 발생시킴

  2. Kernel mode로 들어가서 page fault handler가 invoke됨

  3. 아래와 같은 순서로 page fault를 처리

     (1) invalid reference인지 확인 : 프로세스가 사용하지 않는 주소, 접근 권한에 대한 violation

        => abort

     (2) 빈 page를 획득 (없으면 뺏어온다 : replace)

     (3) 해당 page를 disk에서 memory로 읽어온다

     ​	a. disk I/O가 끝나기까지 이 프로세스는 CPU를 preempt 당함 (block) : disk I/O는 오래 걸리므로 CPU 들고 있으면 낭비

     ​	b. disk read가 끝나면 page tables entry 기록, valid/invalid bit = "valid"

     ​	c. ready queue에 process를 insert -> dispatch later

  4. 이 프로세스가 CPU를 잡고 다시 running

  5. 아까 중단되었던 instruction을 재개



## Demand Paging의 성능

- Page Fault Rate 0 <= p <= 1.0

  - p = 0 : page fault 발생하지 않음
  - p = 1 : 모든 요청이 page fault 발생시킴

- Effective Access Time

  = (1 - p) * memory access에 걸리는 시간

     +p * **(OS와 하드웨어에 의한 page fault 처리 overhead)**

  ​		**+[필요에 따라 page swap-out]**

  ​		**+page swap-in**

  ​		**+OS와 하드웨어 재시작 overhead**



## Page Replacement

### a. **Page Replacement란**

- Free frame이 없는 경우 **어떤 frame을 빼앗아올지** 결정해야 함

- **곧바로 사용되지 않을 page**를 쫓아내는 것이 좋음

- 동일한 page가 여러 번 메모리에서 쫓겨났다가 다시 들어올 수 있음

- Page Replacement는 운영 체제에 의해, 다음 과정을 통해 이루어진다.

  ![image-20230312220847680](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230312220847680.png) 

  - 단, victim page를 swap out할 때 변경된 내용이 있다면 backing store에 써줘야 한다

### b. **Replacement Algorithm**

#### b-1. 의의와 평가 기준

- Page Replacement가 효율적으로 이루어지기 위해서는, 미래에 참조될 가능성이 가장 적은 page를 swap out하고

  참조될 가능성이 높은 page들을 메모리에 둘 수 있는 replacement algorithm이 구현되어야 함

​		-> **page-fault rate을 최소화**하는 것이 목표

- 알고리즘의 평가

  - 주어진 **page reference string**에 대해 page fault를 얼마나 내는지 조사

    - reference string의 예 

      - 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5

      - 1번에 접근 시도를 했는데, replacement algorithm이 1번을 쫓아낸 상태라면 => page fault

        ​																								   쫓아내지 않았다면 => 메모리에서 직접 참조

#### b-2. 종류

##### Optimal Algorithm

- 가장 먼 미래에 참조되는 page를 replace

  - 미래에 참조되는 page (page reference string)를 미리 다 안다고 가정

    -> 실제 시스템에서 사용될 수 없음

    -> online으로 사용되는 것이 아니므로, **offline algorithm**이라고 불림

- 예시) 메모리의 크기가 4 page frame인 경우

  ![image-20230312222308349](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230312222308349.png) 

  - 5번 page를 참조할 때, 1,2,3,4 중 가장 먼 미래에 참조되는 4번 page를 5번 page로 replace
  - page fault가 6번 발생 (빨간색 숫자)

- 다른 **알고리즘의 성능에 대한 상한선(upper bound)**를 제공

  - Optimal Algorithm이기 때문에, 다른 어떤 알고리즘도 page fault를 이보다 적게 낼 수는 없음
  - **Belady's optimal algorithm**, **MIN** (page fault가 최소), **OPT** 등으로 불림



###### FIFO (First In First Out) Algorithm

- 먼저 들어온 것을 먼저 내쫓음

- 예시) 메모리가 3 / 4 page frame인 경우

  ![image-20230312223019632](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230312223019632.png) 

- **FIFO Anomaly** (Belady's Anomaly)

  - 메모리 frame을 늘렸는데 page fault가 줄어들지 않는 현상 (성능이 좋아지지 않을 수 있다)



##### LRU (Least Recently Used) Algorithm

- **가장 오래 전에 참조된 페이지**를 쫓아냄

  ![image-20230312223450680](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230312223450680.png) 

  - 5번 page를 참조하는 시점 기준, 가장 오래 전에 참조한 page는 3번

- 실제로 가장 많이 사용되는 algorithm

- 단점

  - 참조 빈도가 가장 높은 page라도 가장 오래 전에 참조되었다는 이유로 replace될 수 있다

- 구현

  - 운영체제가 Linked list 형태로 page들의 참조 시간 순서를 관리

    - 어떤 page가 참조되면, list의 가장 아랫쪽으로 이동
    - 쫓아낼 때는 가장 위에서 쫓아냄

  - 시간복잡도는 O(1) 

    - page들끼리 time stamp를 비교 (이 경우 시간복잡도는 O(n))할 필요 없이, 

      list에서 가장 위에 있는 page를 쫓아내면 됨

    ![image-20230312224611966](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230312224611966.png) 



##### LFU (Least Frequently Used) Algorithm

- **참조 횟수(reference count)가 가장 적은 페이지**를 쫓아냄

  - 최저 참조 횟수인 page가 여러 개인 경우
    - LFU 알고리즘 자체에서는 임의로 선정
    - 성능 향상을 위해 가장 오래 전에 참조된 page를 지우게 구현할 수도 있음

- 구현

  - List

    - 다른 page들과 참조 횟수를 비교해서 크면 그 page보다 아랫쪽으로 이동

    - 시간 복잡도가 O(n)

      - 최악의 경우, 다른 모든 page들과 참조횟수를 비교해야 함
      - replacement 알고리즘으로는 부적합

      ![image-20230312225235597](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230312225235597.png) 

  - **Heap**<sup>[2](#footnote_2)</sup>

    - 완전 이진 트리(complete binary tree)의 형태

    - 최소 힙(min heap): 부모 노드의 frequency <= 자식 노드의 frequency

      - 자식 노드 두 개외 비교해 참조 횟수가 크다면 자리를 바꿔 아랫쪽으로 이동

    - 시간 복잡도가 O(log n)

      - Heap을 재구성하는 overhead
      - 최악의 경우, 트리의 높이(log 2 n)만큼의 노드와 비교

      ![image-20230312225622113](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230312225622113.png) 

- Paging System에서 LRU, LFU 알고리즘을 사용할 수 있는가?

  - OS가 페이지를 쫓아내고자(evict) 할 때, LRU/LFU 알고리즘을 사용한다면, **가장 오래 전에 참조된/참조 횟수가 가장 적은 페이지**를 알 수 있어야 하는데, **OS는 이를 알 수 없음**

    - 메모리에 참조하려는 페이지가 있으면, (운영체제를 거칠 필요 없이) 하드웨어적으로 주소 변환을 해서 해당 페이지를 읽어들임

      -> 이 경우 운영체제가 페이지를 언제/몇 번 참조했는지, 알 수 없음 (정보가 page fault가 발생한 경우에 한해서만 주어짐)

      -> **Paging System, Virtual Memory System에서 LRU LFU 알고리즘 사용할 수 없음** 

      ​	(단, buffer caching, web caching 등에선 사용 가능)

##### Clock Algorithm

![image-20230313152709718](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230313152709718.png) 

- **Second chance algorithm**, NUR(Not Used Recently) 또는 NRU 등으로 불림

- **Reference bit가 set되어 있지 않은( = 0인) 페이지**를 찾을 때까지 Page table을 따라 포인터를 이동 (circular list) 

  찾으면 해당 페이지를 쫓아냄

  - HW는 참조된 페이지의 reference bit을 1로 set (HW의 도움을 받아 OS의 한계를 극복),

    OS는 쫓아낼 페이지를 결정하는 과정에서 Reference bit이 1이라면 0으로 변경

  - 한 바퀴 돌아와서도 (=second chance) 0이면 그 때는 replace 됨

    자주 참조되는 페이지라면 second chance가 올 때, reference bit가 1일 것

  - 가장 오래 전에 참조한 페이지는 아니지만, 적어도 포인터가 한 바퀴 도는 동안 참조가 안 된 페이지를 쫓아내므로

    LRU의 근사(approximation) 알고리즘이라고 할 수 있음

- Clock algorithm의 개선

  - reference bit과 **modified bit** (dirty bit)을 함께 사용

    - reference bit = 1 : 최근에 참조된 페이지
    - modified bit = 1 : 최근에 변경된 페이지 (쫓아낼 때 disk에 변경사항을 기록해야 하므로 **I/O를 동반**하는 페이지)

  - modified bit이 0인 페이지를 1인 페이지보다 우선적으로 쫓아낸다면 disk에 쓰는 작업이 필요없으므로

     replacement를 빠르게 진행할 수 있음



## Caching

- Caching 기법

  - 한정된 빠른 공간(=Cache)에 요청된 데이터를 저장해 두었다가 후속 요청시 캐쉬로부터 직접 서비스하는 방식

    - paging system 역시 disk보다 상대적으로 빠른 공간인 메모리에 요청된 page에 대한 데이터를 저장하는 caching 기법 활용

  - paging system 외에도 cache memory, buffer caching, web caching 등 다양한 분야에서 사용

    - paging system, cache memory, buffer caching은 단일 시스템에서 저장 매체 간의 속도 차이를,

      web caching은 떨어져 있는 다른 시스템에서 데이터를 읽어오는 데 드는 시간 제약을 극복하기 위해 사용된다는 차이

- Cache 운영의 시간 제약

  - Replacement Algorithm에서 삭제할 항목을 결정하는 일에 지나치게 많은 시간이 걸리는 경우 실제 시스템에서 사용할 수 없음

  - Buffer caching이나 Web caching의 경우

    - O(1)에서 O(log n) 정도까지 허용

  - Paging system의 경우

    - page fault인 경우에만 OS가 관여함

    - 페이지가 이미 메모리에 존재하는 경우 참조 시각 등의 정보를 OS가 알 수 없음

    - O(1)인 LRU의 list 조작조차 불가능

      

## Page Frame의 Allocation

- Allocation problem : **각 process에 얼마만큼의 page frame을 할당**할 것인가?

- Allocation의 필요성

  - 메모리 참조 명령어 수행시, 명령어와 데이터 등 여러 페이지 동시 참조

    - 명령어 수행을 위해 최소한 할당되어야 하는 frame의 수가 있음

  - Loop를 구성하는 page들은 한꺼번에 allocate되는 것이 유리함

    - 최소한의 allocation이 없으면 매 loop마다 page fault

      ex) process에 할당된 page frame이 3인데 loop를 구성하는 page 수가 4라면?

  - 특정 process가 page frame을 전부 장악하는 문제가 발생할 수도 있음

- Allocation scheme

  - **Equal allocation** : 모든 프로세스에 동일한 갯수 할당
  - **Proprotional allocation** : 프로세스 크기에 비례하여 할당
  - **Priority allocation** : 프로세스의 priority에 따라 다르게 할당



## Global vs Local Replacement

- **Global replacement**
  - 미리 allocation을 하지 않고, **replacement algorithm이 allocation의 효과**를 냄
  - replace 시 **다른 process에 할당된 frame을 빼앗아** 올 수 있음 (=다른 프로그램의 페이지도 쫓아낼 수 있음)
  - FIFO, LRU, LFU 등의 알고리즘을 global replacement로 사용할 경우
  - **Working set, PFF 알고리즘** 사용
- **Local replacement**
  - 프로세스 별로 allocation을 한 뒤, **각 프로세스는 자신에게 할당된 frame 안에서만 replacement**
  - FIFO, LRU, LFU 등의 알고리즘을 process  별로 사용할 경우



## Thrashing

### a. Thrashing의 정의와 발생 원인

- Thrashing이란?

  - page fault가 너무 자주 발생하여 시스템이 프로세스의 처리보다 페이지 교체에 더 많은 시간과 자원을 사용하는 현상
  - 프로세스의 원활한 수행에 필요한 최소한의 page frame 수를 할당 받지 못한 경우 발생
  - 프로세스의 실행보다 paging을 위해 더 많은 시간이 소요
  - thrashing이 발생하면 page fault rate 높아짐 , CPU utlization rate 낮아짐

- Thrashing의 발생 원인

  ![image-20230313165416442](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230313165416442.png) 

  - x축은 degree of multiprogramming (메모리에 올라와 있는 프로그램의 수. 이하 MPD) 

    y축은 CPU 사용률

  - 어느 정도까지는 메모리에 올라온 프로그램이 많을수록 (MPD가 높을수록) CPU utilization이 높아짐<sup>[3](#footnote_3)</sup>

    - 한 프로세스가 CPU를 쓰지 않고 보내는 시간의 비율이 P이라고 할 때, 

      N개의 프로세스가 I/O를 기다리고 있을 확률은 P^N

      -> N (=degree of multiprogramming)이 커질수록 CPU utilization = 1 - P^N은 커짐

  - 그러나 메모리에 너무 많은 프로그램이 올라와 각 프로세스가 원활한 수행을 위한 최소한의 frame조차 할당받지 못하게 되면

    -> page fault가 빈번하게 발생

    -> 어떤 프로그램이 CPU를 사용하더라도 page fault가 발생하므로 CPU utilization이 낮아짐

    -> 그런데 OS는 CPU utilization이 낮으니 MPD를 높여야겠다고 판단하고 또 다른 프로세스가 추가됨

    -> 프로세스 당 할당된 frame 수 더욱 감소하고 프로세스는 page의 swap in/out으로 바쁨

    -> 대부분의 시간동안 CPU는 한가하고 throughput 감소

    

### b. Thrashing 발생 방지 알고리즘

- Thrashing을 막으려면, MPD를 조절하고 각 프로세스가 어느 정도는 메모리를 확보할 수 있게 해야 하며,

  이를 위해 다음에 소개할 알고리즘을 활용



#### b-1. Working Set Algorithm

##### Working Set Model

- **참조의 지역성** (**Locality of reference** = the principle of locality)<sup>[4](#footnote_4)</sup>

  - **동일한 값 또는 해당 값에 관계된 스토리지 위치가 자주 액세스**되는 경향

  - 3가지 기본형 (공간, 시간 지역성만을 일컬어 2가지 기본형이라고 칭하기도 함)

    - **공간(spatial) 지역성** : 특정 클러스터의 기억 장소들에 대해 참조가 집중적으로 이루어지는 경향

    - **시간(temporal) 지역성** : 최근 사용되었던 기억 장소들이 집중적으로 액세스되는 경향

      ​										  참조했던 메모리는 빠른 시간 안에 다시 참조될 확률이 높음

    - 순차(sequential) 지역성 : 정렬된 데이터가 순차적으로 액세스되는 경향

      ​											 ex) 일차원 배열의 요소를 처음부터 끝까지 차례대로 순회

      ​											 공간 지역성에 포함하기도 한다

  - 집중적으로 참조되는 해당 page들의 집합을 locality set이라고 함

- **Working Set Model**

  - Locality에 기반하여 **프로세스가 일정 시간동안 원활하게 수행되기 위해 한꺼번에 메모리에 올라와 있어야 하는 page들의 집합**을 **Working Set**이라고 함

  - Working set model은 working set 개념을 기반으로 메모리를 관리하는 기법을 의미

    - Working set model에서는 **프로세스의 working set 전체가 메모리에 올라와 있어야 수행**되고,

      그렇지 않으면, 모든 frame을 반납한 후 swap out(suspend)

  - Thrashing을 방지함

  - Multiprogramming Degree를 조절함



##### Working Set Algorithm

- Working set의 결정

  - **Working set window** (어떤 프로그램의 working set이 추적/기록되는 fixed time interval)를 통해 알아냄

  - window size가 Δ인 경우

    - **시각 t i 에서의 working set WS(t i)**

      - Time interval [t i - Δ, t i] 사이에 참조된 서로 다른 페이지들의 집합

    - Working set에 속한 페이지는 메모리에 유지, 속하지 않은 것은 버림

      (즉, 참조된 후 Δ 시간 동안 해당 page를 메모리에 유지한 후 버림)

  ![image-20230313214656681](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230313214656681.png) 

- 단점

  - page fault가 없어도 항상 working set을 관리해야 하므로 overhead 발생



#### b-2. PFF (Page Fault Frequency) Algorithm

- 프로세스의 Page Fault Rate를 주기적으로 조사하고 이 값에 근거해서 각 프로세스에 할당할 메모리 양을 동적으로 조절하는 알고리즘

![image-20230313220910053](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230313220910053.png) 

- Page fault rate의 상한값과 하한값을 둔다
  - 상한값을 넘으면 frame을 더 할당
  - 하한값 이하이면 할당 frame 수를 줄인다
- frame을 더 할당해야 하는데 빈 frame이 없으면 일부 프로세스를 swap out



## Other Considerations<sup>[5](#footnote_5)</sup>

### a. Page Size

- Page Size는 적당한 것이 좋음

  | Page size가 작은 경우                                        | Page size가 큰 경우                                          |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | page 수가 많고, page table 크기가 큼 <br />-> 관리하는 데 드는 kernel overhead 큼 | page 수가 적고, page table 크기가 작음<br />-> 관리하는 데 드는 kernel overhead 작음 |
  | 내부 단편화 감소                                             | 내부 단편화 증가                                             |
  | I/O 시간 증가<br />: 많은 page를 읽기 때문                   | I/O 시간 감소<br />: 적은 page를 읽기 때문                   |
  | Locality 향상<br />: 주로 사용할 영역만을 가져올 수 있기 때문에 | Locality 저하<br />: 사용하지 않을 영역도 많이 포함해서 가져올 수 있기 때문에 |
  | page fault 증가                                              | page fault 감소                                              |

- page size가 점점 커지는 경향이 있음

  - 다음과 같은 하드웨어 발전 경향으로 인해 page fault의 처리 비용이 증가했기 때문

    - Memory Size 증가 : Page Size가 상대적으로 너무 작으면 page 수가 많아져 kernel의 부담이 증가할 수 있기 때문

    - disk에 비해 CPU가 더 빠르게 발전(속도 측면) : 속도 차이로 인해 I/O 병목 현상이 발생할 확률 증가

      ​																				-> I/O 시간 감소시키기 위해 page size ↑

### b. Program Restructuring

- 사용자가 가상 메모리 관리 기법을 이해하고 있다면 그 특성에 맞도록 프로그램을 재구성하여 성능을 높일 수 있음

- 예시

  - Paging system, Page size = 1KB

  - 자료형이 int인 데이터의 크기 4 Byte

    256개의 int 값으로 이루어진 하나의 행(row)의 크기는 4 * 256 = 1KB

    ```c
    //program 1
    int main() {
        int zar[256][256];
        int i, j;
        
        for (j = 0; j < 256; j++)
            for (i = 0; i < 256; i++)
                zar[i][j] = 0;
        
        return 0;
    }
    ```

    ![img](https://blog.kakaocdn.net/dn/o2EqN/btqPZJBCwJM/iE9kPKjSJvz7F1WkClw2Q0/img.png) 

    - program 1처럼 코드를 작성하면 행 단위로 연속해서 메모리에 올라가는 c언어의 특성상

      내부 반복문을 순회하면서 i값이 바뀔 때마다 행이 바뀌므로

      매번 page fault 발생

    ```c
    //program 2
    int main() {
        int zar[256][256];
        int i, j;
        
        for (i = 0; i < 256; i++)
            for (j = 0; j < 256; j++)
                zar[i][j] = 0;
        
        return 0;
    }
    ```

    ![img](https://blog.kakaocdn.net/dn/HUJcE/btqPWZykomU/5YL9tvWtME099roODZ7oJ1/img.png) 

    - program 2처럼 코드를 작성하면, 

      외부 반복문에서 i값이 바뀌고 행을 이동할 때만 page fault가 발생

### c. TLB Reach

- TLB가 닿는(reach) 범위

  = TLB(Translation Lookaside Buffer)를 통해 접근할 수 있는 메모리의 양을 의미

  = (TLB를 구성하는 entry의 수) * (page의 크기)

  - 각각의 entry는 하나의 page를 가리킴

- TLB의 hit ratio를 높이려면

  - TLB의 크기 증가

    - 그러나 TLB는 비쌈

  - Page의 크기 증가 or 다양한 page size 지원

    - OS의 지원이 필요 : 최근 OS의 발전 경향

      

## 참고

<a name="footnote_1">1</a>. Page Fault란?

http://web.mit.edu/rhel-doc/4/RH-DOCS/rhel-isa-ko-4/s1-memory-virt-details.html

<a name="footnote_2">2</a>. Heap이란?

https://gmlwjd9405.github.io/2018/05/10/data-structure-heap.html

<a name="footnote_3">3</a>. Degree of Multiprogramming과 CPU utilization의 관계

http://denninginstitute.com/modules/vm/green/multip.html

<a name="footnote_4">4</a>. 참조 지역성(Locality of reference)이란?

https://anyflow.net/311

<a name="footnote_5">5</a>. 가상 메모리 관리할 때 고려할 기타 요소(Other considerations)

https://www.youtube.com/watch?v=_QpNwu_MYck&list=PLBrGAFAIyf5rby7QylRc6JxU5lzQ9c4tN&index=37

https://conkjh032.tistory.com/120