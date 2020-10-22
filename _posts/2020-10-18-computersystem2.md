---
title: "컴퓨터구조"
date: 2020-10-18 22:26:00 -0400
categories: computersystem
---

- Instructions
  - binary로 인코딩된 것 -> machine code

  - R format

    - op rs rt rd shamt funct

      - op(6bits) : operation code
      - rs(5bits) : 첫번째 register 번호
      - rt(5bits) : 두번째 register 번호
      - rd(5bits) : 목적지 register 번호
      * register는 총 32가지이므로, 2^5, 즉 5bits 로 다표현가능
      - shamt(5bits) : 얼마만큼 shift 할건가를 결(현재는 00000)

        -> 왼쪽으로 shift 할경우(sll)
          - 왼쪽으로 bit를 옮긴 후 0으로 채운다
          - sll은 i개의 비트 2^i 곱한다.
            ex) 2 -> 4
                0010 ->  0100 (1bit 움직일때 2를 곱한다.)

        ->  오른쪽으로 shift 할 경우(srl)
          - 오른쪽으로 bit를 옮긴 후 0으로 채운다
          - srl은 i개의 비트 2^i로 나눈다
          ex) 4 -> 2
              0100 ->  0010 (1bit 움직일때 2를 나눈다.)

      - funct(6bits) : 현재 op는 64개의 operation 만 표현할 수 있으므로, extend 함

      - ex) add $t0, $s1, $s2
        ->  add opcode = 0, funct = 0x20 이므로
            "0, $1, $s2, $t0, 0, 32"
            "0, 17,  18,   8, 0, 32"
            "000000, 10001, 10010, 01000,00000,100000"
            "0x02324020" -> 하드웨어로 들어감

  - I format

      - op rs rt constant_or_address

        - op(6bits) : operation code
        - rs(5bits) : 메모리 어느위치에서 읽어올지 표현
        - rt(5bits) : 넣어주는 레지스터 표현
        - constant_or_address(16bits) : 상수거나 주소를 표현함
          -> 대표적 load/store
            : 레지스터에 접근하려면 레지스터 주소를 알아야하고, 메모리에 접근하려면 메모리의 주소를 알아야함.
          -> 상수 같은 경우 -2^15~2^15-1까지 표현 가능

  - J format

      - op address

          - op(6bits) : operation code
          - address(26bits)
            -> 대표적 jump(j and j)
              : 어디로 이동할지 메모리 주소를 담는 것

- Stored program computer

    - processor(연산장치) 와 memory(기억장치)가 있다.
      -> instruction과 data는 메모리안에 존재,
      -> ssd 등 저장장치-> 메모리 -> processor -> 연산 -> 다시 메모리

- logic operation

  ---
  sll : Shift left
  srl : Shift Right
  and, andi : AND
  or, ori : OR
  nor : NOT
  ---

  and 같은 경우 : 0 , 1이 나오면 1로 나온다.
  xor : 둘다 0, 1 -> 0
        하나라도 1 -> 1
  not : 0 -> 1 ,1 -> 0

- Instruction for making decision

  - Conditional operation

    - beq
      ex) beq rs, rt, L1
        -> if(rs == rt), L1 label로 jump

    - bne
      ex) bne rs, rt, L1
        -> if(rs != rt), L1 label로 jump

    - J
      ex) J L1
        -> 조건없이 L1 label로 jump

    - ex) C code

        if(i == j)
          f = g + h;
        else
          f = g - h;

        * f, g, h, i, j 는 $s0, $s1, $s2, $s3 그리고 $s4에 저장되어있다.

        MIPS code 변환시
        ->    bne $s3 $s4 Else
              add $s0 $s1 $s2
              J   Exit
        Else: sub $s0 $s1 $s2
        Exit: ...

    - slt
      ex) slt   rd, rs, rt
        -> if(rs<rt) rd =1, else rd =0  

    - slti
      ex) slti rt, rs, constant
        -> if(rs<constant) rt =1, else rt =0

      ex) slt $t0, $s1, $s2 // if($s1<$s2) t0 =1,
          bne $t0, $zero, L1 // branch to L1

      ex) signed라면( '-' 인식)

          $s0 = 1111 1111 1111 1111 1111 1111 1111 1111
          $s1 = 0000 0000 0000 0000 0000 0000 0000 0001

          slt   $t0, $s0, $s1
              -1  <  +1 => $t0 = 1

          unsigned 라면( '-' X)

          sltu  $t0, $s0, $s1
              +4,294,967,295 > +1 => $t0 =0

  - Loop 하는법

    - ex) C code

        while (save[i] == k)
          i += 1;

        * i는 $s3, k는 $s5, save의 주소는 $s6
          save 배열은 word 타입

        MIPS code 변환
        ->   Loop:    sll $t1, $s3, 2
                      add $t1, $t1, $s6 //$t1 = &save[i]
                      lw  $t0, 0($t1) // $t0= save[i]
                      bne $t0, $s5, Exit
                      addi $s3, $s3, 1
                      j Loop
             Exit:    ....

             s3값을 left shift 2bit 한 후 t1에 저장, ex) 0-> 4 , word는 4byte단위이므로, shift 2번으로 4byte를 곱해짐.
             그런다음 시작주소 + 4byte 곱해진 i를 해서 t1에 저장
             그러고 그걸 load를 t0에 저장

  - Basic Blocks

      - 중간에 이동하게 없는 block(branch X)
      - compiler는 코드를 basic block 단위로 자름.

  하드웨어의 효율성을 위해서는 beq,bne를 자주사용(blt,bge X)
