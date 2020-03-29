---
title: "Proxmox入れるときに手こずった"
description: "余ってるマシンにProxmoxを入れる話"
date: 2020-03-26T13:57:06Z
author:      "zinntikumugai"
tags:        ["proxmox","kvm","openvz"]
categories:  ["Tech" ]
---

使ってないマシンにProxmoxを入れてみる

<!--more-->
## Proxmoxとは？
仮想環境を提供するサーバーソフトウェア  
無料でWeb管理,CPUフルパワーで使えるのが強い(VMwareとかね...制限がね..)

## 通常インストール
公式イメージを使ってインストールをしてみる  
![](https://i.imgur.com/qguK8Rq.jpg)
`no cdrom found`と出て、なんかうまく行かない  

> Download Rufus from https://rufus.ie/. Either install it or use the portable version. Select the destination drive and the Proxmox VE ISO file. 
> 
> Once you click Start a dialog asking to download a different version of GRUB will show up. Click No. In the next dialog select the DD mode.
> 
> An alternative to Rufus is Etcher. Download Etcher from https://etcher.io. It will guide you through the process of selecting the ISO and your USB Drive.

公式wiki よりRufus等を使う場合はisoモードではなくddモードを使えとのこと  
いつものノリでやってたのでRufusのisoモードを使ってました

ddモードにして行くとすんなり起動してくれたけどインストールはコケる...  
DHCPでコケるみたいだけど、debugモードにして`/etc/networking/interface`をいじってもだめ(おそらく見てない)

何回やってもだめなので諦めてDebianを入れてからprxmoxを入れることにする

## DebianからProxmox
### Debianを入れる
Debian公式からイメージを取得して適当にインストールする  
https://www.server-world.info/query?os=Debian_10&p=install
### Prxmoxを入れる
https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_Buster  
公式wikiを参照の上インストールしていく
#### `/etc/hosts` の修正
今回のマシンは`192.168.0.19`、ホスト名は`node3`だったので
```
127.0.0.1       localhost
#127.0.1.1      node3
192.168.0.19    node3

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
コメントアウトの`127.0.1.1`は不要  
というかこれを追加したせいで進まなかった()
```
hostname --ip-address
```
コマンドで指定したIPが帰ってくるか確認しておく
#### install
下準備(リポジトリの登録)
```
echo "deb http://download.proxmox.com/debian/pve buster pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list
wget http://download.proxmox.com/debian/proxmox-ve-release-6.x.gpg -O /etc/apt/trusted.gpg.d/proxmox-ve-release-6.x.gpg
chmod +r /etc/apt/trusted.gpg.d/proxmox-ve-release-6.x.gpg  # optional, if you have a non-default umask
apt update && apt full-upgrade
```
そしてインストール
```
apt install proxmox-ve postfix open-iscsi
```
何故かコケたりしたがhostsのミスやホスト名のミスをしてた  
後処理
```
apt remove os-prober
```

無事インストールができると`https://<ip>:8006`に接続すると管理パネルにつなげることができる

## 懸念点
### マシンスペックがゴミ
CPUにAtom,メモリが4Gとかもうね...  
NASにしておいたほうが良かった説もあるがそのうちやっていこう(多分)

### RAIDが弊害？
もともとSMB鯖だったのでRAID5?がしてあった(多分)  
OSのインストール先をSATADOM(8GB)にしたためデータドライブとして使ってくれると思ったが使ってくれない  
ストーレジ自体は認識してるので使えそうだけど勝手にやってくれないっぽい  
![](https://i.imgur.com/cRqmTSO.png)  
後でRAID解除してフォーマットしたりしたが未使用のディスクとしては認識してくれないようで...  
ハードウェアRAIDのほうが楽だなぁ...
