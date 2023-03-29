---
title:  "[운영체제] File Systems"
excerpt: "File and File System, Directory and Logical Disk, open(), File Protection, File system의 Mounting, Accessing Methods"

categories:
  - OS
tags:
  - [OS]
  
toc: true
toc_sticky: true
 
date: 2023-02-20T12:10:55-04:00
last_modified_at: 2023-02-20T12:10:55-04:00

---

# File Systems

## File and File System

- **File**

  - A **named** collection of related information

    - 메모리에 주소를 통해 접근하듯이, 파일은 이름을 통해 접근

  - 일반적으로 **비휘발성의 보조기억장치**에 저장

  - 운영체제는 다양한 저장 장치를 file이라는 동일한 논리적 단위로 볼 수 있게 해줌

  - File의 연산

    - create, read, write, reposition (= lseek, 파일의 포인터 위치 수정), delete, 

      **open (file의 metadata를 메모리에 올려놓음)**, close 등

- **File attribute** (혹은 파일의 **metadata**)

  - 파일 자체의 내용이 아니라 파일을 관리하기 위한 각종 정보들
    - 파일 이름, 유형, 저장된 위치, 파일 사이즈
    - 접근 권한 (읽기/쓰기/실행), 시간 (생성/변경/사용), 소유자 등

- **File system**

  - 운영체제에서 파일을 관리하는 부분
  - 파일 및 파일의 메타데이터, 디렉토리 정보 등을 관리
  - 파일의 저장 방법 결정, 파일 보호



## Directory and Logical Disk

- **Directory**
  - 속한 파일의 이름 등 metadata 중 일부를 보관하고 있는 일종의 특별한 파일
    - 일부는 Directory에 직접 저장하고 일부는 다른 곳에 저장하기도 함
  - Directory의 연산
    - search for a file, create a file, delete a file
    - list a directory, rename a file, traverse the file system
- **Partition (=Logical Disk)**
  - 하나의 (물리적) 디스크 안에 여러 파티션을 두는 것이 일반적
  - 여러 개의 물리적인 디스크를 하나의 파티션으로 구성하기도 함
  - (물리적) 디스크를 파티션으로 구성한 뒤 각각의 파티션에 **file system**을 깔거나 **swapping** 등 다른 용도로 사용할 수 있음



## open()

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230327152639360.png" alt="image-20230327152639360" style="zoom:80%;" /> 

- **open("/a/b")**

  - 디스크로부터 **파일 b의 메타데이터**를 메모리로 가져옴

  - 이를 위하여 **directory path를 search**

    - 루트 디렉토리 "/"를 open하고 그 안에서 파일 "a"의 위치 획득
      - 루트 디렉토리의 위치는 알려져 있음
    - 파일 "a"를 open한 후 read하여 그 안에서 파일 "b"의 위치 획득
    - 파일 "b"를 open

  - 그런데 directory path의 search에 너무 많은 시간이 소요

    - Open을 read/write와 별도로 두는 이유
    - 한 번 open한 파일은 read / write 시 directory search 불필요

  - **Open file table**

    - 현재 open 된 파일들의 메타데이터 보관소 (in memory)
    - 디스크의 메타데이터보다 몇 가지 정보가 추가
      - open한 프로세스의 수
      - File offset : 파일 어느 위치 접근중인지 표시 (별도 테이블 필요)

  - **File descriptor** (file handle, file control block)

    - OS가 파일에 접근할 때 사용하는, 각 파일에 부여된 식별자 (0 이상의 정수)
      - 특정 프로세스에 의해 open된 file들의 정보를 저장하는 프로세스 별 file descriptor table의 인덱스
    - open file table
      - Open file table에 대한 위치 정보 (프로세스별)

  - 예시

    ![image-20230327154037744](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230327154037744.png) 

    1. Process A의 open("/a/b") 명령어 수행 (시스템 콜)

    2. 파일 b를 열기 위해 Directory path를 search하면서 경로 상 파일들의 metadata를 open file table에 올려놓음

       - b의 metadata가 메모리에 올라오면 open("/a/b") 수행 종료

    3.  open() 연산의 결과값으로 해당 파일의 file descriptor ( = file descriptor table의 인덱스)를 반환

    4. 어떤 파일에 대해 read(), write() 연산을 할 경우 그 file descriptor를 인자로 넘겨줌

    5. file descriptor에 해당하는 file descriptor table 상의 엔트리에서 b의 metadata의 위치 획득하고, 

       b의 위치 정보(metadata)를 통해 b의 content를 읽어옴

    6. OS는 읽어온 내용을 자신의 메모리 공간 일부에 저장하고 이를 copy해서 사용자 프로그램에 전달

       -> **Buffer cache**

       - 이후 어떤 프로세스가 해당 데이터를 요청할 경우 disk에서 읽어오지 않고 메모리에서 직접 전달

       - paging system과 달리 buffer cache에 요청한 데이터가 있는지 여부와 상관없이 운영체제에 CPU 제어권이 넘어감

         -> OS가 모든 정보를 알고 있으므로 LRU, LFU 등의 알고리즘 사용 가능

         

    ※ 참고: OS의 구현에 따라서 그림 상의 2개 외에도 관련 table이 추가될 수 있음

    - disk에 있는 metadata를 메모리에 올려놓을 경우 어떤 프로세스가 해당 파일의 어느 위치를 접근하고 있는지에 대한 metadata인 offset이 추가되어야 함

      -> 프로세스마다 offset이 다를 수 있으므로 이를 프로세스 별로 관리하는 테이블을 따로 두는 것이 일반적



## File Protection

- 각 파일에 대해 **1) 누구에게** **2)어떤 유형의 접근(read / write / execution)**을 허락할 것인가?

- Access Control 방법

  - **Access Control Matrix**

    ![image-20230327230034974](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230327230034974.png) 

    - 그런데 적은 수의 유저만 접근할 수 있는 파일이 많다면 이 행렬은 그 값이 대부분 0인 희소 행렬이 될 가능성이 높음

      -> 행렬의 칸을 전부 만드는 것은 낭비

    - 대신 다음과 같은 방법들을 고려할 수 있음

      - **Access control list** : **파일별**로 누구에게 어떤 접근 권한이 있는지 표시
      - **Capability list** : **사용자별**로 자신이 접근 권한을 가진 파일 및 해당 권한 표시

    - 그러나, 여전히 오버헤드가 크기에 일반적인 OS에서는 Grouping이라는 방법 통해 파일의 접근 권한 제어

  - **Grouping**

    - 전체 user를 owner, group, public 세 그룹으로 구분

    - 각 파일에 대해 세 글부의 접근권한(rwx)을 3비트씩으로 표시

      ex) UNIX : rwx r-- r-- (owner, group, public 순)

  - **Password**

    - 파일마다 password를 두는 방법 (디렉토리 파일에 두는 방법도 가능)
    - 모든 접근 권한에 대해 하나의 password: all-or-nothing
    - 접근 권한별 password 둔다면 암기해야 하거나 따로 관리하기가 어렵다는 문제 발생



## File System의 Mounting

- **Mounting** 
  - 어떤 file system의 **특정 디렉토리에 다른 file system을 연결**하여, 운영 체제와 사용자가 해당 파일 시스템에 접근할 수 있게 만드는 것
  - <img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230327233210519.png" alt="image-20230327233210519" style="zoom:80%;" /> 
    - disk3를 disk1의 usr 디렉토리에 mount하면, usr에 접근하는 것이 disk3 file system의 루트 디렉토리에 접근하는 것과 같음



## Access Methods

- 시스템이 제공하는 파일 정보의 접근 방식

  - **순차 접근 (sequential access)**

    - 카세트 테이프를 사용하는 방식처럼 접근
      - 첫 부분을 다시 들으려면 되감아야 함
    - 읽거나 쓰면 offset은 자동으로 증가

  - **직접 접근 (direct access, random access)**

    - LP 레코드 판과 같이 접근하도록 함

    - 파일을 구성하는 레코드를 임의의 순서로 접근할 수 있음

      - a -> b -> c 순서로 저장되어 있는 경우, 순차 접근은 a에 접근한 이후 c를 접근하려면 b를 거쳐야 하는 반면

        ​																 직접 접근은  b를 건너뛰고 c에 바로 접근할 수 있음



## Allocation of File Data in Disk

- Disk에 File을 저장할 때는 동일 크기의 sector로 나누어 저장
- File data를 sector에 할당하는 방법에는 다음과 같은 것들이 있음
  - Contiguous allocation
  - Linked Allocation
  - Indexed Allocation

### a. Contiguous Allocation (연속 할당)

- **리스트**처럼 할당
- 각 파일에 대해 **Disk 상의 연속된 블록**을 할당
- 단점
  - **external fragmentation**
  - **File 크기를 키우기 어려움**
    - file 생성시 얼마나 큰 hole을 할당할 것인지가 문제
    - 크기가 커질 것을 대비해 예비 공간을 할당하면 다른 파일이 해당 공간을 사용하지 못하므로 internal fragmentation이 발생
- 장점
  - **Fast I/O**
    - 한 번의 seek/rotation으로 많은 바이트 trasnfer (파일이 연속적으로 저장되어 있으므로)
    - Realtime file(deadline이 있는) 용 혹은 이미 run 중인 process의 swapping 용으로 적합
  - **직접 접근 (direct access)**과 **순차 접근 (sequential access)** 모두 가능
- 아래 그림에서 연속 할당 시에 디렉토리가 포함하고 있는 파일들의 metadata를 확인할 것

<img src="C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230328221204188.png" alt="image-20230328221204188"  /> 



### b. Linked Allocation (연결 할당)

- **연결 리스트**처럼 할당
- 각 블록이 **다음 블록을 가리키는 포인터**를 저장
  - 디렉토리는 file의 시작 위치만 가지고 있고 이후에는 특정 위치가 다음 위치를 기록해두고 있음
  - 각 블록은 포인터를 저장하기 위해 4바이트 혹은 그 이상 소모
- 장점
  - external fragmentation 발생 안 함
- 단점
  - **직접 접근이 불가**하고 순차 접근만 가능
  - **Reliability 문제**
    - 한 sector가 고장나 포인터가 유실되면 많은 부분을 잃음
  - 포인터를 위한 공간이 블록의 일부가 되어 공간 효율성을 떨어뜨림
    - 512 bytes/sector, 4 bytes/pointer
- 변형
  - **File Allocation Table (FAT)** 파일 시스템
    - **포인터를 별도의 위치에 보관**하여 reliability 문제와 공간효율성 문제 해결

![image-20230328222412761](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230328222412761.png) 



### c. Indexed Allocation (색인 할당)

- 데이터 블록 외에도 파일마다 **포인터의 모음인 인덱스 블록**을 한 개씩 둠
  - 디렉토리는 인덱스 블록을 가리킴
- 장점
  - external fragmentation이 발생 안 함
  - 직접 접근 가능
- 단점
  - small file의 경우 공간 낭비 (실제로 많은 파일들이 small)
  - Too large file의 경우 하나의 블록으로 인덱스를 저장하기에 부족
    - 해결 방안
      - linked : 인덱스 블록이 다른 인덱스 블록을 가리키게 함
      - multi-level : 인덱스 블록을 여러 계층으로 구성 (인덱스 블록 -> 하위 인덱스 블록 -> ... -> 데이터 블록)
      - combined : linked와 multilevel을 합침

![image-20230328223745055](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230328223745055.png) 

- 위의 파일 시스템의 할당에 관한 이론이 실제 파일 시스템에서 어떻게 적용되는지 확인해보자



## UNIX의 파일 시스템

![image-20230328224900715](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230328224900715.png) 

- 파일 시스템의 가장 기본적인 구조이며, 이를 효율적으로 개선시킴으로써 많은 파일 시스템들이 파생되었음

- 유닉스 파일 시스템의 중요 개념

  - **Boot block**

    - **부팅에 필요한 정보** (**bootstrap loader**)
      - 파일 시스템에서 OS 커널의 위치를 찾고 메모리에 올려 정상적인 부팅이 이루어지게 함
    - 어떤 파일 시스템이든 Boot block이 제일 앞에 위치

  - **Super block**

    - 파일 시스템에 관한 **총체적인 정보**
      - 어디까지 Inode list이고 어디부터 data block인지, 어느 블록이 비어 있고 어디가 사용중인지 등

  - **Inode list**

    - **파일 이름을 제외**한 **파일의 모든 메타 데이터**를 저장

      - directory file은 파일 이름과 inode 번호를 포함

    - 파일 하나 당 inode가 하나씩 할당

      - 크기가 큰 파일도 하나의 inode로 저장 위치를 나타낼 수 있어야 함

        -> 크기가 작은 경우 data blocks만으로, 

        ​	크기가 클 경우 single, double, triple indirect 등 여러 계층의 인덱스를 활용

        ​	(indexed allocation의 **multilevel** indexing 참조)

  - **Data block**

    - 파일의 실제 내용을 보관



## FAT 파일 시스템

- Microsoft가 MS-DOS를 개발했을 때 처음 만든 파일 시스템

![image-20230328230957794](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230328230957794.png) 

- Linked Allocation 활용하되 어떤 블록의 다음 블록을 가리키는 포인터를 FAT이라는 테이블에 저장
- FAT 시스템은 Linked allocation의 문제를 대부분 해결할 수 있음
  - No direct access -> FAT을 한 번만 읽으면 직접 접근 가능
  - Reliability 문제 -> FAT에 문제가 없다면 중간 블록에 문제 생겨도 다음 블록을 읽을 수 있음
    - FAT은 중요한 정보를 담고 있기 때문에 손실 시 복구를 위해 이중 저장
  - 공간 효율성 문제 -> 별도로 저장되기 때문에 섹터에 할당된 512 byte 온전히 활용 가능
- FAT은 컴퓨터가 부팅될 때 OS에 의해 메모리에 적재됨



## Free Space Management

- 비어있는 블록을 관리하는 방법들

### a. Bit map or bit vector

![image-20230328233131952](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230328233131952.png) 

- Bit map은 Disk에 부가적인 공간을 필요로 함
- 연속적인 가용 공간을 찾는 데 효과적
  - 연속적으로 0인 영역

### b. Linked List

- 모든 free block들을 링크로 연결 (free list)
- bit map과 달리 공간의 낭비 없음
- 연속적인 가용공간을 찾는 것 쉽지 않음

### c. Grouping

- linked list 방법의 변형
- 첫 번째 free block이 n개의 포인터를 가짐
  - 처음 n-1개의 포인터는 free data block을 가리킴
  - 마지막 포인터가 가리키는 블록은 또 동일한 구조의 n 개의 포인터를 가짐
- 연속적인 가용공간을 찾는 것 쉽지 않음

### d. Counting

- 프로그램들이 종종 여러 개의 연속적인 block을 할당하고 반납한다는 성질에 착안
- (first free block, # of contiguous free blocks) 의 구조



## Directory Implementation

- 디렉토리를 구현하는 방법

### a. directory file의 구조

#### a-1. Linear List

![image-20230328234251240](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230328234251240.png) 

- <file name, file의 metadata>의 list
- 구현이 간단
- 디렉토리 내에 파일이 있는지 찾기 위해서 linear search 필요하므로 비효율적

#### a-2. Hash Table

![image-20230328234313986](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230328234313986.png) 

- linear list + hashing
- Hash table은 file name을 이 파일을 linear list의 위치로 바꾸어 줌
- search time을 없앰
  - hashing을 적용한 값에 해당하는 위치로 바로 접근하므로
- collision 발생 가능 (hashing)

### b. File의 metadata 보관 위치

- 디렉토리 내에 직접 보관
- 디렉토리에는 포인터를 두고 다른 곳에 보관
  - inode, FAT 등

### C. Long file name의 지원

- <file name, file의 metadata>의 list에서 각 entry는 일반적으로 고정 크기
- file name이 고정 크기의 entry 길이보다 길어지는 경우 entry의 마지막 부분에 이름의 뒷부분이 위치한 곳의 포인터를 두는 방법
- 이름의 나머지 부분은 동일한 directory file의 일부에 존재

![image-20230328235039811](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230328235039811.png) 



## VFS와 NFS

- **VFS (Virtual File System)**
  - 서로 다른 다양한 FIEL SYSTEM에 대해 동일한 시스템 콜 인터페이스(API)를 통해 접근할 수 있게 해주는 OS의 계층

- **NFS (Network File System)**
  - 분산 시스템에서는 네트워크를 통해 파일이 공유될 수 있음
  - NFS는 분산 환경에서의 대표적인 파일 공유 방법임

![image-20230328235540038](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230328235540038.png) 

- 클라이언트는 어떤 file system이든 상관없이 VFS Interface를 통해 접근

- NFS를 통해 다른 컴퓨터의 file system도 접근할 수 있음

  - 클라이언트와 서버 모두 NFS 모듈을 가지고 있어야 함

  - 참고 : RPC와 XDR<sup>[1](#footnote_1)</sup>

    - **RPC(Remote Procedure Call)**

      - 상호 미리 정의된 규격을 준수하여 원격에서 동작하고 있는 프로세스에 포함된 함수를 호출 가능하게 하는 프로세스 간 통신기술
      - 일반적으로 프로세스는 자신의 주소공간(address space) 안에 존재하는 함수를 호출하여 실행가능하지만, 

      RPC를 사용하면 다른 주소공간에서 동작하는 프로세스의 함수를 실행할 수 있음

      - 분산 컴퓨팅 환경에서 프로세스 간 상호 통신 및 컴퓨팅 자원의 효율적인 사용을 위해 발전된 기술

    - **XDR(eXternal Data Representation)**

      - 하드웨어/소프트웨어 아키텍쳐가 서로 다른 컴퓨터 시스템들 간의 데이터 전송을 위한 표준 데이터 형식
      - 로컬 데이터 표현 형식(local representation)을 XDR로 변환하는 것을 인코딩(encoding),

        XDR을 로컬 표현 형식으로 변환하는 것을 디코딩(decoding)이라고 함
      



## Page Cache와 Buffer Cache 비교

- **Page Cache**
  - **Virtual memory**의 paging system에서 사용하는 page frame을 caching의 관점에서 설명하는 용어
  
  - Memory-mapped I/O를 쓰는 경우 file의 I/O에서도 page cache 사용
    - **Memory-mapped I/O**<sup>[2](#footnote_2)</sup>
      - **File의 일부를 virtual memory에 mapping** 시킴
      - **매핑시킨 영역에 대한 메모리 접근 연산**은 파일의 입출력을 수행하게 함
      - open() -> read() / write() 시스템 콜을 통해 접근하는 방법과 대비됨
  
  - 이미 메모리에 존재하는 데이터에 대해서는 하드웨어적인 주소변환만 하고 시스템 콜이 발생하지 않음
  
    -> OS가 접근 시간 /  횟수 등의 정보를 완전하게 알 수 없음
  
    -> LRU, LFU 사용 불가, Clock 알고리즘 사용
  
- **Buffer Cache**
  - **파일시스템**을 통한 I/O 연산은 메모리의 특정 영역인 buffer cache 사용
  
  - File 사용의 locality 활용
    - 한 번 읽어 온 block에 대한 후속 요청시 buffer cache에서 즉시 전달
    
  - 모든 프로세스가 공용으로 사용
  
  - Replacement algorithm 필요 (LRU, LFU 등)
  
  - 파일에 접근할 때는 시스템 콜이 반드시 발생
  
    -> OS가 접근 시간 / 횟수 등의 정보를 완전하게 알 수 있음
  
    -> LRU, LFU 사용 가능

![image-20230329163348819](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230329163348819.png) 



- **Unified Buffer Cache**

  - 최근의 OS에서는 기존의 buffer cache가 page cache에 통합됨

  - buffer cache도 페이지 단위로 관리

    - OS가 buffer cache 공간을 따로 두지 않고 필요에 따라서 page cache 공간을 할당해서 사용

      

- 여러가지 file I/O 방식 비교![image-20230329223142476](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230329223142476.png) 

  - **read(), write() 시스템 콜을 사용하는 I/O**

    - 운영체제가 해당하는 파일 내용이 buffer cache에 있는지 확인하고 없으면 파일 시스템에서 가져와서 프로세스에 전달

  - **memory-mapped I/O**

    - **mmap()** 시스템 콜을 통해 메모리의 특정 공간에 파일을 매핑<sup>[3](#footnote_3)</sup>

    - 운영체제가 파일 시스템에서 buffer cache로 파일 내용을 읽어오는 것까지는 동일하지만

      그 내용을 page cache에 copy

    - 이후 page cache에 올라온 내용에 대해서는 OS의 간섭 없이 자신의 메모리를 접근하듯이 파일 I/O를 수행할 수 있음

  - **Unified Buffer Cache를 이용하는 File I/O**

    - Memory Mapped I/O의 경우 buffer cache가 별도로 있지 않기 때문에 보다 단순한 경로를 거침

- ![image-20230329232939646](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230329232939646.png) 

  - 프로그램은 파일 시스템에 저장되어 있다가 

    실행파일을 실행시키면 프로세스가 되고 독자적인 주소 공간이 생성됨

    당장 필요한 부분은 물리적인 메모리에 올라가고 나머지는 swap area로 쫓겨남

  - 프로세스의 메모리 영역 중 code 부분은 쫓겨날 때 swap area로 내려가지 않음

    -> code는 read only이고 이미 파일 시스템의 실행파일에 저장되어 있기 때문에

  ![image-20230329234725736](C:\Users\jodic\AppData\Roaming\Typora\typora-user-images\image-20230329234725736.png) 

 

## 참고

<a name="footnote_1">1</a>. RPC와 XDR

https://leejonggun.tistory.com/9

https://www.geeksforgeeks.org/remote-procedure-call-rpc-in-operating-system/

https://www.geeksforgeeks.org/presentation-layer-in-osi-model/

<a name="footnote_2">2</a>. Memory Mapped I/O와 I/O Mapped I/O

https://code-lab1.tistory.com/204

<a name="footnote_3">3</a>. mmap()

https://devraphy.tistory.com/428
