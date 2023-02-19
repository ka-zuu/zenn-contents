---
title: "RaspiでカメラモジュールからYoutube配信するときの決定版"
emoji: "🍣"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Raspberrypi", "カメラモジュール", "Youtube配信"]
published: true
---

## 概要
ラズパイでカメラモジュールを接続してYoutubeにライブ配信をしようとしたら、意外と手こずったのでそのメモ。
ただ、結果的にはシンプルな解決方法だったので、ザッと経緯と結論だけ残します。

## 前提
うちで買っているカエルの水槽を配信するのをテーマに、定点カメラによるYoutubeライブ配信をテスト。
手元にはRaspberry pi 3Bと4がある。
カメラモジュールはAliExpressで適当に買ったV2 camera (IMX219)。

## 結論
ラズパイ4を使うことと、`libcamera-apps`を使うこと、ffmpegはaptでインストールできるものでOK。
シンプルに上記でやれば、配信自体は可能でした。
こちらのサイトが参考になります。
[ラズパイ新OS「Bullseye」でのカメラモジュールの使い方｜libcameraの使い方も詳しく解説](https://hellobreak.net/raspberry-pi-bullseye-libcamera/)
ただ、フレームレートが出ないこと（だんだん下がってくる）、画像サイズが切り出したようになってしまうので、突き詰める前にやめてしまいました。

## 経緯
最初はラズパイ3＋Rasberry Pi OS(32bit)でやろうと思ったけど、どうしてもエラーになった。
参考にしたのはこちら
[Raspberry PiでYouTubeによるライブ配信を行う！（音声なし）](https://lab4ict.com/website/articles/771)

コードを参考にさせていただき、実行したが、エラー。
```bash
$ raspivid -w 1920 -h 1080 -fps 10 -o - -t 0 | \
ffmpeg -thread_queue_size 512 -f h264 -r 10 -i - \
-f lavfi -i anullsrc=channel_layout=stereo:sample_rate=44100 \
-c:v h264_omx -c:a aac -g 20 -s 1920x1080 \
-f flv rtmp://a.rtmp.youtube.com/live2/<STREAM KEY>
```

エラー内容は以下。
```
libOMX_Core.so not found
libOmxCore.so not found
```

ここから、色々調べながらOMX＝ラズパイのハードウェアエンコーダをどうにかできないかを1日かけてやったけど、無理でした。
で、結果諦めてラズパイ4で最初からやってみたら、結論のとおりにうまくいきました、というお話でした。

OMXのインストールや、ffmpegのソースからのビルド（ライブラリの取捨選択が多すぎてわけわからん・・・）を経て、
何もしなくて良いということに落ち着いて、力が抜けた・・・というお話でした。
ラズパイ3でやる場合の解決策は結局不明なので、この記事はあまり意味がない内容ですみません。

## 補足
今やるなら、TP-Linkあたりの安いカメラ買って、配信するかなー
[RTSPをPC/RasPi経由でYouTube Live](https://qiita.com/nakamiri/items/78e4c8c2bfee85527f0f)

