---
title: "컴퓨터구조"
date: 2020-10-21 22:26:00 -0400
categories: computersystem
---

- Character Data
  : 'a'라는 숫자가 매핑되어있는 ASCII 코드를 이해, 한글,기호는 UNICODE 사용

->  sign extended
  - lb rt, offset(rs)
    : 한 바이트를 읽어옴

  - lh rt, offset(rs)
    :16 bit data를 읽어옴

-> zero extended
  - lbu rt, offset(rs)
    :unsigned 한 바이트를 읽어옴

  - lhu rt, offset(rs)
    :unsigned 16 bit data를 읽어옴

  - sb rt, offset(rs)
    :한 바이트를 저장한다.

  - sh rt, offset(rs)
    : 16 bit data를 저장한다.

- 32bit immediate and addresses

  - 32bit constant
    : 16 bit가 사실 적당하다.
      가끔 32bit가 필요할 수도 있다.

      ex) 0000 0000 0011 1101 0000 1001 0000 0000 이면
      절반으로 나누어

      lui $s0, 61 // 61= 0000 0000 0011 1101
      ori $s0, $s0, 2304 // 2304 = 0000 1001 0000 0000

      lui: 61이라는 값을 $s0에 저장
      ori: 상위 16bit는 61저장하고, 하위 16bit에는 2304 저장후 전체 결과 $s0 에 저장.

  - I format (op, rs, rt, constant_or_address)

    - PC-relative addressing
      : 현재 내가 수행하고 있는 레지스터 주소를 가리킴
        - Target address = PC + offset x4
          * offset : 내가 현재 있는 instruction에서 다음 instruction 까지의 거리.

  - J format(op,address)

    - 26bit는 32bit를 다 표현하지 못함,
      -> 26bit를 x4를 하면(left shift 2번) = 28 bit로 표현이 가능하다.
      -> 나머지 4bit는 PC 31~28번째(상위4개)에서 가져옴.

      => PC 상위 4bit는 같아야됨

    - ex) MIPS code

      * loop의 시작은  80000

        Loop: sll   $t1,  $t3,  2 // 80000   R(0,0,19,9,2,0)
              add   $t1,  $t1,  $s6 // 80004  R(0,9,22,9,0,32)
              lw    $t0,  0($t1) // 80008   I(35,9,8,0) -> t1(8번레지스터 값)을 읽어서 0만큼 떨어진곳을 읽어 t0에 올라올수 있게 함
              bne   $t0,  $s5,  Exit // 80012    I(5,8,21,2) //2: PC기준으로 얼마만큼 떨어져있는지 확인 , PC는 addi일것이므로 2칸떨어진 Exit이 저장됨
              addi  $s3,  $s3,  1 // 80016  
              j     Loop // 80020 J(2,20000) // PC 상위 4bit를 가져옴, 0가져옴, 20000 * 4 = 80000이므로 

        Exit: ... //80024
