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

#### b-1. 목표와 평가 기준

- **page-fault rate을 최소화**하는 것이 목표

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

- **가장 오래 전에 참조된 것**을 지움

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

- 참조 횟수(reference count)가 가장 적은 페이지를 지움

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

  - Heap<sup>[2](#footnote_2)</sup>

    - 완전 이진 트리(complete binary tree)의 형태

    - 부모 노드의 frequency <= 자식 노드의 frequency

      - 자식 노드 두 개외 비교해 참조 횟수가 크다면 자리를 바꿔 아랫쪽으로 이동

    - 시간 복잡도가 O(log n)

      - 최악의 경우, 트리의 높이(log 2 n)만큼의 노드와 비교

      ![image-20230312225622113](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230312225622113.png) 

## 참고

<a name="footnote_1">1</a>. Page Fault란?

http://web.mit.edu/rhel-doc/4/RH-DOCS/rhel-isa-ko-4/s1-memory-virt-details.html

<a name="footnote_2">2</a>. Heap이란?

https://gmlwjd9405.github.io/2018/05/10/data-structure-heap.html