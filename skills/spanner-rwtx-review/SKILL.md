---
name: spanner-rwtx-review
description: Cloud SpannerのReadWriteTransactionを使うコードをレビューし、ReaderSharedLockの範囲・Mutation数・冪等性を確認する。「このトランザクションをレビューして」「ロックの範囲が心配」「リトライで問題ないか見て」といった依頼に使う。
---

# Spanner ReadWriteTransactionの処理のレビュー

ユーザの言語に合わせて回答してください。ユーザが英語で質問してきた場合は英語で、日本語で質問してきた場合は日本語で回答します。

## ReaderSharedLockの範囲

ReadWriteTransaction内でScanしたRowはすべてReaderSharedLockを取得します。
Resultで取得したRowではなく、ScanしたRowであることに注意してください。

Transaction内で実行するQueryに以下がないか確認してください。

* WHEREの指定がPrimary Keyではないクエリ → 想定外に広い範囲をLockしていないか
* 実行計画に Residual Condition や Table Full Scan があるクエリ → 広範囲のLockを取る

実行計画を確認したい場合は `spanner-query-plan` スキルを使ってください。
ReaderSharedLockの具体例（phantom）は [spanner-locking-mutations.md](./spanner-locking-mutations.md) を参照してください。

## Mutationの数

複数行を更新する場合、1 Transactionに入れられるMutationの数（80,000）に気を付けてください。
Mutation数の計算式とIndex数を数えるためのDDL取得方法は [spanner-locking-mutations.md](./spanner-locking-mutations.md) を参照してください。

## 冪等性

ReadWriteTransactionの関数内は冪等になるようにしてください。
TransactionがAbortされた場合、Client Libraryの中でRetryが自動的に走るようになっています。
そのため、Transaction関数の外への副作用（カウンタのインクリメント、外部APIの呼び出し、メッセージ送信など）がRetryで二重に実行されないか確認してください。
