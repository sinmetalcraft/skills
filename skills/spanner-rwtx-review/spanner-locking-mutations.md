# Spanner Locking & Mutations 共通リファレンス

`spanner-query-plan` と `spanner-rwtx-review` の両スキルから参照する共通知識です。

## DDLの取得

Mutation数の計算やLock範囲の判断には、対象Tableに存在するIndexの定義が必要です。
以下でDatabaseのDDL（`CREATE TABLE` / `CREATE INDEX`）をまとめて取得できます。

```
gcloud spanner databases ddl describe {DB_NAME} --instance={INSTANCE_NAME} --project={PROJECT_ID}
```

出力に含まれる `CREATE INDEX` から、対象Tableに紐づくIndexの数を数えてください。
STORINGしているColumnもMutation数に影響するため、`STORING (...)` 句も確認してください。

## ReaderSharedLock

ReadWriteTransaction内でScanしたRowは、すべてColumn毎にReaderSharedLockを取得します。
**Resultで取得したRowではなく、ScanしたRow**であることに注意してください。

例えば `WHERE Status = 2` のクエリを実行した場合、他のTransactionで `Status = 2` のRowをINSERTしようとすると、Lock待ち状態になります。
`Status = 2` の範囲はすべて最初のTransactionがReaderSharedLockを取得しているので、そのTransactionが終わるまで他のTransactionは待たされます。

実行計画に Residual Condition や Table Full Scan がある場合は特に注意してください。
ScanしたRowはすべてReaderSharedLockを取得するので、広範囲のLockを取ることになります。

## Mutationの数

1 Transactionに入れられるMutationの数は **80,000** です。
1行あたりのMutationの計算は概ね以下の通りです。
STORINGしているColumnなど細かな考慮は入っていないため、80,000ぎりぎりになっている場合は注意が必要です。

* **Insert**: INSERTに含むColumnの数 + INSERTするTableに存在するIndexの数
* **Update**: UPDATEに含むColumnの数 + UPDATEするColumnを含むIndexの数 * 2
* **Delete**: 1 + DELETEするTableに存在するIndexの数
