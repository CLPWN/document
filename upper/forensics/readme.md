# Digital Forensics

## 目次
1. [初めに](#prolog)
2. [What is Forensics ?](#WIF)
3. [CTFでのForensics](#ctfF)

## <a id="prolog"></a>初めに
 ここでは主にCTFでの(Digital)Forensicsの話をする。何故こんな話をするかと言うと、CTFでのForensicsがDigital Forensicsにおける学びになるかは怪しいところ。要は、CTFでは本来のDigital Forensicsは学べないということを覚えておいて欲しい。所詮CTFはクイズゲームでeSportsのような側面があることを忘れてはいけない。なので、本来のDigital Forensicsが学びたい場合は[ENISA-Technical](https://www.enisa.europa.eu/topics/trainings-for-cybersecurity-specialists/online-training-material/technical-operational)等から触れるのが良い。CTFからのアプローチはあまりおススメしない。

## <a id="WIF"></a>What is Forensics ?
　デジタルフォレンジックは、コンピュータに関連する痕跡の収集や解析のための技術を示す。具体的には、デバイスからのメモリダンプやディスクイメージ、怪しいプログラムやネットワーク上にある情報の解析を行うことである。これは主に企業のインシデントが発生した場合、事件・事故のデバイス調査に用いられる。<br><br>
 CTFにおいては主に、ファイル解析、Steganography、Memory Forensics、Diskimage Forensicsが出題される。プログラムの解析はReversing、ネットワーク解析はNetworkという括りになっている。本来これら全ては、デジタルフォレンジックの一部である。<br><br>
 加えて、多くのCTFにおいてForensicsはあまり重要視されておらず問題の質は良くない。Miscと区別がつかないような出題の仕方がほとんどを占める。CTF全体を楽しむ場合は、Forensicsは主要ツールのみ把握しておき、あまり学習に時間を割く必要はないのではないだろうか。Digital Forensicsに踏み込んでいくことはCTFから離れることを意味すると考えている。<br><br>

## <a id="ctfF"></a>CTFでのForensics
CTFで出会うForensicsの分野を大きく分けて、独断と偏見で4つくらいになる。
- **よく分からんファイルの解析**<br>
What’s this file的なもの。シグネチャが壊れていたり、ファイルフォーマットがぐちゃぐちゃだったり、バイナリ的にファイルが混じっていたり、zipファイルの解凍に奮闘したり色々。<br><br>
- **Steganography**<br>
主に画像ファイルに入れられた情報を探す。2,3年前まではツールを動かすのみで解ける問題が多かったが、今は画像のシグネチャやフォーマットからバイナリ構造まで把握できていないとチャレンジすら難しい問題が少なくない。音声ファイルを解析する場合もあるが、音声ファイルの問題はツール([audacity](https://www.audacityteam.org/))の機能が上手く扱えれば知識が無くとも解ける問題が多い。<br><br>
- **Memory Forensics**<br>
　マシンが動いているときのメモリダンプが与えられる。多くの問題は、Windows OS(XP以前)のメモリダンプである。よって、何が怪しいプロセスなのか見極めるために基本的なWindowsのプロセス関する知識を持っていることが望ましい。flagにたどり着くまでにReversingのような解析が含まれることがある。<br>
　ツールは[Volatility](https://github.com/volatilityfoundation/volatility)のみ扱えれば良い。さらに、[Volatility](https://github.com/volatilityfoundation/volatility)の機能を多く理解できていればいるほど解答は楽になるようなことが多い。<br><br>
- **Diskimage Forensics**<br>
　マシンのディスクイメージを解析する。マシン全体だったり、ddイメージだったりするがファイル容量がデカい。まず、WindowsとLinuxのディレクトリ構造に関してある程度理解していなければ無駄に時間をかけてしまう。その上、どこに重要な情報がありそうか、怪しいプログラムが置かれそうな場所はどこなのか等の知識があることが望ましい。<br>
　ツールは主に二つ、[FTK® Imager](https://accessdata.com/products-services/forensic-toolkit-ftk/ftkimager)か[Autopsy](https://www.autopsy.com/)のどちらかを用いることが多い。ただ目的のファイルを探し、確認・抽出のみを行う場合はFTK® Imagerで十分だが、その他にブラウザの履歴やイメージを取得したマシンの様々な痕跡を解析する必要がある場合はAutopsyの方が良い。ただ、Autopsyの解析スピードはマシンスペックに大きく依存する。<br><br><br><br>
 
 以上4つの分野に分けたが、先に述べたようにCTFではMiscのような認識があるようなのでこれらとは大きく異なった問題が出てくる可能性に注意していただきたい。
