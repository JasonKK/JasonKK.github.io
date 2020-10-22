---
title: "시스템프로그래밍4"
date: 2020-09-03 22:26:00 -0400
categories: systemsoftware
---

XE 버전

3개의 record를 만드는 과정에서 instruction format, addressing mode를 영향을 받음

1. indirect addressing

  - "@" 를 operand에 추가함.

  ex) J   @RETADR (jump라는 instruction에 indirect addressing 함)

2. immediate Addressing

  - "#" 를 operand에 추가함

  ex) Comp #0(immediate addressing 으로 사용)

3. Format 4

  - "+"를 operation code에 붙임

  ex) +JSUB RDREC

4. Base relative Addressing(directive 만)

  - register B를 사용함

  ex) BASE LENGTH


  * line number : 5씩 증가가 되나, 추후에 사이사이에 다른 instruction이 들어오는 것을 대비함.


  ex) +LDT #4096 : format 4(+), immediate addressing(#) 이다.

-> SIC, SIC/XE 의 차이 : Location counter가 0000부터 사용함.(relative 함)
                    : Start라는 directive 는 프로그램 시작주소, 프로그램 명을 의미함. (header record 표현)
                  ex) 0000 START 0 -> 시작주소가 0이 아닌 상대적인 주소, loader가 재배치를 함.
                    : register 와 resgister 간 instruction이 있음(format 2)
                    -> 2byte 라 사이즈도 작고, register 간 연산이라 memory fetch가 없음

                  ex) COMP ZERO -> COMPR A,S
                    : immediate(value를 직접 operand에 써줌)&indirect addressing 을 많이 사용함.
                    -> translation time 을 줄이기위해! 성능 상승!


- Translation

  - register to register instruction

    - 해당하는 operation code를 machine language 로 바꾸고 , register에 해당하는 것을 바꿈
    - XE에서 제공하는 register 이름이 있지만, 일반적으로는 symbol table을 사용한다.

    ex) 1049 COMPR A,S  A004


  - register to memory instruction

    - relative addressing mode

        - TA = PC or B + displacement

        * base relative: (disp: 0~4095), program counter relative: (disp: -2048~2047)
        -> 12 bit 로 표현해야함

  ex) 0000  FIRST   STL   RETADR  17202D
      .
      .
      .
      0030  RETADDR RESW  1

      -> STL 에 해당하는 opcode : 14, n=1, i=1  ==> 17
         x=0,b=0,p=1,e=0  ==> 2
         PC는 3(0000에서 다음 instruction 은 0003), p flag p =1 , 0030 - 3 = 002D(hexa)

  ex) 0006  CLOOP +JSUB RDREC
      .
      .
      .
      0017        J     CLOOP  3F2FEC

      -> +JSUB : format 4,
          0017 : location counter ,

    - Base relative addressing

      - base register 를 이용, 프로그래머가 base register를 어떻게 제어하느냐에 영향을 받음, assembly를 작성하는 프로그래머 어셈블러에게 어떤값이 들어있어야하는지 알려줘야 그래야 어셈블러가 TA, base register 값을 사용해서 displacement 값을 계산함.

      - 이걸 위해 assembler directive BASE 를 사용함

      ex) BASE LENGTH 라는 뜻은, base register가 address 의 길이를 담고 있음


  * PC 는 프로그래머가 제어 X , Base register 는 프로그래머가 제어할 수 있음

      ex) 0003          LDB   #LENGTH
                        Base  LENGTH
                        .
                        .
          0033  LENGTH  RESW  1
          0036  BUFFER  RESB  4096
          .
          .
          104E          STCH  BUFFER,X    57C003


          -> B register 에 immediate addressing 으로 값을 넣음.
          length의 address 값은 0033 을 B register 에 넣으라 하고 있음. Buffer의 주소값은 0036, 현재 B register 에 있는 값은0033
          0036 -0033 = 0003 , index addressing, base relative 이므로 flag x, b =1 그러면 이 값이  C 가 됨, STCH (54, n,i=1 =57) , x,b=1 -> 1100 -> C , displacement = TA값(buffer의 주소) - B의 값

- Format 4 이면 displacement 계산 안해도됨. (+로 표현해야함 -> 1.PC 2.Base relative 우선으로 파악함  )

 -> address 값이 바로 표현됨

- immediate Addressing

  - memory reference 를 하지 않음
  - assembler 입장에서는 immediate operand에 들어있는 값을 상응하는 값으로 바꿔주고 instruction에 해당하는 부분에다가 값을 넣어 주면 됨.

  ex) 0020    LDA   #3    010003
  -> 3이라는 값을 바로 넣어줌.
  (Extension 예제)
  ex) 103C    +LDT  #4096 75101000

  만약 operand가 20bit보다 더크면 -> immediate addressing 사용불가!

- Program Relocation(프로그램 조정)

: multiprogramming(하나 이상의 프로그램을 사용할 떄 ) 할때 유용
: 메모리의 공간이 있을때 적절히 활용
: assembler 입장에서는 직접 메모리 로딩을 안하고 loader가 하기, 실제 시작 주소를 모름.

시작주소에 따라서 실제 instruction에 들어가는 address 값이 달라짐.
-> 이게 실제주소가 반영이 되면 바뀔수 있다는 정보를 담아주면 됨(program relocation)
-> 시작주소로 부터 얼만큼 떨어져있다는 주소를 만들면 되고, loader가 begin address 를 알았을때 더하면 됨. 이정보를 object program 에 담음.

modification record

- 1. col 1 : M
  2. col 2-7 : 수정이 필요한 시작주소(20 bit address field의 시작주소)
  3. col 8-9 : 얼만큼 수정되야하는지를 담는다(half byte)

  -> main memory가 들어있는 경우에만 해당함.
  (register to register, displacement값을 가지고 있는 relative addressing은 필요가 없음, format 4만 해당)

ex) 4B101036  ->    M00000705+COPY
    4B10105D  ->    M00001405+COPY
    4B10105D  ->    M00002705+COPY

    ->실제 modification이 있어야하는 부분은 01036 -> 00007
      05라고 하는것은, half byte이므로 5*4=20bit 를 바꿔주라
      -> 01036 을 바꾸라는 의미


정리: assembler 가 object code 를 만들어냄, 만들어내는 과정에서 machine architecture에 따른 , instruction 별로 addressing mode와 format을 고려해 바꿔야함 -> 실제로 주소값이 아닐땐 loader가 반영할수 있도록 정보를 포함해준다(M record, 어디서부터 얼만큼까지 바꿔야하는지)
