---
name: spanner-query-plan
description: Cloud SpannerのSQLクエリの実行計画(EXPLAIN / EXPLAIN ANALYZE)を取得し、負荷の高い箇所やLock範囲を分析する。「実行計画を見て」「このSQLは重い？」「ロックの範囲を確認したい」といった依頼に使う。
---

# Spanner Queryの実行計画の分析

ユーザの言語に合わせて回答してください。ユーザが英語で質問してきた場合は英語で、日本語で質問してきた場合は日本語で回答します。

## 接続情報の決定

ProjectIDやInstanceなどは以下の順で決定してください。

1. ユーザから事前に指定があれば、それを使う
2. 以下の環境変数に設定があれば、それを使っていいかユーザに確認する

* `CLOUDSDK_CORE_PROJECT`
* `CLOUDSDK_SPANNER_INSTANCE`
* `CLOUDSDK_SPANNER_DATABASE`

## SQLの取得

SQLはユーザからチャットで貰ってもいいし、SQLファイルのpathが渡された場合はそのファイルを読み込んでください。

## EXPLAIN と EXPLAIN ANALYZE

実行計画を取得するには、SQLの先頭に `EXPLAIN` または `EXPLAIN ANALYZE` を付ける必要があります。先頭に付いていない場合は追加して実行してください。

* `EXPLAIN`: 推定の実行計画のみを取得する。クエリは実行されない。
* `EXPLAIN ANALYZE`: 実際にクエリを実行し、実測の統計（Scanした行数など）を含む実行計画を取得する。

Lockの範囲は実際にScanした行数に依存するため、可能なら `EXPLAIN ANALYZE` で実測を取る方が精度が上がります。
ただし `EXPLAIN ANALYZE` はクエリを実際に実行する点に注意してください。**UPDATE文やDELETE文では実際にデータが変更されます。** 破壊的になる場合は `EXPLAIN`（推定のみ）に留めるか、ユーザに確認してください。

## CLIの実行

実行計画を取得するCLIは複数あり、どれが使えるかはユーザの環境次第です。
ユーザから指定があればそれを優先し、なければ環境にインストールされているものを使ってください（`gcloud spanner cli --help` や `spanner-cli --help` で利用可否を確認できます）。

`gcloud spanner cli`（gcloud標準）:

```
gcloud spanner cli {DB_NAME} --instance={INSTANCE_NAME} --project={PROJECT_ID} --execute "{SQL}"
```

`spanner-cli`（cloudspannerecosystem製のサードパーティツール）:

```
spanner-cli -p {PROJECT_ID} -i {INSTANCE_NAME} -d {DB_NAME} -e '{SQL}' -t
```

## 結果の報告

実行計画を取得できたら、それをユーザに必ず返してください。
そして、負荷が高そうな箇所（Table Full Scan、Residual Condition、広範囲のScanなど）があれば解説してください。

## UPDATE文 / DELETE文の分析

UPDATE文やDELETE文の分析を依頼された場合、特に以下に注意してください。

* WHEREの指定がPrimary Keyではない場合、Lockの範囲が想定通りか確認してください。SpannerはReadWriteTransactionの中でScanしたRowをColumn毎にReaderSharedLockで取得します（更新・削除した行ではなく、読み込んだ行です）。
* 複数行を更新・削除する場合、1 Transactionに入れられるMutationの数（80,000）に気を付けてください。

ReaderSharedLockの詳細とMutation数の計算式、Index数を数えるためのDDL取得方法は [spanner-locking-mutations.md](./spanner-locking-mutations.md) を参照してください。
