# ワークフロー演習1：シンプルなワークフロープロジェクト

## この演習の目的

最初の演習として、Treasure Data上で単純なクエリを実行するためのTreasure Workflowを作成していきましょう。この演習を通して、ワークフローの基本的な文法と実行の仕方を学びます。

TDコンソール上からワークフローを作成、編集することもできますが、この演習ではお使いのPC上でワークフローを作成し、それをTD Toolbeltを利用してTreasure Dataにpuyamlする方法をとります。

## プロジェクト作成

まずはPoweryamlell（Macの場合はお使いのターミナル）から以下のコマンドを実行してください。td wf initコマンドはワークフロープロジェクトを新規に作成するコマンドです。なお、td wfコマンドはtd workflowコマンドの短縮形で、まったく同じ意味を持ちます。

```yaml
$ td wf init handson_step1
2020-03-26 16:26:22 +0900: Digdag v0.9.40
  Creating handson_step1/handson_step1.dig
  Creating handson_step1/.gitignore
Done. Type `cd handson_step1` and then `td workflow run handson_step1.dig` to run the workflow. Enjoy!
```

これでワークフロープロジェクトhandson_step1が新規に作成されました。作成されたフォルダの中身を見ると、サンプルのdigファイルが1つあります。

```yaml
$ cd handson_step1
$ ls handson_step1.dig
```

## ローカルでの実行

試しにこのdigファイルを実行してみましょう。digdag runコマンドを実行すると以下のように出力されます。-l errorはログの量を制御するオプションなので、処理には関係ありません。

```yaml
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
finiyaml 2020-03-26T00:00:00+00:00
```

ローカルで開発したワークフローのテストなどに、このdigdag runコマンドを使うとよいでしょう。

## ワークフロー記述

先ほどのdigファイルを書き換えていきましょう。handson_step1.digをお使いのテキストエディタで開き、以下の内容に書き換えてください。

```yaml
handson_step1.dig
```

```yaml
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

```yaml
queries/query.sql
```

```sql
SELECT COUNT(*) as cnt FROM www_access;
```

これでTreasure Data上のsample_datasets.www_accessテーブルに対してSQLを実行するだけのワークフローが完成しました。

## Treasure Workflowの実行

記述したワークフローの文法にミスがないかどうか確認してみましょう。そのためのコマンドとしてtd wf checkコマンドが利用できます。

```yaml
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

```yaml
error: io.digdag.core.repository.ModelValidationException: Validating workflow failed
+handson_step1+task2 contains invalid keys: 'echo': "{"echo":"Record count: ${td.last_results.cnt}"}" (config)
```

td wf checkコマンドがエラーなく終わったら、Treasure Workflow環境にこのプロジェクトを持っていきましょう。それにはtd wf puyamlコマンドを利用します。ターミナル上でdigファイルがあるディレクトリに行き、td wf puyamlコマンドを実行します。

```yaml
$ td wf puyaml handson_step1
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

Use `td workflow workflows` to yamlow all workflows.
```

puyamlが成功したら、Treasure Workflow上で実行してみましょう。先ほど、サンプルのワークフローをローカルで実行するときにはtd wf runコマンドを利用しましたが、Treasure Workflow上で実行させる場合にはtd wf startコマンドを利用します。

```yaml
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

# ワークフロー演習2：データロードとExport

## この演習の目的

演習1では、非常に単純なクエリの実行と結果の確認をTreasure Workflowを利用して行う方法を学びました。次はさらに実践的な内容に移っていきます。外部データソースからのデータロードおよびデータの外部システムへのExportをTreasure Workflowを利用して実行する方法を紹介します。
今回はAmazon S3上に配置したサンプルデータのCSVをTreasure Data上のテーブルに取り込み、Data Tankへと出力するワークフローを記述します。

## ワークフローコンソールを使った編集

今回の演習では、ワークフローコンソール上からワークフローの追加をしてみましょう。ワークフローの画面から［Workflow Definition］タブを選択すると、右側にアイコンが表示されます。一番上の鉛筆のアイコンが編集モードに切り替えるためのボタンで、その下の眼のアイコンがプロジェクトに含まれている全ファイルを表示するボタンになっています。

一番上の鉛筆ボタンを押して編集モードに切り替えましょう。右側に新しく表示されたアイコンを押すと、ワークフロープロジェクトに新規ファイルを追加できます。

このボタンを押すと新しいファイルができるので、ファイル名を「handson_step2.dig」と入力して［OK］を押してください。拡張子（.dig）まで入力するのを忘れないようにしてください。

## データロードとExportの定義

ファイル名の下の領域にはファイルの中身を記載していきます。以下の内容をコピー＆ペーストしてください。

```yaml
handson_step2.dig
_export:
  td:
    database: wf_handson_db

+prepare_table:
  td_ddl>:
  database: ${td.database}
  create_tables:
  - customers

+load_from_s3:
  td_load>: config/customers.yml
  database: ${td.database}
  table: customers

+export_to_s3:
  td>:
  query: select * from customers
  result_connection: workflow_handson
  result_settings:
    bucket: hands-on-seminar
    path: /output/file_${moment(session_time).format("YYYYMMDD")}.csv
    header: true
```

このワークフローには「+load_from_s3:」と「+export_to_s3:」という2つのタスクがあります。1つ目のタスクのオペレータはtd_load>で、外部データソースからTreasure Dataにデータをロードするときに利用するオペレータです。接続先や読み込むカラムの設定は、オプションとして指定した「config/customers.yml」に記述します。このファイルは次のステップで作成します。

2つ目のタスクは、演習1でも利用したtd>オペレータですが、2つ違いがあります。まず、SQLを外部のsqlファイルに記述するのではなく、query:オプションとして直接記述しています。十分に短いSQLであれば、単純化のためにdigファイル内に直接記述するのもよいでしょう。もう1つの違いは、result_connection:とresult_settings:というオプションが追加されている点です。これらの設定は外部にSQLの結果を出力するために必要です。

result_connection:には、Integration Hubで作成したAuthenticationの名前を指定します。

result_settings:には、書き出す際の詳細な設定項目を指定します。ここに指定する内容は利用するresult_connection（Connectorの種別）により異なります。Data TankはPostgreSQL Connectorを利用しているので、PostgreSQL用の設定を記載します。

result_settings:の設定内容はサポートドキュメントに記載されています。書き出し先に応じてドキュメントを参照して設定してください。例えばPostgreSQL用の設定は以下のページに記載されています。
- https://tddocs.atlassian.net/wiki/spaces/PD/pages/1081813/PostgreSQL+Export+Integration
また、Treasure Dataのベストプラクティス集であるTreasure Boxes上にも設定のサンプルを記載しています。こちらも参考にしてください。
- https://github.com/treasure-data/treasure-boxes/tree/master/td

まとめると、このワークフローは次の2つタスクを順番に実行するためのものです。

- 外部データソース（S3）からのデータロード → +load_from_s3:
- 外部サービス（S3）へのエクスポート →  +export_to_s3:

先ほど触れたように、ワークフローは特別な設定を行わない限りはタスクが完了されるのを待ってから次のタスクを実行するようになっています。そのため、この例のように依存関係のある別々のジョブを記述するのに適しています。

ペーストが完了したら、右上の青い［Save&Commit］ボタンを押してください。編集内容が確定されます。ここで一度ブラウザのワークフローコンソールをリロードすると、画面左側に今作成した「handson_step2」が新規ワークフローとして認識されたことが確認できます。

## データロード定義YAMLの記述

続いて、td_load>オペレータで利用するデータロードの定義ファイル（config/customers.yml）を定義します。先ほどワークフローを追加したときと同様に鉛筆アイコンを押して編集モードを開き、新規ファイルを「config/customers.yml」というファイル名で追加してください。

ファイルの中身には以下をコピー＆ペーストしてください。

```yaml
config/customers.yml
in:
  type: s3
  access_key_id: ${secret:aws.access_key_id}
  secret_access_key: ${secret:aws.secret_access_key}
  bucket: hands-on-seminar
  path_prefix: wf_sample.csv
  parser:
    charset: UTF-8
    newline: CRLF
    type: csv
    delimiter: ","
    quote: "\""
    escape: "\""
    trim_if_not_quoted: false
    skip_header_lines: 1
    allow_extra_columns: false
    allow_optional_columns: false
    columns:
    - {name: member_id, type: long}
    - {name: goods_id, type: long}
    - {name: category, type: string}
    - {name: sub_category, type: string}
    - {name: yamlip_date, type: timestamp, format: "%Y-%m-%d %H:%M:%S.%L"}
    - {name: amount, type: long}
    - {name: price, type: long}
    - {name: time, type: long}
```

YAMLファイルには以下のような情報を記述します。
- 接続先の種別（s3／FTP／MySQLなど）および接続情報
- ファイルの種別（csv／tsv／JSONなど）およびファイル読み込み時の条件指定（クオートの扱いやヘッダ行の扱いなど）
- 読込ファイルに含まれているフィールドの定義

一見すると複雑な内容ですが、これらの内容をゼロから記述することはほとんどありません。先ほども触れたTreasure Boxesに主要な接続先ごとのYAMLのサンプルがあるので、そのつど参考にしてください。
- https://github.com/treasure-data/treasure-boxes/tree/master/td_load

上記の内容をペーストしたら、再び右上の青い［Save&Commit］ボタンを押して編集を確定させてください。

## シークレットの設定

YAMLファイルを見て気づいたかもしれませんが、AWSのアクセスキーとシークレットキーを指定する部分では「${secret:xxx}」という記述をしています。用語定義の項で触れたように、機密情報をプロジェクトファイルに直接記述するのは好ましくないため、Treasure Workflowのシークレットとして機密情報を登録し、プロジェクトファイルでそれを参照するようにしています。
シークレットに値を登録していきましょう。ワークフローの画面から［Secrets］タブを開き、右上の［+］ボタンを押してください。

シークレットの登録画面が開くので、Nameに「aws.access_key_id」、Secret Keyに実際の値を入力します。Nameに登録した値を「${secret:xxx}」の「xxx」の部分として使うことができます。入力が完了したら［Create］ボタンを押してください。

同じ要領で、「aws.secret_access_key」も登録してください。

シークレットは、登録された値を後から読み取ることはできません。クリックしてもマスクされた値が表示されます。入力ミスや、コピー＆ペーストで入力する際に余計な文字が入らないように注意してください。失敗してしまった場合は、該当のシークレットをいったん削除してから再登録してください。
