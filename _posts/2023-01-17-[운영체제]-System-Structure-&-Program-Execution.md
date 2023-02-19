---
title:  "[운영체제] System Structure & Program Execution"
excerpt: "컴퓨터의 시스템 구조(CPU, 메모리, 입출력 장치 등), 동기식/비동기식 입출력, 시스템 콜, 인터럽트, 저장장치 계층 구조, 프로그램의 실행(메모리 적재), 커널 주소 공간의 내용, 사용자 프로그램이 사용하는 함수"

categories:
  - OS
tags:
  - [OS]
  
toc: true
toc_sticky: true
 
date: 2023-01-17T12:48:05-04:00
last_modified_at: 2023-01-17T12:48:05-04:00
---

# System Structure & Program Execution

> 이 글은 이화여자대학교 반효경 교수님의 '운영체제' 강의(2014년 1학기)를 학습한 내용을 정리한 것입니다.

##  컴퓨터 시스템 구조와 기능

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230129175415551.png" alt="image-20230129175415551" style="zoom:67%;" />

- **Computer**

  - **CPU**

    - 기능

      - 매 클럭 사이클마다 Memory에서 Instruction을 하나씩 읽어서 실행한다.<sup>[1](#footnote_1)</sup>

    - 구성 요소

      - **Interrupt line**<sup>[2](#footnote_2)</sup>

        - **인터럽트(Interrupt)**란 CPU가 프로그램을 실행중일 때, I/O device 등의 장치에 예외상황이 발생해 처리가 필요한 경우에 CPU에 알려 처리할 수 있도록 하는 것이다.
          - 인터럽트가 들어오면, 인터럽트 당한 시점의 레지스터와 program counter를 save한 후 CPU의 제어를 인터럽트 처리 루틴에 넘긴다.
          - 인터럽트(넓은 의미)는 하드웨어 인터럽트와 소프트웨어 인터럽트로 나뉜다.<sup>[3](#footnote_3)</sup>
            - **하드웨어 인터럽트**는 키보드, 마우스와 같은 하드웨어가 발생시킨 인터럽트를 말한다. 일반적으로 인터럽트는 하드웨어 인터럽트를 지칭한다.
            - **소프트웨어 인터럽트(Trap)**는 프로그램이 오류를 범하여 **Exception**이 발생하거나 사용자 프로그램이 커널함수 사용을 위해 호출하는 **System Call**이 발생하는 경우를 말한다.
          - 인터럽트 관련 용어
            - **인터럽트 벡터**
              - 인터럽트는 여러가지가 있으며 종류별로 해야 하는 일이 다르다.
                - 예를 들어, 키보드 컨트롤러로부터 인터럽트가 들어오면 키보드 buffer에 들어온 내용을 메모리에 copy하고 키보드 I/O를 요청한 프로세스가 CPU를 얻을 수 있다는 걸 표시하고, timer로부터 인터럽트가 들어오면 CPU를 뺏어서 다른 프로세스에 준다.
              - 해당 인터럽트의 처리 루틴 주소를 가지고 있으며, 어떤 인터럽트가 들어왔을 때 어떤 함수를 실행해야 하는지 정리해놓은 테이블이다.
            - **인터럽트 처리 루틴**(Interrupt Service Routine, 인터럽트 핸들러)
              - 해당 인터럽트를 처리하는 커널  함수를 인터럽트 처리 루틴이라고 부른다. 
          - 현대의 운영체제는 인터럽트에 의해 구동된다.
            - 인터럽트가 들어올 때만 CPU가 운영체제에 넘어가며 그렇지 않으면 CPU는 사용자 프로그램이 쓴다.
        - 인터럽트의 발생은 Interrupt Line이 감지하는데, CPU는 명령을 수행하고 나서 다음 명령을 수행하기 직전에 매번 Interrupt Line이 세팅되었는지 확인한다. 

      - **mode bit**

        - CPU에서 실행되는 것이 운영체제인지 사용자 프로그램인지 구분해준다.

        - 사용자 프로그램의 잘못된 수행으로 다른 프로그램 및 운영체제에 피해가 가지 않도록 하기 위한 보호장치

        - mode bit을 통해 하드웨어적으로 두 가지 모드의 **operation**을 지원한다.

          **1**  **사용자 모드**: 사용자 프로그램 수행

          **0**  **모니터 모드**(=**커널 모드**, **시스템 모드**): OS 코드 수행

          - 보안을 해칠 수 있는 중요한 명령어(ex_ I/O device 접근)는 모니터 모드에서만 수행 가능한 '**특권명령**'으로 규정
          - Interrupt나 Exception 발생시 하드웨어가 mode bit을 0으로 바꿈
          - 사용자 프로그램에게 CPU를 넘기기 전에 mode bit을 1로 세팅
          - mode bit이 0일 때에는 모든 instruction을 수행할 수 있지만, 1일 때는 한정된 instruction만 수행 가능

      - **register**

        - Memory보다 빠르면서 정보를 저장할 수 있는 작은 공간

  - **Memory**

    - CPU의 작업 공간
    - CPU에서 매 클럭 사이클마다 Memory에서 기계어를 읽어서 실행

  - **timer**

    - 특정 프로그램이 CPU를 독점하는 것을 막는다.
      -  처음 부팅 시에는 운영체제가 CPU를 가지고 있다가 사용자 프로그램에 CPU를 넘겨준다.
      -  이때, 사용자 프로그램이 CPU를 독점하지 않게끔 timer에 시간을 설정한다.
      -  시간이 다 되면 timer가 interrupt를 걸고, CPU의 제어권이 사용자 프로그램에서 운영체제로 넘어간다.

  - **DMA(Direct Memory Access) controller**

    - 잦은 Interrupt로 인해 CPU가 방해받는 것을 막기 위해 I/O device에 접근하는 작업이 끝나면 CPU 대신 local buffer의 데이터를 직접 Memory에 복사하고 interrupt를 한 번만 걸어 CPU에 이를 알린다.
    - **DMA(Direct Message Access)**란?<sup>[4](#footnote_4)</sup>
      - 하드디스크, 그래픽 카드 등 주변장치들이 CPU의 개입 없이 메모리에 직접 접근하여 읽거나 쓸 수 있도록 하는 기능
      - 빠른 입출력 장치(인터럽트가 빈번하게 발생한다)를 메모리에 가까운 속도로 처리하기 위해 사용한다.
      - CPU의 중재 없이 device controller가 device의 buffer storage의 내용을 메모리에 block 단위로 직접 전송
      - 바이트 단위가 아니라 block 단위로 인터럽트를 발생시킴

  - **memory controller**

    - Memory로부터 들어오고 나가는 데이터의 흐름을 관리하는 장치

    

- **I/O device**

  - **Input**: device -> computer, **Output**: computer -> device

  - **Disk**

    - Disk는 보조기억장치라고도 하지만 Disk의 데이터를 메모리로 읽어들이거나 처리 결과를 저장하기도 한다는 면에서 I/O device로 볼 수 있음

  - **device controller**

    - 각각의 I/O device 유형을 전담하여 관리하는 일종의 작은 CPU
    - 제어 정보를 위해 각각 명령/상태를 저장하는 control register, status register를 가진다.
      - 예를 들어, 어떤 데이터를 파일에 저장하는 instruction을 실행할 때 데이터는 local buffer에 넣고 데이터를 저장하라는 명령은 control register를 통해 CPU가 device controller에 전달한다.
    - 참고
      - **device driver**(장치구동기): OS 코드 중 각 장치별 처리 루틴 -> software (반면, device controller는 hardware)
        - 단, device driver는 운영체제(혹은 다른 프로그램)가 장치와 소통할 수 있게끔 하는 소프트웨어일 뿐, 실질적으로 장치에 내장되어 이를 동작하게 하는 소프트웨어는 **firmware**이다.<sup>[5](#footnote_5)</sup>

  - **local buffer**

    - device controller의 작업 공간
    - 일종의 data register
    - I/O는 실제 device와 local buffer 사이에서 일어난다.

  - 입출력(I/O)의 수행

    - 모든 입출력 명령은 특권 명령

    - 사용자 프로그램은 어떻게 I/O를 하는가?

      - **시스템 콜(System call**)
        - 사용자 프로그램이 운영체제의 서비스를 받기 위해 커널 함수를 호출하는 것
        - 일반적으로 프로그램 내에서 main과 같은 여러 함수를 호출할 때에는 메모리 주소를 변경한다.
          - 반면 I/O를 수행하기 위해서는 운영체제에 해당하는 주소로 넘어가야 하는데 memory bit가 1이므로 이는 불가능하다.
          - 대신 사용자 프로그램은 CPU에 인터럽트를 걸고 OS에 의해 I/O가 수행되게끔 한다.
        - 소프트웨어 인터럽트에 해당한다.
      - trap을 사용하여 인터럽트 벡터의 특정 위치로 이동
      - 제어권이 인터럽트 벡터가 가리키는 인터럽트 서비스 루틴으로 이동
      - 올바른 I/O 요청인지 확인 후 I/O 수행
      - I/O 완료시 제어권을 시스템콜 다음 명령으로 옮김

    - 과정

      1. I/O device에 접근하는 모든 Instruction(ex_ Disk에서 데이터를 읽어오기, 모니터 출력)은 보안 등의 이유로 사용자 프로그램이 아닌 오직 운영체제를 통해서만 실행되게끔 되어있으므로 해당 Instruction을 포함하는 사용자 프로그램은 즉시 CPU 제어권을 운영체제에 넘겨준다.

      2. 운영체제는 해당 작업을 device controller에 맡기고 다른 사용자 프로그램에 제어권을 넘긴다.

      3. 요청한 작업이 끝나 데이터가 local buffer에 들어오면 controller가 interrupt를 건다.

      4. interrupt를 걸면 운영체제가 제어권을 넘겨받아 데이터를 해당 사용자 프로그램의 메모리 공간에 copy해주고 직전에 interrupt된 프로그램에 제어권을 넘겨준다.

## 동기식 입출력과 비동기식 입출력

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230129224617980.png" alt="image-20230129224617980" style="zoom:67%;" />

- **동기식 입출력(synchronous I/O)**
  - I/O 요청 후 입출력 작업이 완료된 후에야 제어가 사용자 프로그램에 넘어감
  - 구현 방법 1
    - I/O가 끝날 때까지 기다리면서 CPU를 낭비시킴
    - 매시점 하나의 I/O만 일어날 수 있음
  - 구현 방법 2
    - I/O가 완료될 때까지 해당 프로그램에게서 CPU를 빼앗음
    - I/O 처리를 기다리는 줄에 그 프로그램을 줄 세움
    - 다른 프로그램에게 CPU를 줌
- **비동기식 입출력(asynchronous I/O)**
  - I/O가 시작된 후 입출력 작업이 끝나기를 기다리지 않고 제어가 사용자 프로그램에 즉시 넘어감
- 두 경우 모두 I/O의 완료는 인터럽트로 알려줌

## 서로 다른 입출력 명령어

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230130013104259.png" alt="image-20230130013104259" style="zoom:67%;" />

- CPU가 I/O 기기에 접속하는 방법은 CPU가 I/O 포트가 사용하는 I/O 주소와 Memory가 사용하는 주소를 분리하는지에 따라 다음과 같이 분류된다.<sup>[6](#footnote_6)</sup>
  - **I/O Mapped I/O**
    - I/O와 Memory의 주소 공간을 분리한다.
    - I/O를 수행하는 special instruction이 필요하다.
  - **Memory Mapped I/O**
    - 메모리의 일부 공간을 I/O 포트에 할당한다.
    - 많은 프로세스가 하나의 파일을 메모리에서 공유하는 것이 가능해진다.

## 저장장치 계층 구조(Hierarchy of Storage)<sup>[8](#footnote_8)</sup>

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230130013249296.png" alt="image-20230130013249296" style="zoom:67%;" />

|              | Primary Storage                                              | Second Storage                                               |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Speed & Cost | 위로 갈수록 속도가 빠르고 단위 공간 당 가격이 비싸서 용량이 적다. | 아래로 갈수록 속도가 느리고 단위 공간 당 가격이 싸서 용량이 크다. |
| Volatility   | 전원이 꺼지면 내용이 사라지는 휘발성 매체                    | 전원이 꺼져도 내용이 사라지지 않는 비휘발성 매체             |
| CPU 접근     | byte 단위로 접근할 수 있으므로 CPU가 접근할 수 있다(executable) | HDD의 경우 byte가 아닌 섹터 단위로 CPU가 접근할 수 없다      |

- **Caching**: copying information into faster storage system(Disk -> Main Memory -> Cache Memory)<sup>[9](#footnote_9)</sup>
  - **Cache Memory**: 주기억장치와 CPU 사이에 위치하고 자주 사용되는 데이터들을 기억하는 메모리. 컴퓨터의 성능을 향상시키기 위해 사용한다.
  - 캐싱(Caching)이란 캐시 영역으로 데이터를 가져와서 접근하는 방식을 말한다.

## 프로그램의 실행(메모리 load)

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230130203430930.png" alt="image-20230130203430930" style="zoom:67%;" />

- 실행파일을 실행시키면 메모리에 올라가 프로세스가 된다. 

  - 파일을 실행하면 해당 프로그램만의 주소 공간(Address space)이 생긴다.

    - 주소 공간은 code, data, stack으로 구성되어 있다.

      - code: CPU에서 실행할 프로그램의 기계어 코드를 포함
      - data: 변수 등 프로그램이 사용하는 자료구조를 포함
      - stack: 함수를 호출하거나 반환할 때 데이터를 쌓거나 꺼내는 용도로 사용

    - 메모리의 낭비를 막기 위해, Physical memory에는 주소 공간 중에서 당장 필요한 부분만을 올린다.

    - 또한, 필요하지 않은 부분을 Disk의 Swap area에 내려 놓는다.<sup>[10](#footnote_10)</sup>

      *주의)  동일한 Disk지만 Swap area는 Memory의 연장 공간의 용도로 사용되며 메모리의 휘발성 때문에 전원이 꺼지면 의미가 없어지는 반면,

      ​			 File system은 비휘발성의 용도로 사용된다.

    - Address translation: 프로세스마다 0부터 시작하는 Virtual Memory의 주소를 Physical Memory의 주소로 변환하는 것

## 커널 주소 공간의 내용

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230130205649965.png" alt="image-20230130205649965" style="zoom:67%;" />

- code
  - 운영체제의 역할과 관련된 코드들
- data
  - 운영체제가 사용하는 자료구조들이 정의되어 있음
    - CPU, memory, disk 등의 하드웨어를 관리하기 위한 자료구조
    - PCB(Process Control Block): 실행중인 프로그램을 관리하기 위한 자료구조
- stack
  - 운영체제는 함수구조로 코드가 짜여져 있으므로 함수를 호출하거나 반환할 때 스택 영역을 사용
  - 운영체제의 코드는 여러 사용자 프로그램들이 시스템 콜을 통해 불러서 사용할 수 있는데, 어떤 사용자 프로그램이 커널 코드를 실행중인지에 따라서 커널 스택을 개별적으로 둔다.

## 사용자 프로그램이 사용하는 함수

- 함수(function)

  - **사용자 정의 함수**

    - 자신의 프로그램에서 정의한 함수

  - **라이브러리 함수**

    - 자신의 프로그램에서 정의하지 않고 갖다 쓴 함수
    - 자신의 프로그램의 실행 파일에 포함되어 있다.

  - **커널 함수**

    - 운영체제 프로그램의 함수

    - 커널 함수의 호출 = 시스템 콜

      - 메모리 주소의 변경(jump)는 Virtual memory 주소 상에서의 jump를 의미 

        -> 어떤 프로세스의 메모리 주소에서 커널영역의 메모리 주소로의 jump는 불가능

        -> 시스템 콜을 통해 제어권을 운영체제로 넘겨서 커널 함수를 실행

## 프로그램의 실행

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230130220536344.png" alt="image-20230130220536344" style="zoom:67%;" />



## 참고

<a name="footnote_1">1</a>. 컴퓨터가 코드를 읽는 아주 구체적인 원리(Instruction, 기계어 관련)

https://parksb.github.io/article/25.html

<a name="footnote_2">2</a>. 인터럽트(Interrupt)와 인터럽트 핸들링

https://primayy.tistory.com/43

https://zangzangs.tistory.com/106

https://mikiplace.tistory.com/11

<a name="footnote_3">3</a>. 하드웨어 인터럽트와 소프트웨어 인터럽트의 차이

https://www.geeksforgeeks.org/difference-between-hardware-interrupt-and-software-interrupt/

<a name="footnote_4">4</a>. DMA(Direct Memory Access)란? (+PIO, 채널제어방식)

https://kkhipp.tistory.com/168

<a name="footnote_5">5</a>. device driver와 firmware의 차이

http://www.differencebetween.net/technology/difference-between-device-driver-and-firmware/

<a name="footnote_6">6</a>. I/O Mapped I/O와 Memory Mapped I/O

https://do-rang.tistory.com/76

<a name="footnote_7">7</a>. 주기억장치와 보조기억장치

https://www.naukri.com/learning/articles/difference-between-primary-memory-and-secondary-memory/

<a name="footnote_8">8</a>. 저장장치 계층 구조

https://en.wikipedia.org/wiki/Computer_data_storage#Hierarchy_of_storage

<a name="footnote_9">9</a>. 캐시와 캐싱

https://velog.io/@effypark/%EC%BA%90%EC%8B%9C-%EC%BA%90%EC%8B%B1%EC%9D%B4%EB%9E%80

https://opentutorials.org/course/697/3839

<a name="footnote_10">10</a>. swap과 swap area(=swap space)

https://hyeo-noo.tistory.com/101

https://zangzangs.tistory.com/132