# collect-daily-news

* [collect-daily-news-action](https://github.com/hirakawamizuki/collect-daily-news-action) アクションのデモ用リポジトリです
* 自身で設定した複数キーワードに対して、毎日定刻にGoogle Newsからニュースを収集し、Slackに通知しています。

## ディレクトリ構成

* たったこれだけです
* `.github/workflow/main.yml` と同様のファイルを任意のリポジトリに配置することで、任意のリポジトリでも同様のアクションを実行できます
    ```
    $ collect-daily-news
    ├── .github/workflow/main.yml  # GitHub Actionsの動作を定義する場所
    └── README.md
    ```

## `.github/workflow/main.yml` の解説

### ①定期実行：

```
on:
  schedule:
    - cron: '0 2 * * *'  # UTCの2:00（JSTの11:00）に毎日実行する
```

* GitHub Actionsでは、リポジトリのイベント（git push、プルリクエスト作成、など）以外にも、cron記法に従って定期的なスケジュールイベントをトリガにすることもできます

### ②ニュースの収集：

```
uses: hirakawamizuki/collect-daily-news-action@v0.1.1  # ニュースを収集する自作アクション
id: collect-news
with:
  keywords: "Docker,GitHub+Container+Registry,Docker+Hub,CentOS+終了"
  how-many-days: "1"  # 過去1日分のニュースを収集
  output-format: "mrkdwn"  # slack mrkdwn形式で出力
```


* GitHub Actionsの中では、自作のアクションを設定して実行することもできます
* [collect-daily-news-action](https://github.com/hirakawamizuki/collect-daily-news-action) という自作アクションを `uses:` で呼び出しています
* アクション独自のパラメータを `with:` で指定できます
* `keywords` パラメータでは、検索ワードを複数設定しています
* 上記例では、「`Docker`」「`GitHub Container Registry`」「`Docker Hub`」「`CentOS 終了`」というそれぞれの検索キーワードにおいて [Google News](https://news.google.com/topstories?hl=ja&gl=JP&ceid=JP:ja) で検索を行う振る舞いをします
* collect-daily-news-actionアクションの独自パラメータの詳細については、 [README](https://github.com/hirakawamizuki/collect-daily-news-action#collect-daily-news-action) をご確認ください

### ③Slackへの通知：

```
uses: tokorom/action-slack-incoming-webhook@main  # Slack通知用のアクション
env:
  INCOMING_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
with:
  text: ${{ steps.collect-news.outputs.result }}  # アクション間でアウトプットを受け渡している
```

* [GitHub Actions Marketplace](https://github.com/marketplace?type=actions) にある [action-slack-incoming-webhook](https://github.com/tokorom/action-slack-incoming-webhook) というアクションを利用して、Slackへの通知を実現しています
* `${{ steps.<アクションid>.outputs.<output名> }}` という形式で、異なるアクション間でアウトプットを受け渡しできます
* 公開したくないパラメータについては、リポジトリのSettings > Secretsに設定し、その値を `${{ secrets.<パラメータ名> }}` という形式でGitHub Actions側から呼び出せます
