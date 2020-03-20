---
title:       "Hugo をDocker-Composeでビルド、デプロイできるようにする"
subtitle:    ""
description: "環境構築とかめんどいのでDockerで全部できるようにしてみました"
date:        2020-03-21 04:32:00 +0900
author:      "zinntikumugai"
image:       ""
tags:        ["hugo","travis","docker","docker-compose"]
categories:  ["Tech" ]
---

環境構築とかめんどいのでDockerで全部できるようにしてみました

## HugoのDocker
HugoのDockerイメージはなぜか公式から提供されていません  
ローカルでの動作はバイナリを使うべきと言われても実際は
- Hugoを入れるのがめんどい
- Moduleを使うとなるとGoも入れるとかめんどい
- なんかめんどい

と、まぁめんどいんですよ(適当)

実際のところ、初期の環境はUbuntu、メイン環境はWindows、持ち歩くのはWindows  
どの環境もDockerは入っていますが、近々どのマシンもOSレベルのお掃除を検討しているので今からHugoのみ入れるのも不便です  
また、CIとのバージョン対応が今までは`travis.yml`に書き込んでいたのでバージョン差異があるとめんどかったのです

ということでDockerでやりましょ(強引)

## HugoのDockerImage
`peaceiris`さんが作成されたモノを使います  
https://github.com/peaceiris/hugo-extended-docker  
最近のHugoに対応されていますし、Hugoのリリース後その日のうちに対応するとのことです

## ファイル修正
https://github.com/zinntikumugai/sublog/commit/5f64259773826502c6c8c3d0b7f7acf822b8ec0c  
こちらの修正の通りですが多少解説も兼ねて細かく書きます

今回は`docker-compose`を使うので以下の通り

```yml
version: '3'

services:
  hugo:
    container_name: hugo
    image: peaceiris/hugo:v0.67.1
    # image: peaceiris/hugo:v0.67.1-mod  # Hugo Modules
    ports:
      - 1313:1313
    volumes:
      - ${PWD}/src:/src
    command:
      - server
      - --bind=0.0.0.0
      - --buildDrafts
```

`image`にHugoのバージョンを書いていますがdocker-composeの環境変数にしたほうがいいかもしれない(正確にはymlの環境変数だっけかな)

Hugoで必要なファイル類は`src`ディレクトリ以下に配置してください(あくまでも好みですが)

追加で修正が必要なのはHugoのテーマでしょうか  
基本的に

    - git submodule
    - zipでDLして配置

のいずれかですが、submoduleを使っていたので`.gitmodules`を書き換えています  
実際はフォルダ削除とか色々してみましたが、初期pull状態でsubmoduleのpullをしてなかったところsubmoduleを再同期しても動作していなかったので、改めて
```bash
rm -rf themes/cleanwhite
git submodule add https://github.com/zhaohuabing/hugo-theme-cleanwhite.git src/themes/cleanwhite
```
と実行しました

これで
```
docker-compose up
```
を実行して、`http://localhost:1313`にアクセスすることでローカルテストすることができます

## CIの対応
https://github.com/zinntikumugai/sublog/commit/a59e6088fdc853ef1908b2d44e0bb3e004b058cb  
TravisCIでGitHubPages にデプロイしているのでこれらに対応する必要があります  

```yml
language: go
dist: trusty

# using docker
services:
    - docker

# update docker
addons:
    apt:
        packages:
            - docker-ce

env:
    - DOCKER_COMPOSE_VERSION=1.25.4

install: true

before_install:
    # update docker-compose
    - sudo rm /usr/local/bin/docker-compose
    - curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > docker-compose
    - chmod +x docker-compose
    - sudo mv docker-compose /usr/local/bin

script:
    - docker-compose run --rm hugo ""

deploy:
    provider: pages
    local-dir: src/public
    skip-cleanup: true
    github-token: $GITHUB_TOKEN
    keep-history: true
    fqdn: sublog.zinntikumugai.com
    on:
        branch: master
```

以下に注意すればTravisCIでもDocker-Composeは動くようです

- Dockerを更新する必要がある
- Docker-Composeも更新する必要がある
- deployの参照フォルダの修正(`src`ディレクトリに変更した場合)

Dockerの更新ですが通常のインストール(`apt`)同様に行う方法と`addons`の項目があるようです  
なんとなく`addons`にしてみましたが3/21の時点で`18.06.3~ce~3-0~ubuntu`でした

## 使ってみて
各環境を用意しなくてもいいが、デプロイ周りが余計複雑になってる気がする  

今回はHugoのみのDockerイメージを使いましたが、Hugo module込のイメージはかなり大きくなるとのこと(100MB未満が400MBほどに...)  
まぁ、テーマが対応してなかったので使う必要がないと割り切りましたが使おうと思えばすぐ対応できるのでそのときに考えましょ

## 参考
- [Hugo が動く最小構成の Docker イメージを作る](https://qiita.com/peaceiris/items/14d1a0f17dd25911e33b)
- [Hugoフォルダの構成](https://hugo.nakaken88.com/master/directory-structure/)
- [Travis CIでDockerを利用したらハマったしょうもない日記](https://qiita.com/niisan-tokyo/items/2f4a0c904a7c6bfcc367#docker-compose%E3%81%AE%E3%83%90%E3%83%BC%E3%82%B8%E3%83%A7%E3%83%B3%E3%81%8C%E5%8F%A4%E3%81%84%E3%82%88)
- [Install Docker Compose](https://docs.docker.com/compose/install/)
- [Using Docker in Builds](https://docs.travis-ci.com/user/docker/)
- [Hugo v0.56 新機能 Modules と deploy](https://qiita.com/peaceiris/items/fa71db25080756e9a9bf)