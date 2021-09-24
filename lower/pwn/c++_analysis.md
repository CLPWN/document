# Birdcage
## C++の要点
### C++の標準出力
```
std::cout << "hello";
```

### C++の標準入力
```
std::cin >> x ;
```
 
## C++のクラスについて
たとえば、複数の点を表現するとき
```
int main()
{
    int x1 = 0 ;
    int y1 = 0 ;

    int x2 = 0 ;
    int y2 = 0 ;

    int x3 = 0 ;
    int y3 = 0 ;
}
```
と書くのはわかりづらい。また、これ以上店を増やしていくのも大変である。
ここでクラスを使うと
```
struct point
{
    int x = 0 ;
    int y = 0 ;
} ;

int main()
{
    point p ;

    std::cout << p.x << p.y ;
}
```
とかけ、点を複数表現する際も
```
point p1 ;
point p2 ;
point p3 ;
```
と書くだけで済む。

これがクラスの変数をまとめる機能だ。

クラスを定義するには、キーワードstructに続いてクラス名を書く。変数は{}の中に書く。

C++のクラスにはアクセス指定がある。public:とprivate:だ。アクセス指定が書かれたあと、別のアクセス指定が現れるまでの間のメンバーは、アクセス指定の影響を受ける。

structとclassの違いはデフォルトのアクセス指定。


```
struct Bird {
string name() {return typeid(*this).name();};
virtual void sing() = 0;
virtual ~Bird() = default;
};
```
structはデフォルトでpublicとなる。
publicメンバーはクラスの外から使うことができる。



```
class Parrot: public Bird {
string memory;
public:
Parrot() {
cout<<"Talk to: ";
cin>>memory.data();
}
void sing() override {cout<<memory.c_str()<<endl;}
};
```
classはデフォルトでprivateとなる。
privateメンバーはクラスの外から使うことができない。

アクセス指定を明示的に書く場合、違いはなくなる

### クラスの継承とオーバーライド

クラスは使いまわせることに利点がある(継承)。
例えば
```
struct Bird {
string name() {return typeid(*this).name();};
virtual void sing() = 0;
virtual ~Bird() = default;
};
```
について、鳴き声を鳥の種類ごとに
```
struct Cock: Bird {
    void sing() override {cout<<"Cock-a-doodle-doo!"<<endl;}
};
```
と書き換えることができる。

このとき、親クラスのメソッド(継承元)にvirtualキーワードをつける。

また、子クラスのメソッド(継承先)にoverrideキーワードをつける。

## C++でobjdumpを使う
- C++プログラムを objdumpで逆アセンブルすると、関数名などのシンボルは_ZN4Cock4singEv のような形式になry。
 - C++ では引数の違う関数を複数定義すること(オーバーロード)ができるため、シンボルに引数などの情報を含める必要がある名前修飾(name mangling)と呼ばれる仕組み 
- c++filt コマンドを通すとソースコード中の表記で関数が表示されるので，可読性が高まる。
```
$ objdump -d -M intel birdcage | c++filt | grep sing
```

## Birdcageの脆弱性

birdcage の脆弱性はソースコードの 35 行目の cin>>memory.data();が相当する。

簡単に説明すると，Cにおける gets 関数のような存在である。

そして cin>>memory.data(); は (C++11において) **string::c_data()**　と同一の動作になる。

### string について
string には small-string optimization という仕組みがある。

実行速度やメモリの効率、そして「実際のプログラムで使われる文字列は短い」という実情に則して，string にバッファを持たせ、短い文字列であれば文字列のメモリ確保を不要にしている、という仕組みをとっている。

しかしその結果、cin>>に char *を渡すと、cin は gets 関数のようにバッファサイズ以上の文字列を読み込んでしまう...という脆弱性を生んでいる。

(より詳細な説明は，テキストや https://github.com/melpon/qiita/tree/master/items/stdstring を参照してください)

String 周辺のスタック構造は次の通りになる: 

|オフセット| サイズ| 名前| 内容|
|---|---|---|---|
|0x00| 0x08| _M_p| 文字列のポインタ(_M_local_buf)|
|0x08 |0x08 |_M_string_length |文字列長|
|0x10| 0x10| _M_local_buf| 文字列|

- 複数の文字列を扱う場合、上記のスタック構造が連結して配置される。
- _M_local_buf に 0x10 超過の文字列を書き込むことで、後続のデータを改竄できる。
- 後続の別の string の_M_p を書き換えれば、任意のアドレスの値を読み出すことが可能。

実装の本体は https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/bits/basic_string.h


## クラスとvtable(仮想関数テーブル)

クラスの挙動として，子クラスのメンバ関数を親クラスのポインタ経由で呼び出したとき、**親クラスのメンバ関数ではなく子クラスのメンバ関数**(C++ では，virtual で修飾したメンバ関数)が呼び出される

> 例: 63 行目の sing() の呼び出しでは、cage[index] の型は Bird *だが，Parrot::sing() などが呼び出されている

この仕組みは vtable(仮想関数テーブル)によって実現されている。そして各クラスは仮想メンバ 関数のアドレスを並べた vtable を持っている

1. クラスのコンストラクタはオブジェクトの生成時、オブジェクト先頭に自分の vtable のアドレスを書き込む
2. ポインタ経由でメンバ関数を呼び出すとき、vtable からアドレスを取得することで、オブジェクトを生成したクラスのメンバ関数を呼び出す
 1. なお、メンバ関数のアドレスの前には、多重継承(Override)時に使う情報と typeinfo のアドレスが置かれていて、シンボルはその位置を指しているため、逆アセンブルコード中の vtable のアドレスを代入する所の表記は **vtable for Parrot+0x10** 
3. 偽の vtable を用意してオブジェクトの vtable のアドレスを書き換えることで、**メンバ関数の呼び出し時に本来とは異なる処理を実行させる**

## malloc のチャンク

birdcage では new 演算子で動的にメモリを確保している。
その new 演算子の内部処理では malloc 関数が呼ばれている。

malloc 関数はユーザーと OS との間でOS か ら大きなサイズでメモリを確保してユーザーに配分したり，free 関数で解放されたメモリを保持し、新しいメモリが要求されたときにを返したり，解放されたメモリを統合したりする関数。

malloc 関数 を使用しているプログラムに脆弱性があった場合の保護機能もある。

> malloc 関数と free 関数は glibcの glibc/malloc/malloc.c に実装されており，読むこともできる

malloc は、**チャンクという単位でメモリを管理**している。

これはユーザーが扱うメモリに管理用の情報を追加したものであり，チャンクのサイズは 0x20 バイト以上の 0x10 バイトの整数倍

### チャンクの先頭 0x30 バイトの構造

| 名前 | サイズ| | 
|---|---|---|
| prev_size | 0x08 |直前のチャンクのサイズ | 
| size | 0x08 | このチャンクのサイズ| 
| fd | 0x08 | 連結リストの前のチャンク| 
| bk|  0x08 |連結リストの後のチャンク | 
| fd_nextsize | 0x08 |連結リストの自分よりサイズが小さいチャンク | 
| bk_nextsize | 0x08| 連結リストの自分よりサイズが大きいチャンク| 

チャンクのサイズは 16 バイト(x86_64)の整数倍なので、サイズの下位3bit は常に0
size の下位 3 ビットは下位から順に次のように別の用途に使われている:

### PREV_INUSE(1)
直前のチャンクが使用中, またはtcache, fastbin(後述)に格納されている場合 1 

tcache と fastbin 以外の bin に格納されているチャンクは他のチャンクと統合される可能性があり，直前のチャンクが統合の対象になるかどうかのフラグと考えられる

### IS_MMAPPED(2)
128 KiB 以上のチャンクが必要なとき、malloc は自身が管理しているメモリで足りなければ、mmap 関数で直接 OS から確保する。

これはmmap 関数で確保されたかを表すチャンク


### NON_MAIN_ARENA(4)
malloc の管理領域を arena という。通常は 1つだけ存在

複数のスレッドから malloc 関数や free 関数を呼び出すときは排他制御をしてから使用するが，他のスレッドが使用していてロックに失敗したときに新たな arena が作成

この仕組みによって、1 個の arena を使用するよりも実行速度が速く、スレッド毎に arena を用意するよりもメモリ使用量が少なくなる

このフラグが 1 ならば main_arena 以外で管理されているチャンク。

### その他

ユーザーが使用していない malloc が管理するチャンクは、bin と呼ばれる 単方向もしくは双方向の連結リストに繋がれている

fd と bk はリストの前後の要素を指すポインタ。 他の bin と異なり、largebin には複数のサイズのチャンクがサイズ順にソートされて繋がれている

あるサイズのチャンクを探すとき、fd_nextsize と bk_nextsize によって前後のサイズに飛べるよう な仕組み

チャンクの最小サイズは 0x20 バイトであり，fd_nextsize と bk_nextsize が使用されるのは大きなチャンクのみ

チャンクをユーザーが使用しているときは、fd から次のチャンクの prev_size までの 領域が使用可能

チャンクが使用中のときには次のチャンクの prev_size が参照されないことを利用して、メモリ効率を高めている。ユーザーが s biteのメモリを要求したとき、s+8 を 0x20 以上の 0x10 の整数倍に切り上げたサイズのチャンクが確保される

## ヒープについてと、birdcageの攻略
これまではメモリのスタックの動作に注目することが多かった。

スタック領域が「CPU のレジスタを一時的に退避させたり、また C 言語の自動変数 (多くのローカル変数) が置かれる」ものであるのに対し、

**ヒープ領域**：メモリの動的管理 (**C 言語の malloc 関数や C++ の new 演算子でメモリを確保する**こと) で用いられる

ものである。

![ヒープとスタック](https://brain.cc.kogakuin.ac.jp/~kanamaru/lecture/MP/final/part06/img6.png "ヒープとスタックの図")

攻略方法としては、前述のとおり string::_M_p を書き換えることで、GOT から libc のアドレスを読み出す。

birdcage の脆弱性で直接書き換えることができるのはヒープだけなので、ヒープ上に偽の vtable を作り、オブジェクトの vtable へのポインタを書き換える。

偽の vtable のアドレスのためにヒープのアドレスが必要になるので、cage[0] から読み出す。


## 参考
- 江添亮のC++入門 https://ezoeryou.github.io/cpp-intro/#class%E3%81%A8%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E6%8C%87%E5%AE%9A
- Malleus CTF Pwn 2nd Edition
