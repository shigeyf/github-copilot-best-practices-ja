---
description: "TypeScript SDK を使用した Model Context Protocol (MCP) サーバー構築の指示"
applyTo: "**/*.ts, **/*.js, **/package.json"
---

# TypeScript MCP サーバー開発

## 指示

- **@modelcontextprotocol/sdk** npm パッケージを使用: `npm install @modelcontextprotocol/sdk`
- 特定のパスからインポート: `@modelcontextprotocol/sdk/server/mcp.js`、`@modelcontextprotocol/sdk/server/stdio.js` など
- 自動プロトコル処理を備えた高レベルサーバー実装には `McpServer` クラスを使用
- 手動リクエストハンドラーによる低レベル制御には `Server` クラスを使用
- 入力/出力スキーマ検証には **zod** を使用: `npm install zod@3`
- より良い UI 表示のため、ツール、リソース、プロンプトには常に `title` フィールドを提供
- `registerTool()`、`registerResource()`、`registerPrompt()` メソッドを使用（古い API より推奨）
- zod を使用してスキーマを定義: `{ inputSchema: { param: z.string() }, outputSchema: { result: z.string() } }`
- ツールからは `content`（表示用）と `structuredContent`（構造化データ用）の両方を返す
- HTTP サーバーには、Express または類似のフレームワークと共に `StreamableHTTPServerTransport` を使用
- ローカル統合には、stdio ベースの通信用に `StdioServerTransport` を使用
- リクエスト ID の衝突を防ぐため、リクエストごとに新しいトランスポートインスタンスを作成（ステートレスモード）
- ステートフルサーバーには `sessionIdGenerator` によるセッション管理を使用
- ローカルサーバーでは DNS リバインディング保護を有効化: `enableDnsRebindingProtection: true`
- ブラウザベースのクライアント向けに CORS ヘッダーを設定し、`Mcp-Session-Id` を公開
- URI パラメータを持つ動的リソースには `ResourceTemplate` を使用: `new ResourceTemplate('resource://{param}', { list: undefined })`
- より良い UX のため、`@modelcontextprotocol/sdk/server/completable.js` の `completable()` ラッパーを使用して補完をサポート
- クライアントから LLM 補完を要求するには `server.server.createMessage()` でサンプリングを実装
- ツール実行中に追加のユーザー入力を要求するには `server.server.elicitInput()` を使用
- 一括更新には通知のデバウンスを有効化: `debouncedNotificationMethods: ['notifications/tools/list_changed']`
- 動的更新: 登録済みアイテムに対して `.enable()`、`.disable()`、`.update()`、`.remove()` を呼び出して `listChanged` 通知を発行
- UI 表示名には `@modelcontextprotocol/sdk/shared/metadataUtils.js` の `getDisplayName()` を使用
- MCP Inspector でサーバーをテスト: `npx @modelcontextprotocol/inspector`

## ベストプラクティス

- ツールの実装は単一責任に焦点を当てる
- LLM が理解しやすいよう、明確で説明的なタイトルと説明を提供
- すべてのパラメータと戻り値に適切な TypeScript 型を使用
- try-catch ブロックで包括的なエラー処理を実装
- エラー状態ではツール結果に `isError: true` を返す
- すべての非同期操作に async/await を使用
- データベース接続を適切に閉じ、リソースを適切にクリーンアップ
- 処理前に入力パラメータを検証
- stdout/stderr を汚染しない構造化ログをデバッグに使用
- ファイルシステムやネットワークアクセスを公開する際のセキュリティ影響を考慮
- トランスポートのクローズイベントで適切なリソースクリーンアップを実装
- 設定には環境変数を使用（ポート、API キーなど）
- ツールの機能と制限を明確にドキュメント化
- 互換性を確保するため複数のクライアントでテスト

## 共通パターン

### 基本サーバー設定（HTTP）

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import express from "express";

const server = new McpServer({
    name: "my-server",
    version: "1.0.0",
});

const app = express();
app.use(express.json());

app.post("/mcp", async (req, res) => {
    const transport = new StreamableHTTPServerTransport({
        sessionIdGenerator: undefined,
        enableJsonResponse: true,
    });

    res.on("close", () => transport.close());

    await server.connect(transport);
    await transport.handleRequest(req, res, req.body);
});

app.listen(3000);
```

### 基本サーバー設定（stdio）

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new McpServer({
    name: "my-server",
    version: "1.0.0",
});

// ... ツール、リソース、プロンプトを登録 ...

const transport = new StdioServerTransport();
await server.connect(transport);
```

### シンプルなツール

```typescript
import { z } from "zod";

server.registerTool(
    "calculate",
    {
        title: "Calculator",
        description: "基本的な計算を実行",
        inputSchema: {
            a: z.number(),
            b: z.number(),
            op: z.enum(["+", "-", "*", "/"]),
        },
        outputSchema: { result: z.number() },
    },
    async ({ a, b, op }) => {
        const result = op === "+"
            ? a + b
            : op === "-"
            ? a - b
            : op === "*"
            ? a * b
            : a / b;
        const output = { result };
        return {
            content: [{ type: "text", text: JSON.stringify(output) }],
            structuredContent: output,
        };
    },
);
```

### 動的リソース

```typescript
import { ResourceTemplate } from "@modelcontextprotocol/sdk/server/mcp.js";

server.registerResource(
    "user",
    new ResourceTemplate("users://{userId}", { list: undefined }),
    {
        title: "User Profile",
        description: "ユーザープロファイルデータを取得",
    },
    async (uri, { userId }) => ({
        contents: [{
            uri: uri.href,
            text: `User ${userId} data here`,
        }],
    }),
);
```

### サンプリング付きツール

```typescript
server.registerTool(
    "summarize",
    {
        title: "Text Summarizer",
        description: "LLM を使用してテキストを要約",
        inputSchema: { text: z.string() },
        outputSchema: { summary: z.string() },
    },
    async ({ text }) => {
        const response = await server.server.createMessage({
            messages: [{
                role: "user",
                content: { type: "text", text: `Summarize: ${text}` },
            }],
            maxTokens: 500,
        });

        const summary = response.content.type === "text"
            ? response.content.text
            : "要約できませんでした";
        const output = { summary };
        return {
            content: [{ type: "text", text: JSON.stringify(output) }],
            structuredContent: output,
        };
    },
);
```

### 補完付きプロンプト

```typescript
import { completable } from "@modelcontextprotocol/sdk/server/completable.js";

server.registerPrompt(
    "review",
    {
        title: "Code Review",
        description: "特定の観点でコードをレビュー",
        argsSchema: {
            language: completable(
                z.string(),
                (value) =>
                    ["typescript", "python", "javascript", "java"]
                        .filter((l) => l.startsWith(value)),
            ),
            code: z.string(),
        },
    },
    ({ language, code }) => ({
        messages: [{
            role: "user",
            content: {
                type: "text",
                text: `Review this ${language} code:\n\n${code}`,
            },
        }],
    }),
);
```

### エラー処理

```typescript
server.registerTool(
    "risky-operation",
    {
        title: "Risky Operation",
        description: "失敗する可能性のある操作",
        inputSchema: { input: z.string() },
        outputSchema: { result: z.string() },
    },
    async ({ input }) => {
        try {
            const result = await performRiskyOperation(input);
            const output = { result };
            return {
                content: [{ type: "text", text: JSON.stringify(output) }],
                structuredContent: output,
            };
        } catch (err: unknown) {
            const error = err as Error;
            return {
                content: [{ type: "text", text: `Error: ${error.message}` }],
                isError: true,
            };
        }
    },
);
```
