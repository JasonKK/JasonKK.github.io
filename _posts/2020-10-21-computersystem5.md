---
title: "컴퓨터구조"
date: 2020-10-21 22:26:00 -0400
categories: computersystem
---
- 프로그램이 어떻게 시작되는지!

- Translating and Starting a program.

  Compiler producing object module directly

  - C program -> compiler(translation) -> assembly language program -> assembler(1:1 mapping) -> maching language module

  Static linking

  - maching language module -> linker -> excutable machine language program -> loader -> memory -> 운영체제에 의해 수행

  ex) Excel 사용 : Excel binary 만들어져 하드에 저장되어있음. 클릭할시 binary가 memory에 올라오고 운영체제가 실행.

  * Static linking : 미리 linking을 다 끝내놓는것
  * dynamic linking : 프로그램 실행 중간에 필요할 때마다 linking 작업을 함.
                      ex) printf 를 사용했을때, stdio에 저장되어 있으므로 include 함. 불렀을 때 linking 을 함.

                      -> 내가 사용하는 부분에서만 만드므로, 이미지 실행 파일이 크지 않음

                      -> static은 과정이 다해놨으므로 프로그램 수행이 더 빠름, dynamic은 함수 처음쓰일때 특히 오래걸림

  - Pseudo Instruction

    - ex) move $t0, $t1
          -> add $t0, $zero, $t1

    - ex2) blt  $t0,  $t1,  L (t0,t1비교해서 작으면 L로 점프)
          -> slt $at, $t0, $t1 (t0,t1비교한 값 저장)
             bne $at, $zero,  L (저장한값이랑 zero랑 비교)

  - Object module 만들기

    - assembler가 machine instruction으로 바꿈.

      - header : module 마다 있음.description 존재
      - text segment : code에 대한 segment
      - static data segment : static 한것에 대한 segment
      - relocation info : 메모리의 상대적위치를 알고 파악하는데 쓰임.
      - symbol table : 메모리 주소 테이블
      -  debug info  : 디버깅 정보

     - Linking 작업들

      - segment 합치기
      - label을 resolve
      - 외부 파일 변수 참조할때 메모리 위치

  - 프로그램을 로딩한다는 것

    - 이미지파일이 디스크에 있고 메모리 파일에 놓는 것.

      - header를 읽어 프로그램의 segment size가 얼마인지 알아내
      - 프로그램이 돌아갈 virtual address space 를 만듬.
        (컴퓨터안에 나혼자만 쓴다는 착각 불러옴)
      - text segment, initialized data를 메모리에 넣음
      - argument는 stack에 넣음
      - 레지스터 초기화
      - 시작함수를 통해 main 불러오고 끝났다면 exit을 만들어 끝난 것 확인



  - relocate code and data

    - relocatable object file -> executable object file

      - .text, .data 로 나누어서 생각

        * bss: 초기화되지 않은것
        * data: global 한 값


    - 실행 한다는 것 -> loader를 통해 memory 상에 올려놓음

      - process에 stack, static data, text, reserved 로 나누어지며, code는 text, data는 static data에 저장된다.

      - process는 PC가 가리키는 영역에서부터 하나의 instruction 32bit씩 가져와서 무슨 instruction인지 알기위해 상위 6bit opcode로 format 확인


    - ex) MIPS code

            .data
    array:  .word   3 // word type 3 저장
            .word   123 // word type 123 저장
            .word   4346 // word type 4346 저장
    array2: .word   0x11111111 // word type 16진수 저장
            .text  // main 시작, code 부분
    main:
            addiu   $2,  $0, 1024
            addu    $3,  $2, $2
            or      $4, $3, $2
            sll     $6, $5, 16
            addiu   $7, $6, 9999
            subu    $8, $7, $2
            nor     $9, $4, $3
            ori     $10, $2,  255
            srl     $11, $6, 5
            la      $3, array2
            and     $13, $11, $5
            andi    $14, $4, 100
            lui     $17, 100
            addiu   $2, $0, 0xa

        * .data -> static data, .text-> text 부분에 저장
        *  PC는 addiu 주소 저장
        *  static data 의 시작 주소는 1000 0000, 즉 .word 3 일것임.

  - Array, pointer

      -  array는 index를 가지고 움직임, pointer는 메모리 주소로 이동함.

      - ex) C code

          clear1(int array[], int size)
          {
            int i;
            for(i=0;i<size;i+=1)
            array[i]=0;
          }

              MIPS code

              move $t0, $zero // i = 0
      loop1:  sll  $t1, $t0, 2 // $t1 = i * 4
              add  $t2, $a0, $t1 // $t2 = &array[i]
              sw   $zero, 0($s2) // array[i] = 0
              addi $t0, $t0, 1 // i = i + 1
              slt  $t3, $t0, $a1 // $s3 = (i < size)
              bne  $t3, $zero, loop1 // if() goto loop1

              * $a0는 array의 시작주소를 나타냄
              * $a1은 size를 의미함

        - ex2) C code

            clear2(int *array, int size)
            {
              int *p;
              for(p= &array[0]; p< &array[size]; p=p+1)
              *p=0;

            }

              MIPS code

              move $t0, $a0 // p = &array[0]
              sll  $t1, $t1, 2 // $t1 = size * 4
              add  $t2, $a0, $t1 // $t2 = &array[size]
    loop2:    sw   $zero, 0($t0) // Memory[p] = 0
              addi $t0, $t0, 4 // p = p + 4
              slt  $t3, $t0, $t2 // $t3 = (p<&array[size])
              bne  $t3, $zero, loop1 // if() goto loop1

              * loop가 적으므로 최적화
