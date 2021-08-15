# 演習

参考: CTFを解きながら学ぶバイナリ解析

---

## はじめに
以下のコードをコンパイルした実行ファイルを解析していきます。
```c
#include<string.h>
#include<stdio.h>

int main(int argc, char *argv[]){
    char str1[]="def";
    char str2[]="abc";

    int ret = strcmp(str1, str2);
    printf("%d", ret);

    return ret;
}
```

コンパイルしてみましょう。今回は解析を容易にするため，メモリ保護を無効化するオプションをつけてもらいます。

参考図書の関係で，32bitバイナリとしての変換をお願いします。

```bash
$ sudo apt-get install gcc-multilib
$ gcc -m32 intro.c -o intro -fno-stack-protector -no-pie
$ gdb ./intro
```

今回は参考にした問題に倣い，32bit環境を想定して演習していきます。

## 解析
```pdisas main```でmain関数を逆アセンブルします。

```intro.asm
gdb-peda$ pdisas main
Dump of assembler code for function main:
   0x08049172 <+0>:     lea    ecx,[esp+0x4]
   0x08049176 <+4>:     and    esp,0xfffffff0
   0x08049179 <+7>:     push   DWORD PTR [ecx-0x4]
   0x0804917c <+10>:    push   ebp
   0x0804917d <+11>:    mov    ebp,esp
   0x0804917f <+13>:    push   ebx
   0x08049180 <+14>:    push   ecx
   0x08049181 <+15>:    sub    esp,0x10
   0x08049184 <+18>:    call   0x80490b0 <__x86.get_pc_thunk.bx>
   0x08049189 <+23>:    add    ebx,0x2e77
   0x0804918f <+29>:    mov    DWORD PTR [ebp-0x10],0x666564
   0x08049196 <+36>:    mov    DWORD PTR [ebp-0x14],0x636261
   0x0804919d <+43>:    sub    esp,0x8
   0x080491a0 <+46>:    lea    eax,[ebp-0x14]
   0x080491a3 <+49>:    push   eax
   0x080491a4 <+50>:    lea    eax,[ebp-0x10]
   0x080491a7 <+53>:    push   eax
   0x080491a8 <+54>:    call   0x8049030 <strcmp@plt>
   0x080491ad <+59>:    add    esp,0x10
   0x080491b0 <+62>:    mov    DWORD PTR [ebp-0xc],eax
   0x080491b3 <+65>:    sub    esp,0x8
   0x080491b6 <+68>:    push   DWORD PTR [ebp-0xc]
   0x080491b9 <+71>:    lea    eax,[ebx-0x1ff8]
   0x080491bf <+77>:    push   eax
   0x080491c0 <+78>:    call   0x8049040 <printf@plt>
   0x080491c5 <+83>:    add    esp,0x10
   0x080491c8 <+86>:    mov    eax,DWORD PTR [ebp-0xc]
   0x080491cb <+89>:    lea    esp,[ebp-0x8]
   0x080491ce <+92>:    pop    ecx
   0x080491cf <+93>:    pop    ebx
   0x080491d0 <+94>:    pop    ebp
   0x080491d1 <+95>:    lea    esp,[ecx-0x4]
   0x080491d4 <+98>:    ret
End of assembler dump.
```

---

## まず、関数が呼び出されたら...

mainなど、関数が呼び出された際，ebp退避，スタックの作成を最初にやる。

スタックには，関数で利用するローカル変数や，関数終了後に戻るべき元の関数への戻り先アドレスなどを保存する。

スタックは高位の(数字が大きい)アドレスを底として，拡張する場合は低位のアドレスに広がっていく。

これが初期状態のスタックである:

|Address|Memory|Note|
|---|---|---|
||.............................||
|低位アドレス|||
|0xffffd1a8|||
|0xffffd1ac|前の関数のスタックの頂点|<-esp|
|0xffffd1b0||<-ebp|
||.............................||

---

これより、スタックの動作を段階的に確かめていくため、次の通りに9つのBreakpointを設置してください。

各Breakpointに対応したスタックの様子も示していくので、参考にしてください。

```
   0x080491b6 <+0>:     endbr32
   0x080491ba <+4>:     lea    ecx,[esp+0x4]
   0x080491be <+8>:     and    esp,0xfffffff0
   0x080491c1 <+11>:    push   DWORD PTR [ecx-0x4]
   0x080491c4 <+14>:    push   ebp
1->0x080491c5 <+15>:    mov    ebp,esp
   0x080491c7 <+17>:    push   ebx
   0x080491c8 <+18>:    push   ecx
2->0x080491c9 <+19>:    sub    esp,0x10
3->0x080491cc <+22>:    call   0x80490f0 <__x86.get_pc_thunk.bx>
   0x080491d1 <+27>:    add    ebx,0x2e2f
   0x080491d7 <+33>:    mov    DWORD PTR [ebp-0x10],0x666564
   0x080491de <+40>:    mov    DWORD PTR [ebp-0x14],0x636261
4->0x080491e5 <+47>:    sub    esp,0x8
   0x080491e8 <+50>:    lea    eax,[ebp-0x14]
   0x080491eb <+53>:    push   eax
   0x080491ec <+54>:    lea    eax,[ebp-0x10]
   0x080491ef <+57>:    push   eax
5->0x080491f0 <+58>:    call   0x8049070 <strcmp@plt>
6->0x080491f5 <+63>:    add    esp,0x10
7->0x080491f8 <+66>:    mov    DWORD PTR [ebp-0xc],eax
   0x080491fb <+69>:    sub    esp,0x8
   0x080491fe <+72>:    push   DWORD PTR [ebp-0xc]
   0x08049201 <+75>:    lea    eax,[ebx-0x1ff8]
   0x08049207 <+81>:    push   eax
   0x08049208 <+82>:    call   0x8049080 <printf@plt>
8->0x0804920d <+87>:    add    esp,0x10
   0x08049210 <+90>:    mov    eax,DWORD PTR [ebp-0xc]
   0x08049213 <+93>:    lea    esp,[ebp-0x8]
   0x08049216 <+96>:    pop    ecx
   0x08049217 <+97>:    pop    ebx
   0x08049218 <+98>:    pop    ebp
   0x08049219 <+99>:    lea    esp,[ecx-0x4]
   0x0804921c <+102>:   ret
```

---

関数のスタックを作成するためには，前の関数で使われていたスタックの領域を保存し，新しいebp, espを再設定しなければいけない。順序としては:

1. ebpをスタックに退避(保存)
2. espのアドレスをebpへコピー
3. 前の関数のスタックの上に新しい関数のスタックが積み上がる

よってスタックは下図のようになる: (Breakpoint 1)

|Address|Memory|Note|
|---|---|---|
||.............................||
|0xffffd1a4|||
|0xffffd1a8|退避した ebp|<- esp, ebp|
|0xffffd1ac|前の関数のスタックの頂点||
|0xffffd1b0|||
||.............................||

これを実現する処理がこちら:
```
0x8048460 <main+10>: push ebp
0x8048461 <main+11>: mov ebp,esp
```

ここで，push命令はスタックの先頭に引数のレジスタの値を積み上げ，esp-4を実行。

> mov命令: mov x y #yのデータをxへコピー

次のブロックでは，同様のpush操作でebx, ecxを退避。
```
0x8048463 <main+13>: push ebx
0x8048464 <main+14>: push ecx
```

実行後のスタックはこちら:　 (Breakpoint 2)
|Address|Memory|Note|
|---|---|---|
||.............................||
|0xffffd1a0|退避したecx|<- esp|
|0xffffd1a4|退避したebx||
|0xffffd1a8|退避した ebp|<- ebp|
||.............................||

## スタックの作成
- sub命令でスタックの長さを拡張
> sub命令: sub x y  #x=x-y

```
# espを -0x10
0x08048465 <main+15>: sub esp,0x10
```

実行後のスタック:  (Breakpoint 3)
|Address|Memory|Note|
|---|---|---|
||.............................||
|0xffffd190||<- esp|
|0xffffd194|||
|0xffffd198|||
|0xffffd19c|||
|0xffffd1a0|退避したecx||
|0xffffd1a4|退避したebx||
|0xffffd1a8|退避した ebp|<- ebp|
||.............................||

## 引数をスタックに積み，関数を呼び出すまで
- 次に, strcmp関数で比較する
  - def\0
  - abc\0
  
  をスタックに格納。

```
0x08048473 <+29>: mov DWORD PTR [ebp-0x10],0x666564
0x0804847a <+36>: mov DWORD PTR [ebp-0x14],0x636261
# []をつけることで，[ebpに格納されているアドレス-数値] のアドレスの意になる
```

- 全ての英数字・記号は，**ASCIIコードを用いて16進数で表せるようになっている**。
- ASCIIコード表より
  - 0x64=d
  - 0x65=e
  - 0x66=f
  
  なので，"def\0"は0x00666564を特定のメモリアドレスへコピーすれば良い。

> メモリへの格納順はリトルエンディアンであることに注意！

実行後のスタック:  (Breakpoint 4)
|Address|Memory|Note|
|---|---|---|
||.............................||
|0xffffd190||<- esp|
|0xffffd194|0x00636261 = abc\0||
|0xffffd198|0x0055564 = def\0||
|0xffffd19c|||
|0xffffd1a0|退避したecx||
|0xffffd1a4|退避したebx||
|0xffffd1a8|退避した ebp|<- ebp|
||.............................||

次に, strcmp関数に引数を渡して処理を行う。
```
0x08048481 <main+43>: sub   esp,0x8
0x08048484 <main+46>: lea   eax,[ebp-0x14]
0x08048487 <main+49>: push  eax
0x08048488 <main+50>: lea   eax,[ebp-0x10]
0x0804848b <main+53>: push  eax
```

引数はeaxに”abc\0”という文字が格納されたアドレス([ebp-0x14])を代入し，スタックにpush.

> lea(Load Effective Address) 命令: lea x, y # x <- y(で表現できるアドレス)

---

(参考: ジャンプ直前のスタックの状態:(Breakpoint 5))
|Address|Memory|Note|
|---|---|---|
||.............................||
|0xffffd17c|0x08048491|<- esp|
|0xddddf180|0xffffd198||
|0xffffd184|0xffffd194||
||............省略.............||
|0xffffd1a8|退避したebp|<- ebp|
||.............................||

- ジャンプ先で再度ebpをスタックにpushし，espをebpへコピー
  - ->新しいスタックがこの上(低位方向)に出来上がる
- ここで，この戻り先を任意に書き換えれば，関数の終了時に任意のアドレスへ処理を飛ばすことができる。(低レイヤのテクニック)

---


## strcmp 関数の呼び出し
```
0x0804848c <main+54>: call  0x8048300 <strcmp@plt>
# call命令は関数へ制御を移す命令。
```

実行後のスタック: (Breakpoint 6)
|Address|Memory|Note|
|---|---|---|
||.............................||
|0xffffd180|0xffffd198|<- esp|
|0xffffd184|0xffffd194||
|0xffffd188|||
|0xffffd18c|||
|0xffffd190|||
|0xffffd194|0x00636261 = abc\0||
|0xffffd198|0x0055564 = def\0||
|0xffffd19c|||
|0xffffd1a0|退避したecx||
|0xffffd1a4|退避したebx||
|0xffffd1a8|退避した ebp|<- ebp|
||.............................||

- call命令は関数へ制御を移す。
- 実際には２つの命令の組み合わせ
  - 関数が実行終了した際に次に実行すべきアドレス(戻り先アドレス，ここでは0x08048491)をスタックへpush
  - jmp命令を使って制御をジャンプさせる



## スタックの解放

関数を呼び出した後は，引数を渡すため利用したスタックの領域を解放。
```
0x08048491 <main+59>: add esp,0x10
```

解放後のスタックの状態: (Breakpoint 7)
|Address|Memory|Note|
|---|---|---|
||.............................||
|0xffffd190||<- esp|
|0xffffd194|0x00636261 = abc\0||
|0xffffd198|0x0055564 = def\0||
|0xffffd19c|||
|0xffffd1a0|退避したecx||
|0xffffd1a4|退避したebx||
|0xffffd1a8|退避した ebp|<- ebp|
||.............................||

## printf関数の呼び出し

- 関数の呼び出し結果はeaxに格納
- これをprintfで表示するため，eaxの結果をスタックに積む。

```
0x08048494 <+62>: mov    DWORD PTR [ebp-0xc],eax 
0x08048497 <+65>: sub    esp,0x8
0x0804849a <+68>: push   DWORD PTR [ebp-0xc]
0x0804849d <+71>: lea    eax,[ebx-0x1ac0]
0x080484a3 <+77>: push   eax
0x080484a4 <+78>: call   0x8048310 <printf@plt>
```
- ここでは一度 [ebp-0xc] にeaxの結果を退避している
  - 直接eaxをpushしても良いのでは?
  - 実際，gccでコンパイルする際に最適化オプション(-O1~-O3)を使うとそのように直される

printf呼び出し直前のスタックの状態:<br>
([ebx-0x1ac0]の領域に"%d"という文字へのアドレスが配置されていることに注意！)

実行後のスタックの状態: (Breakpoint 8)

|Address|Memory|Note|
|---|---|---|
||.............................||
|0xffffd180|0x08048540|<- esp|
||"%d"という文字へのポインタ||
|0xffffd184|0x00000001||
|0xffffd188|||
|0xffffd18c|||
|0xffffd190|||
|0xffffd194|0x00636261 = abc\0||
|0xffffd198|0x0055564 = def\0||
|0xffffd19c|||
|0xffffd1a0|退避したecx||
|0xffffd1a4|退避したebx||
|0xffffd1a8|退避した ebp|<- ebp|
||.............................||

1. print関数は第一引数に書式文字列を指定
2. 指定されたフォーマットして石に従って第2引数以降に指定されたポインタを参照
3. フォーマットに従い出力

- 実際は上図のように，フォーマット指定子の個数だけスタックを走査し，該当のアドレスのデータを出力
- 従って，第1引数を任意に操作できる場合，フォーマットして石を増やせばスタックの中身を自由に見ることができる
  - この性質を使い，任意のデータを参照したり書き込んだりする攻撃が**書式文字列攻撃**!
