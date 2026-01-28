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
