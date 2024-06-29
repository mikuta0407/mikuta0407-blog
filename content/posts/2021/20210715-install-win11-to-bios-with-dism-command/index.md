---
title: 'Windows11 Insider PreviewをBIOS機に入れてみる(Dism展開)'
date: 2021-07-15T19:55:00+09:00
draft: false
categories: ["Windows"]
description:  
# original: https://mikuta0407.net/wordpress/index.php/2021/07/15/install_win11ip_to_bios/
---

Windows 11はUEFI機にしか正常な手段だとUEFI機にしか入りません。

正当じゃない手段だと入れる方法はあります。  
例えばWindows10→Windows11のアップグレードだとレジストリ編集で突破できるようです。

ただ、クリーンインストールでいい場合はそんなめんどくさいことしなくていいです。

ということで、今回はWindows11 Insider PreviewをBIOS機(LegacyBoot機)にクリーンインストールでインストールしてみた記録を記載していきます。

**注1: 当たり前ですが、推奨するものでもないですし、正規リリース後にWindowsUpdateが降ってくる保証もありません(Win7をRyzen機に入れたときみたいなサポート対象外扱いになる可能性がある)。  
自己責任でどうぞ。あくまでも「やってみた」の記事です**

**注2: あくまでもWindows11 Insider Preview(初期リリース)での実験です。記事を書いているのは2021/07/15です**

**注3: この記事の方法は、パーティションを分けずに1つのパーティションにインストールする方法を記載しています。ブートパーティションとシステムパーティションを分けたいなどの場合は、適宜それらしいことをしてください。**

## 必要な?環境
- TPMもUEFIもない環境
	- 記事の写真ではVirtualBoxで試していますが、Windows11のインストーラはEFI有効化してないVirtualBoxの仮想マシンを通すので、VirtualBoxだけで使う場合は必要ない記事です。
	- 同様の手順で、TPM1.2+UEFIオフ+MBRのマシンにインストールできたのは事実です(CF-RZ4)
- Windows11 Insider PreviewのISO
	- UUPから作成します。(これは正規の方法です。)
	- 参考: https://miyagadget.tokyo/archives/6
	- 今回の記事では22000.51.210617-2050の日本語イメージを使っています。

## Diskpartでディスクのフォーマット

**ディスクを初期化します**

インストーラのISO/USBメモリ/DVD等から起動して、コマンドプロンプトを出します。  
よくあるやつなので出し方は割愛します。

![](https://i.imgur.com/oE7LbOg.png)

DiskpartでNTFSパーティションを作ります。  
list diskのところでGPTがオフなのが確認できます。  
NTFSでプライマリパーティションを作り、Cドライブとして割り当てました。  
![](https://i.imgur.com/k61Tyzf.png)

ここで、今作ったパーティションをアクティブにします。  
![](https://i.imgur.com/XeTooYv.png)

Diskpartを終わります。

## Dism展開する

今回はDismで直接install.wimを展開する手法を使い、要件検出をスキップします。  
まずはwimの中身のindexを確認します。  
今の状態では、インストールメディアはDドライブにマウントされています。

![](https://i.imgur.com/GRA4rKu.png)

```cmd
dism /get-wiminfo /wimfile:D:\sources\install.wim
```

を実行して、Windows 11 Proのindexが1であることを確認します(複数エディション対応とかだと増えるので)

![](https://i.imgur.com/Or3zu3l.png)

次に、Cドライブに展開します。

```cmd
dism /apply-image /imagefile:D:\sources\install.wim /index:1 /applydir:C:\
```

展開が終わるまで待ちます

![](https://i.imgur.com/7UMvizV.png)

最後に、

```cmd
bcdboot c:\windows /l ja-JP /s c:
```

を実行します
![](https://i.imgur.com/EBeJoHl.png)

## 内蔵ストレージから起動する

コマンドプロンプトを閉じて、電源を切り、インストールメディアを抜きます

![](https://i.imgur.com/nW5FxZV.png)

普通にセットアップが始まるので、やっていきます

![](https://i.imgur.com/XNEdHTG.png)

![](https://i.imgur.com/0oKbErq.png)

## 完成

VirtualBoxだと元々要件検出を突破してしまうので画像に説得力がないですが、以上の手順で、**一応、現時点では、** Windows11をLegacyBoot機にインストールすることができました。

![](https://i.imgur.com/WP6BJ3v.png)