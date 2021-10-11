# Web Fundamentals


## ウェブサイトのロード

### サーバーの検索

最初にDNSに問い合わせを行う。DNSはURLを渡すとIPアドレスを返す電話帳のようなものです。これのおかげで私たちはIPアドレスを覚える必要がありません。

IPアドレスはインターネットに接続したデバイスを識別します。バージョン４と呼ばれる現在のIPにおいてアドレスは32ビットの正数値で表されます。これを通常8ビット（0-255)ずつ4つに区切り、それぞれを10進法で表して、「.」で繋ぎます。例えば2進法で「10000101.01000001.11000000.00010001」となっているアドレスなら「133.65.192.14」と表せます。


## コンテンツの読み込み

ブラウザ(fire foxなど)はサーバーのIPアドレスを認識するとサーバーにwebページを要求できます。これはHTTPのGETを使用して要求されます。Web ページのコンテンツを使用してGETに応答します。Web ページがJavaScript、画像、CSS などの追加リソースを読み込んでいる場合、それらは別々のGETで取得されます。

![画像1](https://i.imgur.com/R04qcso.png)


### HTTPの基礎
HTTP(HyperText Transfer Protocol)とはHTMLなどの情報をやり取りするために使われるアプリケーション層の通信プロトコル（通信の約束事のようなもの）です。

![画像２](https://image.itmedia.co.jp/ait/articles/1703/29/wi-httpfig01.png)

<https://www.atmarkit.co.jp/ait/articles/1703/29/news045.html>


![画像３](http://www.kogures.com/hitoshi/webtext/nw-http/request.gif)

<http://www.kogures.com/hitoshi/webtext/nw-http/index.html>


- HTTPメソッド　ブラウザからサーバーに対して出されるお願いのようなもの。
  - GET　ヘッダとコンテンツを要求
  - HEAD　ヘッダのみを要求
  - POST　データを送信
  - PUT　ファイルをアップロード
  - DELETE　データの削除を要求する
  - OPTIONS　使えるメソッドを要求

- ターゲット　閲覧したいWebページ。URLのパスの部分。

- バージョン　現在の主流はHTTP/1.1ですが最新のものだとHTTP/3もあります。
  
- HTTPヘッダ　データや文書の先頭につけられるそれらについての情報を記述した部分。
  - User-Agent　ブラウザの種類、OS情報
  - Referer　ハイパーリンクをクリックしたURL
  - Accept、Accept-Language、Accept-Encoding、Accept-Charset  どのようなデータを受け取りたいのか、言語、文字コード、画像の種類	などの情報.
  - Content-Length  コンテンツの長さ
 

 ![画像4](http://www.kogures.com/hitoshi/webtext/nw-http/response.gif)
<http://www.kogures.com/hitoshi/webtext/nw-http/index.html>

- HTTPステータスコード
  - 100番台　処理が継続
  - 200番台　正常に終了
  - 300番台　リダイレクト
  - 400番台　クライアント側のエラー
  - 500番台　サーバー側のエラー

Cookies
![画像５](https://image.itmedia.co.jp/ait/articles/1704/20/l_wi-cookiefig01.png)
<https://atmarkit.itmedia.co.jp/ait/articles/1704/20/news024.html>
CookieとはWebサーバーがクライアントに預けておく小さなファイルのことです。
CookieにはWebサーバーによって様々な情報が格納されます。具体例としてはユーザー名などやショッピングサイトの買い物かごの情報などの管理に利用されています。

URLについて
スキーム://ホスト名.ドメイン名(:ポート番号)/(パス名/)(ドキュメント名)
具体例として
例1 <https://www.google.co.jp/>
例2 <https://www.example.com/home/list.php?page=2&sort_by=price>

- URL
  - スキーム　アクセスがどのようなプロトコルで行われているかを表しています。例 http,https,ftp
  - ホスト名　ドメイン名の横に「.」区切りで表される。またホスト名は以下の制約があります。
    - 255字以内
    - 英字[a～z]、数字[0～9]、ハイフン[-]（ハイフンが最初と最後にきてはいけない）
    - 大文字小文字は無視される
  - ドメイン名　ホスト名や組織名を区別するために階層構造を持った名前の体系のことです。上の例ではトップレベルドメインとして「jp」セカンドレベルドメインとして「co」サードレベルドメインとして「google」が使われています。
  - ポート番号　「ポート番号とは、インターネットで標準的に用いられるプロトコル（通信規約）であるTCP/IPにおいて、同じコンピュータ内で動作する複数のソフトウェアのどれが通信するかを指定するための番号。単に「ポート」と略されることもある。」<https://e-words.jp/w/%E3%83%9D%E3%83%BC%E3%83%88%E7%95%AA%E5%8F%B7.html>
  - パス名　サーバー内でのは場所を表します。
  - ドキュメント名　ドキュメント名前です。
  - クエリ文字　「パラメータ名=値&…」で表され、例２では「page=2」と「sort by=price」が「&」で区切られています。プログラムに対する命令内容を表します。

Cookieについて（追記）
Cookieの操作については「開発者ツール」を用いて行うことができます。またFirefox では、F12 で開発ツールを開くことができます。
Cookieの詳細についてはこの[ページ](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)で詳細を学ぶことができます。

## curlコマンドの使い方

$ curl [オプション] [URI]

- オプション

    - -X メソッドの指定
    - -d リクエストのHTTPボディの設定
    - -c 受け取ったcookieの保存
    - -b 実行時のcookieの設定

```
write up

5-1
curl $MACHINE_IP/ctf/get
5-2
curl -X POST -d "flag_please" $MACHINE_IP/ctf/post
5-3
curl -c outcookie.txt $MACHINE_IP/ctf/getcookie
cat outcookie.txt
5-4
curl -b "flagpls=flagpls" $MACHINE_IP/ctf/sendcookie

```
