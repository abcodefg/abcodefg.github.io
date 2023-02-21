---
title:  "[운영체제] Process Management"
excerpt: "프로세스 생성, 프로세스 종료, 프로세스와 관련된 시스템콜, 프로세스 간 협력"

categories:
  - OS
tags:
  - [OS]
  
toc: true
toc_sticky: true
 
date: 2023-01-22T13:10:22-04:00
last_modified_at: 2023-01-22T13:10:22-04:00
---

# Process Management

> 이 글은 이화여자대학교 반효경 교수님의 '운영체제' 강의(2014년 1학기)를 학습한 내용을 정리한 것입니다.

## 프로세스 생성(Process Creation)

- 부모 프로세스(Parent process)가 자식 프로세스(children process) 생성
- 프로세스의 트리(계층 구조) 생성
- 프로세스는 자원을 필요로 함
  - 운영체제로부터 받는다.
  - 부모와 공유한다.
- 자원의 공유
  - 부모와 자식이 모든 자원을 공유하는 모델
  - 일부를 공유하는 모델
  - 전혀 공유하지 않는 모델 - 일반적
- 수행(Execution)
  - 부모와 자식은 공존하며 수행되는 모델
  - 자식이 종료(terminate)될 때까지 부모가 기다리는(wait = blocked) 모델



- 주소 공간(Address space)

  - 자식은 부모의 공간을 복사함(binary and OS data)
  - 자식은 그 공간에 새로운 프로그램을 올림(덮어씌움)

- 유닉스의 예 (프로세스 생성의 2가지 단계)<sup>[1](#footnote_1)</sup>

  - **fork()** 시스템 콜이 새로운 프로세스(PID가 다름)를 생성

    - 새로운 프로세스를 위한 메모리 할당

    - 부모(=fork를 호출한 프로세스)를 그대로 복사(OS data except PID + binary)

    - 주소 공간 할당

      *참고: **Copy-On-Write(COW)**<sup>[2](#footnote_2)</sup>`

      <img src="https://blog.kakaocdn.net/dn/kLZGX/btrDUU2o2TQ/lqFsFX7DKyQ7SsfrhaIOk0/img.png" alt="img" style="zoom: 50%;" />

      <img src="https://blog.kakaocdn.net/dn/df2wMN/btrDV08plX5/m98FQQ4TNbHkdkuXHK6RPK/img.png" alt="img" style="zoom:50%;" />

      	- 부모 프로세스를 복사할 때 생기는 메모리 overhead 문제를 해결하는데 사용되는 자원 관리 기법

       - 자식 프로세스를 생성하면 부모 프로세스의 주소 공간을 공유하고, 운영체제는 자식 프로세스에게 독립적인 page table을 만들어준다.
         - 이때, 부모 page table의 read-write bit를 read only로 초기화해서 복사하고 만든다.
         - 자식 프로세스가 공유된 page에 write 작업을 시도하면, 해당 page를 복사해 메모리에 새로 올리는데, 보안상의 이유로 page는 0으로 초기화하여 복사한다.

  - fork 다음에 이어지는 **exec()** 시스템 콜을 통해 새로운 프로그램을 메모리에 올림

    - 새로운 프로세스를 위한 별도의 메모리를 할당하지 않음
    - exec()을 호출한 프로세스는 새로운 프로세스에 의해 덮어 쓰여짐 (=exec() 을 호출한 프로세스가 아닌 exec() 에 의해 호출된 프로세스만 메모리에 남음)
    - fork와 독립적이기 때문에 복제만 하고 덮어씌우지 않거나 복사하지 않고 덮어씌우기만 할 수도 있음

  - 사용자 프로세스가 다른 프로세스를 생성하는 게 아니라 **운영체제를 통해서 프로세스 생성** 가능

## 프로세스 종료(Process Termination)

- 프로세스가 마지막 명령을 수행한 후 운영체제에게 이를 알려줌 (**exit**)
  - 자식이 부모에게 output data를 보냄 (via **wait**)
  - 프로세스의 각종 자원들이 운영체제에게 반납됨
- 부모 프로세스가 자식의 수행을 종료시키는 경우 (**abort**)
  - 자식이 할당 자원의 한계치를 넘어섬
  - 자식에게 할당된 태스크가 더 이상 필요하지 않음
  - 부모가 종료(exit)하는 경우
    - 운영체제는 부모 프로세스가 종료하는 경우 자식이 더 이상 수행되도록 두지 않는다.
    - 단계적인 종료 (... -> 자식의 자식 -> 자식 -> 부모)

## 프로세스와 관련한 시스템 콜

### 개요

- **fork()** : create a child(copy)
- **exec()** : overlay new image
- **wait()** : sleep until child is done
- **exit()** : frees all the resources, notify parent



### fork() 시스템 콜

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230204153218988.png" alt="image-20230204153218988" style="zoom:67%;" />

- 프로세스는 fork() 시스템 콜을 통해 생성된다.
  - creates a new address space that is a duplicate of the caller 
  - 부모 프로세스의 program counter 역시 복사하기 때문에, fork()가 이루어진 다음 시점부터 자식 프로세스가 실행하게 된다(프로세스 재귀적 무한 생성 x).
    - program counter가 부모 프로세스가 실행한 다음 부분을 가리키고 있는 동일한 형태의 다른 함수가 생성된다 생각하면 이해하기 쉽다.
  - 부모와 자식을 구분하기 위해 부모의 경우 fork()의 결과값이 양수, 자식의 경우 결과값이 0이 된다.

 

### exec() 시스템 콜

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230204153830506.png" alt="image-20230204153830506" style="zoom:67%;"  />

- 프로세스는 exec() 시스템 콜을 통해 다른 프로그램을 실행할 수 있다.
  - 어떤 프로그램을 새로운 프로세스로 태어나게 한다.
  - replaces the memory image of the caller with a new program
- 위의 예시에서는 부모 프로세스에 의해 생성된 자식 프로세스가 printf 명령을 수행한 후 새로운 프로세스
  - execlp() 는 현재 프로세스를 대신하여 새로운 프로세스를 실행하는 함수<sup>[3](#footnote_3)</sup>



### wait() 시스템 콜

- 프로세스 A가 wait() 시스템 콜을 호출하면
  - 커널은 child가 종료될 때까지 프로세스 A를 sleep시킨다 (block 상태)
  - Child process가 종료되면 커널은 프로세스 A를 깨운다 (ready 상태)



### exit() 시스템 콜

- 프로세스의 종료
  - 자발적 종료
    - 마지막 statement 수행 후 exit() 시스템 콜을 통해
    - 프로그램에 명시적으로 적어주지 않아도 main 함수가 리턴되는 위치에 컴파일러가 넣어줌
  - 비자발적 종료
    - 부모 프로세스가 자식 프로세스를 강제 종료시킴
      - 자식 프로세스가 한계치를 넘어서는 자원 요청
      - 자식에게 할당된 태스크가 더 이상 필요하지 않음
    - 키보드로 kill, break 등을 친 경우
    - 부모가 종료하는 경우
      - 부모 프로세스가 종료하기 전에 자식들이 먼저 종료됨



## 프로세스 간 협력

- **독립적 프로세스(Independent process)**

  - 프로세스는 각자의 주소 공간을 가지고 수행되므로, 원칙적으로 하나의 프로세스는 다른 프로세스의 수행에 영향을 미치지 못함

- **협력 프로세스(Cooperating process)**

  - 경우에 따라서는 프로세스 간에 정보를 주고받아야 더 효율적으로 실행됨
  - 프로세스 협력 매커니즘을 통해 하나의 프로세스가 다른 프로세스의 수행에 영향을 미칠 수 있음

- **프로세스 간 협력 매커니즘(IPC: Interprocess Communication)**

  - 메시지를 전달하는 방법

    - **Message passing**: 커널을 통해 메시지 전달

      - **Message system**

        - 프로세스 사이에 공유 변수(shared variable)를 일체 사용하지 않고 통신하는 시스템
        - message passing의 방법은 아래 두 가지

      - **Direct Communication**

        - 통신하려는 프로세스의 이름을 명시적으로 표시

          <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205002021917.png" alt="image-20230205002021917" style="zoom:50%;" />

      - **Indirect Communication**

        - mailbox(또는 port)를 통해 메시지를 간접 전달

        - 꺼내가는 게 누구인지 명시하지는 않음

          <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205002111120.png" alt="image-20230205002111120" style="zoom:50%;" />

  - 주소 공간을 공유하는 방법

    - **shared memory**: 서로 다른 프로세스 간에도 일부 주소 공간을 공유하게 하는 매커니즘

      - 예를 들어, process A가 shared memory에 무언가를 적으면 process B 역시 이를 볼 수 있다.
      - 먼저 kernel에 shared memory를 사용하기 위한 시스템 콜을 하여 매핑해야 한다.
      - shared memory를 쓰려는 두 프로세스는 신뢰할 수 있는 관계여야 한다.

      <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230205002558577.png" alt="image-20230205002558577" style="zoom:50%;" />

      ***참고) thread 간의 협력**: thread는 사실상 하나의 프로세스이므로 프로세스 간 협력으로 보기는 어렵지만, 동일한 process를 구성하는 thread들 간에는 주소 공간을 공유하므로 협력이 가능

      

## 참고

<a name="footnote_1">1</a>. fork()와 exec()

https://woochan-autobiography.tistory.com/207

<a name="footnote_2">2</a>. Copy-on-write

https://en.wikipedia.org/wiki/Copy-on-write

https://wpaud16.tistory.com/290

<a name="footnote_3">3</a>. execlp()

https://bubble-dev.tistory.com/entry/CC-execl3-execv3-execle3-execve2-execlp3-execvp3