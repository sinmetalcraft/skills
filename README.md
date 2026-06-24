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
npx skills add sinmetalcraft/skills -a claude-code
```

```bash
# 特定のスキルだけインストール
npx skills add sinmetalcraft/skills -a claude-code --skill spanner-query-plan
```

```bash
# グローバル（~/.claude/skills/）にインストール
npx skills add sinmetalcraft/skills -a claude-code -g
```

インストールするとスキルフォルダがClaude Codeのskillsディレクトリ（プロジェクトなら `.claude/skills/`、グローバル（`-g`）なら `~/.claude/skills/`）に配置され、`SKILL.md` の frontmatter から自動で認識されます。

### バージョン指定

`sinmetalcraft/skills` のように指定すると、デフォルトブランチ（main）の最新が入ります。
特定のバージョンに固定したい場合は、Gitタグを指す `tree/<tag>` 形式のURLを使ってください（最新は `v0.0.1`）。

```bash
# v0.0.1 を固定してインストール
npx skills add https://github.com/sinmetalcraft/skills/tree/v0.0.1 -a claude-code
```

インストール後は入れた時点の内容で固定され、自動では追従しません。最新に更新するには `npx skills update` を実行してください。

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
