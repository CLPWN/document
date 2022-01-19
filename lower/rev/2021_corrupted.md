# Corrupted Flag (SECCON2021 Rev)

> 参考: [SECCON CTF 2021 writeup - st98 の日記帳 - コピー(st98さん)](https://nanimokangaeteinai.hateblo.jp/entry/2021/12/14/223037)

## 問題
テキストファイルをエンコード(暗号化?)する実行ファイル"corrupt"と，壊れたFlagファイル"flag.txt.enc"
```
�R�9d^O�o��;{`�53s`^^f�i��\^Z���!pX2?^Ff3`8^Tg��!|X^Z2�!��l6^C�i���4���!s(4�ǂ�j�^^��̂ai^H^_>�I�aȜf��Ai�^Y2��!a^H^X�͜�^AÜ^^���!Ӝl63�М!��^^^^WZ^@
```
が渡される。

"corrupt"の処理を読み解き，flag.txt.encを元のflagに復元する処理を作る。


## 動作の確認
1. ex) "SECCON{2394089abf908932043890c890da2}"とかかれた flag.txt を作成して "./corrupt flag.txt" を実行
1. 中身が ``` "�҉9d���1{����f���a�6��&s8��p���af����,f��!є7��է" ``` のflag.txt.enc が生成される
1. これだと読んだり操作したりできないので，16真数表記に変換。 pythonなどを使って
``` b'\xd3\xd2\x899d\x0e\x99\x7f\xa6\x951{\xa8\x86\xc1\xe0f\xc8\x14\x02\x86\xf0a\xee\x986\xab\xcc&sn\x088\xc2\x87\x99\xe1p\xa8\n"\x82\xdcap\x08\x1ff\x87\x80\xc1\xd1,\x1ff\x86\x80!\xd1\x947\xab\x86\xd5\xa7\x16\x00' ```
という形に変換

> 注意: 元の flag.txt.enc は，事前に"cp flag.txt.enc orig.enc" するなどしてバックアップをとっておく！

## 方針
ブルートフォース式で，変換後のファイルと問題に添付されていた flag.txt.enc とを1bitずつ比較したときにもっとも一致していたものを採用していく。

# 解法のソースコード
- 処理が完全に理解できなくてもよいので，まずはpython3の基本的文法とやライブラリの使い方を理解することを目指す。
  - Python3の基本的な使い方について: [Pythonに関する情報 | note.nkmk.me](https://note.nkmk.me/python/)
  - 疑問が噴出しがちなアンダースコアの使い方について：　[Pythonのアンダースコア( _ )を使いこなそう！](https://medium.com/lsc-psd/pythonic%E8%89%B2%E3%80%85-python%E3%81%AE%E3%82%A2%E3%83%B3%E3%83%80%E3%83%BC%E3%82%B9%E3%82%B3%E3%82%A2-%E3%82%92%E4%BD%BF%E3%81%84%E3%81%93%E3%81%AA%E3%81%9D%E3%81%86-3c132842eeef)


> 引用元: [SECCON CTF 2021 writeup - st98 の日記帳 - コピー(st98さん)](https://nanimokangaeteinai.hateblo.jp/entry/2021/12/14/223037)
```a.py
import os

def test(s):
  with open('flag.txt', 'w') as f: # writeオプションでファイルを開く。この時fはなんでもよい
    f.write(s)
  os.system('./corrupt') # python内部から外部コマンドを実行する
  with open('flag.txt.enc', 'rb') as f: #これはバイナリファイルなので，binary readオプションでファイルを開く。これで16進数表記で読めるようになる。
    s = f.read()
  return s

def to_bin(s):
  return ''.join(bin(x)[2:].zfill(8) for x in s) 
  #binは2進数に変換 
  # 空の文字列""にjoin関数とforループでひたすら連結していく。
  # [2:]スライスと呼ぶ。0,1番目の配列を切り捨て 
  # zfill(8)は，渡された文字が8字に満たない場合0で埋める
      
def compare(a, b):
  a, b = to_bin(a), to_bin(b)
  return sum(x == y for x, y in zip(a, b))
  #zipは，複数の配列の同列の要素をまとめていく関数
  # ex)
  # names = ['Alice', 'Bob', 'Charlie']
  # ages = [24, 50, 18]
  # for name, age in zip(names, ages):
    # print(name, age)
    # Alice 24
    # Bob 50
    # Charlie 18

with open('orig.enc', 'rb') as f: # orig.encは問題に添付されていたflag.txt.enc
  orig = f.read()

known = '' # 空の文字列だが，ここにブルートフォース処理で判明したflag.txtの部分部分を解ったところから埋めていく。

for i in range(72):
  mc, ms = '', 0
  #for c in range(0x20, 0x7f):
  for c in b'0123456789abcdefSECCON{}': # SECCON{} の中身は 0-9, a-f(10-15)? なので
    s = 0
    for _ in range(100): # _ で最後の表示を持ってくる。
      s += compare(test(known + chr(c)), orig) #chrはunicode16進数表記と文字を変換
    if s > ms:
      mc, ms = chr(c), s 
  known += mc
  print(known)


# SECCON{9e469af5f60e7f0c98854ebf0afd254c102154587a7491594900a8d186df4801}
```
