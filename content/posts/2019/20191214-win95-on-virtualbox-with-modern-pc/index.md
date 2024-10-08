---
title: '今のパソコン上のVirtualBoxにWindows95を入れる'
date: 2019-12-14T00:00:00+09:00
draft: false
categories: ["Windows", "VirtualBox"]
description:  
# original: https://mikuta0407.net/wordpress/index.php/2019/12/14/post-79/
---

(この記事はQiitaからのコピーです。 本記事の投稿日はQiitaでの投稿日としています。)

この記事は[Tokyo City University Advent Calendar 2019](https://adventar.org/calendars/4282)の14日目の記事です。

13日目は[軍事基地に行こう(提案)](https://bob563.hatenablog.com/entry/2019/12/13/000000)でした。自衛隊のイベントとか、｢おーやるんだ｣と思ってもなかなか行かないことが多いのですが、たしかに最近はTwitter等でも情報が回ってくるので、来年は一度くらい行ってみたいですね。

## はじめに
今年はWindows10 1903、1909が出て、WSL2なども出ましたね。一昔前とはぜんぜん違う方向でのWindowsの進化をひしひしと感じます。そして数カ月後にはついにWindows7のサポートが終わってしまいます・・・(そんなWindows7をRyzen 5 3600とRTX2060で使おうとしてる記事はこちら→ https://blog.mikuta0407.net/posts/2019/20190905-windows7-with-ryzen3thgen-and-rtx2000-series/)

そんな年の瀬ですが、ここであえて、"今の"Windowsの原点とも言えるWindows95を触って、懐かしさに浸ってみるのも面白いなと思い(？)、VirtualBoxへのWindows95のインストールにチャレンジしてみました。

思っていたより~~めんどくさかったです~~時代を感じる罠があるので驚きました。

ちなみに~~実際にやらなくても体験できるように~~VirtualBoxをほとんど触ったことのない人でもできるようにスクリーンショット多めです。

## 必要なもの
- VirtualBox(6.xを使用しました)
- Windows95~WindowsMeの起動ディスクイメージ(またはFreeDOS)
- Windows95のインストールイメージ(今回はCDイメージです)

## 1. 400MBくらいのVHDを作る
人によってはこの工程を行いませんが、確実性を考慮し、この手順を行いました。  
作成するVHDには、このあとWindows95のCDイメージをコピーします(詳しいことは後述)  
今回は"ディスクの管理"を使って行いましたが、VHDが作れるなら何でも良いです。(Diskpartでも)

1. Win+X→「ディスクの管理」→「操作」タブ→「VHDの作成」から、VHD作成画面を呼び出す  
![  ](img/193cdd2b-fefb-de46-718f-9ffcd8e6992b.png)
2. 200MBの容量可変VHDを作成する  
CDの内容を書き込むだけなので、400MBで良いです。
ちなみに画像では1000MBで作ってしまっています  
![  ](img/7fad552a-4de4-aef4-7fe0-319de627014a.png)
3. 自動でマウントされたVHDを初期化する  
![  ](img/44661059-a941-ae2f-6c24-0009f4d52a65.png)
4. GPTなんてDOSは読めないので、MBRにします。  
![  ](img/d3fe71ca-1998-92e3-8d4d-029b3e6a057d.png)
5. 普通にFAT32でフォーマットする  
![  ](img/be123e91-5f12-410e-07d6-ed646030c93c.png)  
![  ](img/7c0d985b-5d81-4609-10bd-d7f01dc9edac.png)

これで、フォーマット時に指定したドライブレターに、作成したVHDがマウントされました。

## 2. Windows95のCD内のデータをすべてVHDにコピー
さて、なんでいちいちCDのデータをVHDにコピーするかというと、Windows95の起動ディスクが**CD-ROMを読まない**からです。ドライバがないんですね。  
ちなみにWindows98の起動ディスクであったり、FreeDOSを使う場合にはCDの認識が可能なんですが、どちらにせよCDではなくHDDにデータを入れたほうが成功率があがる(気がする)ので、HDD経由でのインストールをします。データ転送方法ですが、わざわざCDの読めるDOSの起動ディスクを使ってxcopy駆使するくらいならホストマシン側で楽をしましょうということで、

![  ](img/402faea9-3ad2-5fd0-74a3-bb9138ae2e9f.png)

コピーしました。

VHDはこのあと仮想マシンで使うので、切り離しておきましょう。切り離さないとVirtualBoxに怒られます。  
「ディスクの管理」→画面下部から対象ドライブ(青いやつ)を右クリック→「VHDの切断」→「OK」

## 3. VirtualBoxで仮想マシンを作成

![  ](img/9c0f8562-29e0-9e71-3b8a-e3af843edb65.png)

メモリは贅沢に256MB割り当てちゃいましょう。  
ハードディスクは新規作成で8GBくらいで作っておきましょう。(仮想イメージ形式はVHDがおすすめです。ホスト側Windowsでも特別なソフトを使わずに簡単に読み書きできるので。)

## 4. 仮想マシンの設定
作った仮想マシンを起動する前に、設定→ストレージより、IDEプライマリスレーブに先程作ったVHDを割り当てます。  
「既存のディスクを選択」→「追加」→ファイル選択ダイアログから選ぶ→一覧に追加されているので「選択」  
![  ](img/1ce8c3d7-1c1f-efb8-c8b1-8c939cdf7877.png)

そしてHDDと同じ要領でフロッピードライブに、起動ディスクを割り当てます。  
「ディスクを選択」→「追加」→ファイル選択ダイアログから選択→「選択」

最終的にこうなります。  
![  ](img/077ca328-8fa8-b475-b5ed-b9a5c3a904a6.png)

次に、USBを切ります。  
Windows95の頃にはUSBなんて普及してなかったので、インストール時にコケたりしないように切ります。  
設定→USBより、「USBコントローラを有効化」のチェックを外します。  
![  ](img/bea23e9e-fe61-ec8c-e3f1-b9344a056718.png)

これでひとまず設定は終了です。  
仮想マシンを起動しましょう。

## 5. Cドライブをフォーマットする
仮想マシンを起動すると、「Starting Windows 95...」の表示のあと、(日本語版の起動ディスクなので)キーボード配列の選択が出てきました。「半角/全角」キーを押すことでJISと認識させます。(FreeDOS等ではこの画面は出なかったはずです)  
![  ](img/d1b1448d-5d17-a6c1-8936-dc407a2c2cac.png)

選択後はDOSが使えます。ここから、fdiskを利用して、先程OS用に作成したハードディスクのフォーマットを行います。(FreeDOSでも、表記は英語ですが手順はほぼ同じです)

1. FDISKの実行  
fdiskコマンドでFDISKを起動させます。  
![](img/17703321-4591-ed94-22fb-d9060315ace8.png)  
ここではYキー→Enterで、大容量ディスクのサポートをオンにします。  
![](img/b4309b9a-c7de-052f-157b-8239f5aa9a2a.png)  
1番で「MS-DOS領域または論理MS-DOSドライブを作成」を選択します。  
![](img/612258f4-bf77-e248-63f0-3d59991a9329.png)  
1番で「基本MS-DOS領域を作成」を選択します。  
MBRで初期化してくれます。  
![](img/154a69cc-a738-e41c-e8a7-cb147fa73001.png)  
最大サイズを割り当てましょう。  
終了後はEscキーを2回押してFDISKを終了しようとすると、再起動しろとのメッセージが表示されます。  
![](img/976357a8-c85d-0c41-556c-cc90db9ef7c4.png)  
もう一度EscキーでDOSでに戻り、仮想マシンを再起動させます。(仮想マシン→リセット)

2. ドライブをフォーマット
もう一度半角/全角キーでJIS認識させ、プロンプトになったら、今度は  
`format c:`  
を実行します。  
(もしJIS認識されていない場合は、「:」はShift+「;」です。)  
![](img/35cfe3b0-b87d-72f9-a94c-8f0dcf32c687.png)
ファイルなんてなにもないので、y→Enterでフォーマットします。  
![](img/b06f4d7c-a770-8720-aea7-13fac9ad7e63.png)  
ボリュームラベルを決められます。安直にWindows95にしました。  
![](img/5173759d-ca0c-2c17-5fd1-b7539ebd348c.png)  
フォーマットが完了し、使えるようになりました。

## 6. インストーラを起動、インストール開始
先程、CDの内容が入っているHDDをプライマリスレーブに割り当てたので、そのHDDはDドライブになっているはずです。`d:`でドライブを変更しましょう。  
変更後、`SETUP`と打ち込んでインストーラを起動しましょう。

実行するとこうなります  
![  ](img/55446aba-d6af-86d5-36d6-92ce5ef748a6.png)  
チェックをしてくれるらしいのでお願いしましょう。  
![  ](img/c1d120a2-6c51-9e65-4c56-d468962b372d.png)  

ScanDiskが実行されました。Xキーを押してScanDiskを終了させます。  
すると、以下のようにWindows 95のGUIインストーラが起動してきます。  
![](img/57bc8a67-9ce3-d435-f740-06d30cb4078f.png)  
普段のWindowsのインストールのようにライセンスに同意したら、セットアップウィザードです。ここらへんは昔から変わってません。どんどん進めていきましょう。  
![](img/424e9c4c-cf04-0396-7835-2ddb55d3630a.png)

ドライバは全部入れちゃいましょう  
![](img/96ee01e8-3ad1-e988-9ce6-cb488b02920a.png)

起動ディスクの項目は、FreeDOSからインストールしている場合は念の為に作っておきましょう。「はい」を選び、VirtualBoxで「デバイス」→「新規フロッピーディスクの作成」からイメージを作っておきましょう。  
![](img/a95e8c53-df69-19e8-fde1-f8efcf412534.png)

ファイルコピーの進捗度に合わせてスライドが変わるのですが、速くてなんにも読めません。  
![](img/937ef1d5-f648-2a4e-3141-b3e2c5d0504a.png)

ファイルのコピーが終わり、マシンの再起動を求められます。まずフロッピーを抜きましょう。(デバイス→フロッピーディスク→仮想ドライブからディスクを除去)  
![](img/886e9f9c-75ad-2f0d-c4f1-ab4bd74186f2.png)

そうしたら、**OKを押した瞬間からF12を連打** します。(VirtualBoxのブートメニューを表示させる)  
![](img/5041eaef-fa10-fdf0-a91d-904d7e04aad7.png)

表示されたら、**一切ブートせず**、仮想マシンの電源を切ります。(画面を閉じます)

## 7. 高速CPU対応用のパッチを当てる
実はここが今のPCの仮想マシンにWindows95を入れるときの罠です。  
仮想マシンなんだから古いOSが簡単に動いてもいいのですが、Windows95は(今に比べたら)遥かに遅いCPUを前提に作られているOSです。今のPCのCPUは速すぎるのです。それゆえ、このまま再起動してしまうとこうなります。  
![  ](img/3df5e4cb-d0d0-ae08-c676-7cd6512f6a62.png)

これでは何も出来ません。なので、高速CPUに対応させるためのパッチを当てます。(作った人すごいですね)  
http://www.tmeeco.eu/9X4EVER/GOODIES/FIX95CPU_V3_FINAL.ZIP  
↑これをダウンロードすると、中にISOが入っているZIPが手に入りますので、展開してISOを取り出しておきます。

次に、仮想マシンの設定を開き、「ストレージ」→光学ドライブからイメージ割当→先程手に入った「FIX95CPU.ISO」を割り当てます。  
![  ](img/4df22289-d1d4-f27a-8ba9-b39ab151d0d4.png)  
![  ](img/106673d6-6c88-9df1-123c-5eea0df3c7e7.png)

ついでに、「ディスプレイ」→「ビデオメモリ」を贅沢に128MBにしちゃいましょう。(しなくてもいい)  
![  ](img/f2778e9b-66ce-1378-d66b-087869c8a8d2.png)

では、改めて仮想マシンを起動しましょう。

起動すると、こんな真っ赤な画面が出てきます。  
![](img/2bb5f4a1-8a12-d21a-0e33-824d8d4f7b2f.png)  
High-Speed Processor Support、直球ですね。

指示通り、なにかキーを押します。  
すると、README読むか聞かれますが、読まなくても別にいいのでNキーで飛ばします。  
![](img/57899411-86f6-8d7e-59e1-7474fb8c92fc.png)

また何かキーを押すと、パッチが当たります  
![](img/4b8c2f3d-1ecf-8441-e839-0fc68bb24a21.png)

これで高速CPU対応が出来ました。何かキーを押し、「デバイス」→「光学ドライブ」→「仮想マシンからディスクを除去」でCDを抜いて再起動しましょう  
![](img/4e542521-85f6-60d9-6095-47b9b4e42e8e.png)

## 8. セットアップ続行
もし、Startup Menuが出てきたら、Safe ModeではなくNormalを押して通常起動させます。  
![](img/8cdd8d5a-1dec-ca18-75d8-7c6b5a1ce9ae.png)

コンピュータ名などの入力が始まります。  
ひとまずコンピュータ名を入力し、ワークグループも入力(WORKGROUPで良いと思います)したら、左側のタブ「ﾈｯﾄﾜｰｸの設定」に行きます。

このままではTCP/IPが使えないので、「追加」を押し、  
![  ](img/bb3c0a65-bb95-71cc-babe-d92bf9d53bc6.png)

「プロトコル」をクリックしてから「追加」を押します。  
![  ](img/dde164a4-5f82-4bee-185d-969b52a2b767.png)

「Microsoft」から「TCP/IP」を選択し、「OK」で決定します。  
![  ](img/04456770-6e4f-cace-41da-a7660ca9e70c.png)

ネットワーク構成欄にTCP/IPがあることを確認したら、「閉じる」で続行します。  
![  ](img/fefa651f-f728-8478-aec2-5a04fb282268.png)

バージョンの競合とか言ってくることがありますが、コピー元はどうせ一緒なので「現在のファイルをそのまま使う」にしましょう。

すると、今のPCでもたまに見かけるような画面が出てきます。  
![](img/e09ed589-42b7-09b2-a199-7b9ca423302c.png)

このあとはタイムゾーンやプリンタ等の設定ですが、タイムゾーンは別に弄らなくていいので「閉じる」で続行します。  
プリンタも仮想マシンには必要無い(と思っている)ので、「ｷｬﾝｾﾙ」します。

セットアップが完了し、再起動を求められます。再起動しましょう。  
![](img/4466d4c4-e36c-de0f-5284-2dd0a18081eb.png)

ログイン画面が出てきました。ユーザー名は先程入力したものを、パスワードは今考えたものを入力しましょう。  
![](img/31c9d3a0-397b-89c2-21d4-0c2a17f84b1c.png)

![](img/6cb1eef9-f1c9-fe50-303c-24e1193c6bc6.png)  
お疲れ様でした。Windows95がインストールできました。  
最後に、グラフィックドライバを入れましょう。

## 9. グラフィックドライバを当てる
このままだと16色しか使えないので、  
![](img/1e828f04-fe05-dcde-3419-edf5111bd0e7.png)

↓を落としてきます。  
https://web.archive.org/web/20190210203844/http://bearwindows.boot-land.net/140214.zip  
(出典:https://web.archive.org/web/20190210203844/http://bearwindows.boot-land.net/vbe9x.htm)

展開するとこんな感じです。  
![  ](img/0bdfd2e6-9f3b-0776-32f1-4bde030343bb.png)

次に、仮想マシンのメニューから、「デバイス」→「光学ドライブ」→「新規アドホックVISOの作成」に行きます。  
これは、ホストマシンからゲストマシンへ、拡張機能等入れずにも仮想マシンのCDドライブ経由でデータが渡せるｽｯｺﾞｲ便利な機能です。(本当にありがたい)  
上部のファイラーから、先程展開したフォルダを下部にドラッグします。  
![  ](img/06a0233c-c722-5dec-d4c1-06b2d82c65b2.png)

OKを押せば、今下側に入れたファイルが入った(ように見える)仮想ディスクが仮想マシンに挿入されます。  
![  ](img/d128b53d-0076-3574-fa68-0e791846db3a.png)

デスクトップ→右クリック→「画面のﾌﾟﾛﾊﾟﾃｨ」→「ﾃﾞｨｽﾌﾟﾚｲの詳細」→「詳細ﾌﾟﾛﾊﾟﾃｨ」→「変更」でアダプタの変更ができます。  
現在は「ｽﾀﾝﾀﾞｰﾄﾞ PCI ｸﾞﾗﾌｨｯｸｽ ｱﾀﾞﾌﾟﾀ(VGA)」になってるはずです。

右下の「ﾃﾞｨｽｸ使用」から、  
![  ](img/b5ff2111-378c-0ec3-2f36-d91f5688d451.png)

「参照」をクリックし、ドライブ欄から先程のディスクを選択、`140214\128mb`を選択し「OK」を押します。  
![  ](img/b4ce2705-31f1-b6e8-95a5-27bcdeb612c7.png)  
もう一度OKを押すと、このようなダイアログが出てくるので、また「OK」を押します。  
![  ](img/f9914a93-65bf-9687-1239-89eefc66ce1a.png)

ディスプレイアダプタが変更されました。  
![  ](img/16fdac73-7a5c-a9d4-cf6c-44ef935cfa78.png)

このままOKをするとこんなのが出てきますが、閉じて、  
![](img/4f224e79-7dbb-a73f-29cf-b81794729600.png)

閉じましょう  
![  ](img/9da539a8-7bd1-7672-78f7-d3e75c44edd3.png)

このままだとRundll32がエラー吐いたせいで何も出来ないのでスタートメニューから再起動します。  
![  ](img/b8e9bd9a-53df-4c95-ddff-84255945e541.png)

再起動したのに勝手に電源が落ちた場合は、手動で起動し、セーフモードではなくNormalで起動します。

再起動すると、色がたくさん選べるように。どうせなので「~~TRUE COLORS~~ True Color (32ﾋﾞｯﾄ)」を選択しましょう。  
![](img/b3180091-e21e-bd9a-e201-fafa936fe397.png)

右下の「詳細ﾌﾟﾛﾊﾟﾃｨ」から、「ﾓﾆﾀｰ」→「変更」に行きます。  
![  ](img/a53f8fbf-d7d8-9f91-cb13-b56de13fc0cb.png)  
「Super VGA」の中から好きな解像度を選びましょう。  
ダイアログを閉じるとリフレッシュレートの云々がですが、指示に従いましょう。

解像度が800x600や、それ以外にも自分で今選んだモニタの解像度までの解像度がいくつか選択できるようになっています。  
![  ](img/52903e41-6a62-1254-5e94-90b177451a5a.png)

好きな解像度にして、「OK」を押して再起動すれば・・・  
![  ](img/00aee441-a3cd-dda2-d7af-0780576f1ee4.png)  
きれいなフルカラーの表示、1024x768の広大な画面領域が手に入ります!

あとはFirefox1.0を入れてブラウジングするなりして、Windows95を楽しみましょう!!

## あとがき
さて、この記事需要あるんでしょうか・・・  
まぁ作業記録でもありますので、価値は0じゃないと信じています。  
ちなみに25日にらぴーとさんによる、VMwareにWindows98を入れてみる回もあるそうなので、そちらもぜひ。

### 宣伝
4日目に[C言語開発環境を構築しよう! ～超遠回り編～](https://blog.mikuta0407.net/posts/2019/20191204-c-dev-env-for-msdos/)
10日目に[個人シンクライアント運用のすゝめ](https://blog.mikuta0407.net/posts/2019/20191210-recommendations-for-thin-client/)
も書いてます。そちらもぜひ
次の記事: @ke9000による[スマホに音楽データ入れるとストレージ食うけど家にあるCDを持ち運びたくない？解決法、あります！]() です!お楽しみに!(僕はGooglePlayMusicのロッカー機能派)

