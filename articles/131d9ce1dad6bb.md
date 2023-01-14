---
title: "VSCodeのRemote-SSHからDevContainerを起動するやつをやってみた"
emoji: "🖥️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Docker,VS Code,DevContainer,Linux,VSCode]
published: false
---

## 概要
Docker DesktopでのDevContainerを使った環境構築は楽でサイコーなんだけど、Docker Desktopの有償化云々の件を考えると、Desktop使わずに済ませたいなーと思っていたところ、こちらの記事でRemote-SSHからできるようになっていたと知り、これだ！！ということで試してみた。
[Visual Studio Codeで使えるリモート環境のdevcontainerが意外と便利そうだったのでまとめ](https://zenn.dev/bells17/articles/remote-ssh-devcontainer)
ただ、必要なことはすべて参照元にあるので、こちらの記事では最低限DevContainerやりたいときどうする？だけ書く。あと感想。

## やること
1. Linuxサーバを用意してssh接続できるようにする
1. Dockerをインストール
1. VSCodeからRemote-SSH接続
1. DevContainer起動
1. 開発が捗る

## 内容

### Linuxサーバを構築する
ここは割愛。ssh接続できるようにしてね

### Dockerをインストール
Docker Desktopではなく、コマンドラインでDocker engineを入れておく。
公式のドキュメントに従って入れればOK。
https://docs.docker.com/engine/install/ubuntu/

### VSCodeにRemote-SSHを入れて、サーバへ接続
VSCodeをインストールして、拡張機能でRemote-SSHをインストール、サーバへ接続する。
こちらのページがとてもわかり易いです。
[VSCodeを使ってサーバー環境にSSHリモート接続手順](https://blog.masuyoshi.com/vscode%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E7%92%B0%E5%A2%83%E3%81%ABssh%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%88%E6%8E%A5%E7%B6%9A%E6%89%8B%E9%A0%86/)

### DevContainer起動
さて、Docker Engineが入ったサーバへssh接続できれば、あとはローカルのDocker DesktopでDevContainer作成する場合とほぼ同じです。^[拙記事もぜひ  https://zenn.dev/kazuu/articles/fa84bec08f855b]
VSCodeで「フォルダを開く」→作業ディレクトリを選択。
作業ディレクトリがなければ、VSCodeでターミナルを開いて、 `mkdir` しましょう。
その状態でコマンドパレットを開いて「Dev Conteiners: Open Folder in Container...」、もしくは左下の「SSH:サーバ名」をクリックして「Reopen in Container」
.devcontainerがなければ、作成するウィザードが表示されるので、選んでいけば自動で.devcontainerが作成され、コンテナに接続されます。

## 感想
さて、上記の通り、簡単にサーバ側のDockerでDevContainerを起動することができました。
どうしてもDocker Desktopを使いたくない、法人利用で費用を払いたくない向けにはいいかもですね。

ただ、手順を追ってみて考えるのは、これDevContainerである必要あるのかなぁということでした。
例えば、コンテナ側でホットリロードでデバッグしようとしても、ポートフォワーディング設定しないと別PC（手元のPC）から見えませんし、運用も考慮したり複数コンテナ（アプリとDBサーバとか）やり始めると、Dockerfileやdocker-compose.ymlとか書きたくなるので、だったら最初からDockerでいいんじゃないとちょっと思いました。

個人でやっている分には、結局Docker Desktop使うのがいいですねぇ。

## 余談
ちなみに最近は [](https://vscode.dev) にアクセスすれば、ウェブでVSCodeができる。
ただし当然ながらDevContainerはできないし、ローカルファイルを扱いにくい。
一応、VSCode CLIを自前サーバにインストールして、Github Tunneleをすればほぼローカルと変わらない体験ができるらしいです。
[VSCode の Remote Tunnels で「いつもの開発環境へ」お手軽リモート接続](https://zenn.dev/hankei6km/articles/connect-my-machine-via-vscode-remote-tunne)
これでAndroidタブレットやiPadでリモート開発だ！なんて思ったけど、やっぱりDevContainerは扱えないらしく、だとするとホットリロードや自動テストは、CI環境用意しないといけない・・・？
まだちょっと難しそうですね。

結局サーバ側でVim動かすほうがいいのではとちょっと思っている・・・