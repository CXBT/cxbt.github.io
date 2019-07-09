---
layout: post
title:  "알고 쓰자 쉘코드?"
date:   2019-07-09 16:55:43 +0900
categories: pwn
tags: pwnable.tw
---

알고 쓰는건 중요하다. 좆문가 소리를 듣기 전에 본인이 조질 수 있기 때문이다. 내가 그랬다 허허.

`pwnable.tw` 한번 해보려고 첫번째 문제를 봤다. 첫번째 문제 답게 BOF로 뭐 하는 거였는데 그냥 실행 흐름 바꾸고  `read`를 한번 더 호출해서 스택 주소 알아내고 쉘코드 쪽으로 실행 흐름 다시 바꾸면 될 것만 같았다.

간단하게 코드 짜서 했는데 안된다. 아니 아무리 생각해도 안될리가 없는데 안된다. `checksec`으로 뭐 다른거 있나 봤는데 그것도 없었다. 으헝

3일 후... 삽질을 하다가 도저히 안되겠길래 살짝만...(?) 살짝만 다른 사람들이 어떻게 풀었는지 봤다. 근데 방법은 똑같더라. 근데 좀 좋은 걸 알았다.

```python
from pwn import *

r = process('./start')
context.log_level='debug'
gdb.attach(r,'b* 0x08048097')

출처: https://chp747.tistory.com/173 [만두만두]
```

그 전에는 GDB 따로 코드 따로 막 터미널 몇개 따로따로 띄워놓았다. 어딘가 디버거랑 저 프로세스랑 연결하는 거 있지 않을까 계속 찾아봤는데 드디어 우연히도 찾을 수 있었다 이히

코드 실행하면서 디버깅 했는데 이상하게 쉘코드도 잘 올라가고, 시스템 콜 호출까지 잘 갔는데, 이상하게 `/bin/sh`이 호출됬으면 디버거 상에서 뭔가 반응이라도 있어야 할 것 같았는데 스무스하게 넘어가더라. 여기서 딱 쉘코드 쓸때 문제가 발생한 걸 알았다 으흑

```python
0xffc60429 in ?? ()
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
─────────────────────────────────[ REGISTERS ]──────────────────────────────────
 EAX  0xb
 EBX  0xffc60408 ◂— '/bin//sh'
 ECX  0xffc60400 —▸ 0xffc60408 ◂— '/bin//sh'
 EDX  0x3c
 EDI  0x0
 ESI  0x0
 EBP  0x0
 ESP  0xffc60400 —▸ 0xffc60408 ◂— '/bin//sh'
 EIP  0xffc60429 ◂— 0x55ff80cd
───────────────────────────────────[ DISASM ]───────────────────────────────────
   0xffc60421    mov    ebx, esp
   0xffc60423    push   eax
   0xffc60424    push   ebx
   0xffc60425    mov    ecx, esp
   0xffc60427    mov    al, 0xb
 ► 0xffc60429    int    0x80 <SYS_execve>
        path: 0xffc60408 ◂— '/bin//sh'
        argv: 0xffc60400 —▸ 0xffc60408 ◂— '/bin//sh'
        envp: 0x3c
   0xffc6042b    call   dword ptr [ebp + 0x22]
```

쉘코드 원리는 학교에서 귀 닳듯 들었고 내가 직접 만들어 보기까지 해서 문제 풀이 할때는 그냥 검색해서 가져다 쓰는 편이다. `linux shellcode` 구글에 쳐서 맨 처음 나오는 걸 이번에도 쓴거였다. ([23바이트 짜리 쉘코드](http://shell-storm.org/shellcode/files/shellcode-827.php)) 이번엔 맨 처음꺼 아래 쉘코드를 써봤다. ([28바이트 짜리 쉘코드](http://shell-storm.org/shellcode/files/shellcode-811.php))

된다 ~~씨발~~

어머 욕하면 안되는데 아무튼 잘 돌아갔다. 허헣

28바이트짜리 쉘코드는 아예 `ECX`, `EDX`, 딱히 필요 없는 레지스터, 그러니까 매개변수 값으로 사용되는 레지스터를 초기화 하는 코드가 들어 있는데 23바이트 짜리 쉘코드는 그렇지 않더라. 위에 디버거 화면을 보면 멀쩡하게 `ECX`에 `/bin//sh` 문자열 주소가 들어가 있는 것을 볼 수 있다. 그리고 혹시나 해서 설마 `argv`에 저게 들어간다고 안되려나 했는데

```
jhyun@ubuntu:~/Desktop/Hacking/pwnable.tw$ /bin/sh /bin/sh
/bin/sh: 3: /bin/sh: Syntax error: end of file unexpected (expecting ")")
```

안되더라ㅋㅋㅋ

나의 무지를 탓했다. 꺼진 쉘코드도 다시보자
