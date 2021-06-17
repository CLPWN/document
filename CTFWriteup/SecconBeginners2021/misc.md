# SECCON Beginners 2021 Misc

## Git-leak

Gitでコミット上書きされる前の情報を読み出す問題。


---

Gitはバージョン管理ツールであり，コマンドによって以前の状態を復元することができる。

> Gitのオブジェクトには、ファイルの内容のみが格納されているblob(圧縮ファイル)とblob等をツリー構造で保持するtreeオブジェクトがあります。これらのオブジェクトをコミットハッシュ(SHA-1)と紐づけることで管理しています。今回は、flagを含むblobを特定して中身を見れればflagが得られます。

問題のディレクトリに移動し，コミット履歴を見るために
```
git reflog
```
を実行。

```
e0b545f (HEAD -> master) HEAD@{0}: commit (amend): feat: めもを追加
80f3044 HEAD@{1}: commit (amend): feat: めもを追加
b3bfb5c HEAD@{2}: rebase -i (finish): returning to refs/heads/master
b3bfb5c HEAD@{3}: commit (amend): feat: めもを追加
7387982 HEAD@{4}: rebase -i: fast-forward
36a4809 HEAD@{5}: rebase -i (start): checkout HEAD~2
7387982 HEAD@{6}: reset: moving to HEAD
7387982 HEAD@{7}: commit: feat: めもを追加
```

コミットハッシュ "7387982" に対して上書きが行われていることがわかる。

git checkout コマンドでコミットハッシュの状態に戻ることができるので

```
git checkout 7387392 .
```

を実行。

lsコマンドでflag.txtが存在することを確認できる。

## Mail Address Validator
ReDoSの脆弱性。正規表現チェックの落とし穴
https://speakerdeck.com/expajp/sofalsezheng-gui-biao-xian-yi-yi-ari-redosnituite?slide=12

正規表現チェックがあるので変な記号や一般的なアドレスフォーマット(example@domain.com)に従っていないアドレスははじかれる。

ただ，学生メール(x00x000x@mail.cc.niigata-u.ac.jp)のように， .xxx.xxx.xxx というメールアドレスも存在し，文法上でも正しい。
ただ， ".xxx"の部分が多ければ多いほどチェックの行程が多くなるので，

xxx@xxx.xx.xx.xx.xx.xx.xx.xx.xx

のようなアドレスを打つと組み合わせが爆発して処理落ちする。

ソースコード上，処理落ちでタイムアウトさせればFLAGが表示される。
