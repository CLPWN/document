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

## 逆アセンブラ実行結果
```a.asm
0       : push 4096(=0x1000)
5       : push 3735879680(=0xdead0000)
10      : MAP
11      : push 598(=0x256)
16      : CALL
17      : push 522(=0x20a)
22      : CALL
23      : push 1195461702(=0x47414c46)
28      : push 3735879684(=0xdead0004)
33      : STORE32
34      : push 8250(=0x203a)
39      : push 3735879688(=0xdead0008)
44      : STORE16
45      : push 6(=0x6)
50      : push 3735879684(=0xdead0004)
55      : WRITE
56      : push 64(=0x40)
61      : push 3735879690(=0xdead000a)
66      : READ
67      : push 8(=0x8)
72      : push 270(=0x10e)
77      : JMP
78      : push 0(=0x0)
83      : DUP
84      : push 8(=0x8)
89      : SUB
90      : push 8(=0x8)
95      : MUL
96      : push 0(=0x0)
101     : DUP
102     : push 3735879690(=0xdead000a)
107     : ADD
108     : push 0(=0x0)
113     : DUP
114     : LOAD32
115     : push 0(=0x0)
120     : XCHG
121     : push 2(=0x2)
126     : XCHG
127     : push 3735879694(=0xdead000e)
132     : ADD
133     : push 0(=0x0)
138     : DUP
139     : push 3(=0x3)
144     : XCHG
145     : push 1(=0x1)
150     : XCHG
151     : LOAD32
152     : push 3(=0x3)
157     : push 221(=0xdd)
162     : JMP
163     : push 2(=0x2)
168     : DUP
169     : push 2(=0x2)
174     : DUP
175     : push 0(=0x0)
180     : DUP
181     : push 534(=0x216)
186     : CALL
187     : push 2(=0x2)
192     : DUP
193     : XOR
194     : push 4(=0x4)
199     : XCHG
200     : POP
201     : push 4(=0x4)
206     : XCHG
207     : POP
208     : POP
209     : push 1(=0x1)
214     : push 1(=0x1)
219     : XCHG
220     : SUB
221     : push 0(=0x0)
226     : DUP
227     : push 0(=0x0)
232     : push 163(=0xa3)
237     : push 243(=0xf3)
242     : JEQ
243     : push 4(=0x4)
248     : XCHG
249     : STORE32
250     : push 1(=0x1)
255     : XCHG
256     : STORE32
257     : POP
258     : push 1(=0x1)
263     : push 1(=0x1)
268     : XCHG
269     : SUB
270     : push 0(=0x0)
275     : DUP
276     : push 0(=0x0)
281     : push 78(=0x4e)
286     : push 292(=0x124)
291     : JEQ
292     : POP
293     : push 0(=0x0)
298     : push 8(=0x8)
303     : push 405(=0x195)
308     : JMP
309     : push 0(=0x0)
314     : DUP
315     : push 1(=0x1)
320     : push 1(=0x1)
325     : XCHG
326     : SUB
327     : push 8(=0x8)
332     : MUL
333     : push 3735879690(=0xdead000a)
338     : ADD
339     : push 0(=0x0)
344     : DUP
345     : push 64(=0x40)
350     : ADD
351     : LOAD64
352     : push 2(=0x2)
357     : XCHG
358     : LOAD64
359     : push 2(=0x2)
364     : XCHG
365     : SUB
366     : push 2(=0x2)
371     : XCHG
372     : SUB
373     : OR
374     : push 1(=0x1)
379     : XCHG
380     : push 2(=0x2)
385     : XCHG
386     : OR
387     : push 1(=0x1)
392     : XCHG
393     : push 1(=0x1)
398     : push 1(=0x1)
403     : XCHG
404     : SUB
405     : push 0(=0x0)
410     : DUP
411     : push 0(=0x0)
416     : push 309(=0x135)
421     : push 427(=0x1ab)
426     : JEQ
427     : push 438(=0x1b6)
432     : push 477(=0x1dd)
437     : JEQ
438     : push 1852797527(=0x6e6f7257)
443     : push 3735879680(=0xdead0000)
448     : STORE32
449     : push 169943399(=0xa212167)
454     : push 3735879684(=0xdead0004)
459     : STORE32
460     : push 8(=0x8)
465     : push 3735879680(=0xdead0000)
470     : WRITE
471     : push 516(=0x204)
476     : JMP
477     : push 1920102211(=0x72726f43)
482     : push 3735879680(=0xdead0000)
487     : STORE32
488     : push 175399781(=0xa746365)
493     : push 3735879684(=0xdead0004)
498     : STORE32
499     : push 8(=0x8)
504     : push 3735879680(=0xdead0000)
509     : WRITE
510     : push 516(=0x204)
515     : JMP
516     : push 0(=0x0)
521     : EXIT
522     : push 114514893(=0x6d35bcd)
527     : push 3735879680(=0xdead0000)
532     : STORE32
533     : JMP
534     : push 3735879680(=0xdead0000)
539     : LOAD32
540     : push 1919(=0x77f)
545     : MUL
546     : push 810(=0x32a)
551     : push 1(=0x1)
556     : XCHG
557     : SUB
558     : push 811512810(=0x305eb3ea)
563     : push 1(=0x1)
568     : XCHG
569     : MOD
570     : push 0(=0x0)
575     : DUP
576     : push 3735879680(=0xdead0000)
581     : STORE32
582     : push 2(=0x2)
587     : DUP
588     : XOR
589     : NOT
590     : push 2(=0x2)
595     : XCHG
596     : POP
597     : JMP
598     : push 1182143011(=0x46761223)
603     : push 3735879754(=0xdead004a)
608     : STORE32
609     : push 1421780421(=0x54bea5c5)
614     : push 3735879758(=0xdead004e)
619     : STORE32
620     : push 2049108214(=0x7a22e8f6)
625     : push 3735879762(=0xdead0052)
630     : STORE32
631     : push 1572115401(=0x5db493c9)
636     : push 3735879766(=0xdead0056)
641     : STORE32
642     : push 89986910(=0x55d175e)
647     : push 3735879770(=0xdead005a)
652     : STORE32
653     : push 36687155(=0x22fcd33)
658     : push 3735879774(=0xdead005e)
663     : STORE32
664     : push 1120168934(=0x42c46be6)
669     : push 3735879778(=0xdead0062)
674     : STORE32
675     : push 1829806312(=0x6d10a0e8)
680     : push 3735879782(=0xdead0066)
685     : STORE32
686     : push 1408549496(=0x53f4c278)
691     : push 3735879786(=0xdead006a)
696     : STORE32
697     : push 1920592938(=0x7279ec2a)
702     : push 3735879790(=0xdead006e)
707     : STORE32
708     : push 1418853177(=0x5491fb39)
713     : push 3735879794(=0xdead0072)
718     : STORE32
719     : push 1236025887(=0x49ac421f)
724     : push 3735879798(=0xdead0076)
729     : STORE32
730     : push 1235958327(=0x49ab3a37)
735     : push 3735879802(=0xdead007a)
740     : STORE32
741     : push 1199921170(=0x47855812)
746     : push 3735879806(=0xdead007e)
751     : STORE32
752     : push 1461238533(=0x5718bb05)
757     : push 3735879810(=0xdead0082)
762     : STORE32
763     : push 88144731(=0x540fb5b)
768     : push 3735879814(=0xdead0086)
773     : STORE32
774     : JMP
```
## 処理の流れ

大まかに

- 作業用のメモリ領域[0xDEAD0000 - 0xDEAD1000)をmap
- そのメモリ領域に符号化されたフラグを展開(0xDEAD004A から0x40B)
- 0xDEAD0000番地に値0x06D35BCDを格納
- ユーザーからの入力を受け取る(0x40バイト、0xDEAD000A~)
- 上の擬似コードに書いたような処理を行って、ユーザーからの入力を符号化する
  -　 dead0000 = 0x06D35BCD と4Bite入力をXOR演算(return ~(v ^ *dead0000))し、スタックのトップに格納
  -　 dead0000の値は、演算終了ごとに dead0000 = ([dead0000] * 0x77F - 0x32A) % 0x0x305EB3EA をして更新
- 符号化した入力(0xDEAD000Aからの0x40バイト)と、展開しておいた符号化済みのフラグ値(0xDEAD004Aからの0x40バイト)を比較
  - 入力を8byte単位で処理し、計算結果が特定8byteに一致するか確認。それを0x40byte分繰り返す
- それらが一致したらCorrect, 一致しなければWrongと出力
```
# フラグをメモリ上に展開
# push 値指定 => push アドレス指定 => STOREで格納
598     : push 1182143011(=0x46761223)
603     : push 3735879754(=0xdead004a)
608     : STORE32
609     : push 1421780421(=0x54bea5c5)
614     : push 3735879758(=0xdead004e)
619     : STORE32
...
...
...
752     : push 1461238533(=0x5718bb05)
757     : push 3735879810(=0xdead0082)
762     : STORE32
763     : push 88144731(=0x540fb5b)
768     : push 3735879814(=0xdead0086)
773     : STORE32

# 0xDEAD0000番地に値0x06D35BCDを格納。これは何かのキー?
522     : push 114514893(=0x6d35bcd)
527     : push 3735879680(=0xdead0000)
532     : STORE32

# 入力結果を0xdead000a以降に格納------------
55      : WRITE
56      : push 64(=0x40)
61      : push 3735879690(=0xdead000a)
66      : READ

# [0xdead0000](=0x6d35bcd :初期値)をLOAD後、
# [0xdead0000]へ ((0x77f*[0xdead0000]-0x32a)%0x305eb3ea) としてSTORE
# 597で スタックトップに「~((呼び出し時のtop) xor (更新後の[0xdead0000]))」をpushした状態でreturn
# ~ はビット反転
534     : push 3735879680(=0xdead0000)
539     : LOAD32
540     : push 1919(=0x77f)
545     : MUL
546     : push 810(=0x32a)
551     : push 1(=0x1)
556     : XCHG
557     : SUB
558     : push 811512810(=0x305eb3ea)
563     : push 1(=0x1)
568     : XCHG
569     : MOD
570     : push 0(=0x0)
575     : DUP
576     : push 3735879680(=0xdead0000)
581     : STORE32
582     : push 2(=0x2)
587     : DUP
588     : XOR
589     : NOT
590     : push 2(=0x2)
595     : XCHG
596     : POP
597     : JMP
```


## 攻略スクリプト
```
#!/usr/bin/env python3

# 0xdead0000の値の推移

dead0000=[0x6d35bcd,0x2a5d2289,0x2f6875f9,0x2fad9e73,0x5b9550f,0x26c9c89f,0x11c0d0f,0x20e72c5d,0x13c96e3b,0x22929531,0x28cc5725,0x12466b89,0xc06913b,0x253aa61b,0x12a3213b,0x23b9fa5d,0xda0ec51,0x294b7005,0x2bbf4a7d,0x2d748c31,0x2b8ac467,0x478d51b,0x25080a67,0x629db31,0x3635d3b]

print(len(dead0000))


# [0xdead004a, 0xdead008a)の設定値
expected = [0x46761223,0x54bea5c5,0x7a22e8f6,0x5db493c9,0x055d175e,0x022fcd33,0x42c46be6,0x6d10a0e8,0x53f4c278,0x7279ec2a,0x5491fb39,0x49ac421f,0x49ab3a37,0x47855812,0x5718bb05,0x0540fb5b]

print(len(expected))




def invert(lower, upper, loopCount):
    for i in range(3):
        tilde = upper ^ lower
        xor1 = ~tilde
        [lower, upper] = [xor1 ^ dead0000[(3*loopCount)+(2-i)+1], lower]
    return [lower, upper]



def get_bytes(s):
    return int.from_bytes(s.encode("utf-8"), byteorder="little")
def get_str(n):
    return n.to_bytes(length=4, byteorder="little").decode("utf-8")


for i in range(8):
    [lower, upper] = invert(expected[2*i], expected[2*i+1], i)
    print(f"{get_str(lower)}{get_str(upper)}", end="")
print()
```
