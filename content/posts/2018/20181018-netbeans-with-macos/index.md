---
title: 'MacでNetBeansを使いたい→使えた！'
date: 2018-10-18T16:58:05+09:00
draft: false
categories: ["macOS", "Java"]
description: 
# original: https://mikuta0407.net/wordpress/index.php/2018/10/18/post-91/
---

(この記事はQiitaからのコピーです。 本記事の投稿日はQiitaでの投稿日としています。)

某大学のプログラミングの授業ではJavaのプログラミングをNetBeansを使用して行っています。  
自分のマシンにもNetBeansを入れれば､課題の提出ができる(コーディングはVSCodeでやってる)ので､ぜひ入れたいのですが､これまた幾ら頑張ってもMacで使えません。  
とりあえずやるところまでやったので､だれか助けてください。  
**追記: 無事に動きました**

## 環境
- マシン: MacBook 2017
- OS: macOS Mojave

## Oracle JDKを入れてみた
とりあえず､最初はOracle JDKをGUIでインストールしました。

しかし､NetBeansはメインウィンドウが出てきても､新しいプロジェクトのボタンや､その他ボタンを押してもほとんど反応がありません。

以前LinuxでJDKやJREを複数入れたことによって同じ現象が起きたので､そのへんかな､と思い､とりあえずJREとJDKをすべて削除､Mac内からJavaの実行系のものをすべて取っ払いました。

## OpenJDKを入れてみた。

JREは入れずに､OpenJDK11だけを入れてみましたが、症状が改善しないです・・・  
ではnetbeans.confにjdk-homeにopenjdk11の場所を書いてあげます。

うーん改善しないぞ

## OpenJDKのバージョンを落としてみる

JDK11がNetbeans8.2では動かないという情報を聞いたので、OpenJDK9,10を入れてみました。  
それぞれnetbeans.confに書いてみたりもしました。

それでも症状が改善しません。

## NetBeansの開発版を入れてみる
http://bits.netbeans.org/download/trunk/nightly/latest/  
ここにあるNetBeansの開発版(Build 201804200002)を入れてみました。

うーん、LaunchPadに追加はされますが、起動させようとしてもウィンドウが出てきません・・・

## 助けて

万策尽きました。

どなたか対策方法をご存じの方、教えていただけませんか・・・・？

## 追記 動いた
https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html  
ここからjdk8をダウンロードしてきます。
普通にインストールして、

```config {name="netbeans.conf"}
netbeans_jdkhome="/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home"
```

と書いてあげることで動きました。

JDK8がMojaveでも動いてよかったです。

