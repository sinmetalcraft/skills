# Spanner Skills

## Queryの実行計画の分析

Queryの実行計画を取得する方法はいくつかあります。ユーザの環境次第なので、使えるものを使ってください。
ProjectIDやInstanceなどは環境変数に設定があればそれを使っていいか、ユーザに確認してください。
ユーザから事前に指定があれば、それを使ってください。

* CLOUDSDK_CORE_PROJECT
* CLOUDSDK_SPANNER_INSTANCE
* CLOUDSDK_SPANNER_DATABSE

SQLはユーザからチャットで貰ってもいいし、SQLファイルのpathが渡された場合はそのファイルを読み込んでください。
実行計画を取得する場合は、SQLの先頭に `EXPLAIN` を付ける必要があります。
SQLの先頭に `EXPLAIN` がない場合は、追加して実行してください。

以下はコマンドのサンプルです。

```
gcloud spanner cli {DB_NAME} --instance={INSTANCE_NAME} --project={PROJECT_ID} --execute "$(cat sample.sql)"
```

```
spanner-cli -p {PROJECT_ID} -i {INSTANCE_NAME} -d {DB_NAME} -e 'SELECT * FROM users;' -t
```

実行計画を取得できたら、それをユーザに必ず返してください。
そして、負荷が高そうな箇所があれば、解説してください。

UPDATE文やDELETE文の分析を依頼された場合、特に以下に注意してください。
WHEREの指定がPrimary Keyではない場合、Lockの範囲が思っている範囲になっているか確認してください。
SpannerはReadWriteTransactionの中で読み込んだRowをColumn毎にReaderSharedLockを取得します。
更新した行、削除した行ではなく、読み込んだ行です。

複数行を更新する場合、Mutationの数にも気を付けてください。
1 Transactionに入れれるMutationの数は80000です。
1行あたりのMutationの計算は概ね以下の通りですが、STORINGしているColumnなど細かな考慮は入っていないため、80000ぎりぎりになっている場合は注意が必要です。

Insert
INSERTに含むColumnの数 + INSERTするTableに存在するIndexの数
Update
UPDATEに含むColumnの数 + UPDATEするColumnを含むIndexの数 * 2
Delete
1 + DELETEするTableに存在するIndexの数

## ReadWriteTransactionの処理のレビュー

ReadWriteTransaction内でScanしたRowはすべてReaderSharedLockを取得することに注意してください。
Resultで取得したRowではなく、ScanしたRowであることに注意してください。
例えばWHERE Status = 2のクエリを実行した場合、他のTransactionでStatus = 2のRowをINSERTしようとすると、Lock待ち状態になります。
Status = 2の範囲はすべて最初のTransactionがReaderSharedLockを取得しているので、そのTransactionが終わるまで、他のTransactionは待たされます。
ReadWriteTransaction内で実行するQueryの実行計画にResidual ConditionやTable Full Scanがある場合も注意してください。
ScanしたRowはすべてReaderSharedLockを取得するので、広範囲のLockを取ることになります。

複数行を更新する場合、Mutationの数にも気を付けてください。
1 Transactionに入れれるMutationの数は80000です。
1行あたりのMutationの計算は概ね以下の通りですが、STORINGしているColumnなど細かな考慮は入っていないため、80000ぎりぎりになっている場合はちゅういが必要です。

Insert
INSERTに含むColumnの数 + INSERTするTableに存在するIndexの数
Update
UPDATEに含むColumnの数 + UPDATEするColumnを含むIndexの数 * 2
Delete
1 + DELETEするTableに存在するIndexの数

ReadWriteTransactionの関数内は冪等になるようにしてください。
TransactionがAbortされた場合、Client Libraryの中でRetryが自動的に走るようになっています。