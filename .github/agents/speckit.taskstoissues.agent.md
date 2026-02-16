---
description: 既存のタスクを設計成果物に基づいて、依存関係順に整理されたアクション可能な GitHub Issue に変換します
tools: ['github/issue_write']
---

# `/speckit.taskstoissues` エージェント

## ユーザー入力

```text
$ARGUMENTS
```

続行する前に、ユーザー入力を**必ず**確認してください (空でない場合)。

## ルール

**Copilot 既定の指示の適用**:

- `.github/copilot-instructions.md` に記述されたルールを適用すること。

## 実行ステップ

以下の実行ステップに従ってください:

1. リポジトリルートから `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` を実行し、FEATURE_DIR と AVAILABLE_DOCS リストを解析します。すべてのパスは絶対パスである必要があります。引数内のシングルクォートは、エスケープ構文を使用してください (例: "I'm Groot" → 'I'\''m Groot'、またはダブルクォートを使用: "I'm Groot")。
1. 実行したスクリプトから **tasks** へのパスを抽出します。
1. 以下のコマンドで Git リモートを取得します：

```bash
git config --get remote.origin.url
```

> [!CAUTION]
> リモートが GitHub URL の場合のみ次のステップに進んでください

1. リスト内の各タスクについて、GitHub MCP サーバーを使用して、Git リモートに対応するリポジトリに新しい Issue を作成します。

> [!CAUTION]
> リモート URL と一致しないリポジトリには絶対に Issue を作成しないでください
