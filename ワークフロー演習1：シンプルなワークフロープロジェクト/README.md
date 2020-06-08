# ワークフロー演習1：シンプルなワークフロープロジェクト

## この演習の目的

最初の演習として、Treasure Data上で単純なクエリを実行するためのTreasure Workflowを作成していきましょう。この演習を通して、ワークフローの基本的な文法と実行の仕方を学びます。

TDコンソール上からワークフローを作成、編集することもできますが、この演習ではお使いのPC上でワークフローを作成し、それをTD Toolbeltを利用してTreasure Dataにpushする方法をとります。

## プロジェクト作成

まずはPowerShell（Macの場合はお使いのターミナル）から以下のコマンドを実行してください。td wf initコマンドはワークフロープロジェクトを新規に作成するコマンドです。なお、td wfコマンドはtd workflowコマンドの短縮形で、まったく同じ意味を持ちます。

```sh
$ td wf init handson_step1
2020-03-26 16:26:22 +0900: Digdag v0.9.40
  Creating handson_step1/handson_step1.dig
  Creating handson_step1/.gitignore
Done. Type `cd handson_step1` and then `td workflow run handson_step1.dig` to run the workflow. Enjoy!
```

これでワークフロープロジェクトhandson_step1が新規に作成されました。作成されたフォルダの中身を見ると、サンプルのdigファイルが1つあります。

```sh
$ cd handson_step1
$ ls handson_step1.dig
```

## ローカルでの実行

試しにこのdigファイルを実行してみましょう。digdag runコマンドを実行すると以下のように出力されます。-l errorはログの量を制御するオプションなので、処理には関係ありません。

```sh
$ digdag run -l error handson_step1.dig

2020-03-26 16:33:33 +0900: Digdag v0.9.40
start 2020-03-26T00:00:00+00:00
2020-03-26 00:00:00 +00:00
second cat
second dog
third cat
third dog
first cat
first dog
finish 2020-03-26T00:00:00+00:00
```

ローカルで開発したワークフローのテストなどに、このdigdag runコマンドを使うとよいでしょう。

## ワークフロー記述

先ほどのdigファイルを書き換えていきましょう。handson_step1.digをお使いのテキストエディタで開き、以下の内容に書き換えてください。

```sh
handson_step1.dig
```

```sh
_export:
  td:
    database: sample_datasets

+task1:
  td>: queries/query.sql
  store_last_results: true

+task2:
  echo>: "Record count: ${td.last_results.cnt}"
```
  
このワークフローで重要なのは7行目の「store_last_results: true」という記述です。td>オペレータにこのオプションを付与すると、SQLの実行結果の１行目がtd.last_resultsという名前の変数に格納されます。ここで取得した値をその後のワークフロー内で利用することができます。今回のワークフローでは、+task2の中で、${td.last_results.cnt}をecho>オペレータに渡すことで変数の内容を表示しています。最後の.cntは「cntカラム」という意味で、例えばSQLで取得するカラムの名前がpathであれば、変数名は${td.last_results.path}になります。

次にqueriesという名前のディレクトリを作り、その中にquery.sqlというファイルを作成してください。

```sh
queries/query.sql
```

```sql
SELECT COUNT(*) as cnt FROM www_access;
```

これでTreasure Data上のsample_datasets.www_accessテーブルに対してSQLを実行するだけのワークフローが完成しました。

## Treasure Workflowの実行

記述したワークフローの文法にミスがないかどうか確認してみましょう。そのためのコマンドとしてtd wf checkコマンドが利用できます。

```sh
$ td wf check
2020-04-07 18:06:39 +0900: Digdag v0.9.40
  System default timezone: Asia/Tokyo

  Definitions (1 workflows):
    handson_step1 (3 tasks)

  Parameters:
    {}

  Schedules (0 entries):
```

文法に間違いがある場合は以下のようなエラーが表示されます。

```sh
error: io.digdag.core.repository.ModelValidationException: Validating workflow failed
+handson_step1+task2 contains invalid keys: 'echo': "{"echo":"Record count: ${td.last_results.cnt}"}" (config)
```

td wf checkコマンドがエラーなく終わったら、Treasure Workflow環境にこのプロジェクトを持っていきましょう。それにはtd wf pushコマンドを利用します。ターミナル上でdigファイルがあるディレクトリに行き、td wf pushコマンドを実行します。

```sh
$ td wf push handson_step1
2020-03-26 16:58:42 +0900: Digdag v0.9.40
Creating .digdag/tmp/archive-7691553520296721209.tar.gz...
  Archiving queries/query.sql
  Archiving handson_step1.dig
Workflows:
  handson_step1.dig
Uploaded:
  id: 239010
  name: handson_step1
  revision: f19933c2-66b1-49e2-b015-5469846d48bd
  archive type: s3
  project created at: 2020-03-26T07:58:45Z
  revision updated at: 2020-03-26T07:58:45Z

Use `td workflow workflows` to show all workflows.
```

pushが成功したら、Treasure Workflow上で実行してみましょう。先ほど、サンプルのワークフローをローカルで実行するときにはtd wf runコマンドを利用しましたが、Treasure Workflow上で実行させる場合にはtd wf startコマンドを利用します。

```sh
$ td wf start handson_step1 handson_step1 --session now
2020-03-30 10:22:12 +0900: Digdag v0.9.40
Started a session attempt:
  session id: 20193101
  attempt id: 95412094
  uuid: 1cc896bc-7b2b-4f69-b7e6-6be6dbe12e68
  project: handson_step1
  workflow: handson_step1
  session time: 2020-03-30 01:22:13 +0000
  retry attempt name:
  params: {}
  created at: 2020-03-30 10:22:15 +0900
```
  
ワークフローの実行結果をCLIから確認することも可能ですが、GUIのほうが一覧で見やすいので、今回はTDのコンソールから確認してみましょう。
https://console.treasuredata.com/app/workflows にアクセスすると、下記のようにワークフローの一覧が見えます。ワークフローの数が多い場合はCtrl+F（Macの場合は⌘+F）を押して検索ダイアログを開き、「handson_step1」で検索してください。

［Run History］タブに、このワークフローの実行履歴が表示されます。緑の「●」マークが付いているセッションは成功、赤の「●」マークが付いているセッションは失敗を表しています。いずれかのセッションをクリックすると、実行状況やログの情報が確認できます。それらのうち、［WORKFLOW LOGS］タブの情報を見ると、td>オペレータに指定したクエリが実行され、その結果がecho>オペレータで表示されていることが確認できます。

