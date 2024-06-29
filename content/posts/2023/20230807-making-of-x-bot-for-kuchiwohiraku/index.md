---
title: 'リスナーが勝手にPodcast番組の更新通知Botを作って遊んだ話'
date: 2023-08-07T18:58:00+09:00
draft: false
categories: ["盆栽システム開発", "PHP", "Twitter", "本格雑談くちをひらく"]
description:  
image: "img/スクリーンショット-2023-08-07-18.58.06.png"
# original: https://mikuta0407.net/wordpress/index.php/2023/08/07/making-of-x-bot-for-kuchiwohiraku/
---

{{< alert warning >}}本記事の内容は2023年8月当時の実装です。現在はGoを使って完全にリファクタリングされ、更新検知の仕組みも変更されています。いつか記事を書きます。{{< /alert >}}

{{< alert info >}}現在のBotのソースコードは [mikuta0407/kuchihira-bot](https://github.com/mikuta0407/kuchihira-bot)で公開しています。

[![mikuta0407/kuchihira-bot](https://gh-card.dev/repos/mikuta0407/kuchihira-bot.svg)](https://github.com/mikuta0407/kuchihira-bot) {{< /alert >}}

2023/05/31から、「本格雑談くちをひらく」というPodcast番組の更新通知Botを自アカウントで、2025/06/26からは [（@kuchihira_bot）さん / X](https://x.com/kuchihira_bot)で、公黙認(公式的に黙認)という立場で運営させていただいています。その開発経緯等について、なんとなくまとめておこうかと。

## TwitterのAPI周りの改悪と個人的な影響

ツイッt……XがAPI周りを大幅に変更し、v1.1系を潰すことで便利な連携ツールを一層したり、極端に有料化して各所が悲鳴を上げてるのは記憶に新しいかと思います。  
Togetter等の広告収入がある媒体ではAPIの有料化に対応出来ましたが(Togetterは元々エンタープライズ契約だったそうですが)、サブスクリプション形式でユーザーからの直接的な収益によって運営していたIFTTTは、無料プランでのツイッt……X連携を辞めてしまいました。

[「IFTTT」無料プランのTwitter連携が終了へ、アプレット数にも制限　ほか - ダイジェストニュース - 窓の杜](https://forest.watch.impress.co.jp/docs/digest/1501656.html)  
[Updates to IFTTT free tier - IFTTT](https://ifttt.com/explore/updates-to-free-tier-2023)

これにより影響を受けた方々はかなりいらっしゃるかと思いますが、僕の場合は「普段聴いているPodcast番組 [中村繪里子・吉田尚記の本格雑談くちをひらくのエピソード - Omny.fm](https://omny.fm/shows/kuchiwohiraku) の公式アカウントで行われている更新告知ツイートが止まる」という影響がありました。

上記IFTTTの告知を見ると5/23以降無料プランでは終了、となっていますが、[@kuchiwohiraku](https://twitter.com/kuchiwohiraku) で最後に投稿された更新通知ツイートが、

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">[ May 23, 2023 at 05:00PMup!] 『アナウンス技術論と人間の強み。[中村繪里子・吉田尚記の本格雑談くちをひらく] 』 podcast→<a href="https://t.co/SwAoG0LX9p">https://t.co/SwAoG0LX9p</a> Web→<a href="https://t.co/Gke1033vNc">https://t.co/Gke1033vNc</a> <a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a> <a href="https://t.co/fvjFwVVD9y">https://t.co/fvjFwVVD9y</a> <a href="https://twitter.com/yoshidahisanori?ref_src=twsrc%5Etfw">@yoshidahisanori</a> <a href="https://twitter.com/eriko_co_log?ref_src=twsrc%5Etfw">@eriko_co_log</a></p>&mdash; 本格雑談くちをひらく (@kuchiwohiraku) <a href="https://twitter.com/kuchiwohiraku/status/1660924535846076416?ref_src=twsrc%5Etfw">May 23, 2023</a></blockquote>

これです。見事に5/23の分を最後に止まっています。

## 無いなら作ればいいじゃない

僕は普段、このアカウントからのツイートも見つつ、Spotifyアプリの新着通知を見ていたので、ツイートが止まっていることに気がついていなかったのですが、出演者の一人である中村繪里子さんが

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">本格雑談〜くちをひらく〜<a href="https://twitter.com/kuchiwohiraku?ref_src=twsrc%5Etfw">@kuchiwohiraku</a> の更新したよツイートが公式からできなくなってるうううう！！  
  
月〜金 毎日配信してるのよー！ <a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a></p>&mdash; 中村繪里子@中☆吉の中のほう＆#ぶればんのパン担当 (@eriko_co_log) <a href="https://twitter.com/eriko_co_log/status/1663520183254933505?ref_src=twsrc%5Etfw">May 30, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

とツイートされており、「そう言えばツイートが無いぞ」と気づくことになります。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">アプリの通知があったから気づいてなかったけど本当にTwitterとまってる・・・  
 <a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a>  
  
5/23が最後ってのは、見事にIFTTTの無料プランでのTwitter連携が終わったやつだなぁ・・・<a href="https://t.co/evTkU7R0mS">https://t.co/evTkU7R0mS</a></p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1663570182458191878?ref_src=twsrc%5Etfw">May 30, 2023</a></blockquote>

気づいたのは0時を過ぎた頃。まだ眠くなってなかったので、盆栽プログラミング魂が騒ぎ出します。深夜テンションかもしれません。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">くちをひらくのアカウントの投稿は1日1回=最大でも31回/月なので、Twitter APIのFreeでも余裕で賄える  
・・・とすると、何らかの手段でBot作ってしまうのが手っ取り早いのかもしれない。</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1663570721258479616?ref_src=twsrc%5Etfw">May 30, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">IFTTTはあれかな、omny. fmのRSSフィードを使って更新されたらっていうアプレットだったのかな</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1663571851451449345?ref_src=twsrc%5Etfw">May 30, 2023</a></blockquote>

普段聴いてるPodcastの告知が無い、というのは、いろんな意味で危機です。リスナーとしては不安です。  
ならば、「**無いなら作ればいいじゃない**」の精神ということで、勝手に作ってみることにしました。

(本当はここで、アカウントを生やしてやっていこうとしたのですが、アカウント作成がどうやってもエラーで進めない、という現象に陥ったので、最初の1ヶ月(公黙認されるまで)は一旦自分のアカウントでBotを追加する方針にしました)

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">突破できたと思ったらこれだよ <a href="https://t.co/CmZcUmpmQa">pic.twitter.com/CmZcUmpmQa</a></p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1663583576523444224?ref_src=twsrc%5Etfw">May 30, 2023</a></blockquote> 

## 突貫プログラミングの開始

普段、僕は盆栽プログラミングをするときはPHPをシェルスクリプトのwrapper的な使い方をしながら使っています。ありがとうshell_exec。あんまりよくない。  
それ以外の言語だとGoは扱えますが、omny.fmから提供されるRSSなので、突貫でXMLを扱おうとすると、PHPが手持ちの武器では最適です。

どうにかPHPでTwitterのBotが出来ないか、とググったところ、  
[abraham/twitteroauth: The most popular PHP library for use with the Twitter OAuth REST API.](https://github.com/abraham/twitteroauth)  
という、v2のAPIに対応した素晴らしいライブラリがありました。

慣れない、というか良くわからないTwitterのdeveloperポータルと戦いながら、(時間の無駄になったアカウント作成を含め、)1時間後にはツイートできました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">API v2のテスト</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1663590945433227266?ref_src=twsrc%5Etfw">May 30, 2023</a></blockquote>

ここまでくれば後はXMLのパースと、本文の組み立てで済みます。
どんな挙動で動いてるのかは後述しますが、とりあえずRSSのXMLから最新投稿を取得し本文の組み立てをしてツイートするところまで難なくたどり着けました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">[2023/05/30 17:00 up!] 『月末ですが、徳島の思い出を語り始めます。[中村繪里子・吉田尚記の本格雑談くちをひらく]』 podcast→<a href="https://t.co/eqgXh7WKLF">https://t.co/eqgXh7WKLF</a> Web→<a href="https://t.co/i3pNDEpcDp">https://t.co/i3pNDEpcDp</a> <a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a> <a href="https://t.co/sAF622ibtO">https://t.co/sAF622ibtO</a></p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1663593795802497024?ref_src=twsrc%5Etfw">May 30, 2023</a></blockquote>
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">おーーー (ざっくりBotができたかもしれない)  
  
まぁとりあえず寝る</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1663593975155134465?ref_src=twsrc%5Etfw">May 30, 2023</a></blockquote>

この後、cronで回して、「新規投稿があればツイートする」ところまで出来たので寝ました。

翌日、ドキドキしながら17:00を待ちます。スクリプトからは標準出力に状況を出すようにしたので、ターミナルを眺めながらドキドキ待っていると・・・

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">[2023/05/31 17:00 up!] 『邪神ちゃんなイベント裏話。[中村繪里子・吉田尚記の本格雑談くちをひらく]』 podcast→<a href="https://t.co/eqgXh7WKLF">https://t.co/eqgXh7WKLF</a> Web→<a href="https://t.co/i3pNDEpcDp">https://t.co/i3pNDEpcDp</a> <a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a> <a href="https://t.co/ykPxrZb1Wf">https://t.co/ykPxrZb1Wf</a></p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1663818324575412225?ref_src=twsrc%5Etfw">May 31, 2023</a></blockquote>

無事に更新に追従してツイートされました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">うおおお更新追従に成功した・・・</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1663818448110260224?ref_src=twsrc%5Etfw">May 31, 2023</a></blockquote>

## 約1ヶ月、個人アカウントでツイートし続ける

とりあえず番組のメール宛に「こんなBot作って動かしてみています。もし良かったらそちらのアカウント等で動作できますが、どうでしょうか(でもトークン問題とか難しいですよね。こっちはただのいちリスナーですし。)」といった内容を送りつつ、毎日17:02にBotがツイートする日々を過ごしていました。

## 公式黙認される

公式アカウントでのツイートが止まってから初めての収録があり、その新録分での配信が始まったな、と思っていたところ、
こんな更新がありました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">[2023/06/26 17:00 up!] 『公式黙認宣言。[中村繪里子・吉田尚記の本格雑談くちをひらく]』 podcast→<a href="https://t.co/eqgXh7WKLF">https://t.co/eqgXh7WKLF</a> Web→<a href="https://t.co/i3pNDEpcDp">https://t.co/i3pNDEpcDp</a> <a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a> <a href="https://t.co/2lJQKLVhfG">https://t.co/2lJQKLVhfG</a> (このツイートは勝手にやってるBotです)</p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1673244004685873152?ref_src=twsrc%5Etfw">June 26, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

内容としては、「僕がやってるこのツイートBotが**非公式だけど黙認**される」という驚きの内容でした。そんなことあるんだ。……確かに阿部寛のホームページはそういう由来か……

ただ、公式黙認された=公式アカウントではツイートされないということでもあります。

僕の個人アカウントでのツイートでは、全く界隈の異なる方はRT/いいねし辛いでしょうし、そもそも声優さんの公式アカウント等が、いくら番組で公式黙認とはいえ個人のアカウントのツイートを毎日RTするのも相当難しいかと思い、流石にやるならちゃんとアカウントを別で用意しよう、と考えます。

だた、先述の通り何故か新規アカウント作成に失敗し続ける状況なので、新規アカウント作成は諦め、過去に作っていてほとんど使っていなかったアカウントを、「くちをひらくの更新通知Bot」として流用することにしました。これが、公式アカウントが2017年8月スタートに対してBotが2014年1月スタートになっている理由です。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ここ1ヶ月、<a href="https://twitter.com/hashtag/%E3%81%8F%E3%81%A1%E3%82%92%E3%81%B2%E3%82%89%E3%81%8F?src=hash&amp;ref_src=twsrc%5Etfw">#くちをひらく</a> の更新通知をするBotを動かしていましたが、本日の回で公式黙認をいただきました。<br>・・・が、僕の個人アカウントをフォローするのも、という方も絶対にいらっしゃると思うので、昔眠らせてたアカウントを掘り起こし、くちをひらくの専用Botとして動かすことにしました <a href="https://t.co/wwVmo6Abjx">https://t.co/wwVmo6Abjx</a></p>&mdash; たっくん (@mikuta0407) <a href="https://twitter.com/mikuta0407/status/1673255345408606209?ref_src=twsrc%5Etfw">June 26, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

最初はいくら公式から黙認されたとはいえ、非公式アカウントではやはり拡散力は大幅に落ちるのではないか、と危惧していましたが、今では公式アカウント時代と同じようなRT/いいね数なので安心しています。石川さん(番組ディレクター)、吉田さん、中村さん、フォローしていただいてるみなさん、ありがとうございます。

## どうやって動作させているのか

ここからは、Botについて、技術的な面を書いておこうと思います。 隠すものはトークンくらいしか無いですし。

Bot自体はOracle Cloud InfrastructureのA1インスタンスで動かしています。4vCPU/24GB RAM/200GB SSDが無料です。なんででしょうね。こわいですね。 このインスタンスではMisskey鯖が2本動いてたりするレベルなので、この程度スクリプトの負荷はほぼ無視できます。いい時代だ。(Raspberry Piでやるにしても軽い処理ですけど)

ソースコード自体は [くちをひらくの更新通知ツイートBot(@kuchihira_bot)のソースコード](https://gist.github.com/mikuta0407/955808cff9eb725a313a17286c43b558) に置いてあります。 (Gitで管理するほどでもないのでGistです。)

どういう動作かはコメントに書いてあるので読んでください……と言ってしまったらこのブログが終わってしまうので、もうちょっと自然言語で解説しようかと思います。

- 16:50になったら、以下の処理をスタート
- "今日"の日時を取得
- OmnyfmからRSSのXMLを取得し、最新投稿分を読み込む
- 最新投稿の投稿日時を確認
- もし一致しない=前日分なら20秒待機し、2番に戻る(360回試してもだめなら諦める)
- 一致=更新があったら、文面を作成し、ツイートする

0番はcronでタイマー実行させています。PHPとは無関係ですね。

2番に関しては、[中村繪里子・吉田尚記の本格雑談くちをひらく clips - Omny.fm](https://omny.fm/shows/kuchiwohiraku?cloudflare-language=ja)「[RSSフィード](https://omny.fm/shows/kuchiwohiraku/playlists/podcast.rss)」というリンクがあるので、ありがたく使わせてもらっています。(多分元々のIFTTTもこれを使ってたのかなと)  
Botを作ってから知ったのですが、表示上の投稿日時は17:00になっているものの、実際にRSSに反映されるのは17:02のようです。(Botの投稿時間が17:02なのはこれが理由です)

4番のif分岐については、何らかの不具合によってRSSの更新に遅延が発生した場合を考慮して約2時間の幅を持たせています。  
2時間という幅自体が有効なわけではありませんが、このif処理は土日にも有効です。番組は基本的に月〜金に投稿されるので、cronでは

```
50 16 * * 1-6 php ...
```

といったように月〜金を指定すれば土日に無駄な処理をする必要は無いのですが、この番組は「プレミアムリスナー」という(ここで説明すると長くなるので詳細は省きますが)特別回があります。  
この特別回は基本的に数ヶ月に1度くらいのペースで、**土曜日に**投稿されます。  
これを逃さないようにするためには、とりあえず毎日処理させて、更新がなかったら諦めさせれば特にツイートもせず影響もありません。(強いて言えばRSSを2時間もの間20秒間隔で取得しにいくのがどうなのか、という程度ですが、omny.fmさん許してね。)

また、Twitter API v2のFreeでは、月1500件までしか投稿できないそうですが、先述(埋め込んだツイート)の通り、土日関係なく毎日投稿したとしても31回が最大値なので、何も問題ないようです。

以上、こんな感じで動いてるよ、のコーナーでした。

## 余談1

個人アカウントの頃、頻繁にBotが「Standalone App」に降格されてしまい、API v2が通らなくなってしまう、という問題がありました。  
どうやらProjectに最初からあるBotを使わず、別で作ってProjectに追加させたのがなにか悪かったようです。よくわかりません。新アカウントではデフォルトのBotをリネームして使っているので、今のところこういった問題は起きていません。

## 余談2

@kuchihira_botはBotの運営者として@mikuta0407を登録しています。  
……@mikuta0407が凍結された場合って、どうなるんでしょうね……。  
凍結されたくはないですが、凍結されないよう気をつけようと思います。  
(一度年齢ロックやらかしたときに免許証を開示してるので許してほしい)

## 〆

突貫で作ったBotがなんだかんだ安定運用していて、嬉しく思っています。  
非公式黙認という立場ですが、今後も安定運用目指していきますので、よろしくお願いします。
