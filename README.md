# skills

Cloud Spannerまわりの作業を支援するAgent Skills集です。

## Skills

| Skill | 用途 |
| --- | --- |
| `spanner-query-plan` | SQLの実行計画（`EXPLAIN` / `EXPLAIN ANALYZE`）を取得し、負荷の高い箇所やLock範囲を分析する |
| `spanner-rwtx-review` | ReadWriteTransactionを使うコードをレビューし、ReaderSharedLockの範囲・Mutation数・冪等性を確認する |

## Install

[`skills` CLI](https://github.com/vercel-labs/skills) でインストールできます。

```bash
# 対話的にスキルを選んでインストール
npx skills add sinmetalcraft/skills
```

```bash
# 特定のスキルだけインストール
npx skills add sinmetalcraft/skills --skill spanner-query-plan
```

```bash
# グローバル（~/.claude/skills/）にインストール
npx skills add sinmetalcraft/skills -g
```

インストールするとスキルフォルダが各エージェントのskillsディレクトリ（Claude Codeなら `.claude/skills/` または `~/.claude/skills/`）に配置され、`SKILL.md` の frontmatter から自動で認識されます。

---

以下はこのリポジトリの開発者向けの情報です。

## ディレクトリ構成

```
skills/
├── spanner-query-plan/
│   ├── SKILL.md
│   └── spanner-locking-mutations.md   # 共通リファレンス（同梱）
└── spanner-rwtx-review/
    ├── SKILL.md
    └── spanner-locking-mutations.md   # 共通リファレンス（同梱）
```

`spanner-locking-mutations.md` は ReaderSharedLock / Mutation数の計算 / DDLの取得方法をまとめた共通リファレンスです。

`skills` CLI のインストール単位は**スキルフォルダ単位**で、フォルダ外のファイルは一緒に配置されません。そのため共通リファレンスは各スキルフォルダ内に**意図的に複製**しています。

> **メンテナンス注意**: `spanner-locking-mutations.md` を更新するときは、両方のフォルダのファイルを同じ内容に揃えてください。
