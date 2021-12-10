# CLPWN - Pwn
pwn関係の勉強会用の資料置き場。

## 備忘録
```a.sh
# 逆アセンブラ
objdump -d -M intel bin > bin.txt

# LIBC GOT 検索
objdump -T libc-2.xx.so | grep ' func'

# 実行ファイルのサイズが大きい場合など、きっちり値を計算したいとき
# 各セクションのファイル中のオフセットと実行時に配置されるメモリアドレスを調べる。
readelf -S bin
hexdump -C bin

# 各種ツールインストール
pip3 install pwntools
apt install binutils gcc g++
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit

# セキュリティ機構チェック
checksec --file=bin

# One_gadget RCE インストールと使用方法
# 最新版の方がより多くのRCEを見つけられるので、自力ビルドを推奨。
apt install ruby-bundler ruby-dev make gcc
cd
git clone https://github.com/david942j/one_gadget.git
cd one_gadget
bundle install --path vendor/bundle
bundle exec one_gadget /lib/x86_64-linux-gnu/libc-2.31.so
```
