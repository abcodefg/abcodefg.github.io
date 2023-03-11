---
title:  "[운영체제] Memory Management"
excerpt: "Logical/Physical Address, 주소 바인딩, MMU(Memory Management Unit), Dynamic Relocation, Hadware Support for Address Translation, Some Treminologies, Dynamic Loading, Overlays, Swapping, Dynamic Linking, Allocation of Physical Memory, Contiguous Allocation, Paging"

categories:
  - OS
tags:
  - [OS]
  
toc: true
toc_sticky: true
 
date: 2023-02-08T13:18:17-04:00
last_modified_at: 2023-02-08T13:18:17-04:00
---

# Memory  Management

## Logical Address와 Physical Address

- **Logical Address** ( = virtual address, 논리적 주소)

  - 실행을 위해 메모리에 적재된 프로세스마다 독립적으로 가지는 주소 공간
  - 각 프로세스마다 0번지부터 시작
  - CPU가 보는 주소는 logical address임
    - CPU가 특정 logical address의 데이터를 요청 -> physical address로 변환 -> 그 주소의 데이터를 읽어들여 CPU에 전달

- **Physical Address** ( = 물리적 주소)

  - 메모리에 실제 올라가는 위치

- 주소 바인딩

  - CPU가 프로세스의 작업을 수행할 때, 논리적 주소를 통해 실제 메모리의 물리적 주소를 알 수 있게끔

    두 주소를 연결하는 작업

  - Symbolic Address -> Logical Address -> Physical Address

    *Symbolic Address : 변수, 함수의 이름과 같이 프로그래머가 사용하는 Symbol로 구성된 주소

## 주소 바인딩(Address Binding)

### a. 주소 바인딩의 종류

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230213173442461.png" alt="image-20230213173442461" style="zoom: 50%;" /> 

- **Compile time binding**

  - physical address가 컴파일시 알려짐

  - 시작 위치 변경시 재컴파일 (run time에 변경 불가능)

  - 컴파일러는 절대 코드(absolute code) 생성

    - **절대 코드** : 어셈블리어가 정한 물리적 메모리 주소에 고정되어 위치되는 코드

  - 컴파일 타임에 결정된 주소를 물리적 주소로 설정해야 하기 때문에 비효율적

    -> 과거 컴퓨터에서 프로그램이 하나만 실행되던 환경에서는 사용되기도 했으나,

    ​	 지금의 컴퓨터 시스템에서는 잘 활용하지 않음

- **Load time binding**

  - Loader의 책임하에 물리적 메모리 주소 부여
  - 컴파일러가 재배치가능 코드(relocatable code)를 생성한 경우 가능

- **Execution time binding (=Run time binding)**

  - 수행이 시작된 이후에도 프로세스의 메모리 상 위치를 옮길 수 있음 (실행 시에 물리적 주소가 결정된다는 점은 Load time binding과 동일)

  - 따라서, CPU가 주소를 참조할 때마다 binding을 점검해야 함 (address mapping table)

    -> 하드웨어적인 지원이 필요 (ex_ base and limit registers, **MMU**)

  - 지금의 컴퓨터 시스템에서 주로 활용

### b. Memory Management Unit (MMU)

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230213194905997.png" alt="image-20230213194905997" style="zoom: 50%;" /> 

- **Memory Management Unit** : CPU 코어 안에 탑재되어 **Logical Address를 Physical Address로 매핑**해 주는 **Hardware** device

- MMU scheme : 사용자 프로세스가 CPU에서 수행되며 생성해내는 모든 주소값에 대해 relocation register의 값을 더한다.

  - **Relocation register (=base register)** : 접근할 수 있는 물리적 메모리의 최솟값 (프로세스의 시작 위치가 저장되어 있음)
  - **Limit register** : 논리적 주소의 범위 (프로그램의 크기가 저장되어 있음)

- MMU의 **memory protection**

  <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230213200324999.png" alt="image-20230213200324999" style="zoom:50%;" /> 

  - **Limit register**는 **접근 가능한 주소의 상한**선을 규정한다.

    이를 통해 어떤 프로세스가 운영체제 혹은 다른 프로세스의 메모리를 침범하지 않도록 막는다 (=**메모리를 보호**한다).

  - 요청한 logical address의 값이 limit register의 값( = 프로그램의 크기)보다 크다면 trap이 걸린다 (**segmentation fault** 발생).<sup>[1](#footnote_1)</sup>

    -> 그 결과 CPU 제어권을 넘겨받은 운영체제는 프로그램을 강제로 abort 시키고 해당 프로그램이 사용중이던 자원을 release한다.

## Allocation of Physical Memory

- 메모리는 일반적으로 **OS 상주 영역** (낮은 주소 영역 사용)과 **사용자 프로세스 영역** (높은 주소 영역 사용), 두 영역으로 나뉘어 사용
- 사용자 프로세스 영역의 할당 방법
  - **Contiguous Allocation**
    - 각각의 프로세스가 메모리의 연속적인 공간에 적재되도록 하는 것 ( = 한 영역에 통째로 올라가는 것)
    - 종류
      - **Fixed partition allocation (고정 분할 방식)**
      - **Variable partition allocation (가변 분할 방식)**
  - **Non-contiguous Allocation (불연속 할당)**
    - 하나의  프로세스가 메모리의 여러 영역에 분산되어 올라갈 수 있는 것
    - 종류
      - **Paging**: Logical/Physical Memory를 고정된 크기의 블록인 Page/Frame으로 나눔
      - **Segmentation**: Logical Memory를 가변적 크기의 논리적 단위로 나눔
      - **Paged segmentation**: Memory를 논리적 단위로 나누고 각 Segment별로 Paging 적용



### 1. Contiguous Allocation (연속 할당)

#### a. 메모리 연속할당 방식의 종류

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230214163235078.png" alt="image-20230214163235078" style="zoom:50%;" /> 

- **Fixed partition allocation (고정 분할 방식)**

  - 프로그램이 들어갈 사용자 프로세스 영역을 미리 **분할(partition)**로 나누어 놓는 방식

  - 분할의 크기가 모두 동일한 방식과 서로 다른 방식이 존재

  - 분할 당 하나의 프로그램만을 load해야 하므로, 

    동시에 **load할 수 있는 프로그램의 수와 수행 가능한 프로그램의 최대 크기가 제한**된다.

  - 프로그램이 종료되거나 분할보다 작은 크기의 프로그램에 할당되면서 **외부조각, 내부조각**이 생길 수 있음

- **Variable partition allocation (가변 분할 방식)**

  - 메모리에 load되는 **프로그램의 크기**에 따라 **분할의 크기, 개수가 동적**으로 변하는 방식

  - 프로그램들의 크기가 균일하지 않기 때문에, 프로그램이 종료된 후 빈 메모리 공간이 남아서 **외부 조각**이 생길 수 있음

    ex) B가 끝나고 상대적으로 용량이 큰 D가 실행될 때 B

- 세부 개념

  - **External fragmentation (외부 단편화)**
    - 프로그램 크기보다 분할의 크기가 작은 경우
    - 빈 곳인데도 프로그램이 올라갈 수 없는 작은 분할

  - **Internal fragmentation (내부 단편화)**
    - 프로그램 크기보다 분할의 크기가 큰 경우
      - 특정 프로그램에 배정되었지만 사용되지 않는 메모리 조각

#### b. 가변 분할 방식과 Hole의 처리

- **Hole**

  - 가용 메모리 공간
  - 프로그램들이 실행되고 종료되는 과정에서 다양한 크기의 hole들이 메모리 여러 곳에 흩어져 있게 됨
  - 프로세스가 도착하면 수용가능한 hole을 할당
  - 운영체제는 다음과 같은 정보를 유지
    - 할당공간
    - 가용공간(hole)

- **Dynamic Storage Allocation Problem**

  - 가변 분할 방식에서 size n인 요청을 만족하는 가장 적절한 hole을 찾는 문제

  - Solution의 종류

    - First-fit
      - Size가 n 이상인 것 중 최초로 찾아지는 hole에 할당
    - Best-fit
      - Size가 n 이상인 가장 작은 hole을 찾아서 할당
      - Hole들의 리스트가 크기순으로 정렬되지 않은 경우 모든 hole의 리스트를 탐색해야 함
      - 많은 수의 아주 작은 hole이 생성됨
    - Worst-fit
      - 가장 큰 hole에 할당
      - 역시 모든 리스트를 탐색해야 함
      - 상대적으로 아주 큰 hole들이 생성됨

    -> 실험적인 결과: First-fit과 best-fit이 worst-fit보다 속도와 공간 이용률 측면에서 효과적인 것으로 알려짐

- **Compaction**

  - external fragmentation 문제를 해결하는 한 가지 방법으로, 사용 중인 메모리 영역을 한 군데로 몰고 hole들을 다른 한 곳으로 몰아 큰 block을 만드는 것
  - 프로그램 하나가 아닌 전체의 바인딩과 관련된 문제이기 때문에 매우 비용이 많이 듦
  - 최소한의 메모리 이동으로 compaction하는 방법이 효율적이지만 복잡함
  - 제약 조건: 프로세스의 주소가 실행 시간에 동적으로 재배치 가능한 경우에만 수행될 수 있다.

### 2. Non-Contiguous Allocation

#### a. Paging

##### a-1. Paging의 정의와 동작 방식

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230220154725396.png" alt="image-20230220154725396" style="zoom:50%;" />  

- Paging이란
  - 프로세스를 **일정 크기의 Page**로 잘라서 메모리에 **불연속적(noncontiguous)**으로 적재하는 방식
    - 앞서 보았듯이 contiguous allocation은 빈 메모리 공간이 있음에도 프로세스보다 크기가 작아 프로세스를 적재할 수 없는 external fragmentation 문제가 발생하고, 이를 해결하기 위한 기법은 복잡하고 비용이 많이 듦
    - 하나의 프로세스를 붙여놓을 필요 없이 여러 조각으로 잘라 메모리 곳곳의 빈 공간에 채워넣을  수 있다면 이러한 문제가 해결됨
  - **일부는 backing storage**에, **일부는 physical memory**에 저장

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230220154853092.png" alt="image-20230220154853092" style="zoom: 67%;" /> 

- 동작 방식 및 특징

  - physical memory를 동일한 크기의 **frame**으로 나눔

  - logical memory를 (frame과) 동일한 크기의 **page**로 나눔

  - **page table**을 이용하여 logical address(**VPN**, Virtual Page Number)를 physical address(**PFN**, Physical Frame Number)로 변환

    - 위 그림의 logical address에서 p는 **page number**로, 각 page별 physical memory 상에서의 시작 주소를 나타냄

    - d는 **page offset**이며, Base Address와 합쳐져 physical memory address로 변환됨

  - **external fragmentation**은 발생하지 않으나, **internal fragmentation**은 발생 가능

    - 프로세스가 필요한 virtual memory가 "**page의 크기 * n**"이 아닐 수도 있음

      -> page 1개의 크기보다 작은 공간이 남을 수 있음 -> **internal fragmentation** 발생

##### a-2. Page table

###### Page table이란?<sup>[2](#footnote_2)</sup>

- logical address와 physical address를 매핑하는 테이블

  - 각 프로세스는 자신만의 page table을 독립적으로 가지고 있음
  - 개별 매핑 데이터는 **Page Table Entry (PTE)**라고 함

- Page table은 **main memory**에 상주

  - register는 속도는 빠르지만 용량이 프로세스의 크기와 그에 따른 page table의 크기에 비해 작으므로 실용적이지 않음

- **Page-table base register(PTBR)**가 각 프로세스의 page table을 가리키고, **Page-table length register(PBLR)**가 테이블 크기를 보관

  - PTBR과 PTLR은 각각 page table의 base 값과 limit 값을 가짐
  - PTBR을 통해서 Physical address의 시작 주소지를 가져온 후, PTE의 Size만큼 VPN을 이동시켜서 주소 변환 작업 수행

- page table은 **OS**로부터 관리되고 **MMU**가 접근해서 읽음

  - 모든 가용 frame들을 관리

- 모든 메모리 접근 연산에는 **2번의 memory access** 필요 (page table 접근 1번 + 실제 data/instruction 접근 1번)

  -> 시간이 2배로 걸림

  -> 속도 향상을 위해 **associative register** 혹은 **translation look-aside buffer(TLB)**라 불리는 고속의 **lookup hardware cache** 사용

###### TLB(Translation Look-aside Buffer)란?

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230220165335981.png" alt="image-20230220165335981" style="zoom:50%;" /> 

- CPU와 Main memory 사이에 존재하는 빠른 주소 변환을 위한 cache

  - Main memory보다 접근 속도가 빠른 hardware로 구성 

- Page table에서 빈번히 참조되는 일부 PTE를 caching하고 있음

- TLB에 의한 Translation 과정

  <img src="http://thumbnail.egloos.net/600x0/http://pds27.egloos.com/pds/201212/19/32/d0014632_50d18ff68818d.png" alt="img" style="zoom: 80%;" /> 

  1. logical address가 physical address로 변환되어야 할 때, 먼저 **TLB를 검색**한다.

  2. (1) 검색이 성공하면 (**TLB hit**) -> 즉시 **physical address를 반환**하며, 프로세스는 해당 주소로 접근이 가능하다. -> **OUT**

     (2) 검색이 실패하면 (**TLB miss**) -> 통상적으로 핸들러는 해당 logical address에 매핑된 physical address가 있는지 **페이지 테이블을 검색**하게 된다. 

     ​															(2번의 메모리 접근 필요)

  3. (1) 페이지 테이블에서 일치하는 매핑 데이터를 찾았다면 (**page table hit**) -> 해당 정보를 TLB에 업데이트(**TLB write**)하고 

     ​																									 1번으로 돌아가 해당 logical address의 translation 요청 재개 -> **OUT**

     (2) 페이지 테이블에서 일치하는 매핑 데이터를 찾지 못했다면 (**page not present**) -> 

  ​																												logical address가 유효하지 않거나,

  ​																											 	(이 경우, OS는 해당 주소 접근을 요청한 프로세스에게 segmentation fault 발생시킴)

  ​																											 	logical address가 physical address에 존재하지 않기 때문 

  ​																											 	(물리 메모리 공간이 부족해 하드디스크의 보조 저장소, **paging file**로 **page-out**된 상황)

  2. 해당 logical address가 physical address에 존재하지 않는다면 -> 

     (1) physical memory가 꽉 차지 않았다면 (empty frame이 존재한다면) -> paging file에 존재하는 데이터를 empty frame으로 로드하고,

     ​																															  page table과 TLB의 내용 갱신 (**page table write & table write**)

     (2) physical memory가 꽉 찼다면 (empty frame이 없다면) -> RAM에서 empty로 변경할 페이지를 찾고,

     ​																										  페이지의 데이터가 변경되었다면 paging file에 쓴 후 empty frame으로 변환한 뒤,

     ​																										  해당 empty frame에 원래 load하려던 데이터를 load하고 page table과 TLB의 내용 갱신

- TLB는 특정 항목이 아닌 전체를 search해야 하므로 시간이 오래 걸림

  -> Translation 성능 향상을 위해 **Associative Register**를 활용

  - **parallel search**가 가능

    - 페이지 번호 p가 주어지면 TLB의 모든 entry를 대상으로 동시에 search

      - p가 있다면 (TLB hit) -> 주소 변환이 이루어짐

      - p가 없다면 (TLB miss) -> Page Table 통해서 주소 변환

        ※ Page Table의 search는 p번째 entry를 조회하므로 

        ​	sequential/parallel search일 필요 X

- TLB는 **context switch 때 flush** (remove old entries) 

  - 프로세스마다 개별적인 page table과 TLB가 존재하고, 다른 프로세스로 CPU가 넘어가면 page table의 주소변환 정보도 달라지므로

###### Effective Access Time (실질 메모리 접근 시간, 유효 접근 시간, EAT)<sup>[3](#footnote_3)</sup>

- CPU가 요구한 데이터를 읽는 데 걸리는 평균 시간을 측정하는 성능 지표

- Associative register 조회(lookup)하는 데 걸리는 시간 = ε
- memory cycle time = 1
- **Hit ratio** (TLB에서 주소를 찾는 경우의 비율)= α

​					(hit)			(miss)

- **EAT** = (1 + ε)α + (2 + ε)(1 - α)

  ​		= 2  + ε - α

  - TLB hit : TLB를 조회(ε)하고 변환된 주소로 메모리를 조회(1) => 1 + ε

  - TLB miss : TLB를 조회(ε)했으나 page number가 없어 메모리의 page table을 조회(1)하고

    ​					변환된 주소로 메모리를 조회(1) => 1 + 1 + ε

  - hit ratio α가 1에 가깝고 TLB를 조회하는 데 걸리는 시간이 메모리 접근 시간인 1보다 작다면, EAT는 page table만 있을 때 접근하는 시간 2보다 작다.

##### a-3. Two-Level Page Table

###### Two-Level Page Table이란?

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230221152058551.png" alt="image-20230221152058551" style="zoom:50%;" /> 

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230221161924702.png" alt="image-20230221161924702" style="zoom: 50%;" /> 

- page table을 위한 memory 공간을 절약하기 위해 사용함

- 현대의 컴퓨터는 address space가 매우 큰 프로그램 지원

  - 32 bit address 사용시 : 2^32B (4GB)의 address space

    ​										-> 각 프로그램이 가질 수 있는 메모리의 최대 크기는 4GB

    - page size가 4KB라면 1M 개의 page table entry가 필요
    - 각 page entry가 4B라면 프로세스당 4M의 page table이 필요
    - 그러나 대부분의 프로그램은 4GB의 주소 공간 중 지극히 일부분만 사용
      - p번째 entry로 접근하는 page table 특성 상 중간에 사용하지 않는 공간이 있어도 비울 수 없고, 모든 page에 해당하는 PTE를 만들어야 함
      - 따라서, page table 공간이 심하게 낭비됨

​	 -> page table 자체를 page로 나누어서 구성

​	 -> 사용되는 주소 공간에 대해서만 inner page table을 생성 

​		 사용되지 않는 주소 공간에 대한 outer page table의 엔트리 값은 NULL

​		 (대응하는 inner page table이 없음)

###### Two-Level Paging의 예시

- page size가 4KB인 32-bit machine의 logical address는 어떻게 구성되어야 하는가?
  - **12 bit**의 **page offset**
    - page offset은 한 page 내에서 시작 주소로부터 얼마만큼 떨어져 있는지를 나타낸 것
    - 메모리는 Byte 단위로 주소가 매겨지는데, 4KB(=2^12 Byte) 크기의 page를 구분하려면 서로 다른 2^12가지의 정보(-> 주소)를 나타낼 수 있어야 하므로 12 bit
  - **20 bit**의 **page number**
    - 10 bit의 page offset
      - 4KB 크기의 page를 구성하는 각 page entry가 4B라면, page entry의 총 개수는 1K = 2^10이고, 서로 다른 2^10가지의 주소를 나타낼 수 있어야 하므로 10 bit
    - 10 bit의 page number
      - 32 bit 중 page의 offset을 나타낼 12 bit, page table의 page의 offset을 나타낼 10 bit를 제외한 나머지는 10 bit
  - <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230223170425984.png" alt="image-20230223170425984" style="zoom: 80%;" /> 
    - p1은 outer page table의 index
    - p2는 outer page table의 page에서의 변위(displacement)

##### a-5. Multi-level Paging

- Address space가 커지면 다단계 페이지 테이블 필요

- 각 단계의 페이지 테이블이 메모리에 존재하므로 logical address의 physical address 변환에 더 많은 메모리 접근 필요

  -> TLB를 통해 메모리 접근 시간을 줄일 수 있음

- 4단계 페이지 테이블을 사용하는 경우

  - 메모리 접근 시간이 100ns, TLB 접근 시간이 20ns이고

  - TLB hit ratio가 98%인 경우

    - EAT (Effective Access Time) = 0.98 * 120 + 0.02 * 520 (120 + 4단계 페이지 테이블의 주소 변환 시간 400)

      ​												 = 128ns

      -> 메모리 접근 시간이 100ns이므로, 

      ​	 주소 변환을 위해 28ns만 소요

##### a-6. Memory Protection을 위한 Page Table Entry의 bit

###### Protection bit

- page에 대한 접근 권한 (read/write/read-only)을 나타내는 bit
  - page가 프로세스의 code 영역을 담고 있다면, 입력된 instruction을 수행할 뿐 내용이 바뀌어선 안 되므로 read-only
  - data / stack 영역은 데이터를 중간에 수정하는 것이 가능하므로 read와 write

###### Valid (v) - Invalid (i) Bit

###### <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230306214742906.png" alt="image-20230306214742906"  />  

- page table 자료구조의 특성상 시작 주소로부터의 index를 통해 각 page에 접근하기 때문에 프로세스의 주소공간 중 사용되지 않는 영역에 대해서도 page table entry가 만들어져야 함

  -> page table entry마다 해당 page가 사용되고 있는지(메모리에 올라와 있는지)를 구분할 수 있는 bit를 사용 

- **valid(1)**는 해당 주소의 frame에 그 프로세스를 구성하는 유효한 내용이 있다는 뜻이고, 

  **invalid(0)**는 frame에 유효한 내용이 없음을 뜻함

  		1. 프로세스가 그 주소 부분을 사용하지 않는 경우
  		1. 해당 페이지가 메모리에 올라와 있지 않고 swap area에 있는 경우

- page table을 통해 logical memory에서 physical memory로 접근할 때 

  - valid하다면 바로 해당 페이지로 접근
  - invalid하다면 **Page Fault**가 발생

##### a-7. Inverted Page Table

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230306222929904.png" alt="image-20230306222929904" style="zoom:67%;" /> 

- page table의 용량이 큰 이유
  - 모든 process 별로 그 logical address에 대응하는 모든 page에 대해 page table entry가 존재
  
  - 대응하는 page가 메모리에 있든 아니든 간에 page table에는 entry로 존재
  
    -> page table의 용량을 줄이기 위해 Inverted Page Table 사용 가능
  
- Inverted Page Table
  
  - Page Table이 프로세스마다 하나씩 있는 게 아니라 시스템 전체에 하나 존재
  - Page frame 하나 당 page table에 하나의 entry를 둔 것
  - 각 page table entry는 각각의 물리적 메모리의 page frame이 담고 있는 내용 표시
    - 단점 : 프로세스 id(pid)와 논리주소의 페이지(p)를 찾기 위해 테이블 전체를 탐색해야 함 -> 시간적인 overhead
    - 조치 : page table을 associative register에 집어넣어 entry들을 병렬적으로 검색 (단, expensive)
  

##### a-8. Shared Pages

###### Shared code

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230306223641164.png" alt="image-20230306223641164" style="zoom:67%;" /> 

- 여러 process가 공유할 수 있는 code를 physical memory 상의 같은 frame으로 매핑하는 기법

  - 동일한 코드가 메모리에 한 번만 적재되므로 메모리를 아낄 수 있음
  - Re-entrant code(재진입 코드)라고도 불림

- read-only로 하여 프로세스 간에 하나의 code만 메모리에 올림

  (ex_ text editors, compilers, window systems)

  <-> Shared Memory가 read와 write가 가능한 반면, Shared code는 read-only 

  ​		(Process Management의 프로세스 간 협력 부분 참고)

-  Shared code는 모든 프로세스에서 동일한 logical address를 가져야 함

  - 동일한 shared code는 프로세스 A의 주소공간에서 1번이라면 프로세스 B의 주소공간에서도 1번이어야 함

- 반면, Private code and data의 경우

  - 각 프로세스들은 독자적으로 메모리에 올라옴
  - Private data는 logical address space의 아무 곳에 와도 무방 



#### b. Segmentation

##### b-1. Segmentation의 정의와 특징

- 프로세스는 논리적(의미) 단위인 여러 개의 segment로 구성
  - 작게는 프로그램을 구성하는 함수 하나하나, 크게는 프로그램 전체를 하나의 세그먼트로 정의 가능
  - 일반적으로는 code, data, stack 부분이 각각 하나의 segment로 정의됨
- Paging이 전혀 가미되지 않은 original segmentation을 사용하는 메모리 시스템은 현실적으로 없음

##### b-2. Segmentation의 구성 요소와 동작 방식

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230310184729247.png" alt="image-20230310184729247" style="zoom:67%;" /> 

- 구성 요소
  - Logical address는 <**segment number**, **offset**> 두 가지로 구성
  - Segment table : Segment별 주소 변환
    - table의 각 entry는 다음을 포함
      - base - 해당 segment의 시작점인 physical address
      - limit - 해당 segment의 길이
  - Segment table base register(STBR)
    - 물리적 메모리에서의 segment table의 위치
  - Segment table length register(STLR)
    - 프로그램이 사용하는 segment의 수 ( = segment table의 크기)
      - 위 그림에서 segment number, s가 STLR

- 동작 방식

  - logical address의 segment number (s) 가 STLR의 값보다 크거나,

    ​									  offset (d) 이 해당 entry의 limit보다 크다면  -> trap 발생

  - 정상적인 요청이라면  -> segment의 시작 위치 (base)에 offset (d)을 더해 주소 변환

    - base 위치만큼 떨어진 곳에서 segment가 시작, segment의 시작점에서 d만큼 떨어진 곳에 원하는 주소의 내용이 있음

- Paging과 Segmentation 비교

  | Paging                                                       | Segmentation                                                 |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | offset의 크기가 고정된 Page 크기에 의해 제한                 | offset을 표현할 수 있는 bit 수에 의해 제한                   |
  | 시작 주소가 frame 번호로 주어짐                              | 시작 주소가 정확한 Byte 단위 주소로 주어짐                   |
  | 내부 단편화 O, 외부 단편화 X                                 | 외부 단편화 O, 내부 단편화 X<br />(segment 길이 가변적이므로 가변분할 방식과 동일한 문제점 발생) |
  | 보통 page 하나의 크기가 4KB로 상대적으로 작아,<br />page table이 상대적으로 더 많은 메모리를 차지 | segment의 크기가 상대적으로 커서, <br />segment table이 상대적으로 더 적은 메모리를 차지 |

- Segmentation의 장점

  - segment는 의미 단위이기 때문에 보안(protection)과 공유(sharing)에 있어 paging보다 효과적임<sup>[4](#footnote_4)</sup>

  - 보안

    - 권한 부여는 의미 단위로 이루어짐 (ex_ code는 read-only, data는 read & write)

      - paging의 경우, 프로그램을 동일한 크기의 page로 자르면 한 page에 code와 data가 들어갈 수 있는데 이때 어떤 권한을 부여할지 불명확

        -> 의미 단위로 protection 해야 하는 경우 다른 page에 위치시키는 등의 부가적인 작업 수행

      - segmentation의 경우, 접근 제어 키를 사용하여 segment 별로 허용되는 작업을 제어하여, 사용자의 잘못된 접근으로부터 보호될 수 있음

  - 공유

    - 공유해야 하는 프로시저가 커서 몇 개의 페이지로 나누어진다면, 이 페이지의 엔트리는 공유하는 프로세스들의 페이지 테이블에서 모두 같은 위치에 있어야 하므로 테이블의 구성이 어려움
    - 공유 프로시저의 크기가 페이지의 크기로 정확하게 나누어떨어지지 않을 경우, 공유할 필요가 없거나 공유해선 안되는 부분이 공유 페이지에 포함될 수도 있음

##### b-3. 예시

![image-20230311212522224](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230311212522224.png) 

- 두 개의 프로세스가 Segment를 공유하는 경우 (shared segment)

![image-20230311212637153](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230311212637153.png) 



#### c. Paged Segmentation

- segment 하나를 여러 개의 page로 구성

- segment table을 통해, segment의 주소(s, d)를 segment의 page table의 주소, page number(p)와 page의 offset(d')로 변환

  -> 해당 segment의 page table을 통해, 해당 page number(p)에 상응하는 frame(f)를 찾아 조회

  ![image-20230311214510187](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230311214510187.png) 

- pure segmentation과의 차이점

  - segment table entry가 segment의 base address를 가지고 있는 것이 아니라, 

    segment를 구성하는 page table의 base address를 가지고 있음

  - 외부 단편화(external fragmentation) 문제 없음

    - 물리적인 메모리에는 page 단위로 올라가고 segment는 n개의 page로 구성

  - 의미 단위로 이루어지는 공유, 보안 등은 segment table level에서 관리

    

## 주소 변환과 OS

- Logical Address를 Physical Address로 변환하는 일련의 과정은 Memory Management Unit 등에 의해 하드웨어적으로 이루어지며, 이때 OS는 개입하지 않는다.

  - CPU를 갖고 있는 프로세스가 매 클럭 사이클마다 메모리에서 데이터를 읽어들여서 CPU를 실행하고, 

    이때 메모리에 접근하기 위해 주소 변환을 할 때마다 CPU가 운영체제로 넘어갈 수는 없음

    


## 용어 정리

### Dynamic Loading

- 프로세스 전체를 메모리에 미리 다 올리는 것이 아니라 해당 루틴이 불려질 때 메모리에 load하는 것

- 메모리 공간을 효율적으로 사용할 수 있음

- 가끔씩 사용되는 많은 양의 코드의 경우 유용

  ex) 오류 처리 루틴

- 운영체제의 특별한 지원 없이 프로그램 자체에서 구현 가능

  (OS는 관련 라이브러리를 제공함으로써 프로그래머가 dynamic loading을 효율적으로 구현할 수 있게 지원)

  - 운영체제가 자체적으로 Loading을 제어하는 Paging을 Dynamic Loading이라고 칭하기도 함

### Overlays

- 메모리에 프로세스의 부분 중 실제 필요한 정보만을 올림

  - 이 부분은 Dynamic Loadingㅇ과 유사하지만 둘은 역사적으로 다름

  - 작은 공간의 메모리를 사용하던 초창기 시스템에서 프로그램 하나를 메모리에 올리는 것도 불가능

    -> 프로그래머가 프로그램을 쪼개서 실행되어야 하는 부분이 메모리에 올라가게끔 수작업으로 코딩

    - 프로그래밍이 매우 **복잡**함

- **프로세스의 크기가 메모리보다 클 때** 유용

- **운영체제의 지원 없이** 사용자에 의해 구현

### Swapping

- **Swapping**

  - 프로세스를 일시적으로 메모리에서 backing store로 쫓아내는 것

- **Backing store (=swap area)**

  - 디스크 : 많은 사용자의 프로세스 이미지를 담을 만큼 충분히 빠르고 큰 저장 공간

- **Swap in** / **Swap out** 

  - 일반적으로 중기 스케줄러(swapper)가 CPU priority가 낮은 프로세스를 선정해 쫓아내고 priority가 높은 프로세스를 메모리에 올려 놓음
    - 이때, priority가 낮은 프로세스를 쫓아내는 것을 swap out, 쫓아낸 프로세스가 메모리로 들어오는 것을 swap in이라고 함
  - Swapping이 효과적으로 동작하려면 Run time binding이 지원되어야 함 
    - swap out 되었다가 메모리의 다른 위치로 swap in 될 수 있음
    - Compile / Load time binding에서는 반드시 정해진 원래 주소로 돌아가야 함
  - swap time은 대부분 transfer time (swap 되는 양에 비례)
    - 프로세스를 통째로 하드디스크에 옮기는 것이기 때문에 방대한 데이터가 이동함
    - 상대적으로 적은 데이터가 이동하는 파일 입출력은 seek time(디스크 헤드가 이동하는 시간)이 대부분을 차지
    - 

  *참고 : Paging에서는 프로세스의 모든 내용이 아닌, 프로세스의 페이지 중 일부를 쫓아내고 메모리에 load하는 것도 Page Swap이라고 부르기도 함

### Dynamic Linking

- **Linking** 

  - 여러 목적 파일(object file)들을 묶어 하나의 실행파일(executable file)을 만드는 것
    - 목적 파일은 컴파일러나 어셈블러가 소스코드 파일을 컴파일 또는 어셈블 해서 생성하는 파일

- **Static Linking**

  - 라이브러리가 프로그램의 실행 파일 코드에 포함됨
  - 실행 파일의 크기가 커짐
  - 동일한 라이브러리를 각각의 프로세스가 메모리에 올리므로 메모리 낭비
    - 100개의 코드가 printf 함수를 사용한다면, "printf 함수의 라이브러리 코드 * 100" 만큼의 용량이 메모리를 차지

- **Dynamic Linking**

  - 라이브러리가 실행파일이 아닌 **별도의 파일**로 존재하고, 프로그램의 **실행시 연결**(link)됨

  - 라이브러리 호출 부분에 라이브러리 루틴의 위치를 찾기 위한 포인터의 역할을 하는 **stub**이라는 작은 코드를 둠

  - 라이브러리가 이미 메모리에 있으면 그 루틴의 주소로 가고 없으면 디스크에서 읽어옴

    -> Dynamic Linking을 하려면 **OS의 도움이 필요**하며, 

    ​	 Dynamic Linking을 위한 운영체제 라이브러리를 **Shared Library**라고 부른다.

    ​	 ex) 리눅스 : Shared Object, 윈도우 : dll



##  참고

<a name="footnote_1">1</a>. Segmentation fault (=Memory access violation)

https://en.wikipedia.org/wiki/Segmentation_fault

<a name="footnote_2">2</a>. Page Table

http://egloos.zum.com/sweeper/v/2988646

https://ddongwon.tistory.com/49

<a name="footnote_3">3</a>. Effective Access Time (EAT)

https://www.cukashmir.ac.in/cukashmir/User_Files/imagefile/DIT/StudyMaterial/OperatingSystemBTech/BTechCSE_3_Rizwana_BTCS304_unit3_B.pdf

<a name="footnote_4">4</a>. Segment 보호와 공유

https://itdexter.tistory.com/409

