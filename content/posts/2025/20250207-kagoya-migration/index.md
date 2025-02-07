---
title: 'KAGOYA CLOUD VPSを新基盤側に移行した'
date: 2025-02-07T15:00:00+09:00
draft: false
categories: ["Linux","VPS","盆栽システム開発", "ネットワーク"]
description: KAGOYAのVPSがリニューアルしたので旧基盤から移行してみました。
# original: 
image: img/eyecatch.png
---

## KAGOYAのVPSが強くなったらしい

2025年2月6日、国内事業者が運営するVPSのKAGOYA CLOUD VPSの新基盤開始が告知されました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">【お知らせ】<br>2月6日(木)より、国内事業者VPSとして13年の実績のある「KAGOYA CLOUD VPS」の全プランにて新基盤を採用し、性能を大幅に強化しました。 <a href="https://t.co/ikoPMLFIQf">pic.twitter.com/ikoPMLFIQf</a></p>&mdash; カゴヤ・ジャパン (@kagoya_inc) <a href="https://twitter.com/kagoya_inc/status/1887381905253048617?ref_src=twsrc%5Etfw">February 6, 2025</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

[【VPS】KAGOYA CLOUD VPS リニューアルのお知らせ | KAGOYA](https://www.kagoya.jp/news/2025020631681/?argument=vqHX23Xs&dmai=a67a44bf8e26c6)

これまで私は大容量プランで、「2コア/2GB/200GB」を月額880円で使用していましたが、新基盤ではなんとストレージも外足速度も強くなって、110円も安い770円です。

| |旧基盤|新基盤|
|:----|:----|:----|
|コア数|2vCPU|2vCPU|
|RAM|2GB|2GB|
|ストレージ|200GB|200GB|
|NW速度(実測)|70Mbps程|200Mbps程|
|値段|880円|770円|

ちなみに年払いにしたら毎月715円相当。OCIのAlways Free以外なら安定性的にもKAGOYAが強すぎます。新基盤すごい。

(WebARENA IndigoのVPSを使っていたことがあるのですが、回線品質が不安定すぎて何度も疎通断アラートが鳴ったことがありましたが、KAGOYA移行後はほぼ発生していないので助かります)

ということで早速移行してみたので、移行手順の記録と、簡単な性能比較をしてみました。

ちなみに:
- 旧プラン情報: [料金・機能・テンプレート | KAGOYA (Wayback Machine)](https://web.archive.org/web/20241003141651/https://www.kagoya.jp/vps/function-plan/#anchor-standard)
- 新プラン情報: [KAGOYA CLOUD VPS | 柔軟でパワフルな環境を格安で](https://www.kagoya.jp/vps/)

## 既存インスタンスは?

新基盤が出はしましたが、告知メール文には

> 2025年2月6日(木)より、「KAGOYA CLOUD VPS」を大幅に刷新しました。
> 本リリースにより、読み込み・書き込み性能が大幅に向上し、業界No.1(※1)のコストパフォーマンスで、さらに快適で高速なVPS環境をご利用いただけるようになりました。

という記載だけで、プレスリリースの文言やFAQを読んでも既存インスタンスがどういう扱いになるのかが読み取れませんでした。

自動でマイグレーションされるのか、はたまた借り換えをしないといけないのか、そこ書いてほしかったな……。

## どうやらスナップショットを使うと移行できるっぽい

どうやらスナップショットは既存インスタンスと新基盤で共有できるようで、既存インスタンスでスナップショットを取って、それを元に新規インスタンスを立てることでIPアドレスは変化してしまいますが移行ができる、という人柱報告を得ることが出来ました。

## 移行手順

ということで、以下の手順を踏んで、式年遷宮で全部を作り直すこと無く、新基盤へ移行します。

1. 既存インスタンスをシャットダウン
2. 既存インスタンスのスナップショット作成
    - 10GBあたり4.4円/日とのこと。200GBなので88円/日ですね。実質タダ。
    - スナップショット作成には2時間弱かかりました。
3. 新規インスタンス作成
    - このときベースイメージをスナップショットにする
    - 1時間くらいかかりました。
4. Webコンソールから起動確認
5. IP直指定でのSSH疎通確認
6. DNS向け先変更
7. 外部からの疎通確認
8. 旧インスタンス削除
9. スナップショット削除

## 細かい修正

外足のIPアドレスが変わるので、いくつかを適当に修正していきます。

- 別ホストのApache2でIPのホワイトリストベースのACL
- HAProxyの掴むIPアドレス
- スタティックルート書いてたホストのルーティング

## 性能確認

### fastfetch結果(HW部分系のみ)

見える変化
- 仮想化基盤がOpenStack NovaからOpenStack Computeに
- CPUがIntel Xeon Silver 4210からIntel Xeon Silver 4416+に
- 仮想GPUがCirrus Logic GD 5446からRedHat Virtio 1.0 GPUに

#### 旧基盤

```
OS: Ubuntu 22.04.5 LTS x86_64
Host: OpenStack Nova (13.1.2-1.el7)
Kernel: Linux 6.8.0-52-generic
CPU: 2 x Intel(R) Xeon(R) Silver 4210 (2) @ 2.19 GHz
GPU: Cirrus Logic GD 5446
Memory: 1.19 GiB / 1.92 GiB (62%)
Swap: 268.00 KiB / 8.93 GiB (0%)
Disk (/): 147.77 GiB / 196.68 GiB (75%) - ext4
```

#### 新基盤

```
OS: Ubuntu 22.04.5 LTS x86_64
Host: OpenStack Compute (25.3.0-1.el9s)
Kernel: Linux 6.8.0-52-generic
Uptime: 14 mins
CPU: 2 x Intel(R) Xeon(R) Silver 4416+ (2) @ 2.00 GHz
GPU: RedHat Virtio 1.0 GPU
Memory: 1.20 GiB / 1.92 GiB (62%)
Swap: 268.00 KiB / 8.93 GiB (0%)
Disk (/): 147.65 GiB / 196.68 GiB (75%) - ext4
```

### 外足の速度測定

新基盤は3倍という謳い文句はそのとおりです。ちなみに新基盤では初速だけ500Mbps以上の数値が見えました。

#### 旧基盤

```
$ ./speedtest -s 7139

   Speedtest by Ookla

      Server: SoftEther Corporation - Tsukuba (id: 7139)
         ISP: KAGOYA JAPAN
Idle Latency:    10.48 ms   (jitter: 0.43ms, low: 10.35ms, high: 11.93ms)
    Download:    76.95 Mbps (data used: 35.5 MB)
                160.79 ms   (jitter: 53.67ms, low: 11.17ms, high: 314.71ms)
      Upload:    77.04 Mbps (data used: 39.6 MB)
                181.38 ms   (jitter: 56.09ms, low: 10.97ms, high: 291.02ms)
 Packet Loss:     0.0%
```

#### 新基盤

```
$ ./speedtest -s 7139

   Speedtest by Ookla

      Server: SoftEther Corporation - Tsukuba (id: 7139)
         ISP: KAGOYA JAPAN
Idle Latency:    10.32 ms   (jitter: 0.22ms, low: 10.23ms, high: 10.92ms)
    Download:   243.23 Mbps (data used: 320.1 MB)
                 31.21 ms   (jitter: 18.27ms, low: 10.25ms, high: 510.81ms)
      Upload:   234.95 Mbps (data used: 365.7 MB)
                 10.46 ms   (jitter: 2.29ms, low: 10.19ms, high: 239.22ms)
 Packet Loss:     0.0%
```

### ストレージ速度測定

[【diskspd】ストレージのベンチマーク方法（Linux版CrystalDiskMarkを作ってみた） – Hacker's High](https://hackers-high.com/linux/storage-benchmark-like-crystaldiskmark/) で作られている、 [DiskMark-linux.sh ](https://github.com/haxyier/DiskMark-linux-sh/blob/master/DiskMark-linux.sh) をお借りしました。

旧基盤と新基盤でシーケンシャルの最大速度が11GB/s前後で、大体9Gbps弱となります。おそらく10GのNICでiSCSI接続しているのでしょう。文句はないです。

大きく進化しているのがランダムR/Wです。旧ではQ32T16が50MB/sを切っていましたが、新では501.66MB/sと10倍以上高速化しています。また、Q1T1に関してもかなりの速度上昇が見られます。

NVMe化によるランダムR/Wの改善がよくわかりますね。

#### MB/s

##### 旧基盤

||Read|Write|
|:----|:----|:----|
|SEQ1M Q8T1|1141.20|2032.80|
|SEQ1M Q1T1|142.60|1372.20|
|RND4K Q32T16|47.43|78.44|
|RND4K Q1T1|0.07|41.50|

##### 新基盤

||Read|Write|
|:----|:----|:----|
|SEQ1M Q8T1|1044.40|1044.20|
|SEQ1M Q1T1|1044.40|1044.40|
|RND4K Q32T16|501.66|429.67|
|RND4K Q1T1|53.23|52.79|


#### IOPS

##### 旧基盤
||Read|Write|
|:----|:----|:----|
|SEQ1M Q8T1|1141.20|2032.80|
|SEQ1M Q1T1|142.60|1372.20|
|RND4K Q32T16|12141.00|20079.60|
|RND4K Q1T1|17.80|10623.60|

##### 新基盤

||Read|Write|
|:----|:----|:----|
|SEQ1M Q8T1|1044.40|1044.20|
|SEQ1M Q1T1|1044.40|1044.40|
|RND4K Q32T16|128424.80|109996.60|
|RND4K Q1T1|13627.00|13513.00|
