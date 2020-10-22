---
title: "시스템프로그래밍3"
date: 2020-09-03 22:26:00 -0400
categories: systemsoftware
---

- C program -> Assembly language program -> Object Machine language modeule -> executable  (그림 첨부))

- Assembler : 시스템 소프트웨어의 일종, 어셈블리랭귀지로 작성된, 코드로 변환이 되었는데  machine 코드로 바꾸는 것(xxx.o) 확장자가 붙은

- 컴퓨터가 이해할 수 있는 symbolic form
- 어셈블러가 처리해야할 기본적 function이 있음.


assembler directive
: assembler에게 특정행동을 하라고 알려주는 명령어

START : 프로그램 이름과 시작주소를 알려줌
END : 프로그램의 끝을 나타, 뒤에 이프로그램의 처음 실행해야할 instruction 표현함
BYTE : character나 constant 값을 생성
WORD : 1 단어의 integer 생
RESB : reserve byte, 메모리 공간을 지정된 byte 만큼 잡음
RESW : reserve word, 메모리 공간을 지정된 word 만큼 잡음

main routine : input device 로부터 record를 읽고, output device에 가져다 씀

subroutine

- RDREC : input device에서 record를 읽어와서 buffer에 가져다 씀
- WRREC : buffer에 쓰여져 있는 record를 output device에 가져다 씀

instruction RD,WD는 하나씩 가져온다.

record의 마지막은 null character

--------------------------------------
- Object code를 어셈블러가 어떤알고리즘에 의해서 만들어내는지 확인!

- Loc : machine address(hexa) 를 나타냄, 1000 이라고 가정함.
Sic standard가 3byte이므로 3byte씩 loc이 바

    ex) STL RETADR =-> object code(141033)

instruction 변환 과정

 1. convert mnemonic operation code to machine language equivalents
 (opcode를 거기에 상응하는 hexa 값으로 변환)
 ex) STL instruction

 2. convert symbolic operands to equivalent machine address
 (operand를 machine address로 변환)
 ex) RETADDR(return address) to 1033

 3. build the machine instruction in the proper Format
 (instruction을 포멧에 맡게 빌드)

 4. data constant값을 internal machine representation에 맞게 바꿔줌
 (EOF-> 454F46)

 5. object program and the assembly listing
  (object program 을 별도에 file 에가져다 써서, output file 로 만든다)


---> assembler가 소스 프로그램 하는 과정

-> 1st Pass: source program 을 한번 스캔한다, label에 대해 정의하고 주소를 찾는다

-> 2nd Pass: 실제 translation이 일어난다.

object program : assembler 가 만드는 output program

-> header record : 프로그램 이름, 시작주소, 프로그램 길이

  co1 1. H
  col 2-7 프로그램 이름
  col 8-13 프로그램 시작주소(hexa)
  col 14-19 프로그램 길이(hexa)
-> text record : 변환된 instruction, data를 포함하면서 실제 machine 주소값을 포함하는 형태

co1 1. T
col 2-7 이 record의 시작주소
col 8-9 이 record의 길이
col 9-69 object code

-> end record : object program의 마지막을 의미하고, 실제 실행이 되야하는 첫번째 instruction 을 담고 있음.

co1 1. E
col 2-7 object code에 실행가능 첫번째 instruction의 주소

* RESW,RESB는 instruction이라 object code 가 생기지 않지만, word, byte는 해당하는 constant를 잡기에 object code로 변환됨

* object code 가 없는 부분은 loader 예약해두는 공간이다.

- UNIX 시스템

  - Header : size, position을 알려줌

  - Text Segment : machine language 에 해당함( text record)

  - Static Data Segment : 고정되어있는 데이터

  - Dynamic Data Segment : Stack 구조로 되어있음

  - Relocation information : 시작주소로 부터 상대적으로 로딩해야하므로,

  - Symbol Table : label 정보가 담겨있음

  - Debuggind information : 디버그 관련 담겨져 있음

(forward reference 문제를 해결하기위해 2pass 알고리즘 사용)
- Pass 1

: 1. 라인 별로 읽음
  2. address를 할당함(loc)
  3. symbol table의 label에대한 address값을 저장함. 실제사용은 pass2
  4. assembler directive에 대한 처리( 상수선언(byte,word), 메모리 할당(RESB,RESW) 등등)

ex)
1. input line의 첫번째를 읽음
2. OPCODE 가 START인지 비교
3. START 면 시작주소를 저장후 location counter(LOCCTR)를 시작주소로 하고, intermediate file을 만듬
4. 아니면 locctr을 0으로 만듬
  * Pass1은 intermediate file에다가 결과물을 가져다가 사용함.

- Pass 2

: 1. 1st Pass에서 만든 file을 input으로 사용함
  2. mnumonic operation code -> machine language(OP code table)
  3. label -> address(symbol Table)
  4. 1st pass에서 처리못한 assembler directive 처리
  5. object program 생성

ex)
1. intermediate file을 input file로 읽음



- Operational Code Table(OPTAB)- 절대 불변

: opcode에 해당하는 Instruction명과 해당하는 부록

SIC은 fixed size, XE는 여러개의 size

Pass 1. look up을 하고, 해당하는 opcode를 찾을 수 있는지
Pass 2. 실질적인 translation이 일어남.

일반적으로 hash table, key로 쉽게 검색이 가능함.

- Symbol table(SYMTAB)

: symbol의 이름(name), 주소(value)를 가짐.

Pass 1. label을 만날시 SYMTAB에 검색을 해서, 있으면 가져오고 없으면 insert함
Pass 2. look up 을함 , 실질적인 address를 바꿈.

일반적으로 hash table, insertion과 retrieval이 있음.
