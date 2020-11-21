# ブラウザ以外のアプリケーションの通信を確認する方法

ここでは、アプリが通信先URLの確認までを行えるようにする。
今回は具体例として burpsuite にて、暗号通貨管理アプリ？[exodus]の通信を確認する。


またブラウザ以外と記述したが、このやり方ではホストOSの通信がすべてburpsuiteを通して通信される。
試してはないが、owaspでも同じ方法でできるはず…
## 環境
- windows
- burp suite 
- exodus
## 環境図
exodusの通信をburpsuiteで挟んで、監視・改ざんを行いテストする。
1. 設定前
![](https://i.imgur.com/DCsCa6e.png)

2. 設定後
![](https://i.imgur.com/LLX7Zdo.png)

## burp側設定
[proxy]->[option]->[Proxy Liseners]にて受け取りたいアドレスとポートを指定する。
このアドレスとポートはwindows側の設定で利用するので再度参照する。


![](https://i.imgur.com/nvsJqX8.png)



## windows側設定
windowsにてプロキシの設定を開く。ここに先ほど入力したアドレスとポートを入力しプロキシサーバを指定する。
これによってwindowsの通信をburpsuiteを介して行うことができる。


![](https://i.imgur.com/xspNfp5.png)


しかしながら、これだとHTTPS通信における証明書問題にぶつかってしまうので、次の項目で解決する。

## windows上の証明書について
この辺見てください
https://www.securesky-tech.com/column/naruhodo/02.html#import-root-ca-for-win


## burpにて通信確認
証明書の問題が解決されたのでburp suite上で通信の確認ができる。

![](https://i.imgur.com/tUFqdsL.png)

![](https://i.imgur.com/evYWzYT.png)


###  エンドポイント確認
先ほどの履歴を見ると、下のようなURLがエンドポイントとなる。

```neo-mag.a.exodus.io/insight/addrs/txs?noScriptSig=&noAsm=&noSpent=&from=&to=```


## 参考資料
- burpsuite 証明書
https://www.securesky-tech.com/column/naruhodo/02.html#import-root-ca-for-win

- wacom ドライバーの通信をburpsuite で観測する
    今回の参考にした
https://robertheaton.com/2020/02/05/wacom-drawing-tablets-track-name-of-every-application-you-open/
