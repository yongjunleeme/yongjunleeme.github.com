---
layout  : wiki
title   : term
summary : 
date    : 2020-02-19 16:01:29 +0900
updated : 2020-03-01 20:56:48 +0900
tags    : 
toc     : true
public  : true
parent  : study
latex   : false
---
* TOC
{:toc}

> 개인이 보는 용도의 낙서장입니다;

## 컴퓨터구조

### 1

- 인터리빙

- 디미니싱 리턴 - 하드웨어는 발전하는데 되레 무엇이 마이너스를 만들어내는 것이었는데 기억이..ㅜㅜ

- 프로그램 - 순서를 가진 기계어 명령의 시퀀스. 이를 순서대로 Fetch해서 디코딩하고 명령어가 시키는 작업을 한다. 그러면서 머신 스테이트를 업데이트하는 것.

- Memory
	* 메인메모리만 DRAM, 나머지는 모두 SRAM. 왜냐면 나머지가 모두 [NAND게이트](https://ko.wikipedia.org/wiki/NAND_%EA%B2%8C%EC%9D%B4%ED%8A%B8) 이기 때문.

	- 명령어를 가져올 때 캐시를 쓰면 한 번에 여러 명령어를 가져오고 캐시는 재활용된다.

- 대역폭 - 일정 시간 내 얼마나 많은 일을 하느냐의 척도

- ILP(Instruction level parallelism) - 사이클당 처리되는 명령어수를 증가시키자. 즉, IPC를 높이자 그러나  한계에 부딪힘. 그래서 
 
- TLP(Thread Level Parallelism)이 나옴 - 칩 안에 하나 코어 복잡하게 만들어봤자 열만 펄펄 난다. 그래서 적당한 코어를 여러 개 배치하는 방식으로 바꿈.

> 멀티코어와 멀티쓰레딩이 비슷한 건가...?

- CPU의 주소는 2개
	- 1. physical address - DRAM의 주소
	- 2. virtual address - 프로그램의 주소

- 32bit - 2^32bit -> 42억 byte -> 4GB
	- 1byte - 8bit(2^8개의 값 가능, 256)
	- 1kB - 1000byte(8000bit), 1KB - 1024bye
	- 1MB = 10^6byte
	- 1GB = 10^9byte
	 
- 컴퓨터 속도측정 -  한 프로그램 내 총 실행할 명령어 수(프로그램당 명령어) * 명령어당 사이클수(CPI) * 1 사이클 시간

### 2

- 프로그래밍 패러다임
	- 1 Impretive - how
	- 2 Procedure 
	- 3 Object-oriented
	- 4 Funcional - What


- 컴파일러 - Translation
- 인터프리터 - Translation과 동시에 바로 실행 예) 셸
- 어셈블리 - 컴파일러의 일부

- lecical -lex, syntax - yark, sementic

- 머신에 상관없이 인터미디어(중간) 코드 생성. 그 이유는 [디커플링](https://books.google.co.kr/books?id=RTJFDwAAQBAJ&pg=PA329&lpg=PA329&dq=%EC%BB%B4%EA%B3%B5+%EB%94%94%EC%BB%A4%ED%94%8C%EB%A7%81+%EB%9C%BB&source=bl&ots=DG0X3EmpQT&sig=ACfU3U36wC3fFZmodaSpB-96aLAa7cuAOg&hl=ko&sa=X&ved=2ahUKEwiCzrm--9_nAhWXHHAKHcmBATEQ6AEwBHoECAkQAQ#v=onepage&q=%EC%BB%B4%EA%B3%B5%20%EB%94%94%EC%BB%A4%ED%94%8C%EB%A7%81%20%EB%9C%BB&f=false) 때문이다. 밀접하게 연결된 두 개체를 떼어 내서 접점을 제거한다는 의미. X86이든 리스크든 어디서도 적용 가능한(?) 코드를 만들기 위해서인가? 아닌가?

- 모든 바이트는 주소를 가진다 (2의 30승 = 1기가)

- 메모리는 프로그램 입장에서 `Linear array byte`

- 주소의 크기가 프로그램 크기 결정(32비트 시대에 최대 프로그램은 4GB에 불과)

- 워드 - 연산의 기본 단위(32비트 머신은 32비트 워드)
- 워드와 레지스터 크기는 같다.

- alignment - 모든 데이터는 배수에 저장 (32비트 워드는 4의 배수에서 시작) int가 2비트인 모양

- Machine Instruction
	- 1. 계산 논리
	- 2. ALU 정수 레지스터
	- 3. Floating pointing unit 실수 레지스터

- jump - 명령 순서 변경 (브랜치)
- Oprerand - 데이터
- Input - Memory destination - 레지스터

- data transfer instruction - load, store

- IO 디바이스는 메모리 주소에 맵돼 있다. 이 주소를 포트라고 한다.

- Control Transfer Instruction - 명령어 순서를 바꿔준다. (브랜치라고도 표현)

- Program Counter - 읽어올 다음 명령어 주소를 가리킨다.

### 3

- 리스크는 프로그램 크기 커지면서 명령어 개수 적어졌지만 그만큼 코드 사이즈는 커졌다.

- 밉스 Instruction format
	- ALU
	- Data transfer instruction
	- branch instruction

- 32bit 명령어 안에 어떻게 인코딩?
	- 오퍼레이션 나타내는 필드를 오프코드?
	- 나머지 부분은 오퍼랜드를 Specify

- input, output이 되는 오퍼랜드는 모두 레지스터(인풋 2개, 아웃풋 1개)
- RS RT RD
- 소스 레지스터 RS RT 각각 소스1, 소스2
- RD Destination 레지스터
- RT 오퍼랜드 레지스터
- RRS RT 메모리 오퍼랜드

- Operation에 6비트 할당, 2의 6승인 64개의 Operation이 있군.

- Shift에는 상수를 인코딩. 이런 것을 이미디엇이라고 부름
- 레지스터는 달러로 표현, 가장 왼쪽이 데스티네이션

- 로드 - 메모리에 있는 것 레지스터로
- 스토어 - 레지스터에 있는 것 메모리로

- 베이스 어드레싱 - 레지스터 상수를 더해서
- RS 베이스 레지스터에 저장된 32비트 부소에 ㅅ어드레스를 더하는데 이 어드레스를 오프셋이라고 한다. 이것이 디스플레이스먼트

### 4

- function 실행 시에만 메모리 할당, 리턴하면 메모리 반환
- 스택 - function마다 사용할 로컬변수 저장한 공간

- 메모리 안의 스택 - 시스템 스택

- 스택 포인터는 스택의 탑을 가리키고 있다.

- 스택 프레임 - function에 국한된 공간. function마다 자신의 스택 프레임을 갖는다. activation record라고도 부름

- 연속적으로 function을 부르면 스택의 프레임이 또 할당

- 리턴은 레지스터를 통해 알려준다.
- V0, V! 레지스터, 실제로는 $2, $3. 왜 2개? 부동소숫점인 경우 떄문
- 연속적으로 function 호출하면 리턴 어드레스 레지스터를 덮어쓰니 콜하기 전에 스택에 저장. 리턴할 때는 다시 복구. 세이브드 레지스터에.












- [Amdahl's law](https://parkcheolu.tistory.com/34](https://parkcheolu.tistory.com/34) - 속도를 계산할 때  병렬화될 수 있는 부분은 병렬가능 개수로 나눈 수치 만큼 향상이 가능하다.








[위키백과](https://ko.wikipedia.org/wiki/NAND_%EA%B2%8C%EC%9D%B4%ED%8A%B8)