---
title: "시스템프로그래밍"
date: 2020-09-03 22:26:00 -0400
categories: systemsoftware
---

시스템 소프트웨어는 계속 발전하지만, 거의 기본적인 구조는 유지되고 있습니다.

- 간단한 컴퓨터 구조

  - 메모리 유닛
  : 프로그램(명령어)과 데이터를 동시 저장

  - CPU
  : 프로그램을 실행하는 역할

  - ALU
  : 수학적 논리적 연산을 수행하는 역할

  - 컨트롤 유닛
  : 제어하는 역할, 주로 CPU , input/output 장치 제어
  : Fetch-Decode-Execute cycle 존재

    * Fetch-Decode-Execute cycle 란?
    : Fetch(메모리에 저장되있는 instruction 들을 CPU register 로 가져옴)
      Decode(무엇을 해야하는지 결정을 함)
      Execute(Decode에 맞춰서 실행을 함)


이 과목 실제 컴퓨터에서 필요없는 부분(성능 향상을 위한 부분 등등)을 가상 컴퓨터를 사용합니다.

크게 2가지, SIC 과 SIC/XE 가 있습니다.

SIC

- 메모리는 결국 하나의 일차원 배열입니다. , 24bit로 고정된 cell을 사용합니다. access는 메모리 주소로 직접 할 수 있습니다. 메인메모리에는 프로그램과 데이터가 같이 들어가 있습니다. SIC에서는 24bit로 되어있으며 , 8bit(word)가 3개 , 3byte words 로 되어있습니다. Maximum memory는 32KB입니다.

- 레지스터같은 경우 CPU 내부에 존재하며, 임시 데이터 저장을 위해 쓰입니다. 어떤 레지스터는 특정 목적에 맞게 사용하는 레지스터가 있습니다. 나머지는 일반적으로 사용됩니다.

- SIC 에서의 레지스터는, 특정한 목적을 가진 5가지 레지스터가 사용됩니다. (*여기서 (num) num은 레지스터 A가 binary로 바뀔때, assembler가 assembly code로 바꿀때 사용되는 숫자입니다.)

  - A(0) : Accumulator(덧셈 뺄셈)
  - X(1) : Index register(Addressing mode에 특화됨)
  - L(2) : Linkage register
  - PC(8) : CPU 내에 fetch해올 다음 instruction을 저장하기 위한 register
  - SW(9) : Status Word(instruction 수행중 조건의 상태를 저장하기 위한 register)

- Data format은 integer, character 사용가능합니다. 소수점은 사용할 수 없습니다.

- Machine Instruction format :
모든 instruction들은 24bit 이며, (끄림첨부 필요)

8bit opcode(instruction이 뭘 하는 건지)
1bit x(flag: addressing mode를 위해 사용)
15bit address(데이터 값, 데이터가 저장되어있는 주소값)

*addressing mode : data를 fetch해오기위한 그 메모리 주소값(target memory address)

- SIC 에서는 두가지 addressing mode 가 있습니다.
x=0 , direct addressing mode이면 뒤에 address 값이 바로 메인 메모리에서 fetch 해올 target address 입니다.

x=1, indexed addressing mode이면, address + (X register)를 할시 target address 가 나옵니다

- Instruction set : 특정한 CPU가 처리할 수 있는 모든 instruction들의 집합

SIC 에서는
  -Load or Store instruction :  LDA, LDX, STA, STX ...
    - LDA m? A <- (m .. m+2)

    m에서 m+2까지 부분을 register A로 읽어온다. (*instruction은 3byte 이므로)

    - STA m? m..m+2 <-(A)

    register A에 들어있는 값을 m~m+2 메모리부분에 저장

  -Integer Arithmetic operation(ADD,SUB,MUL,DIV)
    - 사용하는 register는 A 다!

  - Comparison(COMP)
    - COMP m? (A) : (m..m+2)

    register A의 값과 메모리에들어있는 word 값을 비교

  - Conditional Jump(JLT,JEQ,JGT)
    - JLT m? PC <- m if CC set to <

    - JEQ m? PC <- m if CC set to =

    - JGT m? PC <- m if CC set to >

  - Subroutine Linkage(JSUB, RSUB)
    - JSUB m? L<-(PC); PC<-m

      특정한 subroutine 으로 jump , 실행이 끝나면 실행했던 위치로 돌아간다.

      현재 들어있는 PC 값을 register L에 저장하고, PC값에는 주소값 m을 넣는다.

    - RSUB ? PC <- (L)

      register L에 들어있는 값을 다시 PC에 넣어 원상태로 돌아온다.

    - I/O(Input and Output)

      1 byte 씩 이동하면서 실행된다.

      - TD(test device)

        TD m

        데이터를 주고받을 준비가 되었는지 테스트

        RD m

        m으로 정의되어있는 디바이스로 부터 데이터를 가져와 A에 가져다 씀.

        WD m

        m으로 정의 되어있는 디바이스로 부터 데이터를 가져
