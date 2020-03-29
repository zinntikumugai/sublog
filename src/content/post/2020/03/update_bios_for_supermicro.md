---
title: "supermicroのマザーボードをBIOS更新する"
description: "ちょっと諸事情で調子悪かったのでBIOS更新してみました"
date: 2020-03-26T13:55:23Z
author:      "zinntikumugai"
tags:        ["supermicro","bios","bios update","dos"]
categories:  ["Tech" ]
---

# SupermicroのマザーボードをBIOS更新する気分的に更新してみる

<!--more-->
使うマシン

|HW|Name|
|:-|:-|
|MB| supermicro X7SPA-HF-D525 |
| CPU | Intel Atom D252 |
| Ram | 4GB |

## 公式サイトよりBIOSをダウンロード  
https://www.supermicro.com/products/motherboard/ATOM/ICH9/X7SPA-HF-D525.cfm  
展開してReadmeを見ると
```
	1. Save this file to your computer.

	2. extract the files to a DOS bootable device (such as a bootable USB stick, or CD).

	2. Boot to a DOS prompt and type AMI.BAT BIOSname.###.

	4. Do not interrupt the process until the flashing is complete.

	5. After you see the message of BIOS has completed the update, unplug the AC, clear the CMOS and plug in the AC and power on the system.

	6. Go to the BIOS setup screen and press F9 to load the default and press F10 to save and exit.
	

** If the BIOS flash failed, you can contact our RMA dept. to have the bios chip reprogrammed.
This will require shipping the board to our RMA dept. for BIOS reprogramming.  
The RMA dept's email address is rma@supermicro.com
```
と書かれているのでどうやらDOSが必要みたいです
## DOSの準備
RufusでFreeDosを入れます
![](https://i.imgur.com/vhl0MKU.png)
ブートの種類で選択したぐらいですが適当なUSBメモリに焼きます(microSDを使いましたが)
![](https://i.imgur.com/0IC7VuB.png)
ダウンロードしたファイル類をUSBメモリーの直下に展開して準備OKです

## Update
BIOSの順序を変えるなりしてUSBメモリーから起動します  
FreeDOSが起動したら
```
ami x7spa3.719
```
と実行すればあとは勝手にやってくれます
![Imgur](https://i.imgur.com/n99CQBd.jpg)
![Imgur](https://i.imgur.com/vxRdEHK.jpg)
最近のマザーは普通にBIOS/UEFIに更新機能があるので多少て惑いましたが、DOS更新も大して変わりませんね  
ちなみに本来の目的的にはBIOS更新では治りませんでした(知ってた)

## 参考
- https://www.8volt.com/stk2/index.php/cgi-vfx/system/377-supermicro-7047a-t-x9dai-bios-update

