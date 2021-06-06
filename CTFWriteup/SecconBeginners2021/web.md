# SECCON Beginners 2021 Web

## osoba

問題フォルダをダウンロードするとflagファイルが見つかるが，当然そこには回答は書かれていない。

問題フォルダの階層構造から推測し，[問題webページ](https://osoba.quals.beginners.seccon.jp)のflagに辿り着きたい。

### 攻撃手法:ディレクトリトラバーサル

traverseという単語は横断するという意味であり、悪意のある利用者がパラメータに「 ../」といったディレクトリを横断するキーワードを含め、
サービス提供者が予期していないディレクトリへのアクセスを目論む攻撃のこと。

### 攻略
各種リンクは

https://osoba.quals.beginners.seccon.jp/?page=public/

に存在するので，そこから相対パスを使って遡る。

```
https://osoba.quals.beginners.seccon.jp/?page=public/../../flag
```

が正解

### 参考サイト・おまけ
https://qiita.com/boxboxbax/items/85948e93f367b1289a93
こちらのサイトにディレクトリトラバーサルの説明と対策が記載されている(python, c++,c#)。

## werewolf


配布されているapp.pyではWEREWOLFが抽選の対象から外れており、しかもroleプロパティにsetterが無い状態となっている。(つまりPlayerクラスないの変数の値変更が可能)

今回のプログラムでは抽選結果を__roleに保持しているため、_Player__roleにWEREWOLFが入るようPOSTしてやればよい。

> Pythonでは、プライベート変数の代替機能としてアンダースコア2つ__をアタマに着けたメンバ名は、クラスの外側からは_<CLASSNAME>__<MEMBERNAME>のようにしないとアクセスできない機能がある

```
curl -X POST -F '_Player__role=WEREWOLF' https://werewolf.quals.beginners.seccon.jp/
```

> curl POST の -F について: curl で -F オプションを使うと、curl がよしなに MIME type を multipart/form-data に設定してくれて、key=value 形式で POST することができます。

Python3のオブジェクト指向やクラスの知識が必要なので各自で調べてみてください

## check_url

https://check-url.quals.beginners.seccon.jp/
urlパラメータに渡したurlにサーバがcurlしてくれるのでウマいことループバックアドレスを渡してadminになりすます。

なお，半角英数しか入力できない。localhostもダメ。

そこで，localhostの10進表記 127.0.0.1 の16進表記 **0x7f000001**を入力。

```
https://check-url.quals.beginners.seccon.jp/?url=http://0x7f000001
```