---
title: "시스템프로그래밍4"
date: 2020-09-03 22:26:00 -0400
categories: systemsoftware
---

- Instruction Set architecture

: 컴퓨터 프로그램과 하드웨어가 어떻게 서로 소통하는지, 원리에 대해서 배우는 것, ISA 는 그 사이 interface. MIPS는 ISA중 하나.

컴퓨터 프로그램 작성할때 , 하드웨어가 이해하지 못하므로, compiler로 highlevel language -> low level language translate 함.

low level language(machine language, assembly code) -> binary 형태로 변형시켜주는것

ISA(interface 역할)

application 기반으로 SW,HW 발전

-computer architecture

  - displacement

    - HW, SW 사이 인터페이스역할
    - 한쪽만 바껴도 의사소통 불가, 즉 호환성때문에 변경되기가어려움
    (comercial processor(x86 등등)은 legacy instruction을 현재도 작동되게끔함. 예전에만들어놓은것도 되야하므로)

  - Microarchitecture

    - ISA를 구현해놓은것
      -intel 도 x86 바탕으로 process, AMD도 x86 바탕으로 process만듬. 서로다름. Microarchitecture가 다르므로

    - intel, amd, arm , mips(임베디드에 자주사용됨)

- Operation of computer hardware

  - Instruction Set
  : 컴퓨터를 다루는 녀석
  : 다른 컴퓨터는 다른 Instruction Set(그러나 많은부분은 비슷함)

- MIPS

  - Arithmetic operation

    : two source one destination (three operands)
    ex) add a,b,c // a = b + c  
      -> opcode(add), operands(a,b,c)

    : regular 하므로 만들기 쉽고, 성능을 적은 비용에 높이기 쉽다.

    ex) f = (g + h) - (i + j);

    add $t0,g,h # temp t0 = g+h
    add $t1,i,j # temp t1= i+j
    sub f,t0,t1 # f = t0 - t1

- Operands

  - register operands

    - MIPS 에서 Arithmetic instruction은 register operands 다.
    - MIPS 에선 32x32 bit register 파일이 있음.
    - 자주 접근하는 데이터는 레지스터에 저장한다.
    - 32bit data: word 라고 한다.

    - Assembler는 이름이 있다
      - $t0,$t1 ... $t9 임시 value 저장
      - $s0,$s1 ... $s7 변수 저장

      적은양의 레지스터 -> 더빠름

        ex) f = (g + h) - (i + j);
        -> 단, f,g,h,i,j 는 $s0,$s1,$s2,$s3,$s4에 있음

        add $t0, $s1,$s2
        add $t1, $s3,$s4
        sub $s0, $t0,$t1



      - 32 registers

        - 하드웨어구현을 쉽게하려고 분리시킴.

    - Memory Operands

      - Dram이라고 알려짐, array,structure, dynamic data 가 있음
      - arithmetic operation을 하려면, 메모리에서 데이터를 레지스터로 가져와야함.
        - load : memory -> register
        - store : register -> memory

      - memory는 byte 단위로 접근
      - eoMIPS는 big endian(MSB 사용),값을 저장하는 순서가 달라짐
        -> 서로다른 ISA 라면, 데이터주고받을때 메모리표현방법이 달라 문제가생김

      ex) g = h + A[8];
        -> g는 $s1에 저장, h는 $s2에 저장, A의 시작 주소는 $s3에 있다

        lw  $t0, 32($s3)  //load word(4byte 값을 가져옴 32바이트 만큼 떨어트림)
        add $s1,$s2,$t0

      ex) A[12] = h + A[8];
        -> h 는 $s2, A 시작주소는 $s3

        lw $t0, 32($s3) // word를 레지스터로 가져온다.
        add $t0, $s2, $t0
        sw $t0, 48($s3) // t0 에있는 것을 48($s3)에 저장한다.

    - 메모리에 넣고 연산? -> 연산기는 processor 안, 레지스터접근이더빠름.
    - 컴파일러는 가능한 많이 레지스터 사용, 메모리접근(비용과다, 느림)

    전원을끄면 main memory는 날라감.

    - 결론: CPU는 주로쓰는 데이터를 register에 저장한다.

  - MIPS immediate Instructions(constant를 다루는 것)

    - addi : constant 값을 더할 떄 사용함.
    - $zero : constant 0 를 의미합니다.

    ex) register 를 이동할 때

    add   $t2   $s1   $zero
    -> s1이랑 zero 를 더해, t2에 넣는다. 즉 s1을 t2로 이동하는 듯

  - unsigned binary integer vs 2s-complement signed integer

    : 양수만 표현 vs 반은 음수, 반은 양수 표현
    : 2s complement 같은 경우 1 음수, 0 양수
      -> 가장 양수는 01111111, 가장 음수는 10000000
    : 보수화 하는것, 1->0 0->1 로 바꾸고 +1 하기
    : lb(load byte),lh(load halfword), beq,bne(비트 연장)
      -> -2: 1111 1110 -> 1111 1111 1111 1110


  - MIPS ISA
    - Register 0~31
    - PC : 내가 수행할 instruction의 메모리 주소를 가지고 있음.
    - High, Low : 32bit-> 64bit가 될경우 상위 32bit는 high, 하위 32bit는 low

    - 3가지 format이 있음(항상 32bit)
      - op, rs ,rt, rd, sa, fuct (R format)
      - op, rs, rt, immediate (I format)
      - op, jumptarget (J format)
