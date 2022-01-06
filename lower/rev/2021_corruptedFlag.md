# Corrupted Flag (SECCON CTF 2021 Reversing)

> 参考: [SECCON CTF 2021作問者Writeup - CTFするぞ(ptr-yudaiさん:作問者)](https://ptr-yudai.hatenablog.com/entry/2021/12/19/232158)

## 問題
テキストファイルをエンコード(暗号化?)する実行ファイル"corrupt"と，壊れたFlagファイル"flag.txt.enc"
```
�R�9d^O�o��;{`�53s`^^f�i��\^Z���!pX2?^Ff3`8^Tg��!|X^Z2�!��l6^C�i���4���!s(4�ǂ�j�^^��̂ai^H^_>�I�aȜf��Ai�^Y2��!a^H^X�͜�^AÜ^^���!Ӝl63�М!��^^^^WZ^@
```
が渡される。

"corrupt"の処理を読み解き，flag.txt.encを元のflagに復元する処理を作る。


## 解き方
逆アセンブラソフトを通して処理を見ると，**4ビットを7ビットに拡張する処理をフラグに対して行っている**。 

処理の内容としては，**誤り訂正が可能なビットをXORで作って3ビット拡張**するというもの。 

**エンコード時にフラグのビットはランダムに破壊される**が、**各7ビットのうち1ビットが破壊されるかされないかの2択**なので、落ち着いて各7ビットごとにパリティ検査⇨誤りがある場合は訂正で対処可能。

## 目標
- ある程度x86機械語が読める
- Radare2などの逆アセンブラソフトが使える
- パリティ検査，誤り訂正など(新大では情報理論(情シス3年の講義)でやる予定)を理解する


## 手順
### 逆アセンブル(radare2を使用)
> radare2を入れていない人はここでインストール
> sudo apt install radare2

初めにfileコマンドでファイルの種類を確認しておく。
```a.sh
% file corrupt
corrupt: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=df82b674f179982a41f1d195ca9e95407c0edd87, for GNU/Linux 3.2.0, not stripped
```
corruptが実行ファイルであることが確認できたので， **[r2 ファイル名]**で解析を始める。

```a.asm
% r2 corrupt  
Warning: run r2 with -e bin.cache=true to fix relocations in disassembly
 -- I C dead bugs everywhere!
[0x000012e0]> aaaaaa        (<-aはanalyzeの略コマンド。aが多いほど詳細解析(max6つ))
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Finding and parsing C++ vtables (avrr)
[x] Type matching analysis for all functions (aaft)
[x] Propagate noreturn information (aanr)
[x] Finding function preludes
[x] Enable constraint types analysis for variables
```
 
 **afl** で，使用されている関数を調べる。
 
```a.asm
[0x000012e0]> afl
0x000012e0    1 46           entry0
0x00001310    4 41   -> 34   sym.deregister_tm_clones
0x00001340    4 57   -> 51   sym.register_tm_clones
0x00001380    5 57   -> 54   sym.__do_global_dtors_aux
0x000010e0    1 11           sym..plt.got
0x000013c0    1 9            entry.init0
0x00001000    3 27           sym._init
0x00001610    1 5            sym.__libc_csu_fini
0x00001618    1 13           sym._fini
0x000015a0    4 101          sym.__libc_csu_init
0x000013d0    7 121  -> 116  sym.__corrupt_internal
0x000011b0    7 234          main
0x000011a0    1 15           sym.cleanup
0x00001450    9 334          sym.corrupt
0x000012a0    3 50           sym.setup
0x00001160    1 11           sym.imp.open
0x00001180    1 11           sym.imp.exit
0x000010f0    1 11           sym.imp.free
0x00001100    1 11           sym.imp.fread
0x00001110    1 11           sym.imp.fclose
0x00001120    1 11           sym.imp.__stack_chk_fail
0x00001130    1 11           sym.imp.close
0x00001140    1 11           sym.imp.read
0x00001150    1 11           sym.imp.malloc
0x00001170    1 11           sym.imp.fopen
0x00001190    1 11           sym.imp.fwrite
0x00001030    2 31   -> 28   fcn.00001030
0x00001040    1 15           fcn.00001040
0x00001050    1 15           fcn.00001050
0x00001060    1 15           fcn.00001060
0x00001070    1 15           fcn.00001070
0x00001080    1 15           fcn.00001080
0x00001090    1 15           fcn.00001090
0x000010a0    1 15           fcn.000010a0
0x000010b0    1 15           fcn.000010b0
0x000010c0    1 15           fcn.000010c0
0x000010d0    1 15           fcn.000010d0
```


 なお，大抵は，main(または，それに準ずる関数)に基本の処理が書かれているので，そのあたりを見ていく。
 
 各関数に絞って逆アセンブルを行う場合は **[pdf@関数名]** 。

```a.asm
[0x000012e0]> pdf@main
            ; DATA XREF from entry0 @ 0x1301
┌ 234: int main (int argc, char **argv, char **envp);
│           ; var size_t *nitems @ rsp+0x8
│           ; var void *ptr @ rsp+0x10
│           ; var int64_t var_118h @ rsp+0x118
│           0x000011b0      f30f1efa       endbr64
│           0x000011b4      4155           push r13
│           0x000011b6      488d35470e00.  lea rsi, [0x00002004]       ; "rb" ; const char *mode
│           0x000011bd      488d3d430e00.  lea rdi, str.._flag.txt     ; 0x2007 ; "./flag.txt" ; const char *filename
│           0x000011c4      4154           push r12
│           0x000011c6      55             push rbp
│           0x000011c7      4881ec200100.  sub rsp, 0x120
│           0x000011ce      64488b042528.  mov rax, qword fs:[0x28]
│           0x000011d7      488984241801.  mov qword [var_118h], rax
│           0x000011df      31c0           xor eax, eax
│           0x000011e1      e88affffff     call sym.imp.fopen          ; file*fopen(const char *filename, const char *mode)
│           0x000011e6      4885c0         test rax, rax
│       ┌─< 0x000011e9      0f8481000000   je 0x1270
│       │   0x000011ef      4c8d6c2410     lea r13, [ptr]
│       │   0x000011f4      4889c1         mov rcx, rax                ; FILE *stream
│       │   0x000011f7      ba00010000     mov edx, 0x100              ; size_t nmemb
│       │   0x000011fc      4889c5         mov rbp, rax
│       │   0x000011ff      be01000000     mov esi, 1                  ; size_t size
│       │   0x00001204      4c89ef         mov rdi, r13                ; void *ptr
│       │   0x00001207      e8f4feffff     call sym.imp.fread          ; size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream)
│       │   0x0000120c      4889ef         mov rdi, rbp                ; FILE *stream
│       │   0x0000120f      4989c4         mov r12, rax
│       │   0x00001212      e8f9feffff     call sym.imp.fclose         ; int fclose(FILE *stream)
│       │   0x00001217      488d4c2408     lea rcx, [nitems]           ; int64_t arg4
│       │   0x0000121c      4889e2         mov rdx, rsp                ; int64_t arg3
│       │   0x0000121f      4c89e6         mov rsi, r12                ; int64_t arg2
│       │   0x00001222      4c89ef         mov rdi, r13                ; int64_t arg1
│       │   0x00001225      e826020000     call sym.corrupt
│       │   0x0000122a      488d35e10d00.  lea rsi, [0x00002012]       ; "wb" ; const char *mode
│       │   0x00001231      488d3ddd0d00.  lea rdi, str.._flag.txt.enc ; 0x2015 ; "./flag.txt.enc" ; const char *filename
│       │   0x00001238      e833ffffff     call sym.imp.fopen          ; file*fopen(const char *filename, const char *mode)
│       │   0x0000123d      4889c5         mov rbp, rax
│       │   0x00001240      4885c0         test rax, rax
│      ┌──< 0x00001243      742b           je 0x1270
│      ││   0x00001245      488b542408     mov rdx, qword [nitems]     ; size_t nitems
│      ││   0x0000124a      488b3c24       mov rdi, qword [rsp]        ; const void *ptr
│      ││   0x0000124e      4889c1         mov rcx, rax                ; FILE *stream
│      ││   0x00001251      be01000000     mov esi, 1                  ; size_t size
│      ││   0x00001256      e835ffffff     call sym.imp.fwrite         ; size_t fwrite(const void *ptr, size_t size, size_t nitems, FILE *stream)
│      ││   0x0000125b      4889ef         mov rdi, rbp                ; FILE *stream
│      ││   0x0000125e      e8adfeffff     call sym.imp.fclose         ; int fclose(FILE *stream)
│      ││   0x00001263      488b3c24       mov rdi, qword [rsp]        ; void *ptr
│      ││   0x00001267      e884feffff     call sym.imp.free           ; void free(void *ptr)
│      ││   0x0000126c      31c0           xor eax, eax
│     ┌───< 0x0000126e      eb05           jmp 0x1275
│     │││   ; CODE XREFS from main @ 0x11e9, 0x1243
│     │└└─> 0x00001270      b801000000     mov eax, 1
│     │     ; CODE XREF from main @ 0x126e
│     └───> 0x00001275      488b94241801.  mov rdx, qword [var_118h]
│           0x0000127d      64482b142528.  sub rdx, qword fs:[0x28]
│       ┌─< 0x00001286      750d           jne 0x1295
│       │   0x00001288      4881c4200100.  add rsp, 0x120
│       │   0x0000128f      5d             pop rbp
│       │   0x00001290      415c           pop r12
│       │   0x00001292      415d           pop r13
│       │   0x00001294      c3             ret
│       │   ; CODE XREF from main @ 0x1286
└       └─> 0x00001295      e886feffff     call sym.imp.__stack_chk_fail ; void __stack_chk_fail(void)
[0x000012e0]> 
```
