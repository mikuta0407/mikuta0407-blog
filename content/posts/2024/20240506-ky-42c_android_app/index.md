---
title: '【メモ】 KY-42C (DIGNOのガラホ)でのアプリ動作可否記録'
date: 2024-05-06T21:07:00+09:00
draft: false
categories: ["Android", "ガラホ"]
description:  
# original: https://mikuta0407.net/wordpress/index.php/2024/05/06/ky-42c_android_app/
---

ガラホのKY-42Cでapk突っ込んでAndroidアプリを動かそうとしたときの動作可否のメモ

## 傾向

- 過去にSHF31で遊んだ経験から動くと思っていたアプリが動かないことが多い。
- React Nativeっぽいアプリは結構だめ? (My IIJmioやOpenVPN Connectとかがそれっぽいエラーを吐いてる(logcatした))
- SHF31で遊んだとき動いたアプリでもぬるぽ出して死ぬアプリがあるので、結構Androidを削ってる?
- pixivは完璧に動く
  - 面白いのは、Mode1 RETROⅡよりカーソル操作が安定する

## 動いた

- IIJmioクーポンスイッチ
  - 画面小さすぎてレイアウトは厳しい
- AIDA64
- DSub (daneren2005/Subsonic)
  - かなり謎だがカーソルでの操作がかなり不能。ポインタ併用必須。
- DVCアプリ
  - 事あるたびに「GooglePlay開発者サービスがない」と言ってくるがバーコードは出せる
- pixiv
- Solid Explorer

## 動かない

- IIJmio国際電話
- bs-16i
- Bluesky
- My IIJmio
- OpenVPN Connect
- Spotify
- Voicy
  - ログインはできるし、メイン画面も一瞬見えるが落ちる。惜しい。