# Windows側のVPNをWSL2にも適用する方法

- Windows10 でWSL2を使ってTryhackmeを攻略したい人向け.

1. Windows環境でVPNを設定する
2. ユーザーディレクトリにファイル .wslconfig を生成
3. 値 localhostForwarding=True を書き込む
4. 再起動(一応)

- .wslconfigでメモリやCPUコアの割り当てとかも制御できたかも?
