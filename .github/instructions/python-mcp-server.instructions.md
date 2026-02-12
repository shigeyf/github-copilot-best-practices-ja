---
description: 'Python SDK を使用した Model Context Protocol (MCP) サーバー構築の指示'
applyTo: '**/*.py, **/pyproject.toml, **/requirements.txt'
---

# Python MCP サーバー開発

## 指示

- プロジェクト管理には **uv** を使用する: `uv init mcp-server-demo` と `uv add "mcp[cli]"`
- FastMCP は `mcp.server.fastmcp` からインポートする: `from mcp.server.fastmcp import FastMCP`
- 登録には `@mcp.tool()`、`@mcp.resource()`、`@mcp.prompt()` デコレーターを使用する
- 型ヒントは必須 - スキーマ生成とバリデーションに使用される
- 構造化出力には Pydantic モデル、TypedDict、または dataclass を使用する
- 戻り値の型が互換性のある場合、ツールは自動的に構造化出力を返す
- stdio トランスポートの場合、`mcp.run()` または `mcp.run(transport="stdio")` を使用する
- HTTP サーバーの場合、`mcp.run(transport="streamable-http")` または Starlette/FastAPI にマウントする
- ツール/リソースで MCP 機能にアクセスするには `Context` パラメータを使用する: `ctx: Context`
- ログ送信には `await ctx.debug()`、`await ctx.info()`、`await ctx.warning()`、`await ctx.error()` を使用する
- 進捗報告には `await ctx.report_progress(progress, total, message)` を使用する
- ユーザー入力の要求には `await ctx.elicit(message, schema)` を使用する
- LLM サンプリングには `await ctx.session.create_message(messages, max_tokens)` を使用する
- サーバー、ツール、リソース、プロンプトのアイコン設定には `Icon(src="path", mimeType="image/png")` を使用する
- 自動画像処理には `Image` クラスを使用する: `return Image(data=bytes, format="png")`
- URI パターンでリソーステンプレートを定義する: `@mcp.resource("greeting://{name}")`
- 部分的な値を受け入れて候補を返すことで補完サポートを実装する
- 共有リソースの起動/シャットダウンには lifespan コンテキストマネージャを使用する
- ツール内で lifespan コンテキストには `ctx.request_context.lifespan_context` でアクセスする
- ステートレス HTTP サーバーの場合、FastMCP 初期化時に `stateless_http=True` を設定する
- モダンクライアント向けに JSON レスポンスを有効化する: `json_response=True`
- サーバーテストには: `uv run mcp dev server.py` (Inspector) または `uv run mcp install server.py` (Claude Desktop) を使用する
- Starlette で異なるパスに複数サーバーをマウントする: `Mount("/path", mcp.streamable_http_app())`
- ブラウザクライアント向けに CORS を設定する: `Mcp-Session-Id` ヘッダーを公開する
- FastMCP が不十分な場合は、最大限の制御のために低レベル Server クラスを使用する

## ベストプラクティス

- 常に型ヒントを使用する - スキーマ生成とバリデーションを駆動する
- 構造化ツール出力には Pydantic モデルまたは TypedDict を返す
- ツール関数は単一責任に集中させる
- 明確な docstring を提供する - ツールの説明になる
- 型ヒント付きの説明的なパラメータ名を使用する
- Pydantic Field の description を使用して入力をバリデーションする
- try-except ブロックで適切なエラーハンドリングを実装する
- I/O バウンド操作には async 関数を使用する
- lifespan コンテキストマネージャでリソースをクリーンアップする
- stdio トランスポートを妨げないよう stderr にログ出力する（stdio 使用時）
- 設定には環境変数を使用する
- LLM 統合前にツールを個別にテストする
- ファイルシステムやネットワークアクセスを公開する際はセキュリティを考慮する
- 機械可読データには構造化出力を使用する
- 後方互換性のためにコンテンツと構造化データの両方を提供する

## 一般的なパターン

### 基本サーバーセットアップ (stdio)

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("My Server")

@mcp.tool()
def calculate(a: int, b: int, op: str) -> int:
    """計算を実行する"""
    if op == "add":
        return a + b
    return a - b

if __name__ == "__main__":
    mcp.run()  # デフォルトで stdio
```

### HTTP サーバー

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("My HTTP Server")

@mcp.tool()
def hello(name: str = "World") -> str:
    """挨拶をする"""
    return f"Hello, {name}!"

if __name__ == "__main__":
    mcp.run(transport="streamable-http")
```

### 構造化出力を持つツール

```python
from pydantic import BaseModel, Field

class WeatherData(BaseModel):
    temperature: float = Field(description="摂氏の気温")
    condition: str
    humidity: float

@mcp.tool()
def get_weather(city: str) -> WeatherData:
    """都市の天気を取得する"""
    return WeatherData(
        temperature=22.5,
        condition="sunny",
        humidity=65.0
    )
```

### 動的リソース

```python
@mcp.resource("users://{user_id}")
def get_user(user_id: str) -> str:
    """ユーザープロフィールデータを取得する"""
    return f"User {user_id} profile data"
```

### Context を持つツール

```python
from mcp.server.fastmcp import Context
from mcp.server.session import ServerSession

@mcp.tool()
async def process_data(
    data: str,
    ctx: Context[ServerSession, None]
) -> str:
    """ログ付きでデータを処理する"""
    await ctx.info(f"Processing: {data}")
    await ctx.report_progress(0.5, 1.0, "半分完了")
    return f"Processed: {data}"
```

### サンプリングを持つツール

```python
from mcp.server.fastmcp import Context
from mcp.server.session import ServerSession
from mcp.types import SamplingMessage, TextContent

@mcp.tool()
async def summarize(
    text: str,
    ctx: Context[ServerSession, None]
) -> str:
    """LLM を使用してテキストを要約する"""
    result = await ctx.session.create_message(
        messages=[SamplingMessage(
            role="user",
            content=TextContent(type="text", text=f"Summarize: {text}")
        )],
        max_tokens=100
    )
    return result.content.text if result.content.type == "text" else ""
```

### ライフスパン管理

```python
from contextlib import asynccontextmanager
from dataclasses import dataclass
from mcp.server.fastmcp import FastMCP, Context

@dataclass
class AppContext:
    db: Database

@asynccontextmanager
async def app_lifespan(server: FastMCP):
    db = await Database.connect()
    try:
        yield AppContext(db=db)
    finally:
        await db.disconnect()

mcp = FastMCP("My App", lifespan=app_lifespan)

@mcp.tool()
def query(sql: str, ctx: Context) -> str:
    """データベースをクエリする"""
    db = ctx.request_context.lifespan_context.db
    return db.execute(sql)
```

### メッセージを持つプロンプト

```python
from mcp.server.fastmcp.prompts import base

@mcp.prompt(title="Code Review")
def review_code(code: str) -> list[base.Message]:
    """コードレビュープロンプトを作成する"""
    return [
        base.UserMessage("このコードをレビューしてください:"),
        base.UserMessage(code),
        base.AssistantMessage("コードをレビューします。")
    ]
```

### エラーハンドリング

```python
@mcp.tool()
async def risky_operation(input: str) -> str:
    """失敗する可能性のある操作"""
    try:
        result = await perform_operation(input)
        return f"Success: {result}"
    except Exception as e:
        return f"Error: {str(e)}"
```
