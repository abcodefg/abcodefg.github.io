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

