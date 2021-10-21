---
title: "VSCode + Remote ContainerでGoの開発環境を作ったけどimportできなくて悩んだ話"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Go","Docker","VSCode","DevContainer"]
published: true
---
# 概要
[前回の記事](https://qiita.com/ka-zuu/items/3144780ee528815e7557)でVSCodeでDockerのDevContainerをサクッと起動して、OS環境を汚さない快適開発環境を作れるようになったけど、Goでサンプル実装しようとすると、どうにもうまく行かない。
具体的には、自分の作ったpackageをimportできない。
情報を色々検索したところ、Goの推奨する標準構成が色々変わったことに、インターネッツの記事が追いついていなく、新旧の情報が錯綜しているためだった。
結果的に解決できたので、その備忘録を書いておく。

# 結論
結論から書くと、こちらのサイトを最初から参考にして環境作っておけばよかった・・・という話・・・
[Go 脱初心者への道](https://zenn.dev/shellyln/articles/b2992891f8f3f9381025#1.-%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E4%BD%9C%E6%88%90%E3%83%BB%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E7%AE%A1%E7%90%86)

* GOPATHは今はもう使わず、go modを活用しましょう
* srcフォルダは使わず、cmdを使いましょう

# やったこと

## 失敗の歴史
前回のように構築したあと、main.goのみの実行は問題なく出来、よしよしと思い、入門編をやり始めた。
[gihyo.jpのこちら](https://qiita.com/tenntenn/items/0e33a4959250d1a55045)を参考に、少し古いけど大まか変わらないだろう、と考えて、[こちら]をやり始めたところ、importがうまく行かない・・・
gosampleモジュールをvscode上でも認識してくれないし、実際にビルドもできない。
よくある、コンテナ内でのパスの問題かなと思い、GOPATHやGOROOTの設定を変えようとしてみた。

[こちら](https://dev.classmethod.jp/articles/vscode-remote-containers-golang/#toc-7)とかに書いてあるように
devcontainer.jsonをいじって、`workspaceMount`や`workspaceFolder`を変更してみたが、コンテナのビルド時にエラー・・・
GOPATHを変更しようとしても、うまく行かない・・・ということで色々調べていると、こちらのサイトにたどり着く。
[go mod完全に理解した](https://zenn.dev/optimisuke/articles/105feac3f8e726830f8c)

> go mod周辺を理解してないとGo言語使った開発辛いと思うんですが、ネットの日本語情報少なくないすか？
最近はGOPATH使わないと思うんですが、古い情報が引っかかってきて、検索するのが辛いです。GOPATH使う時とgo mod使うときの比較情報も、あたらしくGo言語やる人的にはいらないと思うし。変更点は良いから、最新版の体系的な情報をくれよ、ってなってました。

ほんこれ・・・

つまるところ、プロジェクトルートにgo.modを起きましょうという話のよう。
喜び勇んでプロジェクトのルートフォルダで`go mod init [プロジェクト名]`としてみた・・・が、変わらず。

最終的には、冒頭で紹介した[こちら](https://zenn.dev/shellyln/articles/b2992891f8f3f9381025#1.-%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E4%BD%9C%E6%88%90%E3%83%BB%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E7%AE%A1%E7%90%86)にたどり着いたことで、無事解決・・・


## 解答編
まず、上記ブログにあるように、フォルダ構成はsrcを使わないようになってきているよう。
（ただ、今回の問題とは根本的には関係ない）
[こちらのGithubリポジトリ](https://github.com/golang-standards/project-layout/blob/master/README_ja.md)にフォルダ構成の有志によるオススメ例があるので、参考にさせてもらう。
日本語READMEもあるし、参考になるリポジトリが紹介されているのも◎。

上記フォルダ構成をベースに、gihyo.jpのサンプルコードを実装すると、図のような感じ。
![](/images/5d1bdff134edf8/5d1bdff134edf8_1.png)


main.goの中でgosampleが相変わらず解決できずエラーになっている。

そこで、ワークフォルダルートで以下を実行。

```shell
vscode ➜ /workspaces/test3 $ pwd
/workspaces/test3
vscode ➜ /workspaces/test3 $ go mod init test3
go: creating new go.mod: module test3
go: to add module requirements and sums:
        go mod tidy
vscode ➜ /workspaces/test3 $ 
```

以下のようなgo.modができる。
ただ、相変わらずgosampleは見つからない。
![](/images/5d1bdff134edf8/5d1bdff134edf8_2.png)


この理由は2つ。
1. go.modを使う場合は、importするときに「相対パス」を指定する必要がある
2. モジュール名がローカルフォルダを指す場合は、`replace`を指定する必要がある

`replace`については[こちら](https://zenn.dev/shellyln/articles/b2992891f8f3f9381025#1.-b.-%E3%81%93%E3%82%8C%E3%81%8B%E3%82%89%E5%A7%8B%E3%82%81%E3%82%8B%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%A7%E3%81%AF%E3%80%81%E6%9C%80%E5%88%9D%E3%81%8B%E3%82%89%E6%96%B0%E3%81%97%E3%81%84%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E7%AE%A1%E7%90%86%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E3%81%A7%E3%81%82%E3%82%8B-go-modules-%E3%82%92%E4%BD%BF%E3%81%86)。

go.modはこうなる

```go
module test3

go 1.17

replace (
  test3 v0.0.0 => ./
)
```

main.goはこう

```go
package main

import (
	"fmt"
	"test3/internal/gosample" //"test3/internal"を追記
)

func main()  {
	fmt.Println(gosample.Message)
}
```

無事、エラーが消えた！！
![](/images/5d1bdff134edf8/5d1bdff134edf8_3.png)

## 補足
ちなみに、上記で`test3`とした部分は、公開リポジトリの場合は例えばGithubのパスを指定するが、非公開であれば何でも良い。
何でもいい理由は、`replace`で読み替えちゃうから、ということでした。
この辺がマジでわかりにくかった・・・
ちなみに、公開リポジトリを指定すると、ビルド時に`$GOPATH/pkg/mod`にインストールされるらしいよ。
（やってないし、あとでまた苦労しそうな予感・・・）

あとたぶん、DevContainerでGoの環境作っただけだと関連ツールがインストールされていないぽいので、[こちら](https://zenn.dev/tomi/articles/2020-10-22-go-docker#%E8%A3%9C%E5%AE%8C%E3%83%84%E3%83%BC%E3%83%AB%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)もやっておいたほうが良さそうですね。

# まとめ
Goは言語の名称のググラビリティが低いだけじゃなくて、挙動や最新情報のググラビリティも低い・・・
動けばええやんじゃなくて、解決にたどり着けてよかったよかった。
というか、ほぼ全て@shellylnさんのおかげでした。ありがとうございます。
