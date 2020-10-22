---
title: "컴퓨터구조3"
date: 2020-10-18 22:26:00 -0400
categories: computersystem
---

- Supporting Procedures in Computer hardware

  - Procedure를 부르는 것

    1. Place parameters in registers(parameter를 레지스터에 넣음)

      - $a0 ~ $a3 : 4개의 레지스터 argument를 이용해서 parameter 읽고 전달

    2. Transfer control to Procedure(내가 현재 수행하고 있는 함수에 호출하는 함수로 control 이동)

    3. Acquire storage for Procedure(procedure에서 필요한 공간을 메모리에 할당, 함수 안에서 사용할 레지스터의 공간도 확보함)

    4. Perform procedure's operations(procedure operation 수행)

    5. Place result in register for caller(리턴값이 있을 경우, caller에게 전달하겠금 올려놓음)

    - $v0 ~ $v1 : caller 가 접근할 수 있는 곳에 callee가 리턴함.

    6. Return to place of call(control을 caller에게 다시 줌.)

    - $ra : 이 return address를 참조해서 돌아감.


  - Procedure call Instruction

    - jal(Procedure call)
      ex) jal ProcedureLabel
        -> 다음 레지스터의 주소를 $ra에 저장, jump 수행후 $ra로 다시 돌아옴.

    - jr(Procedure return)
      ex) jr  $ra
        -> $ra를 PC에 복사

    - ex) C code

    int leaf (int g,h,i,j)
    {
      int f;
      f = (g + h) - (i + j)
      return f;
    }

    * argument g,h,i,j 는 $a0,$a1,$a2,$a3에 있다.
    * f는 $s0에 있다($s0는 스택에 넣어야한다.)
    -> $s는 다 찰수 있으므로, 레지스터 부족현상이 일어날수 있어 메모리에 저장
    * 결과물은 $v0

    MIPS code

    leaf_example :
          addi  $sp,  $sp,  -12  // stack pointer 에서 12(4바이트 3개 필요하므로)만큼 빼준뒤
          sw    $t1,  8($sp) // sp-12 에서 8만큼 올라가 $t1 저장
          sw    $t0,  4($sp) // sp-12 에서 4만큼 올라가 $t0 저장
          sw    $s0,  0($sp) // sp-12 에서 0만큼 올라가 $s0 저장
          // $s1 , $t0, $s0를 stack에 넣는다

          add   $t0,  $a0,  $a1 //(g+h)
          add   $t1,  $a2,  $a3 //(i+j)
          sub   $s0,  $t0,  $t1
          add   $v0,  $s0,  $zero // (s0-> v0에 저장)

          lw    $s0,  0($sp) // 가장 나중에 넣어준걸 먼저 빼낸다
          lw    $t0,  4($sp)
          lw    $t1,  8($sp)
          addi  $sp,  $sp,  12  // SP는 원상복귀!

          jr    $ra (다시 돌아감)

    단순화 MIPS code :
          addi  $sp,  $sp,  -4
          sw    $s0,  0($sp)// $t 함수는 함수 진행동안 사라지지 않으므로, $s0만 선언한다.

          add   $t0,  $a0,  $a1
          add   $t1,  $a2,  $a3
          sub   $s0,  $t0,  $t1
          add   $v0,  $s0,  $zero

          lw    $s0,  0($sp)
          addi  $sp,  $sp,  4
          jr    $ra

          -> 메모리의 읽고 쓰고 하는 작업은, 오래걸리는 operation이므로 줄이는게 가장 좋음!

- Non-leaf Procedure

  : procedure 안에서 또다른 procedure 호출

  - ex) C code

  int fact (int n)
  {
      if(n < 1) return 1;
      else return n * fact(n-1);
  }

  * n은 $a0에 저장
  * result 는 $v0에 저장

    MIPS code:
        fact: addi  $sp,  $sp, -8 // stack 2 칸 만들기
              sw    $ra,  4($sp) //리턴 어드레스 넣기
              sw    $a0,  0($sp) //$a0 넣기

              slti  $t0,  $a0,  1 // $a0< 1 test, 맞으면 $t0=1 틀리면 $t0=0
              beq   $t0,  $zero L1 // 틀리면 L1으로 감

              addi  $v0,  $zero 1 // 결과값은 1
              addi  $sp,  $sp,  8 // stack pointer 재조정
              jr    $ra

        L1:   addi  $a0,  $a0,  -1 // a0값을 -1 해준다
              jal   fact // fact 함수 불러준다

              lw    $a0,  0($sp) //기존 n값을 가져온다.
              lw    $ra,  4($sp) // 리턴할 주소를 가져온다.
              addi  $sp,  $sp,  8  // 스텍 정상 배치

              mul   $v0,  $a0,  $v0 //a0==n * v0==fact(n-1) 을 곱해준다.

              jr    $ra
