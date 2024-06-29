---
title: 'Java AppletをIDEを使わずにコマンドラインで簡単に使いたい'
date: 2018-10-11T16:59:00+09:00
draft: false
categories: ["Java", "macOS", "Linux"]
description:  
# original: https://mikuta0407.net/wordpress/index.php/2018/10/11/post-93/
---

(この記事はQiitaからのコピーです。 本記事の投稿日はQiitaでの投稿日としています。)

ke9000氏が書いた「 [Java AppletをIDE使わずにビルドしたい](https://qiita.com/ke9000/items/3fa9b3645ceeff5ec223) 」を試していたところ、一つのプロジェクトならいいが、学校の授業のように頻繁にソースコードを切り替えるときには、毎回HTMLファイルを書き換え、Classesフォルダを作り・・・っとやるのは大変なことに気づいた。  
そこで、LinuxやMac等のOSに限るが、簡単に操作ができるようにしてみる。  
もちろんやってることは当たり前すぎて、参考になるほどでもないが、備忘録として書いておく。  
基本的に先述の記事を読んだことを前提に進めるが、一応適宜コードは書いていく。  
(**注: Classesフォルダに毎回コピーをしているのは、授業ではNetBeansを使用しているため、srcフォルダから元ソースファイルを動かせないためである**)

## フォルダ構成について
今回は、~/Javaというフォルダ内で作業する。
Java/の中にプロジェクトフォルダがあり、その中にソースコードがあるという前提で話をすすめる。

## HTMLファイルを用意する
```html {name="applet.html"}
<html>
<head><title>Applet Test</title></head>
<body>

<applet code="CLASS_NAME.class" width="150" height="150">
</applet>

</body>
</html>
```

これを**~/Java/applet.html**として保存する

## シェルスクリプトを書く
```shell {name="applet.sh"}
#!/bin/sh

javac $1.java
# 引数として渡された名前の.javaファイルをコンパイル
 
if [ -e $1.html ]; then
    :
else
    cat <Javaフォルダのパス>/applet.html |  sed "s/CLASS_NAME/$1/" > $1.html
fi
# 先程作ったhtmlファイルをベースに、「CLASS_NAME」をそのときに使うファイルに変更したものを作る。あったら作らない。

mkdir Classes 2>/dev/null
# Classesフォルダがなかったら作る

cp *.class Classes
# コンパイルでできた.classファイルをClassesにコピー/上書きコピー

appletviewer $1.html
# Appletを起動
```

mkdirのところはifで分岐させても良かったが、エラーは単純に「すでにあるから作れないよ！」なので、表示させなければそれでいいということでこうした。  
ちなみにapplet.htmlとapplet.shは同じディレクトリにいることを前提としている

## エイリアスに登録する
今作ったシェルスクリプトを、.javaフォルダのあるところで簡単に実行できるようにbashrcに書く

```shell {name=".bashrc"}
alias applet='<applet.shのパス>/applet.sh'
```

## 使ってみる

アプレットを使用したプログラムのソースコード(hoge.java)の位置まで移動して、  
`applet hoge`(.javaはいらない)  
で、コンパイルから指定フォルダまでの移動、起動までやってくれる

以上。

~~元記事でも言ってるけど、JavaAppletって非推奨なんじゃないの？~~

