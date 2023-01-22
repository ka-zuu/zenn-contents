---
title: "SlackからDiscordに移行するので、Googleカレンダーの予定のアップデートをDiscordへ通知する"
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Googleカレンダー, Python, bash, Discord, ChatGPT]
published: true
---

## 概要

今まで家族内の連絡その他にSlackを使っていたけど、無料だと90日分しか過去ログが参照できなくなったことを期に、Discordへ移行を検討することに。
シンプルに済ますなら、IFFFTでもカレンダーのアップデートをDiscordへ通知することはできた。
でも既存のレシピじゃないので枠を消費するし、複数人のカレンダーを読むには人数分の枠が必要・・・
IFFFTの枠はIoTとかのために残しておきたい。

調べたら、ただメッセージ送るだけなら、Bot作るまでもなく、Incoming Webhook使えばいいじゃんということみたい。
というわけで、自宅サーバで動くようにしてみる。

また、今回はPythonを少しいじるが、自分はPythonほとんど書いたことないので、AIの力を借りてみる。


## 前段
Googleカレンダーからの情報取得の方法をググると、いくつかの方法が出てくる。
ここで考えるのは大きく2点。
一つはAPIキーを使うか、OAuthを使うか？
もう一つはOAuthするとして、アカウントを個人アカウントにするか、サービスアカウントにするか？

まずAPIキーは、公開カレンダーに匿名でアクセスする前提になる。公開して良いカレンダーで、誰がアクセスしたかわからなくても良いのであれば、一番楽。
今回はプライベートなカレンダーの情報を取得するので却下。

で、OAuthを使うことになるが、個人アカウントとしてAPIアクセスするか？サービスアカウントとしてアクセスするか？です。
例えばそれぞれが個人アカウントを持っていて、特定の代表者が各人のカレンダーを自身のGoogleカレンダーウェブアプリ上で表示しているような場合で、かつ代表者がカレンダー情報をまとめて取得するような場合には、個人アカウントとしてのAPI利用で十分。

こちらのサイトがわかりやすかったです。
[APIキー、OAuthクライアントID、サービスアカウントキーの違い:Google APIs - 無粋な日々に](https://messefor.hatenablog.com/entry/2020/10/08/080414)


## Discordの対応方法

Webhookの作り方は公式のここに書いてあります。
[Intro to Webhooks](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks)
チャンネルに紐づくので、チャンネルの設定で「連携サービス」を選択すれば、そこにウェブフックの設定がいます。
![](/images/4efb2aac76fd7a/4efb2aac76fd7a_discord.png)
すでにウェブフックが一ついると思うので、これを利用します。
アバター画像は、テキスト投稿時にURL指定もできる。公開されていない画像の場合はここでアップロードしておく。
今回はGoogleカレンダーのアイコンを設定。
![](/images/4efb2aac76fd7a/4efb2aac76fd7a_webhook.png)

Webhookに投げる方法は、こちらにすべてがあり、とても参考になります。
[DiscordにWebhookでいろいろ投稿する - Qiita](https://qiita.com/Eai/items/1165d08dce9f183eac74)

ぼくはシェル芸で書きたいので、curlでできる方法を公式から引っ張ってきます。
[Discord Webhooks Guide](https://birdie0.github.io/discord-webhooks-guide/tools/curl.html)


## Googleの対応方法

Googleカレンダーへのアクセスについては、基本的にはクイックスタートガイドどおりにやればいい説
[Python quickstart | Google Calendar | Google Developers](https://developers.google.com/calendar/api/quickstart/python)

Google CloudでAPIを有効にして、認証を作成、`credentials.json`を作成。
Googleへアクセスするためのライブラリをインストールすれば環境作成はOK。
サンプルコードを一度実行すると、実行ディレクトリにアクセストークン`token.js`を作成してくれる。

トークンを作成するために、サンプルコードを実行するとローカルでサーバを実行して、待機状態になる。
ブラウザを開いて権限を承認すると、`http://localhost/`へリダイレクトして、トークンが作成される。
最初DevContainer内で実行していたんだけど、リダイレクト先のポートが毎回変更されるので、コンテナのポートフォワーディング設定ができず、断念。
コンテナ外で`token.js`を作成した。
`credentials.json`と`token.js`は、別環境経もコピーして持っていけば認証に使えるので、サーバで実装するときはコピーすればOK。
逆に言えば、.gitignoreに追加する等、他人に渡らないようにしないといけない。



## Pythonの実装
設計としては、今以降の予定をすべて取得する→前回の取得結果とdiff→差分を通知。
予定の取得はPythonのサンプルコードをベースに修正する。

ここで、最近はやりのAIを使ってみることにした。
サンプルコードは「今以降の10件の予定を取得する」コードだけど、今回は「指定したカレンダーの、今以降のすべての予定を取得」したい。

まず試しにChatGPTに、サンプルコードの説明をしてもらう。
`次のコードの説明をして`と書いて、コードをベタ貼り。
![](/images/4efb2aac76fd7a/4efb2aac76fd7a_chatgpt_explain.png)
すご・・・

次に、`このコードを、コマンドライン引数でカレンダーIDを指定できるようにして`とお願いして出てきたコードがこれ
```python
from __future__ import print_function

import datetime
import os.path
import sys

from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build
from googleapiclient.errors import HttpError

# If modifying these scopes, delete the file token.json.
SCOPES = ['https://www.googleapis.com/auth/calendar.readonly']


def main():
    """Shows basic usage of the Google Calendar API.
    Prints the start and name of the next 10 events on the user's calendar.
    """

    if len(sys.argv) < 2:
        print("Please provide a calendar ID as a command line argument.")
        return
    calendar_id = sys.argv[1]

    creds = None
    # The file token.json stores the user's access and refresh tokens, and is
    # created automatically when the authorization flow completes for the first
    # time.
    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)
    # If there are no (valid) credentials available, let the user log in.
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                'credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)
        # Save the credentials for the next run
        with open('token.json', 'w') as token:
            token.write(creds.to_json())

    try:
        service = build('calendar', 'v3', credentials=creds)

        # Call the Calendar API
        now = datetime.datetime.utcnow().isoformat() + 'Z'  # 'Z' indicates UTC time
        print('Getting the upcoming 10 events')
        events_result = service.events().list(calendarId=calendar_id, timeMin=now, singleEvents=True, orderBy='startTime').execute()
        events = events_result.get('items', [])

        if not events:
            print('No upcoming events found.')
            return

        # Prints the start and name of the next 10 events
        for event in events:
            start = event['start'].get('dateTime', event['start'].get('date'))
            print(start, event['summary'])

    except HttpError as error:
        print('An error occurred: %s' % error)


if __name__ == '__main__':
    main()
```
実際には多少修正する必要はあったけど、`sys`をインポートして引数を取得すれば良いということがサクッとわかるので、それだけでタスカル・・・！
Python詳しくなくてもできちゃうね。


## シェルスクリプトの実装
スクリプトはこちら。
```bash
#!/bin/bash -xvu

# カレンダーからイベントを取得して、差分をDiscordに通知する

# 初期設定
tmp=$(mktemp)

work_dir="$(dirname $0)" # スクリプトのあるディレクトリ
filename="$(basename $0)" # スクリプトのファイル名
mkdir -p ${work_dir}/log # ログディレクトリ
exec 2> ${work_dir}/log/${filename}.$(date +%Y%m%d_%H%M%S) # 標準エラー出力をログファイルに出力
cd ${work_dir}

# 外部の変数ファイルを取得
source ${work_dir}/keys.sh

# カレンダーイベントの取得
cat ${work_dir}/target_calendars.txt |
while read calendar_id; do
  python3 ${work_dir}/get_event_gcal.py ${calendar_id} > $tmp-${calendar_id}_events

  # 過去のデータを取得して、差分を抽出
  if [ -s ${work_dir}/events_old/${calendar_id}_events ]; then
    diff ${work_dir}/events_old/${calendar_id}_events $tmp-${calendar_id}_events > $tmp-${calendar_id}_diff
  fi

  # 差分があればDiscordに通知
  if [ -s $tmp-${calendar_id}_diff ]; then
    while read line; do
      if [ "${line:0:1}" = ">" ]; then
        # Discordに通知
        curl -X POST -H "Content-Type: application/json" -d "{\"content\": \"${line:2}\"}" ${DISCORD_WEBHOOK_URL}
      fi
    done < $tmp-${calendar_id}_diff
  fi

  # 今回の結果を過去データとして保存
  mkdir -p ${work_dir}/events_old
  mv $tmp-${calendar_id}_events ${work_dir}/events_old/${calendar_id}_events

done

# 一時ファイルの削除
rm -f $tmp-*

# 終了
exit 0
```

コメントに書いてある通りの処理ですが、複数のカレンダーから予定を取得して、それぞれをテキストに毎回保存。
前回の保存結果とdiffを取り、差分をDiscordへ投稿します。
Discordへの通知は、公式サイトの処理のまま、1行ずつ投げています。
[Discord Webhooks Guide](https://birdie0.github.io/discord-webhooks-guide/tools/curl.html)

ちなみに、このスクリプトをChatGPTさんに説明してもらうと、こう。
![](/images/4efb2aac76fd7a/4efb2aac76fd7a_chatgpt_shell.png)

その日の予定一覧を取得するコードも、Githubに上げてあります。
よければどうぞ。
[https://github.com/ka-zuu/GoogleCalendar_to_Discord](https://github.com/ka-zuu/GoogleCalendar_to_Discord)

最後に、crontabに登録して終了。
予定を追加すると、通知が来ます。

## まとめ
GoogleカレンダーからDiscordへの通知ができるようになりました。
OAuthのAPIは面倒だけど、ウェブフックは楽で良い。
あと、ChatGPTの可能性は大いに感じることができました。
ちなみに今回はGithub Copilotも使っていて、コードのサジェストがいちいち素敵でコーディングが楽しい。
プログラマの世界が変わっていく潮目を感じました！

次回はAlexaの買い物リストをDiscordに通知するお話です。
Copilotを絡めて書いてみたい。
