---
title: '本格雑談くちをひらくの更新通知Botを全面的に作り直していた話'
date: 2025-09-07T00:00:00+09:00
draft: false
categories: ["盆栽システム開発", "Go", "Twitter", "本格雑談くちをひらく"]
description: 以前作った本格雑談くちをひらくの更新通知Botをリファクタリングしていたので、なにをしたのかをまとめた記事です。
image: "img/eyecatch.png"
# original: 
---

2023年に [リスナーが勝手にPodcast番組の更新通知Botを作って遊んだ話](https://blog.mikuta0407.net/posts/2023/20230807-making-of-x-bot-for-kuchiwohiraku/) という記事を投稿しました。

記事の内容をざっくり3行で説明します。

- Podcast番組「本格雑談くちをひらく」の公式アカウントからの更新通知が途絶えた。
- 原因はIFTTTとXの連携有料化
- リスナーが勝手にBotを自作して動かし始めたら公式的に黙認された

このBotは2023/06/26の回から正式稼働を始めました。 <blockquote class="twitter-tweet"><p lang="ja" dir="ltr">[2023/06/26 17:00 up!] 『公式黙認宣言。[中村繪里子・吉田尚記の本格雑談くちをひらく]』 podcast→<a href="https://t.co/TP5MAG1NbF">https://t.co/TP5MAG1NbF</a> Web→<a href="https://t.co/vvzhK2UK72">https://t.co/vvzhK2UK72</a> <a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a> <a href="https://t.co/XR3seEd0p7">https://t.co/XR3seEd0p7</a> <a href="https://twitter.com/yoshidahisanori?ref_src=twsrc%5Etfw">@yoshidahisanori</a> <a href="https://twitter.com/eriko_co_log?ref_src=twsrc%5Etfw">@eriko_co_log</a></p>&mdash; 本格雑談くちをひらく 更新通知【公黙認】 (@kuchihira_bot) <a href="https://twitter.com/kuchihira_bot/status/1673254488294178817?ref_src=twsrc%5Etfw">June 26, 2023</a></blockquote>  

それから1年が経過し、先日2024/06/26の回で1周年についてのメールを取り上げていただきました。

<iframe src="https://omny.fm/shows/kuchiwohiraku/45112a2a-27b3-4422-bf40-b19800f29e93/embed?style=Artwork" width="100%" height="180" allow="autoplay; clipboard-write" frameborder="0" title="ダメなディレクターの介護で進化。[中村繪里子・吉田尚記の本格雑談くちをひらく]"></iframe>

番組内のメールでも読まれましたが、実はリリースからの1年の間で、Botの中身をすべて作り直しています。

今回は、稼働から1年の間の変化と、現状のプログラムの仕様についてまとめておきます。

(本当はこの記事は2024/07に出す予定でした。気づいたら1年経過していました。)

ちなみに、僕の所属するサークルが出した [SWSD REPORT VOL.3.5](https://swsd.booth.pm/items/6444529) でも取り上げています。ご興味があればぜひ。

## フォーマット変更

リリース直後は、元々使われていたフォーマットを踏襲していました。

「
[YYYY/mm/dd HH:MM up!] 『タイトル』 podcast→<AppleのPodcastのリンク> Web→<omny.fmの全体リストページのリンク> #くちをひらく <omny.fmの当該回へのリンク> @yoshidahisanori @eriko_co_log
」

ただ、リスナー視点としては流石に1行でまとまっていて見づらかったこともあり、変更を考えます。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">そういえば告知ツイートBotのフォーマット、とりあえず以前のものをそのまま使っていましたが、ちょっと見やすくしてみようと思います。<br>明日からはこんな感じにしてみようかなぁと。(Voicyの(番組の)リンクを掲載。)<br><br> <a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a> <a href="https://t.co/bw0EINLNUi">pic.twitter.com/bw0EINLNUi</a></p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1676872695198412800?ref_src=twsrc%5Etfw">July 6, 2023</a></blockquote> 

すると、めちゃくちゃありがたいことに、吉田さんから受け入れていただけます。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ありがとうー！！！めっちゃ助かります！ <a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a> <a href="https://t.co/3ADeo1Z6ti">https://t.co/3ADeo1Z6ti</a></p>&mdash; よっぴー/吉田尚記 (@yoshidahisanori) <a href="https://twitter.com/yoshidahisanori/status/1676885553319641089?ref_src=twsrc%5Etfw">July 6, 2023</a></blockquote> 

フォーマットについてはこれを導入し、いまに至っています。改行を入れ、AppleのPodcastのリンクを消してVoicyへのリンクを追加、といったところですね。

## アイコンを描いていただいてしまった

導入当初、公式アイコンが緑っぽいということで、真緑のアイコン(mspaintで15秒位で作った)にしていましたが、番組のふつおた枠に「なにか寂しい気がするのでもうちょっと華やかにしてみたい。なにか良い案等あれば」といったメールを送ったところ、

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">『お絵かきコーナー！[中村繪里子・吉田尚記の本格雑談くちをひらく]』 <a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a><br><br>Web: <a href="https://t.co/HAcuUwZSyh">https://t.co/HAcuUwZSyh</a><br>Voicy: <a href="https://t.co/9nC07beZxn">https://t.co/9nC07beZxn</a><a href="https://twitter.com/yoshidahisanori?ref_src=twsrc%5Etfw">@yoshidahisanori</a> <a href="https://twitter.com/eriko_co_log?ref_src=twsrc%5Etfw">@eriko_co_log</a> <a href="https://twitter.com/kuchiwohiraku?ref_src=twsrc%5Etfw">@kuchiwohiraku</a><a href="https://t.co/hfuvllSWG6">https://t.co/hfuvllSWG6</a></p>&mdash; 本格雑談くちをひらく 更新通知【公黙認】 (@kuchihira_bot) <a href="https://twitter.com/kuchihira_bot/status/1692446709593629123?ref_src=twsrc%5Etfw">August 18, 2023</a></blockquote>

<blockquote class="twitter-tweet"><p lang="qme" dir="ltr"><a href="https://twitter.com/hashtag/%E6%96%B0%E3%81%97%E3%81%84%E3%83%97%E3%83%AD%E3%83%95%E3%82%A3%E3%83%BC%E3%83%AB%E7%94%BB%E5%83%8F?src=hash&amp;ref_src=twsrc%5Etfw">#新しいプロフィール画像</a> <a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a> <a href="https://t.co/zMMWJGFEnf">pic.twitter.com/zMMWJGFEnf</a></p>&mdash; 本格雑談くちをひらく 更新通知【公黙認】 (@kuchihira_bot) <a href="https://twitter.com/kuchihira_bot/status/1693564153544487185?ref_src=twsrc%5Etfw">August 21, 2023</a></blockquote>

なんと吉田尚記さんと中村繪里子さんに手書きでアイコンをつくっていただいてしまいました。非公式で、公式に黙認していただいている立場にも関わらずありがたすぎるお話です……。大切に使わせていただいています。


## 更新漏れとの戦い

くちをひらくでは、この1年で数回、想定外の更新が数度発生しました。

### 1日2回の更新

例えば更新設定漏れによる一括投稿では、2023/7/28に、2023/7/26日分と2023/07/27分が同時に更新されたときの話です。(注: ディレクターの石川さんを責めている意図はまったくありません。)

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">これは例の更新通知Botの裏話ですが、7/27分は手動でスクリプト実行、26日分は手でツイート作りました(この方が早い)</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1684583506192769024?ref_src=twsrc%5Etfw">July 27, 2023</a></blockquote> 

このときの更新は7/28の0時頃に2つ更新されました。このとき、当時のBotは「スクリプトが実行されたタイミングでRSSから取得できたアイテムの**最新のものがその日の日付の更新だった場合に**、それをツイートする、という動作になので、0時に2つ投稿されたあとにスクリプトだけを実行してしまうと、7/27予定分のみがツイートされてしまいます。

そのため、7/26分を手でツイートし、7/27分をスクリプトで投稿する、といった運用を行いました。(手でツイート、普通にiPhoneでコピペで作っていました。(たまたまPCが手元になかった))

### マチ★アソビ起因の突発更新

2023年の秋のマチ★アソビでは、吉田さんと中村さんが徳島の居酒屋で録音された突発的な更新がありました。Botは17時台以外の更新を想定していなかったので、Voicyアプリに来た更新通知で更新を認識し、手動でスクリプトの実行をしていました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr"><a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a><br>(Voicy通知に気づいて手動でスクリプト叩いた)</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1718300802224411029?ref_src=twsrc%5Etfw">October 28, 2023</a></blockquote> 

この後も2回ほどあった記憶がありますが、適宜叩いていた気がします。

これが結構悔しいのです。追いつけず人間が出てきて作業した、ということにとても悔しさを感じました。ただ、回数は多くない事象でもあるので、このときは「発生したら発生したで考えよう」という気持ちで一旦改修を見送ります。

## BlueskyへのBot

### 発端
2024年2月、ある意味ではTwitter対抗となるBlueskyが招待制を廃止し、一般開放が行われました。

一時期は僕のTLでも大きくBluesky移行の流行があったのを覚えています。結果としてそこまでの大移動はありませんでしたが、少なくともThreads向けにBotを作るよりは明らかに価値があるのではないか? (本音: ただ作ってみたい)、ということで、Blueskyへの完全非公式なくちをひらくBotの作成を考え始めます。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">(ところで何も関係ないけどBlueskyに完全非公式Botを作るかちょっと悩んでいる) <a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a></p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1755888329214894187?ref_src=twsrc%5Etfw">February 9, 2024</a></blockquote> 

考えているだけでは良くないので、Blueskyへの投稿機能をBotへ実装し始めます。

### ついでにGoへ移行する

Blueskyへの投稿をPHPで行えないか模索してみましたが、ライブラリがあるにはあるも使いづらい、自分で作るにはちょっとめんどくさい、という障壁が立ちはだかります。上手いこと手軽にBlueskyへの投稿ができるものはないか、と模索していると、

[mattn/bsky](https://github.com/mattn/bsky) を発見します。mattnさんのGo製Blueskyクライアントですね。

当初はこれをPHP側から僕の盆栽システム開発伝統の`shell_exec`で叩こうと考えましたが、

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">深夜テンションでくちをひらくBotを書き直している</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1756348324112982440?ref_src=twsrc%5Etfw">February 10, 2024</a></blockquote> 

深夜テンションになり、Goで全部書き直し始めてしまいます。

2022年以降、業務でGolangに触れるようになり、ある程度GoでCLIプログラムがかけるようになっていたこともあり、せっかくならその知見を活かしたいという気持ちもありました。

ただ、後述の通り、Bluesky対応やGoでのリファクタリングをした結果、正直コード量がかなり過剰になってしまいました。流石にどうしようもないのですが、今でもちょっと気になっているところではあります。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">くちをひらくBotをGoでリファクタリングしてるけど、<br>どう考えても元のPHP版のほうがシンプルだし行数少ないのでどうしたもんかなぁってなってる<br><br>まぁ趣味だからやりたいようにやるんだけど、過剰な気はしている</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1756666673338417610?ref_src=twsrc%5Etfw">February 11, 2024</a></blockquote> 

まずGoからTwitter API v2使っての投稿のテストからしていましたが、当時のテストポストは消してしまっていたようです。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">Botテスト投稿してはツイ消しを繰り返している(残す意味が本当に無いので)</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1756703690860478572?ref_src=twsrc%5Etfw">February 11, 2024</a></blockquote> 

その後クソみたいなコミットログを積みながら、

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">進化した <a href="https://t.co/KK8jOTrPa0">pic.twitter.com/KK8jOTrPa0</a></p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1756716676790517993?ref_src=twsrc%5Etfw">February 11, 2024</a></blockquote> 

二晩でリファクタリングしたGo製のBotの本番投入を決定します。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">くちをひらくの公黙認Bot、ちょっと今日の更新分でリファクタリング版の動作確認してみます・・・</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1756708619675161030?ref_src=twsrc%5Etfw">February 11, 2024</a></blockquote> 

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">一応動作確認っぽいことはしてるけどいきなり本番投入なのドキドキで面白いな<br>まぁ内容としてはもう次は本番投入するしかわからないことなんだけど・・・</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1756718376347676806?ref_src=twsrc%5Etfw">February 11, 2024</a></blockquote> 

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">よしcronを新プログラムの方に切り替えた。寝るか。</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1756718937755271366?ref_src=twsrc%5Etfw">February 11, 2024</a></blockquote> 

本番だけど完全公式じゃないからできる技ではありますね。

結果としてTwitterへの投稿とBlueskyがうまく行きました。これはほんとに安心した記憶があります。いくら自分のアカウントでテストしていたとはいえ、本番動作では吉田さんと中村さんにメンションが飛びます。一応は間違った内容を出してはいけないので、かなりの緊張がありましたが、無事にPHP→Goのリファクタリングに成功しました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr"><a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a> のBotをGoでリファクタリングして、今日無事に動いたのでリポジトリ公開しておきます<a href="https://t.co/I4NZ4cRFDu">https://t.co/I4NZ4cRFDu</a></p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1756958545558536588?ref_src=twsrc%5Etfw">February 12, 2024</a></blockquote> 

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">Blueskyの <a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a> の更新通知&quot;非公式&quot;アカウント作ってみました。<br>Bluesky上のBotプログラム作ってみたかっただけです。<br><br>【非公式】本格雑談くちをひらく 更新通知 <a href="https://t.co/EpEM6eSw1K">https://t.co/EpEM6eSw1K</a></p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1758274362095075539?ref_src=twsrc%5Etfw">February 15, 2024</a></blockquote> 

このPHP→GoのリファクタリングとBlueskyへの投稿機能だけを付けたものは、 [Release v1.0.0 · mikuta0407/kuchihira-bot](https://github.com/mikuta0407/kuchihira-bot/releases/tag/v1.0.0) として公開しています。

## 更新検知ロジックの改修 その1

1日に複数回更新されることを考えると、既存の「最新のもののみを見る」というフローでは失敗します。

そこで、前回投稿したものを記録しておき、差分を確認して投稿するように変更することを決意します。
(厳密に言うと、元々ローカルに状態を持ちたくないという気持ちがあり実装していなかったという経緯があるにはありますが、流石にここまでくると無理なことに気づきました。)

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">んー<br>くちをひらくBot、複数同時更新時(意図的な発生も含む)の追従ちゃんとやってみたいな。RSSわからん</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1760588465002156408?ref_src=twsrc%5Etfw">February 22, 2024</a></blockquote> 

改修方針としては、
- 前回投稿時のUUIDをファイルシステムに記録する
- RSSを取得し、最後のUUIDより後に存在しているものを取得する
- 存在していた場合はそれらを投稿する

というものになります。

実装ロジックは割とすぐ思いついたので、2時間くらいで改修ができました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">くちをひらくBot、ファイルシステム側に状態を持ちたくなかったんだけど、複数投稿を検知する、という要件を満たすには最後のGUIDを保持するしかないよなぁとなったので、日付ベースではなくGUIDベースで取得処理をするように変更した。<br>でもまぁこれのほうが正しい気がしてきたな</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1760619582258315735?ref_src=twsrc%5Etfw">February 22, 2024</a></blockquote> 

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">dry runでは問題なく動いてるから多分大丈夫なんだけど、とりあえず月曜日にちゃんと投稿できてればmainにmergeしようかな</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1760620269566349646?ref_src=twsrc%5Etfw">February 22, 2024</a></blockquote> 

ところが改修はしたものの、記録ファイルの読み込み周りの設定を盛大に間違えた結果、間違ったものが投稿されてしまいました。まぁ開発にミスは付きもの……。ごめんなさい。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">(昨日Bot改修したら思ってたのと違う動きして昨日の分だけが投稿されました)<a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a></p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1760948823931912218?ref_src=twsrc%5Etfw">February 23, 2024</a></blockquote> 

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">普通にコード間違えてファイル読み込みに失敗してた。そりゃ16:50に昨日のが出るわ</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1760986258304860177?ref_src=twsrc%5Etfw">February 23, 2024</a></blockquote> 

この後なんとか修正して、検知ロジックを日付ベースではなく差分ベースでみるように完全に変更できました。

ただ、この時点ではBotは17時にcronによって起動する従来の手法をそのまま採用しています。そのため、17時更新以外にはリアルタイム追従ができません(例えば内部的なタイムアウトである2時間を過ぎた後は更新されても検知できない。翌日17時にまとめて検知される)。

## 更新検知ロジックの改修 その2

ここまできたら、マチ★アソビのときのような例外的な更新や、17時から2時間経過するまで以外でも更新されたものをリアルタイムで拾い、元のIFTTT経由での投稿に近い利便性を取り戻したくなります。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">くちをひらくBotの実装について改めて考えています</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1785226282298753534?ref_src=twsrc%5Etfw">April 30, 2024</a></blockquote> 

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">やはりcronではなくサービスとしてやるべきか?</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1785226854078808245?ref_src=twsrc%5Etfw">April 30, 2024</a></blockquote> 

そう、cronで17時に起動させるのではなく、常にデーモンとして動かすように変えるのです。

結果としてまた勢いで実装しました。後ほど説明はしますが、20秒ごとにRSSを掘りに行って差分をチェックし続けるような動作です。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">くちをひらくbot、17時以外の更新もリアルタイムに追従できるようにしました。</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1785611873015964144?ref_src=twsrc%5Etfw">May 1, 2024</a></blockquote> 

10日ほど運用し、無事に動いたので(不具合出さずに動いて本当によかった)、mergeして`v1.2.0`としてリリースしました。  
変更の際、従来の単発実行用の処理も残したので、若干コードが汚くなったのが心残りではありますが、単発実行機能を消す気はないのでいつか考えます。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">mikuta0407/kuchihira-bot: くちをひらくBot (For Twitter + Bluesky) <a href="https://t.co/I4NZ4cRFDu">https://t.co/I4NZ4cRFDu</a><br><br>17時じゃない更新にも追従できるようにデーモンモードを追加して、ここ最近ちゃんと動いてるのでmainにmergeした。v1.2.0としてリリース。</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1788766912798990423?ref_src=twsrc%5Etfw">May 10, 2024</a></blockquote> 

ちなみに、これは詳しい検証をしていないので体感ですが、RSSを取得し差分を確認といった処理が明らかにPHPで記載していたときよりも高速化しています。ここは言語特性の差だとは思いますが、速度差を実感できたのは面白かったです。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">くちをひらくBot、PHP→GoにしたらRSSのXMLの処理周りが結構速くなってびっくりしたんだよな…<br>そもそも速度求めてないプログラムだけども。</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1803094266492879282?ref_src=twsrc%5Etfw">June 18, 2024</a></blockquote> 

## `v1.2.0`以降のデーモン動作に関しての説明

デーモンモードを追加した現在のソースコード自体は [mikuta0407/kuchihira-bot v1.2.1](https://github.com/mikuta0407/kuchihira-bot/releases/tag/v1.2.1) に置いてあります。 (v1.2.0とv1.2.1の違いはREADME.mdのため、動作に関してはv1.2.0と同一です。)

ここからは現状のGo製のkuchihira-botのデーモンモード動作について説明します。  
単発実行の挙動や、ここでは本質ではないBluesky周りについては割愛します。

### パッケージの役割

- cmd
  - cobraでサブコマンドを実装するための部分。サブコマンドは以下
    - `post`: 単発実行用(説明割愛)
    - `login`: BlueSkyログイン
    - `daemon`: デーモンモード実行
- internal
  - core    
    - メインロジックの制御
  - config
    - jsonで保存する各種設定情報の読み取り
  - rss
    - RSSの取得をしてパースを行う
  - twitter
    - Twitterへの投稿
  - bsky
    - BlueSkyのログイン(トークン取得)、投稿、URLカード展開
  - discord
    - 管理人がDiscordで処理結果通知を得るためのWebhook送信

### パッケージの依存関係

パッケージの依存関係は以下のようになっています。

- main
  - cmd
    - internal/bsky (BlueSkyログイン用の依存)
      - internal/config
    - internal/core
      - internal/config
      - internal/discord
        - internal/config (構造体のみの利用)
      - internal/rss
        - internal/config (構造体のみの利用)
      - internal/bsky
        - internal/config (構造体のみの利用)
      - internal/twitter
        - internal/config (構造体のみの利用)

### デーモン実行時の動作概要

- systemctlによって`kuchihira-bot daemon`が叩かれ起動
- `core.init()`によってRSS取得先、連携先に必要なトークン等の情報、使用URL等の読み込みが行われる
- `core.DaemonStart`が呼ばれ、常駐スタート
- 以下の処理が20秒のsleepを挟みながら繰り返し行われ続ける
  - 前回投稿したGUIDを取得
  - RSSの取得先URLから全アイテムを読み出す
  - 前回投稿時のGUIDと、最新アイテムのGUIDが異なる場合、最新アイテムから順番に確認して、前回投稿のGUIDと一致するまで差分アイテムを取得
  - 差分として出てきたアイテムに対してシーケンシャルにそれぞれTwitterとBlueskyへ投稿処理
  - 最後に投稿したアイテムのGUIDをファイルシステムに記録

Omny.fmに対してずっとアクセスし続けてることになりますが、こればかりは許してもらいたいところです。

## 余談

実は元々公式アカウントが使っていたIFTTTでの告知は、Omny.fmのRSSの更新から20〜30分ほど遅延しての投稿が行われていました(VoicyやSpotifyアプリの通知のほうが早かった)。これは単純にIFTTTのRSSをポーリングする時間の頻度が細かくないことによるものと思われます。PHP時代からではありますが、確実に数十秒間隔でRSSのポーリングを行うため、RSSが更新された直後(最大遅延は現状20秒)に投稿ができるようになり、毎日17:01~17:03の間にはTwitterへ更新通知が飛ばせるようになっています。IFTTTのRSSポーリングと比べてはいけないのはわかった上ですが、ちょっとした自慢ではあります。

## 〆

突貫で作ったBotがなんだかんだ安定運用していて、安心して過ごしています。公式黙認という立場ですが、今後も安定運用目指していきます。趣味の中で起きる本番運用というものは基本的に自分か友人等の身内のみで終わるものが多く、比較的広い範囲に影響する運用というものは出会うことが出来ません。運用と言っても基本は放置しているだけですが、やりがいは大きいです。楽しくBot運営をやっていますので、もしこの記事を読んで興味を持ってくださった方は、「本格雑談くちをひらく」、ぜひ聴いてみてください!

<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
