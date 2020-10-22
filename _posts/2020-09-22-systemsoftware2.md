---
title: "시스템프로그래밍2"
date: 2020-09-03 22:26:00 -0400
categories: systemsoftware
---

-SIC/XE

  Memory : 1MB

  Register : 5 register in SIC + 4개 더(B,S,T, and F)
    - B(3) : Base register- addressing mode 에 사용됨
    - S(4), T(5) : 일반적인 목적으로 사용되는 Register
    - F(6) : floating point를 사용가능합니다.

  Data Format : integer, character 로표현됨.
              + float도 사용됨.

  Instruction Format :

        1. Format 1 : No memory reference( 1byte)
          -> 메모리에서 값을 불러오지않음

        2. Format 2 : No memory reference (2 byte)
                      for register operation
          -> 메모리에서 값을 불러오지않음

        3. Format 3 : Relative addressing(3 bytes)

            - flag e = 0

        4. Format 4 : Address field extension to 20 bits( 4 bytes)

            - flag e =  1

    Addressing Modes(format 3,4만 해당) :

        1. Relative addressing(상대 주소값) (Format 3)

          -> 기본이되는 base로 부터 얼만큼 떨어져 있는가

          1. Base reletive Addressing

            - flag b=1, flag p= 0: TA = (B) + disp/address

          2. PC relative Addressing
          *PC는 다음 주소값을 가지고 있는 Register

            - flage b=0, p=1: TA = (PC) + disp/address

          3. Direct addressing

            - flag b=p=0: TA:= disp/address

        * index addressing(flag x) 와 같이 사용 가능함
          ex) x=1 일시, X register에 있는 것 또한 더해서 TA를 구한다.

        *  Relative address 사용하는 이유?

          -> 메모리의 주소사이즈 만큼 address field에 표현하기엔 너무 많음.
            상대주소로 표현하기 힘들땐 format 4를 사용한다.


          4.  Immediate addressing

            - flag i=1,n=0

            -> value값을 직접 표현한

          5.  Indirect addressing

            - flag i=0,n=1

            -> TA의 값 자체가 또다른 메모리 주소라, 그 주소로 찾아가서 해당 값을 가져옴

            * indexing(flag x)는 immediate 과 indirect에선 사용하지 않음


          6. SIC simple Addressing

            - flag i=0,n=0

            * flag b,p,e 는 address field 로 사용된다 -> SIC 버전이기 때문

          7. SIC/XE simple addressing

            - flag i=1,n=1


      Instruction Set

          - SIC에서 사용된건 다 사용됨.
          - B register 관 데이터 전송 가능(LDB.STB..)
          - floating point 연산 가능(ADDF,SUBF,MULF,DIVF...)
          - register to register 가능(format 2)
            (RMO,ADDR,SUBR,MULR,DIVR....)
          - supervisor call
            : os로 시스템 콜




      Assembly language statement


        - Instruction

        - directive

          -> assembler가 어떤 액션을 취해야하는지 알려주는 것

        - macro

        Actual machine architecture

        - CISC(Complex Instruction Set Computers)

          -복잡

        - RISC()

          -단순
