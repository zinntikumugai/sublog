---
title: "Quest(OculusLink) + Vive Tracker"
date: 2020-04-01 00:05:00 +0900
author:      "zinntikumugai"
tags:        ["vr","oculus","vive","trucker"]
categories:  ["VR" ]
---

諸事情でVive Trackerを借りることができたのでフルトラをしていくよ
<!--more-->
## 必要なもの

- HMD: Quest
- PC: OculusLinkは通常の描画に加えてエンコードでGPUパワーを多く使うので余裕のあるものがいい
- USBケーブル(3.0): Type-C to Type-AかOculus公式のOculusLink用のType-C to Type-C
- ベースステーション: 1.0か2.0、最低でも1つ以上(1つあればなんとかなる)
- Vive Trucker: フルトラをするなら3つほど

追加でTruckerを固定する物があるといいです 私は百均のクロックス的なやつと伸びる靴紐で足は固定しています  
腰のトラッカーに関してはそこらへんにあった紐ですが…  

少々かかってしまいますがそれぞれ体に固定するものもあるそうです

## セットアップ

予め、SteamVR、Oculusソフトウェアをインストールしておきます(SteamVRはSteamよりインストールできます)

`C:\Program Files (x86)\Steam\config\stemvr.vrsettings`のstemvrの項目項目を修正します  

```json
{
   "steamvr" : {
      "activateMultipleDrivers" : true,
      "enableHomeApp" : false,
      "haveStartedTutorialForNativeChaperoneDriver" : true,
      "requireHmd" : false,
      "showAdvancedSettings" : true,
      "showMirrorView" : false,
      "supersampleScale" : 0.81999999284744263
   }
}
```

追加するのはこちら

- `activateMultipleDrivers : true`: 複数の種類のドライバを読み込む(これをしないとトラッカーが見えない)
- `requireHmd : false`: HMDを必須としない

この状態でクエストをOculusLinkでSteamVRを立ち上げ、Viveトラッカーを接続すると 
![](https://cdn.discordapp.com/attachments/589079124618903553/691929721990021180/unknown.png)  
この画像のように、トラッカーとベースステーションの位置があらぬところに…  
おそらく、ベースステーションが基準となるHMDが無く、クエストは独自の基準点を持ってしまっていると思われます  
この状態では縦横高さはともかく回転なども異なるためまともに使えません  

## OpenVR-SpaceCalibrator
別のトラッカーを基準に自動調整できるソフトウェアで対処します  
[OpenVR-SpaceCalibrator | GitHub](https://github.com/pushrax/OpenVR-SpaceCalibrator/releases)
よりDL

1. SteamVR終了後、インストール、SteamVRを改めて起動する
1. SpaceCalibratorのメニューをVR上で開き、基準とするデバイス(OculusTouch等)と、適応するデバイス(Vive Trucker)を選択(シリアル番号で出てくるのでものすごいわかりにくい) 
    ![](https://i.imgur.com/k8tMOsD.png)
1. それぞれのデバイスをなるべく近くになるように持つ
1. `Start Calibration`を押して、8の字に動かす

### Truckerがどれかわからん

1. SteamVRのトラッカーの割当より`腰`,`左足`,`右足`のように割り当てる
    ![](https://i.imgur.com/k3kDYhy.png)
1. コントローラーのテストより、各割当のトラッカーがどれかを確認、体につける
    ![](https://i.imgur.com/dPNaU4Y.png)
    `腰`,`左足`,`右足`などにしたとき、トラッカーのボタンは`Power`に割り当てられています
1. キャリブレーションをする  
    おすすめは腰(取り外しがしやすい位置にある)

### VRChatで使うときはどうするの?
トラッカーを認識した状態で起動するとTポーズで固まっているので、アバターの位置に合わせてください(画像では腰のトラッカーが見えてませんが、あります) 
![](https://i.imgur.com/BlzK7tK.jpg)
![](https://i.imgur.com/6aflQwh.jpg)
![](https://i.imgur.com/xB1zu1U.jpg)
おおよそで足などは動きますが、場合によっては右足だけ動かしたのに両足だけ動いてしまうことがあります  
その場合はあらためてVRChat上でキャリブレーションをしてしてください  
また、フルトラ中はstandingPlay/shitedPlayの切り替えはできません(代わりにキャリブレーションボタンになります)

## まとめ

毎度キャリブレーションをしなといけない手間がありますが、足腰が動かせるようになりました  
おそらく基本的に他のVive,Valve製のHDM以外なら応用できると思われます(ViveとValveのHMDはデフォで対応してるので)  
~~フルトラで可愛く動こう!~~

## 参考
- [Oculus Questで無線フルトラをする - 簡易設定編 | トナのブログ](https://tona.hatenablog.jp/entry/2019/12/11/231111)
- [Oculus Rift x Vive trackerセットアップガイド(SteamVR Beta 1.3.7対応ver.)  | しいの脳内日記](http://si-nounai.blogspot.com/2019/03/oculus-rift-x-vive-trackersteamvr-beta.html)