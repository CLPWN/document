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

## 参考
- 江添亮のC++入門 https://ezoeryou.github.io/cpp-intro/#class%E3%81%A8%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E6%8C%87%E5%AE%9A
- Malleus CTF Pwn 2nd Edition
