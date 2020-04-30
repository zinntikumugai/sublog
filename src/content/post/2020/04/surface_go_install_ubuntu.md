---
title: "surface goにUbuntuをインストールする"
description: "Suraface GoにUbuntuを入れていきます"
date: 2020-04-09T21:29:58Z
author:      "zinntikumugai"
tags:        ["surface", "ubuntu"]
categories:  ["Tech" ]
---

~~クソスペ~~Surface GoにVMは向いていない気がするので直接Ubuntuを動かせるようにしてみます  
マルチブートしてもいいですが、なんとなく分けたかったので内蔵SSDには入れない方法で行きます
<!--more-->
## Ubuntu 18.04をインストール

### 用意しておくもの
- Type-Cのハブ
- 8GB程度の記憶メディア
- インストール先の記憶メディア
- USB接続のNIC
 
まず、Surface GoはType-Cしか拡張IOがない上にmicroSDからのブートはできないようなので適当なType-C のハブを用意しておきます  
8GBの記憶メディアはインストール用Liveイメージを書き込んでおきます  
インストール先の記録メディアは今回は64GBのmicroSDカードを使います(インストール先としてはmicroSDスロットは使えます)  
Surface Go内蔵WiFiはデフォルトで使えないため、適当なUSB接続のNICをを用意しておきます  
今回は[バッファローのやつ](https://amzn.to/2XvDuEq)を使いました  
他のものでも構いませんがLinuxで追加ドライバなしで動くものが便利です  

### Liveイメージを書き込む
8GBの記憶メディアにUbuntu公式か、日本語Ubuntuからisoイメージをダウンロードしておきます  
Rufusなどで書き込んでおけばOKです

### インストール
ハブにすべて接続して一旦Windowsを起動します  
Windowsの設定を開き、更新とセキュリティ、回復へと進み、PCの起動をカスタマイズするの`今すぐ再起動`を押し、再起動します  
WindowsBootマネージャーから外部のメディアを起動、ubuntu的なやつを選択します  
正常に立ち上がるとインストーラーが起動するのでそのままインストールします

## セットアップ
初期状態はタッチパネルも使えないので追加のドライバ類をインストールします  
まずは更新
```bash
sudo apt update
sudo apt upgrade
```
(以降はめんどいので`sudo`を省略しています)

### カーネルのインストール
https://github.com/jakeday/linux-surface
に従いインストールします
```bash
apt install git curl wget sed

git clone --depth 1 https://github.com/jakeday/linux-surface.git ~/linux-surface
cd ~/linux-surface
sh setup.sh
```
インストール中にいくつか質問がされるので内容に応じてyes/noで答えます

- サスペンドを休止状態にするか
- xorgのサンプルコンフィグを消すか
- パルスオーディオのサンプルコンフィグを消すか
- RTCをUTCに合わせるか
- カーネルをインストールするか

一旦再起動します
```bash
reboot
```

このまま起動するとSecureBootがONになってる場合はgrubが立ち上がりません  
このままでは進まないので自己証明書で証明します  
参考は[こちら](https://github.com/jakeday/linux-surface/blob/master/SIGNING.md)

`mokconfig.cnf`というファイルで作成します
```conf
HOME                    = .
RANDFILE                = $ENV::HOME/.rnd 
[ req ]
distinguished_name      = req_distinguished_name
x509_extensions         = v3
string_mask             = utf8only
prompt                  = no

[ req_distinguished_name ]
countryName             = <YOURcountrycode>
stateOrProvinceName     = <YOURstate>
localityName            = <YOURcity>
0.organizationName      = <YOURorganization>
commonName              = Secure Boot Signing Key
emailAddress            = <YOURemail>

[ v3 ]
subjectKeyIdentifier    = hash
authorityKeyIdentifier  = keyid:always,issuer
basicConstraints        = critical,CA:FALSE
extendedKeyUsage        = codeSigning,1.3.6.1.4.1.311.10.3.6
nsComment               = "OpenSSL Generated Certificate"
```
`<YOUR~>`とあるところは適時変更してください

```bash
openssl req -config ./mokconfig.cnf \
        -new -x509 -newkey rsa:2048 \
        -nodes -days 36500 -outform DER \
        -keyout "MOK.priv" \
        -out "MOK.der"
openssl x509 -in MOK.der -inform DER -outform PEM -out MOK.pem

```
cnfファイルから公開鍵と秘密鍵を生成します  
また、PEM型に変更します
```bash
sudo mokutil --import MOK.der
```
生成したキーをインストールします  
このとき、パスワードを要求されますがキーを有効化するときに使うので適当に設定します

```bash
reboot
```
そして再起動します

### カーネルの署名

![](https://i.imgur.com/DcyfLyx.jpg)


起動したらこんな画像になるので、`Enroll MOK`を選択

![](https://i.imgur.com/rYVKinJ.jpg)


`View key 0`を選択します

先程生成したキーか確認します

![](https://i.imgur.com/DX49aRk.jpg)

Enterとか押したら有効するか聞かれるのでYesを選択します  
そのままrebootします


```bash
mokutil --list-enrolled
```
一応インストールされたキーを確認してみます

```bash
sbsign --key MOK.priv --cert MOK.pem /boot/vmlinuz-[KERNEL-VERSION]-surface-linux-surface --output /boot/vmlinuz-[KERNEL-VERSION]-surface-linux-surface.signed
```
生成したキーで署名します

```bash
cp /boot/initrd.img-[KERNEL-VERSION]-surface-linux-surface{,.signed}
```
署名したキーと未署名をコピーします

```bash
update-grub
```
これらの変更をgrubに反映します

再起動して起動時ののカーネル選択で、署名したカーネルを選択します

正常に起動すれば、タッチパネル等が機能します

```bash
mv /boot/vmlinuz-[KERNEL-VERSION]-surface-linux-surface{.signed,}
mv /boot/initrd.img-[KERNEL-VERSION]-surface-linux-surface{.signed,}
update-grub
```
不要になった未署名のカーネルを署名済みのカーネルで上書きしてしまいます

次回以降のカーネル更新も同様に署名してください

### スワップファイルの修正

休止状態の変更をした場合スワップファイルを修正します
参考は[こちら](https://fitzcarraldoblog.wordpress.com/2018/07/14/configuring-lubuntu-18-04-to-enable-hibernation-using-a-swap-file/)と[こちら](https://wiki.archlinux.jp/index.php/%E3%82%B5%E3%82%B9%E3%83%9A%E3%83%B3%E3%83%89%E3%81%A8%E3%83%8F%E3%82%A4%E3%83%90%E3%83%8D%E3%83%BC%E3%83%88)

```bash
swapoff -a
```
一旦すべてのスワップを外します

```bash
dd if=/dev/zero of=/swapfile bs=1M count=8192
```
メインメモリと同容量のスワップを作成します(8GB)

```bash
grep swapfile /etc/fstab
```
念の為`/etc/fstab`にswapfileの記述があることを確認します

```bash
blkid
```
接続しているデバイスから、swapfileのあるドライブパーテーションのUUIDをメモしておきます

```bash
apt install uswsusp
```
`filefrag -v /swafpfile`というコマンドでswap offsetを調べられるのですが、非常にわかりにくいため`uswsusp`をインストールします

```bash
swap-offset /swapfile
```
こちらのコマンドでoffsetを調べます  
こちらもメモしておきます

```bash
vim /etc/default/grub
```

`GRUB_CMDLINE_LINUX_DEFAULT`という項目に`resume=UUID=<UUID> resume_offset=<RESUME_OFFSET> resumedelay=15`を追加します

```bash
update-grub
```
grubを更新します

`/etc/initramfs-tools/conf.d/resume`に同様に下記の通り追加します
```
RESUME=UUID=<UUID> resume_offset=<OFFSET>
```
```bash
update-initramfs -u -k all
```
initramfsを更新します

`/etc/polkit-1/rules.d/85-suspend.rules`に下記を書き込みます
```
polkit.addRule(function(action, subject) {
    if (action.id == "org.freedesktop.login1.suspend" ||
        action.id == "org.freedesktop.login1.suspend-multiple-sessions" ||
        action.id == "org.freedesktop.login1.hibernate" ||
        action.id == "org.freedesktop.login1.hibernate-multiple-sessions")
    {
        return polkit.Result.YES;
    }
});
```

`/var/lib/polkit-1/localauthority/50-local.d/50-enable-suspend-on-lockscreen.pkla`ファイルに下記を追加します

```
[Allow hibernation and suspending with lock screen]
Identity=unix-user:*
Action=org.freedesktop.login1.suspend;org.freedesktop.login1.suspend-multiple-sessions;org.freedesktop.login1.hibernate;org.freedesktop.login1.hibernate-multiple-sessions
ResultAny=yes
ResultInactive=yes
ResultActive=yes
```

最後に再起動をかければOKです

ロックをかけ、休止状態をかけるとメモリのデータをすべてswapに書き込むためフリーズみたいになりますが休止状態に移行します  
今回USB 2.0接続のmicroSDに書き込んでいるため非常にメモリのデータに書き込みが遅いですができました