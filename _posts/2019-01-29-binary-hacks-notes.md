---
layout: post
title:  "Notes while reading \"Binary Hacks\""
date:   2019-01-29 16:05:15 +0900
categories: life
tags: linux
---

![이미지](https://books.google.co.kr/books/content?id=vKqYMQAACAAJ&printsec=frontcover&img=1&zoom=1&imgtk=AFLRE70Az-m-T2GujWn__UMIjsHINubX1pvcAg5ZBG7HGxQX6H4Zxfua60X3QEHCIylDyJwnBeO7P_sDY7rr4UHe9LyR_cbxe7UV04_Ee4lvU-YsDnndTCjh_6OTOFiUnoxAg7yoQdPp){: .center-image }

[`BINARY HACKS: 해커가 전수하는 테크닉 100선`](https://books.google.co.kr/books/about/BINARY_HACKS_%ED%95%B4%EC%BB%A4%EA%B0%80_%EC%A0%84%EC%88%98%ED%95%98%EB%8A%94_%ED%85%8C.html?id=vKqYMQAACAAJ&redir_esc=y) 이란 책을 읽으면서 기억해 놓고 싶은 명령어를 여기 끄적여 봤습니다.

## 파일을 16진수랑 ASCII로 덤프하기

```
$ od --format x1z -A x 주간계획.xlsx | head -5
000000 50 4b 03 04 14 00 06 00 08 00 00 00 21 00 21 8c  >PK..........!.!.<
000010 46 3a 73 01 00 00 8c 05 00 00 13 00 08 02 5b 43  >F:s...........[C<
000020 6f 6e 74 65 6e 74 5f 54 79 70 65 73 5d 2e 78 6d  >ontent_Types].xm<
000030 6c 20 a2 04 02 28 a0 00 02 00 00 00 00 00 00 00  >l ...(..........<
000040 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  >................<
```

`od`는 `octal dump`의 약자인것 같다. 

- --format : 표시할 형식 (옵션 안주면 8진수가 기본값)
  - x1 : Hex로 1바이트 씩 표시
  - z : 옆에 ASCII 표시
- -A : 옆에 오프셋 형식 (안주면 8진수가 기본값)
  - x : Hex로 표시

## ELF 파일 심볼 테이블 보기

```
$ readelf -s orw | head -10

Symbol table '.dynsym' contains 7 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND write@GLIBC_2.2.5 (2)
     2: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __stack_chk_fail@GLIBC_2.4 (3)
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND read@GLIBC_2.2.5 (2)
     4: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main@GLIBC_2.2.5 (2)
     5: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
     6: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND open@GLIBC_2.2.5 (2)
```

ELF 바이너리의 심볼 테이블을 볼 수 있다. `nm`은 오브젝트 파일에 포함된 심볼을 볼 수 있다. (살짝 방식이 다르다)

## 특정 섹션의 역어셈블만 보기

```
jhyun@ubuntu:~/Desktop$ objdump -d -Mintel -j .text orw | head -12

orw:     file format elf64-x86-64


Disassembly of section .text:

0000000000400530 <_start>:
  400530:       31 ed                   xor    ebp,ebp
  400532:       49 89 d1                mov    r9,rdx
  400535:       5e                      pop    rsi
  400536:       48 89 e2                mov    rdx,rsp
  400539:       48 83 e4 f0             and    rsp,0xfffffffffffffff0
```

---

이런거 말고도 리눅스를 배우면서 잘 이해못했던 부분을 많이 돌아볼 수 있어서 좋았습니다. 때가 된다면 이 책에서 배운 내용들과 여러가지 조사해서 정리한 다음 포스팅 하려고 생각중입니다 하하.

그래서 책 다 읽었냐고요? ...쉿