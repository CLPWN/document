ssdの中身削除方法を調べると、[SSDをSecure Erase機能で安全に消去する - Qiita](https://qiita.com/nvsofts/items/f46c8a1b3541894ad382)とか`hdparm`を利用した方法が出てくる。<br>
<br>
しかし、例えば[ubuntu - hdparm doesn't read SSD? HDIO_DRIVE_CMD(identify) failed: Inappropriate ioctl for device - Super User](https://superuser.com/questions/1412504/hdparm-doesnt-read-ssd-hdio-drive-cmdidentify-failed-inappropriate-ioctl-fo)とかで言われている通り、`hdparm`はnvmeのssdだと使えないので`nvme-cli`使いましょう。<br>
<br>
インストール(apt)
``` sh
sudo apt install nvme-cli
```
nvmeのssd削除使用例
``` sh
sudo nvme format -s1 /dev/nvme0n1
```
`/dev/nvme0n1`は削除したいデバイスを選択して適宜置き換え。