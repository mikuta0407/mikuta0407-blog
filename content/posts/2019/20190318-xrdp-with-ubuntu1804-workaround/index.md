---
title: 'Ubuntu 18.04.2でxrdp(のXorgモード)が使えないので一時的に凌ぐ'
date: 2019-03-18T16:56:00+09:00
draft: false
categories: ["Linux", "Ubuntu", "RDP"]
description:  
# original: https://mikuta0407.net/wordpress/index.php/2019/03/18/post-85/
---

(この記事はQiitaからのコピーです。 本記事の投稿日はQiitaでの投稿日としています。)

おそらくすでにたくさんある情報のn番煎じなんですが､Ubuntu18.04.2に関する情報がかなり少なかったので､同じような人がいたとき用に書いておきます。  
(ちなみに試した環境はXubuntuですが､コマンド的にどのフレーバーでも動きそうな気がします)

## xrdpがUbuntu18.04.2で使えないらしい
https://note.spage.jp/archives/576  
https://ubuntuforums.org/showthread.php?t=2413157  
ここの情報のおかげでわかったのですが､どうも18.04.2ではxrdpのXorgモードを使う際に必要なxorgxrdpが対応していない?らしく､(少なくとも簡単には)動かないようです。

無理に弄って壊しても嫌ですし､いつか対応することを前提に､xorg経由ではなく、X11VNC経由でXRDPを使うことで凌ぎます。

## 1. x11vncをインストールする
本来であればsudo apt install xrdpと少しの操作で使えるようになるはずのxrdpがXorgモードで使えないということなので､できるだけ手間を省いていきます。
(xrdpは入ってる前提)

```shell

sudo apt install x11vnc
x11vnc -storepasswd #パスワードを設定します
sudo nano /etc/xrdp/xrdp.ini
```

## 2. xrdpの設定を変える
xrdpの設定ファイルで､Xvncに関する設定を少し変更します

```ini {name="/etc/xrdp/xrdp.ini"}
[Xvnc]
name=Xvnc
lib=libvnc.so
username=ask
password=ask
ip=localhost 
# ↑ここを一応127.0.0.1からlocalhostに変えておく
port=5900 
# ↑-1を5900に
```

これで設定は終わりです。

## 3. X11VNCを起動して、RDPクライアント経由でつなぐ
あとは

```
x11vnc -localhost -forever -bg -display :0
```

を叩いておけば､

- localhostのみに(-localhost)､
- ずっと(-forever)､ 
- バックグラウンドで(-bg)

開くVNCサーバーが立ち上がります。

あとは、先程の設定のおかげでXRDPがVNC経由でつないでくれますので、お好きなRDPクライアントで接続しましょう。

VPSみたいに生VNCが危なくて使えないときには､xrdpを経由することでちょっと安全な認証が使えますので､そういうときにいいですね。  
xorgxrdpがちゃんと対応しない間はこれで凌ごうと思います。

(2020/02/03編集: XRDPの代わりにVNCで繋げばいいじゃんみたいな表現に見えた気がするのですこし修正しました)
