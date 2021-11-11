# SECCON CTF 2020 SCSBX:Reversing

- 独自VM問題。
  - https://github.com/SECCON/SECCON2020_online_CTF/tree/main/reversing/scsbx_reversing

> 参考ハンズオン: <br>
> https://tan.hatenadiary.jp/entry/2020/10/11/151926 <br>
> https://hikalium.hatenablog.jp/entry/2020/10/11/205917)

## VMの動作について
- 各命令: https://github.com/SECCON/SECCON2020_online_CTF/blob/main/reversing/scsbx_reversing/DOCS.md

  - PUSH命令は 1バイト + データ4バイト
  - それ以外の命令はすべて1バイトのスタックマシン。

- 各命令のオペコード: https://github.com/SECCON/SECCON2020_online_CTF/blob/main/reversing/scsbx_reversing/files/src/scsbx.hpp

```op.asm
\\ 以下、一部抜粋

/* Memory Operations */
#define INS_PUSH    0x20
#define INS_POP     0x21
#define INS_DUP     0x22
#define INS_XCHG    0x23
#define INS_LOAD32  0x24
#define INS_LOAD64  0x25
#define INS_STORE8  0x26
#define INS_STORE16 0x27
#define INS_STORE32 0x28
#define INS_SHOW    0x70

 /* Branch Operations */
#define INS_JMP   0x30
#define INS_JEQ   0x31
#define INS_JGT   0x32
#define INS_JGE   0x33
#define INS_CALL  0x34

/* Arithmetic Operations */
#define INS_ADD   0x40
#define INS_SUB   0x41
#define INS_MUL   0x42
#define INS_DIV   0x43
#define INS_MOD   0x44

/* Logical Operations */
#define INS_NOT   0x50
#define INS_AND   0x51
#define INS_OR    0x52
#define INS_XOR   0x53
#define INS_SHL   0x54
#define INS_SHR   0x55
#define INS_ROL   0x56
#define INS_ROR   0x57

/* System Operations */
#define INS_READ  0x60
#define INS_WRITE 0x61
#define INS_MAP   0x62
#define INS_UNMAP 0x63
#define INS_EXIT  0x64
```

## 逆アセンブラをPython3で作る
```disas.py
#!/usr/bin/env python3

#出典: https://tan.hatenadiary.jp/entry/2020/10/11/151926

with open("seccon.bin", "rb") as f:
    pc = 0 # 命令カウンタ
    while True: 
        d = f.read(1)
        pc += 1 
        if len(d) == 0: break
        ins = d[0]
        print(f"{pc-1}\t: ", end="") 
        if ins == 0x20: # PUSH命令だけ 1byte + data 4byte
        # ここからは4byte dataの処理
            value = f.read(4)
            if len(value) < 4:
                print("Wrong len in push!")
                exit
            pc += 4
            v = int.from_bytes(value, byteorder="little")
            print(f"push {v}(={hex(v)})")
        elif ins == 0x21: print("POP")
        elif ins == 0x22: print("DUP")
        elif ins == 0x23: print("XCHG")
        elif ins == 0x24: print("LOAD32")
        elif ins == 0x25: print("LOAD64")
        elif ins == 0x26: print("STORE8")
        elif ins == 0x27: print("STORE16")
        elif ins == 0x28: print("STORE32")
        elif ins == 0x70: print("SHOW")
        elif ins == 0x30: print("JMP")
        elif ins == 0x31: print("JEQ")
        elif ins == 0x32: print("JGT")
        elif ins == 0x33: print("JGE")
        elif ins == 0x34: print("CALL")
        elif ins == 0x40: print("ADD")
        elif ins == 0x41: print("SUB")
        elif ins == 0x42: print("MUL")
        elif ins == 0x43: print("DIV")
        elif ins == 0x44: print("MOD")
        elif ins == 0x50: print("NOT")
        elif ins == 0x51: print("AND")
        elif ins == 0x52: print("OR")
        elif ins == 0x53: print("XOR")
        elif ins == 0x54: print("SHL")
        elif ins == 0x55: print("SHR")
        elif ins == 0x56: print("ROL")
        elif ins == 0x57: print("ROR")
        elif ins == 0x60: print("READ")
        elif ins == 0x61: print("WRITE")
        elif ins == 0x62: print("MAP")
        elif ins == 0x63: print("UNMAP")
        elif ins == 0x64: print("EXIT")
        else:
            print(f"unknown ins! {ins}")
            exit
```            
