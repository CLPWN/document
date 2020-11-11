
編集中。追記等ありましたらお願いします！！


# WEB分野について
一括りにwebといっても様々な技術の集合であるので、広い知識が求められる。
そのためweb分野を学習したい場合には、まず一つだけのジャンルに絞って学習する事がおすすめされることが多い。

## 目次
1. [学習リソース](#study)
2. [言語とツール](#program)
3. [ジャンル](#categoly)
    - [sql](#sql)
    - [xss](#xss)


## <a id="study"></a>学習リソース一覧
- [root me](https://www.root-me.org/?lang=en)
    clientとserverサイドのそれぞれのweb学習を行える。
    web分野以外の学習も行える
    後半の答えはネットを探しても見つからない…
- [xss game](https://xss-game.appspot.com/)
    google製のxss学習サイト
    基本的なxssを体験できる
- [dvwa](http://www.dvwa.co.uk/)
    - [環境構築の記事](https://qiita.com/y-araki-qiita/items/131efa82c4205e83fef8)
    徳丸本にあるような脆弱性について、実際に攻撃を行うことができる
    
- [htb](https://www.hackthebox.eu/)
    サーバーのソフトにある既知脆弱性を利用してサーバーを乗っ取ることができる
    - [紹介の記事](https://qiita.com/v_avenger/items/c85d946ed2b6bf340a84)

-[overthewire　natas](https://overthewire.org/wargames/natas/)
    natasは、サイト内からパスワードを見つけ出して、次のレベルの進んでいくweb問題
    overthewireや他にもlinuxなどの問題（bandit)がある。
## <a id="program"></a>言語と便利ツール
### 言語
web分野においてもプログラムを書いて解く必要のある問題が存在する。
その場合に多くはpythonといった軽量プログラミング言語がよく用いられる。勿論C言語などでも可能だが、pythonなどでは有用なモジュールを簡単に利用することができるためよく利用される。

一般的にプログラムでwebを弄ることは、webスクレイピングと呼ばれるので好きな言語で検索してみるとよいかもしれません。

例えばこちらは[pythonでのrequestsモジュールの利用方法](https://note.nkmk.me/python-requests-usage/)です。

### 便利ツール
#### ローカルプロキシ
りブラウザからのリクエストを一旦キャプチャして、内容を変更した上でWebサーバへリクエストすることができるツール
サーバとクライアント間の通信を入念に調査することが可能となる。
- burpsuite
エンタープライズ版あり
- owaspzap
オープンソース

（注）https通信を行う場合はルート証明書の設定が必要となる。

#### エンドポイント作成
- [beeceptor](https://sendgrid.kke.co.jp/blog/?p=11260)
適当な返答を返すように設定をすることが可能
- [requestbin](http://requestbin.net/r/13rerz21)
単にアクセスの詳細を見るのに使えるので,xssなどで使える


## <a id="categoly"></a>ジャンル
- [sqli](sqli/)
- [xss](xss/)
- [csrf](csrf/)