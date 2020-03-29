---
title: "Hugoのテーマをモジュール化する"
subtitle:    ""
description: "バージョン管理も含めHugo Modulesを使ってみました"
date: 2020-03-28T08:38:58Z
author:      "zinntikumugai"
image:       ""
tags:        ["hugo","hugo modules","docker","docker-compose"]
categories:  ["Tech" ]
---

# Hugoのテーマをモジュール化する

なんとなくできたらいいなぐらいでやってみます  
<!--more-->
というより、やりましたの記事が無い(ﾄﾞｳｼﾃ...  
今回もDocker版Hugoでやっていきます  
バイナリ版の場合はgoとgitを使える環境を整えましょう

## git submodule のリポジトリを消す
先に消してしまいます

```bash
$ git submodule deinit -f src/themes/cleanwhite
```
`-f PATH`のところは各自のものに置き換えてください

```bash
$ rm -rf src/themes/
```
テーマフォルダも中身がないので消してしまいます
```bash
$ rm -f .gitmodules
```
他にsubmoduleを使うものも無いので消してしまいます

## `docker-compose.yml`の修正
HugoのDockerイメージは軽量版はモジュールを扱えないので対応版に変更します
```yaml
version: '3'

services:
  hugo:
    container_name: hugo
    # image: peaceiris/hugo:v0.67.1
    image: peaceiris/hugo:v0.67.1-mod  # Hugo Modules
    ports:
# ～～ 以下略 ～～
```

## `config.toml`の修正
```toml
baseURL = "https://sublog.zinntikumugai.com/"
languageCode = "ja"
title = "さぶろぐ!"
# theme = "cleanwhite"

# ～～ 中略 ～～ 

[module]
    [[module.imports]]
    path="github.com/zhaohuabing/hugo-theme-cleanwhite"
```

まず、(おそらくどこでもいいと思いますが)末尾に利用するモジュールのリポジトリを追加します
今回はテーマのみなので一つですね  
そして、__`theme=<テーマ名>`を消します__  
このモジュールとして追加したテーマなんですが、どうも名前が変わってしまうのか、`theme=<テーマ名>`の記載があるときはローカルフォルダを見てしまうようで、そんなテーマはないと言われてしまいました
![](https://i.imgur.com/krVMitt.png)


## モジュールの初期化
最後に初期化をしないと動きません
```bash
$ docker-compose.exe run hugo mod init github.com/zinntikumugai/sublog
```
これはDocker版なのでバイナリ版は
```bash
$ hugo mod init リポジトリURL
```
リポジトリURLはこのプロジェクトのリポジトリ(サイトのリポジトリ)のURLを追加してください
これを行うと`go.mod`というファイルが作成されます

## 動作
```bash
$ docker-compose up
```
をすると
![](https://i.imgur.com/2ib0oF9.png)
![](https://i.imgur.com/FUC9wQo.png)

git submoduleでやってたときのように動かすことができました  
また、モジュールを取得するときに`go.sum`というファイルも追加されました(チェックサム用のようです)

## その他
モジュール関連のコマンドに
```bash
$ hugo mode vendor
```
という`_vendor`ディレクトリにモジュールの依存関係を保存するコマンドがありますが、PHPのComposer的なやつかと思っていまいしたが、試したときはモジュールの外部読み込みがなくなるだけのようなのでこのブログにおいては必要なさそうです

## 参考
- https://qiita.com/peaceiris/items/fa71db25080756e9a9bf
- https://gohugo.io/hugo-modules/use-modules/
- https://gohugo.io/commands/hugo_mod_init/
- https://qiita.com/uegaki-masaaki/items/f6d625c92bb355ccc409
