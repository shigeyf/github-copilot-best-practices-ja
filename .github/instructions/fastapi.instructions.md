---
description: "FastAPI 開発標準とベストプラクティス"
applyTo: "**/*.py, **/pyproject.toml, **/requirements.txt"
---

# FastAPI 開発指示

[https://fastapi.tiangolo.com](https://fastapi.tiangolo.com) の公式 FastAPI ドキュメントに従い、モダンなパターン、非同期処理、ベストプラクティスを用いた高品質な FastAPI アプリケーション構築のための指示です。

## プロジェクトコンテキスト

- 最新の FastAPI バージョン（0.100+）
- 型安全性のための Pydantic v2
- 非同期処理（async/await）をデフォルトとして使用
- Python 3.11+ の最新機能を活用
- プロジェクト管理には **uv** を推奨: `uv init` と `uv add fastapi[standard]`

## 開発標準

### アーキテクチャ

- レイヤードアーキテクチャを採用（Router → Service → Repository）
- 機能またはドメインごとにモジュールを整理
- 依存性注入（Dependency Injection）パターンを活用
- ビジネスロジックをエンドポイントから分離

```python
# 推奨するプロジェクト構成
app/
├── main.py              # アプリケーションエントリーポイント
├── core/
│   ├── config.py        # 設定管理
│   ├── security.py      # 認証・認可
│   └── dependencies.py  # 共通依存関係
├── api/
│   ├── v1/
│   │   ├── endpoints/   # ルーターモジュール
│   │   └── router.py    # API v1 ルーター集約
│   └── deps.py          # API 依存関係
├── models/              # SQLAlchemy モデル
├── schemas/             # Pydantic スキーマ
├── services/            # ビジネスロジック
├── repositories/        # データアクセス層
└── tests/               # テストファイル
```

### Pydantic スキーマ設計

- リクエスト/レスポンスには必ず Pydantic モデルを使用
- 入力スキーマと出力スキーマを分離
- Pydantic v2 の `model_config` を使用した設定

```python
from pydantic import BaseModel, Field, ConfigDict
from datetime import datetime

class UserBase(BaseModel):
    """ユーザーの基本スキーマ"""
    email: str = Field(..., description="メールアドレス", examples=["user@example.com"])
    name: str = Field(..., min_length=1, max_length=100, description="ユーザー名")

class UserCreate(UserBase):
    """ユーザー作成リクエストスキーマ"""
    password: str = Field(..., min_length=8, description="パスワード")

class UserResponse(UserBase):
    """ユーザーレスポンススキーマ"""
    id: int
    created_at: datetime

    model_config = ConfigDict(from_attributes=True)
```

### エンドポイント設計

- RESTful な命名規則に従う
- 適切な HTTP メソッドとステータスコードを使用
- パスパラメータ、クエリパラメータ、ボディを明確に区別
- レスポンスモデルを必ず指定

```python
from fastapi import APIRouter, Depends, HTTPException, status, Query, Path
from typing import Annotated

router = APIRouter(prefix="/users", tags=["users"])

@router.get(
    "/{user_id}",
    response_model=UserResponse,
    summary="ユーザー取得",
    description="指定された ID のユーザー情報を取得します。",
)
async def get_user(
    user_id: Annotated[int, Path(description="ユーザー ID", ge=1)],
    service: Annotated[UserService, Depends(get_user_service)],
) -> UserResponse:
    """ユーザー情報を取得する"""
    user = await service.get_by_id(user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="ユーザーが見つかりません",
        )
    return user

@router.get(
    "",
    response_model=list[UserResponse],
    summary="ユーザー一覧取得",
)
async def list_users(
    skip: Annotated[int, Query(ge=0, description="スキップ件数")] = 0,
    limit: Annotated[int, Query(ge=1, le=100, description="取得件数")] = 20,
    service: Annotated[UserService, Depends(get_user_service)],
) -> list[UserResponse]:
    """ユーザー一覧を取得する"""
    return await service.list(skip=skip, limit=limit)
```

### 依存性注入

- `Depends()` を使用して依存関係を明示
- `Annotated` 型ヒントで可読性を向上
- 共通の依存関係は再利用可能な形で定義
- ライフサイクル管理には `lifespan` を使用

```python
from fastapi import Depends
from typing import Annotated, AsyncGenerator
from contextlib import asynccontextmanager
from sqlalchemy.ext.asyncio import AsyncSession

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    """データベースセッションを提供する依存関係"""
    async with async_session_maker() as session:
        try:
            yield session
        finally:
            await session.close()

# 型エイリアスを使用して可読性を向上
DbSession = Annotated[AsyncSession, Depends(get_db)]

async def get_current_user(
    token: Annotated[str, Depends(oauth2_scheme)],
    db: DbSession,
) -> User:
    """現在のユーザーを取得する依存関係"""
    user = await authenticate_user(token, db)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="認証に失敗しました",
            headers={"WWW-Authenticate": "Bearer"},
        )
    return user

CurrentUser = Annotated[User, Depends(get_current_user)]
```

### 非同期処理

- I/O バウンド操作には `async def` を使用
- CPU バウンド操作には `run_in_executor` を検討
- データベース操作には非同期 ORM（SQLAlchemy 2.0 async）を使用
- 外部 API 呼び出しには `httpx` の非同期クライアントを使用

```python
import httpx
from functools import lru_cache

@lru_cache
def get_http_client() -> httpx.AsyncClient:
    """HTTP クライアントのシングルトンインスタンス"""
    return httpx.AsyncClient(timeout=30.0)

@router.get("/external-data")
async def fetch_external_data() -> dict:
    """外部 API からデータを取得する"""
    client = get_http_client()
    response = await client.get("https://api.example.com/data")
    response.raise_for_status()
    return response.json()
```

### エラーハンドリング

- カスタム例外クラスを定義
- グローバル例外ハンドラーを実装
- 適切なエラーレスポンススキーマを定義
- ログ出力を含める

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
import logging

logger = logging.getLogger(__name__)

class AppException(Exception):
    """アプリケーション基底例外"""
    def __init__(self, message: str, status_code: int = 400):
        self.message = message
        self.status_code = status_code
        super().__init__(self.message)

class NotFoundError(AppException):
    """リソースが見つからない例外"""
    def __init__(self, resource: str, id: int | str):
        super().__init__(
            message=f"{resource}（ID: {id}）が見つかりません",
            status_code=404,
        )

@app.exception_handler(AppException)
async def app_exception_handler(request: Request, exc: AppException) -> JSONResponse:
    """アプリケーション例外ハンドラー"""
    logger.warning(f"AppException: {exc.message}", extra={"path": request.url.path})
    return JSONResponse(
        status_code=exc.status_code,
        content={"detail": exc.message},
    )

@app.exception_handler(Exception)
async def general_exception_handler(request: Request, exc: Exception) -> JSONResponse:
    """一般例外ハンドラー"""
    logger.exception("Unhandled exception", extra={"path": request.url.path})
    return JSONResponse(
        status_code=500,
        content={"detail": "内部サーバーエラーが発生しました"},
    )
```

### バリデーション

- Pydantic の Field バリデーションを活用
- カスタムバリデーターを実装
- Query/Path パラメータにも型制約を適用

```python
from pydantic import BaseModel, Field, field_validator, model_validator
import re

class UserCreate(BaseModel):
    """ユーザー作成スキーマ"""
    email: str = Field(..., description="メールアドレス")
    password: str = Field(..., min_length=8, description="パスワード")
    password_confirm: str = Field(..., description="パスワード確認")

    @field_validator("email")
    @classmethod
    def validate_email(cls, v: str) -> str:
        """メールアドレスの形式を検証する"""
        pattern = r"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$"
        if not re.match(pattern, v):
            raise ValueError("有効なメールアドレスを入力してください")
        return v.lower()

    @model_validator(mode="after")
    def validate_passwords_match(self) -> "UserCreate":
        """パスワードの一致を検証する"""
        if self.password != self.password_confirm:
            raise ValueError("パスワードが一致しません")
        return self
```

### セキュリティ

- OAuth2/JWT 認証を実装
- CORS を適切に設定
- レート制限を実装
- 入力のサニタイズを行う

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

# CORS 設定
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com"],  # 本番環境では具体的なオリジンを指定
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["*"],
)

# OAuth2 スキーム
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/v1/auth/token")
```

### 設定管理

- pydantic-settings を使用して環境変数を管理
- 環境ごとの設定を分離
- シークレットは環境変数から読み込む

```python
from pydantic_settings import BaseSettings, SettingsConfigDict
from functools import lru_cache

class Settings(BaseSettings):
    """アプリケーション設定"""
    app_name: str = "My FastAPI App"
    debug: bool = False
    database_url: str
    secret_key: str
    access_token_expire_minutes: int = 30

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
    )

@lru_cache
def get_settings() -> Settings:
    """設定のシングルトンインスタンスを取得する"""
    return Settings()
```

### データベース統合

- SQLAlchemy 2.0 の非同期モードを使用
- Alembic でマイグレーションを管理
- トランザクション管理を適切に実装

```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase

class Base(DeclarativeBase):
    """SQLAlchemy ベースクラス"""
    pass

engine = create_async_engine(
    settings.database_url,
    echo=settings.debug,
    pool_pre_ping=True,
)

async_session_maker = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,
)
```

### ライフサイクル管理

- `lifespan` コンテキストマネージャを使用
- 起動時の初期化とシャットダウン時のクリーンアップを実装

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    """アプリケーションライフサイクル管理"""
    # 起動時の処理
    await init_database()
    logger.info("アプリケーションを起動しました")

    yield

    # シャットダウン時の処理
    await close_database()
    logger.info("アプリケーションをシャットダウンしました")

app = FastAPI(lifespan=lifespan)
```

### テスト

- pytest と pytest-asyncio を使用
- HTTPX の AsyncClient でエンドポイントをテスト
- テスト用のデータベースを分離
- ファクトリーパターンでテストデータを生成

```python
import pytest
from httpx import AsyncClient, ASGITransport
from sqlalchemy.ext.asyncio import AsyncSession

@pytest.fixture
async def client(app: FastAPI) -> AsyncGenerator[AsyncClient, None]:
    """テスト用 HTTP クライアント"""
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as ac:
        yield ac

@pytest.mark.asyncio
async def test_create_user(client: AsyncClient, db_session: AsyncSession) -> None:
    """ユーザー作成エンドポイントのテスト"""
    response = await client.post(
        "/api/v1/users",
        json={
            "email": "test@example.com",
            "name": "Test User",
            "password": "securepassword123",
        },
    )
    assert response.status_code == 201
    data = response.json()
    assert data["email"] == "test@example.com"
```

### ドキュメント

- OpenAPI スキーマを充実させる
- エンドポイントには summary と description を記述
- レスポンス例を提供

```python
from fastapi import FastAPI

app = FastAPI(
    title="My API",
    description="サンプル FastAPI アプリケーション",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc",
    openapi_tags=[
        {"name": "users", "description": "ユーザー管理 API"},
        {"name": "auth", "description": "認証 API"},
    ],
)
```

### ロギング

- 構造化ログを使用（JSON 形式推奨）
- リクエスト ID でトレーサビリティを確保
- ログレベルを環境ごとに設定

```python
import logging
import uuid
from fastapi import Request

@app.middleware("http")
async def add_request_id(request: Request, call_next):
    """リクエスト ID をログに追加するミドルウェア"""
    request_id = str(uuid.uuid4())
    logger = logging.getLogger("app")

    with logger.contextvars.bind(request_id=request_id):
        response = await call_next(request)
        response.headers["X-Request-ID"] = request_id
        return response
```

## バックグラウンドタスク

- 軽量なタスクには `BackgroundTasks` を使用
- 重いタスクには Celery や ARQ を検討

```python
from fastapi import BackgroundTasks

async def send_notification(email: str, message: str) -> None:
    """通知を送信するバックグラウンドタスク"""
    # メール送信処理
    await email_service.send(email, message)

@router.post("/notify")
async def notify_user(
    email: str,
    message: str,
    background_tasks: BackgroundTasks,
) -> dict:
    """ユーザーに通知を送信する"""
    background_tasks.add_task(send_notification, email, message)
    return {"status": "通知をキューに追加しました"}
```

## パフォーマンス最適化

- レスポンスキャッシュを実装
- データベースクエリを最適化（N+1 問題を回避）
- 適切なインデックスを設定
- コネクションプーリングを活用

## 実装プロセス

1. プロジェクト構成とアーキテクチャを計画
2. `uv init` でプロジェクトをセットアップ
3. Pydantic スキーマを定義
4. データベースモデルを実装
5. リポジトリ層を実装
6. サービス層にビジネスロジックを実装
7. API エンドポイントを実装
8. 認証・認可を追加
9. エラーハンドリングを実装
10. テストを作成
11. ドキュメントを充実させる
12. パフォーマンスを最適化

## 追加ガイドライン

- PEP 8 と PEP 257 に従う
- 型ヒントを必ず使用する
- 非同期処理を優先する
- 依存性注入を活用する
- テストカバレッジを維持する
- セキュリティを常に考慮する
- ドキュメントを最新に保つ
