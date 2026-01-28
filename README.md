# GitHub Copilot ベストプラクティス (日本語)

GitHub Copilot
の構成ファイルのテンプレートとベストプラクティスを提供するリポジトリです。

## ディレクトリ構成

```text
your-repository/
│
├── .github/
│   ├── copilot-instructions.md            # リポジトリ全体のカスタム指示 (必須)
│   ├── instructions/                      # 固有のカスタム指示
│   │   └── *.instructions.md
│   ├── prompts/                           # 再利用可能プロンプト (VS Code)
│   │   └── *.prompt.md
│   ├── agents/                            # カスタムエージェント (VS Code)
│   │   └── *.agent.md
│   └── workflows/                         # GitHub Actions
│
├── .claude/
│   └── skills/                            # プロジェクトスキル (推奨)
│       └── <skill-name>/
│           └── SKILL.md
│
├── .vscode/
│   ├── settings.json                      # VS Code / Copilot 設定
│   └── mcp.json                           # MCP サーバー設定
│
├── AGENTS.md                              # エージェント指示
├── CLAUDE.md                              # Claude 互換指示 (オプション)
└── GEMINI.md                              # Gemini 互換指示 (オプション)
```

## 各ファイルの説明

### GitHub.com / Coding Agent / CLI 共通

| ファイル                                 | 説明                                     | 対象                       |
| ---------------------------------------- | ---------------------------------------- | -------------------------- |
| `.github/copilot-instructions.md`        | リポジトリ全体に適用される基本指示       | 全 Copilot 機能            |
| `.github/instructions/*.instructions.md` | 特定モジュールや特定パスに適用される指示 | Coding Agent, Code Review  |
| `.claude/skills/*/SKILL.md`              | タスク固有のスキル定義                   | Coding agent, CLI, VS Code |
| `AGENTS.md`                              | エージェント向け指示 (階層継承)          | AI エージェント全般        |

### VS Code 専用

| ファイル                      | 説明                               | 対象                 |
| ----------------------------- | ---------------------------------- | -------------------- |
| `.github/prompts/*.prompt.md` | 再利用可能なプロンプトテンプレート | VS Code Copilot Chat |
| `.github/agents/*.agent.md`   | カスタムエージェント定義           | VS Code Copilot Chat |
| `.vscode/mcp.json`            | MCP サーバー連携設定               | VS Code              |

## スキル vs カスタム指示 の使い分け

| 種類             | 用途                         | ロードタイミング |
| ---------------- | ---------------------------- | ---------------- |
| **カスタム指示** | コーディング規約、基本ルール | 常に適用         |
| **スキル**       | 特定タスクの詳細手順         | 関連時のみ       |

**推奨**: スキルは `.claude/skills/` に統一することで、GitHub Copilot と Claude
Code の両方で利用可能

## Copilot Instructions について

このリポジトリの `.github/copilot-instructions.md` には、GitHub Copilot Coding Agent に対する包括的な指示が定義されています。

### 主な指示内容

#### 基本原則
- **言語設定**: すべての出力・応答は日本語で記述
- **例外**: コード内の識別子（変数名、関数名など）は英語を使用

#### Coding Agent 向け指示
- **Git コミットと PR**: 日本語でのコミットメッセージ、PR タイトル・説明文の記述規則
  - コミットメッセージ: `[Copilot]` プレフィックスを付与
  - PR タイトル: `[Copilot]` プレフィックスを付与、Draft PR の場合は `[WIP]` を維持
- **作業プロセス**: タスク理解、コードベース調査、計画共有、小単位の変更、動作確認、完了報告

#### コード生成向け指示
- **コメントとドキュメント**: 日本語でのコメント記述、コーディング規約の遵守
- **コード品質**: 可読性、エラーハンドリング、コーディングスタイルの統一
- **テスト**: テストカバレッジの確保、テスト品質の維持
- **セキュリティ**: 機密情報の保護、セキュアコーディング
- **パフォーマンス**: 効率的なコードの実装

#### Copilot Chat 向け指示
- **対話形式**: 日本語での対話、分かりやすい技術説明
- **情報提供**: 明確で簡潔な回答、具体例の提供、ベストプラクティスの推奨
- **コードレビュー**: 建設的なフィードバック、多角的なレビュー観点

#### 禁止事項
- 機密情報の漏洩
- 著作権侵害
- 悪意のあるコード生成
- 既存機能の不必要な破壊

#### ベストプラクティス
- 段階的な変更
- ドキュメント優先
- 継続的な改善
- 積極的なコミュニケーション

これらの指示により、Copilot は一貫性のある高品質なコード生成とレビューを実現します。

## ファイルフォーマット

### カスタム指示 (`.github/copilot-instructions.md`)

```markdown
# GitHub Copilot Instructions

## 言語設定

すべての応答は日本語で記述すること。

## コーディング規約

- 変数名・関数名は英語で記述
- コメントは日本語で記述

## ビルド・テスト

- ビルド: `npm run build`
- テスト: `npm test`
```

### パス固有指示 (`.github/instructions/*.instructions.md`)

```markdown
---
applyTo: "**/*.py"
---

# Python コーディング規約

- Type hints を必須とする
- docstring は Google スタイル
```

### スキル (`.claude/skills/*/SKILL.md`)

```markdown
---
name: github-actions-debugging
description: GitHub Actions のワークフロー失敗をデバッグする手順
---

# デバッグ手順

1. 失敗したワークフローを確認
2. エラーログを取得
3. 修正して再実行
```

### プロンプトファイル (`.github/prompts/*.prompt.md`) - VS Code

```markdown
---
description: コードレビューを実行
agent: agent
tools: ["search", "read_file"]
---

# コードレビュー

以下の観点でレビューしてください：

1. バグの可能性
2. セキュリティリスク
3. パフォーマンス

対象: ${selection}
```

### カスタムエージェント (`.github/agents/*.agent.md`) - VS Code

```markdown
---
name: planner
description: 実装計画を生成
tools:
    - search
    - fetch
    - githubRepo
model: Claude Sonnet 4
---

# Planning Agent

実装計画を生成します。コードの変更は行いません。
```

## 参考リンク

- [GitHub Docs: Adding repository custom instructions](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot)
- [GitHub Docs: About Agent Skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills)
- [VS Code: Customize chat to your workflow](https://code.visualstudio.com/docs/copilot/copilot-customization)
- [VS Code: Prompt files](https://code.visualstudio.com/docs/copilot/customization/prompt-files)
- [VS Code: Custom agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [GitHub: awesome-copilot](https://github.com/github/awesome-copilot)

## ライセンス

Apache 2.0 License

See [LICENSE](LICENSE) for more information.
