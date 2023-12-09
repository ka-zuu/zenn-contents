---
title: "デスクトップPCにSteam Linkを楽につなげるため、Windows11のWake on Lanを設定したりログイン周りの設定したりした"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Windows11,WoL,"Steam Link","Windows Hello"]
published: false
---

## 概要
最近子どもが生まれまして、2階の書斎ではなく、基本1階のリビングで過ごしています。
自分はゲームはなるべくPCでやりたい派なんですが、ゲーム用のPCは書斎に。
リビングにあるノートPCやAndroid TV（ソニーのブラビア）、中華ゲーム機のAnbernic RG405mでリモートプレイできたらいいのでは？ということで、Steam Linkを使ってみたところ、遅延も少なくとても快適！

ただ、当然母艦のPCが起動していないと接続できず、かといってそのために都度2階に行くというのも億劫・・・
そういえばWake on Lanとかあったなー、今のWindowsでもできるんだろうか、というところからスタートして、諸々実現できたので、まとめをメモします。

## やりたいこと
Todo：ここにネットワーク図を入れる

* 2階のPCに対してLAN内からマジックパケットを送ることでPCが起動する。
* PCは起動するとSteam Linkの受付ができる状態になる
* クライアントからSteam Linkを起動してゲームができる

## やること

### Wake on Lanを有効にする
1. Windows11の設定
2. BIOS(EFI)の設定

### 自動ログインの設定
1. 認証なしログインの設定
2. Windows Helloの無効化

### Steamの設定
1. 自動起動設定
2. 音の出力

## まとめ

## おまけ
RG405mでSteam Linkをやるときの設定


