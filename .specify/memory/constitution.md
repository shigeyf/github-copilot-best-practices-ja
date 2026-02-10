# [PROJECT_NAME] Constitution

<!-- 例: Spec 憲章、TaskFlow 憲章など -->

## 基本原則

### [PRINCIPLE_1_NAME]

<!-- 例: I. ライブラリファースト -->

[PRINCIPLE_1_DESCRIPTION]

<!--
例:
すべての機能はスタンドアロンライブラリとして開始する;
ライブラリは自己完結型で、独立してテスト可能、ドキュメント化されている必要がある;
明確な目的が必要 - 組織上の理由だけのライブラリは不可
-->

### [PRINCIPLE_2_NAME]

<!-- 例: II. CLI インターフェース -->

[PRINCIPLE_2_DESCRIPTION]

<!--
例:
すべてのライブラリは CLI 経由で機能を公開する;
テキスト入出力プロトコル: stdin/args → stdout、エラー → stderr;
JSON と人間が読める形式をサポート
-->

### [PRINCIPLE_3_NAME]

<!-- 例: III. テストファースト (絶対条件) -->

[PRINCIPLE_3_DESCRIPTION]

<!--
例:
TDD 必須: テストを書く → ユーザー承認 → テスト失敗 → 実装;
Red-Green-Refactor サイクルを厳格に遵守
-->

### [PRINCIPLE_4_NAME]

<!-- 例: IV. 統合テスト -->

[PRINCIPLE_4_DESCRIPTION]

<!--
例:
統合テストが必要な重点領域:
新規ライブラリのコントラクトテスト、コントラクト変更、
サービス間通信、共有スキーマ
-->

### [PRINCIPLE_5_NAME]

<!--
例:
V. 可観測性、
VI. バージョニングと破壊的変更、
VII. シンプルさ
-->

[PRINCIPLE_5_DESCRIPTION]

<!--
例: テキスト I/O によりデバッグ性を確保;
構造化ログ必須;
または: MAJOR.MINOR.BUILD 形式;
または: シンプルに始める、YAGNI 原則
-->

## [SECTION_2_NAME]

<!--
例:
追加制約、セキュリティ要件、パフォーマンス基準など
-->

[SECTION_2_CONTENT]

<!--
例:
技術スタック要件、コンプライアンス基準、デプロイポリシーなど
-->

## [SECTION_3_NAME]

<!--
例:
開発ワークフロー、レビュープロセス、品質ゲートなど
-->

[SECTION_3_CONTENT]

<!--
例: コードレビュー要件、テストゲート、デプロイ承認プロセスなど
-->

## ガバナンス

<!--
例:
憲章は他のすべてのプラクティスに優先する;
修正にはドキュメント化、承認、移行計画が必要
-->

[GOVERNANCE_RULES]

<!--
例:
すべての PR/レビューはコンプライアンスを検証する必要がある;
複雑さには正当な理由が必要;
ランタイム開発ガイダンスには [GUIDANCE_FILE] を使用
-->

**バージョン**: [CONSTITUTION_VERSION] | **承認日**: [RATIFICATION_DATE] | **最終更新日**: [LAST_AMENDED_DATE]

<!-- 例: バージョン: 2.1.1 | 承認日: 2025-06-13 | 最終更新日: 2025-07-16 -->
