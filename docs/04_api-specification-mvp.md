# foodllm API仕様書（MVP版）

## 1. ドキュメント情報

| 項目 | 内容 |
|------|------|
| ドキュメント名 | API仕様書（MVP版） |
| プロジェクト名 | foodllm 対話型LLMアプリケーション |
| バージョン | 1.8 |
| 作成日 | 2025-01-02 |
| APIバージョン | v1 |

---

## 2. API概要

### 2.1 基本情報

| 項目 | 内容 |
|------|------|
| ベースURL | `https://{function-app-name}.azurewebsites.net/api/v1` |
| プロトコル | HTTPS |
| 認証方式 | API Key（ヘッダー）、将来：Azure AD B2C |
| レスポンス形式 | JSON |
| 文字コード | UTF-8 |

### 2.2 共通ヘッダー

| ヘッダー名 | 必須 | 説明 |
|-----------|------|------|
| Content-Type | ○ | `application/json` |
| X-API-Key | △ | API認証キー（通常API用） |
| X-Admin-API-Key | △ | 管理者API認証キー（管理者API用） |
| X-Request-ID | - | リクエスト追跡用ID（UUID） |
| X-Use-Case | - | ユースケースID（デフォルト: hondashi） |

> **認証ヘッダーの使い分け**
> - 通常API（Chat, Recipe, Product, Feedback）: `X-API-Key` を使用
> - 管理者API（Admin）: `X-Admin-API-Key` を使用（`X-API-Key` では認証不可）

### 2.3 共通レスポンス形式

#### 成功時

```json
{
  "success": true,
  "data": { ... },
  "metadata": {
    "request_id": "uuid",
    "timestamp": "2025-01-02T12:00:00Z",
    "processing_time_ms": 150
  }
}
```

#### エラー時

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "エラーメッセージ",
    "details": { ... }
  },
  "metadata": {
    "request_id": "uuid",
    "timestamp": "2025-01-02T12:00:00Z"
  }
}
```

### 2.4 エラーコード一覧

| コード | HTTPステータス | 説明 | ユーザー向けメッセージ |
|--------|---------------|------|----------------------|
| INVALID_REQUEST | 400 | リクエスト形式が不正 | 入力内容に誤りがあります。内容を確認してください。 |
| VALIDATION_ERROR | 422 | バリデーションエラー | 入力内容が正しくありません。{field}を確認してください。 |
| UNAUTHORIZED | 401 | 認証エラー | 認証に失敗しました。再度お試しください。 |
| FORBIDDEN | 403 | アクセス権限なし | この操作を行う権限がありません。 |
| NOT_FOUND | 404 | リソースが見つからない | お探しの情報が見つかりませんでした。 |
| RATE_LIMIT_EXCEEDED | 429 | レート制限超過 | リクエストが多すぎます。しばらく待ってからお試しください。 |
| INTERNAL_ERROR | 500 | 内部エラー | システムエラーが発生しました。しばらく待ってからお試しください。 |
| SERVICE_UNAVAILABLE | 503 | サービス一時停止 | 現在メンテナンス中です。しばらくお待ちください。 |
| LLM_TIMEOUT | 504 | LLM応答タイムアウト | 回答の生成に時間がかかっています。再度お試しください。 |
| LLM_ERROR | 502 | LLMエラー | 回答の生成中にエラーが発生しました。再度お試しください。 |
| SESSION_EXPIRED | 440 | セッション期限切れ | セッションの有効期限が切れました。ページを再読み込みしてください。 |
| MESSAGE_LIMIT_EXCEEDED | 429 | 会話長制限超過 | 会話が長くなりすぎました。新しい会話を始めてください。 |

---

## 2.5 CORS設定

| 設定項目 | 値 |
|---------|-----|
| 許可オリジン | `https://{static-web-app}.azurestaticapps.net`, `https://localhost:3000`（開発用） |
| 許可メソッド | GET, POST, DELETE, OPTIONS |
| 許可ヘッダー | Content-Type, X-API-Key, **X-Admin-API-Key**, X-Request-ID, X-Use-Case |
| 公開ヘッダー | X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset |
| プリフライトキャッシュ | 86400秒（24時間） |
| 認証情報 | 許可しない（credentials: false） |

> **管理者API**: ブラウザベースの管理画面から呼び出す場合は `X-Admin-API-Key` がCORSプリフライトで許可される必要があります。サーバー間通信のみの場合はこの設定は不要です。

---

## 2.6 API Key管理

### 2.6.1 API Keyの種類

| 種類 | 用途 | 権限 | ヘッダー | 有効期限 |
|------|------|------|---------|---------|
| 通常API Key | フロントエンドからのアクセス | Chat, Recipe, Product, Feedback | `X-API-Key` | 無期限（ローテーション推奨90日） |
| 管理者API Key | 管理者APIアクセス | 上記 + Admin API | `X-Admin-API-Key` | 30日（自動失効） |

> **認証の区別**: 管理者APIは `X-Admin-API-Key` ヘッダーを使用します。`X-API-Key` では管理者APIにアクセスできません。

### 2.6.2 API Keyライフサイクル

**発行手順**
1. Azure Key Vaultで新規シークレット作成
2. API Management（Consumption）のサブスクリプションに登録
3. 利用者に安全な方法で配布

**更新手順**
1. 新しいAPI Keyを発行
2. 移行期間中は旧Keyも有効（7日間）
3. 旧Key失効

**失効手順**
1. Azure Key Vaultでシークレット無効化
2. API Managementのサブスクリプションから削除
3. アクセスログで不正利用確認

---

## 3. API エンドポイント一覧（MVP）

| メソッド | エンドポイント | 説明 | MVP | Phase 2 |
|---------|---------------|------|-----|---------|
| **POST** | **/auth/verify** | **MVPアクセスパスワード検証** | **✅** | **削除予定** |
| POST | /chat/messages | メッセージ送信・応答取得 | ✅ | |
| POST | /chat/messages/stream | ストリーミング応答取得 | ✅ | |
| POST | /conversations | 新規会話作成 | ✅ | |
| GET | /recipes/search | レシピ検索 | ✅ | |
| GET | /recipes/{id} | レシピ詳細取得 | ✅ | |
| GET | /products/search | 製品検索 | ✅ | |
| GET | /products/{id} | 製品詳細取得 | ✅ | |
| GET | /products/{id}/faq | 製品FAQ取得 | ✅ | |
| GET | /faq/search | FAQ検索 | ✅ | |
| POST | /feedback | フィードバック送信 | ✅ | |
| GET | /health | ヘルスチェック | ✅ | |
| GET | /admin/conversations | 会話一覧取得（管理者用） | ✅ | |
| GET | /admin/conversations/{id} | 会話詳細取得（管理者用） | ✅ | |
| GET | /conversations | 会話一覧取得（ユーザー用） | - | ✅ |
| GET | /conversations/{id} | 会話詳細取得（ユーザー用） | - | ✅ |
| DELETE | /conversations/{id} | 会話削除 | - | ✅ |

---

## 3a. MVPアクセス認証API（MVP期間限定）

> **注記**: このAPIはMVP期間の限定公開用です。正式公開時に削除予定です。

### 3a.1 POST /auth/verify

アクセスパスワードを検証し、認証トークンを発行する。

**リクエスト**

```http
POST /api/v1/auth/verify
Content-Type: application/json
```

```json
{
  "password": "string"
}
```

| フィールド | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| password | string | ○ | アクセスパスワード |

**レスポンス（成功: 200 OK）**

```json
{
  "success": true,
  "token": "mvp_xxxxxxxxxxxxx",
  "expires_at": "2025-01-05T12:00:00Z"
}
```

| フィールド | 型 | 説明 |
|-----------|-----|------|
| success | boolean | 認証成功フラグ |
| token | string | 認証トークン（sessionStorageに保存） |
| expires_at | string | トークン有効期限（ISO 8601形式） |

**レスポンス（失敗: 401 Unauthorized）**

```json
{
  "success": false,
  "error": {
    "code": "INVALID_PASSWORD",
    "message": "パスワードが正しくありません",
    "remaining_attempts": 3
  }
}
```

| フィールド | 型 | 説明 |
|-----------|-----|------|
| success | boolean | 認証失敗フラグ |
| error.code | string | エラーコード |
| error.message | string | エラーメッセージ |
| error.remaining_attempts | number | 残り試行回数 |

**レスポンス（ロックアウト: 429 Too Many Requests）**

```json
{
  "success": false,
  "error": {
    "code": "LOCKED_OUT",
    "message": "試行回数の上限に達しました",
    "retry_after": 60
  }
}
```

| フィールド | 型 | 説明 |
|-----------|-----|------|
| error.retry_after | number | 再試行可能までの秒数 |

**セキュリティ仕様**

| 項目 | 設定値 |
|------|--------|
| 最大試行回数 | 5回 |
| ロックアウト時間 | 60秒 |
| トークン有効期間 | ブラウザセッション（タブ閉じまで） |
| パスワード保存 | 環境変数（`MVP_ACCESS_PASSWORD_HASH`）にSHA-256ハッシュで保存 |

**実装例（FastAPI）**

```python
# api/routers/auth.py
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel
import hashlib
import secrets
import os
from datetime import datetime, timedelta

router = APIRouter(prefix="/api/v1/auth", tags=["auth"])

# 試行回数管理（本番ではRedis等を使用）
attempt_tracker: dict[str, dict] = {}

class VerifyRequest(BaseModel):
    password: str

class VerifyResponse(BaseModel):
    success: bool
    token: str | None = None
    expires_at: str | None = None
    error: dict | None = None

@router.post("/verify", response_model=VerifyResponse)
async def verify_password(request: VerifyRequest, client_ip: str = None):
    # ロックアウトチェック
    if client_ip in attempt_tracker:
        tracker = attempt_tracker[client_ip]
        if tracker.get("locked_until") and datetime.now() < tracker["locked_until"]:
            retry_after = int((tracker["locked_until"] - datetime.now()).total_seconds())
            raise HTTPException(
                status_code=429,
                detail={
                    "success": False,
                    "error": {
                        "code": "LOCKED_OUT",
                        "message": "試行回数の上限に達しました",
                        "retry_after": retry_after
                    }
                }
            )
    
    # パスワード検証
    password_hash = hashlib.sha256(request.password.encode()).hexdigest()
    expected_hash = os.getenv("MVP_ACCESS_PASSWORD_HASH")
    
    if password_hash == expected_hash:
        # 認証成功
        token = f"mvp_{secrets.token_urlsafe(32)}"
        expires_at = datetime.now() + timedelta(hours=24)
        
        # 試行回数リセット
        if client_ip in attempt_tracker:
            del attempt_tracker[client_ip]
        
        return VerifyResponse(
            success=True,
            token=token,
            expires_at=expires_at.isoformat()
        )
    else:
        # 認証失敗
        if client_ip not in attempt_tracker:
            attempt_tracker[client_ip] = {"attempts": 0, "locked_until": None}
        
        attempt_tracker[client_ip]["attempts"] += 1
        remaining = 5 - attempt_tracker[client_ip]["attempts"]
        
        if remaining <= 0:
            attempt_tracker[client_ip]["locked_until"] = datetime.now() + timedelta(seconds=60)
            raise HTTPException(
                status_code=429,
                detail={
                    "success": False,
                    "error": {
                        "code": "LOCKED_OUT",
                        "message": "試行回数の上限に達しました",
                        "retry_after": 60
                    }
                }
            )
        
        raise HTTPException(
            status_code=401,
            detail={
                "success": False,
                "error": {
                    "code": "INVALID_PASSWORD",
                    "message": "パスワードが正しくありません",
                    "remaining_attempts": remaining
                }
            }
        )
```

**環境変数設定**

```bash
# パスワードのハッシュ値を生成（例: パスワードが "hondashi2025" の場合）
echo -n "hondashi2025" | sha256sum
# 結果: a1b2c3d4e5f6...（このハッシュ値を環境変数に設定）

# 環境変数
MVP_ACCESS_PASSWORD_HASH=a1b2c3d4e5f6...
```

---

## 4. API詳細仕様

### 4.1 チャットAPI

#### 4.1.1 POST /chat/messages

メッセージを送信し、AI応答を取得する。

**リクエスト**

```http
POST /api/v1/chat/messages
Content-Type: application/json
X-API-Key: {api-key}
```

```json
{
  "session_id": "session_123456",
  "message": "ほんだしの保存方法を教えてください"
}
```

| パラメータ | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| session_id | string | - | セッションID（未指定時は新規作成） |
| message | string | ○ | ユーザーメッセージ（1〜1000文字） |

**session_id の扱い**
- **未指定の場合**: 新規セッション・会話が自動作成され、レスポンスで `session_id` が返却される
- **指定の場合**: 既存セッション内で会話が継続される
- **推奨フロー**: 初回は `POST /conversations` でセッション取得 → 以降は `POST /chat/messages` で送信

> **POST /conversations との使い分け**
> - `POST /conversations`: ウェルカムメッセージを取得したい場合、明示的に新規会話を開始する場合
> - `POST /chat/messages` (session_id未指定): 即座にメッセージを送信し、セッションも同時に作成する場合

**レスポンス（成功）**

```json
{
  "success": true,
  "data": {
    "session_id": "session_123456",
    "conversation_id": "conv_789012",
    "message_id": "msg_345678",
    "response": {
      "content": "ほんだしの保存方法についてお答えします。\n\n開封後は湿気を避けて、密閉容器に入れて冷暗所で保存してください。冷蔵庫での保存も可能です。",
      "intent": "faq_answered",
      "suggestions": [
        {
          "id": "sug_001",
          "label": "賞味期限について",
          "value": "ほんだしの賞味期限はどのくらいですか？",
          "type": "text"
        },
        {
          "id": "sug_002",
          "label": "使い方を知りたい",
          "value": "ほんだしの使い方を教えてください",
          "type": "text"
        },
        {
          "id": "sug_003",
          "label": "レシピを見る",
          "value": "ほんだしを使ったレシピを教えてください",
          "type": "text"
        }
      ],
      "faqs": [
        {
          "id": "faq_001",
          "question": "ほんだしの保存方法は？",
          "answer": "開封後は湿気を避けて、密閉容器に入れて冷暗所で保存してください。",
          "category": "保存方法",
          "product_id": "prod_001"
        }
      ],
      "recipe": null,
      "products": []
    },
    "usage": {
      "prompt_tokens": 150,
      "completion_tokens": 120,
      "total_tokens": 270
    }
  },
  "metadata": {
    "request_id": "req_abc123",
    "timestamp": "2025-01-02T12:00:00Z",
    "processing_time_ms": 1200
  }
}
```

**レスポンスフィールド**

| フィールド | 型 | 説明 |
|-----------|-----|------|
| session_id | string | セッションID |
| conversation_id | string | 会話ID（管理用） |
| message_id | string | メッセージID |
| response.content | string | AI応答テキスト |
| response.intent | string | 検出された意図 |
| response.suggestions | array | サジェスト配列（0〜4件） |
| response.faqs | array | FAQ情報配列（0〜複数件） |
| response.recipe | object | レシピ情報（該当時） |
| response.products | array | 製品情報配列（該当時） |
| usage | object | トークン使用量 |

**インテント一覧（統一定義）**

| intent値 | 説明 | 使用フェーズ |
|----------|------|-------------|
| `recipe_search` | レシピを探している | 入力意図分類 |
| `recipe_provided` | レシピを提供した | **APIレスポンス** |
| `product_info` | 製品情報を提供 | 入力意図分類 / APIレスポンス |
| `faq_question` | FAQ質問をしている | 入力意図分類 |
| `faq_answered` | FAQに基づいて回答した | **APIレスポンス** |
| `general_chat` | 一般的な会話 | 入力意図分類 / APIレスポンス |
| `out_of_scope` | 対応範囲外 | 入力意図分類 / APIレスポンス |

> **入力意図分類 vs APIレスポンス**
> - **入力意図分類**: ユーザーメッセージを受信した時点で分類（内部処理用）
> - **APIレスポンス**: AI応答と共に返却されるintent（クライアント表示用）
> - 例: `recipe_search`（入力意図） → RAG検索 → `recipe_provided`（レスポンス）

**サジェスト（suggestions）の詳細**

| フィールド | 型 | 説明 |
|-----------|-----|------|
| id | string | サジェストID |
| label | string | 表示ラベル（ボタンに表示） |
| value | string | 送信値（typeがtextの場合） |
| type | string | `text` または `action` |
| icon | string | アイコン名（オプション） |

**type の振る舞い**

| type | クライアント処理 | サーバー処理 |
|------|----------------|-------------|
| `text` | `value`をメッセージとして`POST /chat/messages`に送信 | 通常のメッセージとして処理 |
| `action` | `value`に応じた画面遷移・モーダル表示を実行 | 送信不要（クライアント完結） |

**action の value 一覧**

| value | 動作 |
|-------|------|
| `show_recipe:{recipe_id}` | レシピ詳細モーダルを表示 |
| `show_product:{product_id}` | 製品詳細モーダルを表示 |
| `new_conversation` | 新規会話を開始 |
| `open_external:{url}` | 外部リンクを新規タブで開く |

---

#### 4.1.2 POST /chat/messages/stream

ストリーミング形式でAI応答を取得する（Server-Sent Events）。

**リクエスト**

```http
POST /api/v1/chat/messages/stream
Content-Type: application/json
Accept: text/event-stream
X-API-Key: {api-key}
```

```json
{
  "session_id": "session_123456",
  "message": "味噌汁の作り方を教えてください"
}
```

**レスポンス（SSE）**

```
event: message_start
data: {"session_id":"session_123456","conversation_id":"conv_789012","message_id":"msg_345678"}

event: content_delta
data: {"delta":"基本の"}

event: content_delta
data: {"delta":"味噌汁の"}

event: content_delta
data: {"delta":"レシピを..."}

event: message_complete
data: {"intent":"recipe_provided","suggestions":[...],"recipe":{...}}

event: done
data: {"usage":{"total_tokens":350}}
```

**イベントタイプ**

| イベント | 説明 |
|---------|------|
| message_start | メッセージ開始 |
| content_delta | コンテンツ差分 |
| message_complete | メッセージ完了（メタデータ含む） |
| error | エラー発生 |
| done | ストリーム終了 |
| keep_alive | 接続維持用ハートビート |

**SSE接続仕様**

| 項目 | 値 | 説明 |
|------|-----|------|
| 接続タイムアウト | 60秒 | LLM応答の最大待機時間 |
| Keep-Alive間隔 | 15秒 | `:keep_alive`イベント送信間隔 |
| 最大接続時間 | 5分 | 1回のストリーミング接続の最大時間 |
| 再接続待機 | 1秒→2秒→4秒 | 指数バックオフ（最大3回） |

**再接続ポリシー**
1. ネットワークエラー発生時、クライアントは自動再接続を試行
2. 再接続時は`message_id`を送信し、途中から再開
3. 3回再接続失敗で通常API（非ストリーミング）にフォールバック
4. `error`イベント受信時は再接続せず、エラー表示

**再接続時のパラメータ送信方法**

| 方法 | パラメータ | 説明 |
|------|-----------|------|
| HTTPヘッダー | `Last-Event-ID` | 最後に受信したイベントのID（SSE標準） |
| クエリパラメータ | `resume_from` | 再開位置のmessage_id |

```http
POST /api/v1/chat/messages/stream?resume_from=msg_123456
Content-Type: application/json
Accept: text/event-stream
X-API-Key: {api-key}
Last-Event-ID: evt_789
```

**再接続時のサーバー挙動**

| 条件 | 挙動 |
|------|------|
| `resume_from`が有効 | 指定位置から未送信のコンテンツを再送 |
| `resume_from`が無効/期限切れ | 最初から再生成（新規message_id発行） |
| 再接続上限（3回）超過 | 429エラーを返却、通常APIへのフォールバックを推奨 |

---

### 4.2 会話API

#### 4.2.1 POST /conversations

新規会話を作成する。

**リクエスト**

```http
POST /api/v1/conversations
Content-Type: application/json
X-API-Key: {api-key}
```

```json
{
  "use_case_id": "hondashi"
}
```

| パラメータ | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| use_case_id | string | - | ユースケースID（デフォルト: hondashi） |

**レスポンス**

```json
{
  "success": true,
  "data": {
    "session_id": "session_789012",
    "conversation_id": "conv_456789",
    "use_case_id": "hondashi",
    "started_at": "2025-01-02T12:00:00Z",
    "welcome_message": {
      "content": "こんにちは！ほんだしに関するご質問なら何でもお気軽にどうぞ。\n\n和食やだし料理のレシピ、ほんだしの使い方など、お手伝いします。",
      "suggestions": [
        {"id": "sug_001", "label": "レシピを探す", "value": "ほんだしを使ったレシピを教えてください", "type": "text"},
        {"id": "sug_002", "label": "使い方を知りたい", "value": "ほんだしの基本的な使い方を教えてください", "type": "text"},
        {"id": "sug_003", "label": "製品について", "value": "ほんだしの製品ラインナップを教えてください", "type": "text"},
        {"id": "sug_004", "label": "よくある質問", "value": "ほんだしについてよくある質問を教えてください", "type": "text"}
      ]
    }
  }
}
```

---

### 4.3 製品FAQ API

#### 4.3.1 GET /products/{id}/faq

指定した製品のFAQを取得する。

**リクエスト**

```http
GET /api/v1/products/prod_001/faq?category=保存方法&limit=5
X-API-Key: {api-key}
```

| パラメータ | 型 | 必須 | デフォルト | 説明 |
|-----------|-----|------|-----------|------|
| category | string | - | - | カテゴリフィルタ |
| limit | integer | - | 5 | 取得件数（1〜20）※画面表示は最大5件 |

**レスポンス**

```json
{
  "success": true,
  "data": {
    "product_id": "prod_001",
    "product_name": "ほんだし",
    "faqs": [
      {
        "id": "faq_001",
        "question": "ほんだしの保存方法は？",
        "answer": "開封後は湿気を避けて、密閉容器に入れて冷暗所で保存してください。冷蔵庫での保存も可能です。",
        "category": "保存方法",
        "priority": 1
      },
      {
        "id": "faq_002",
        "question": "ほんだしにアレルギー物質は含まれていますか？",
        "answer": "ほんだしには、小麦・乳成分が含まれています。アレルギーをお持ちの方はご注意ください。",
        "category": "アレルギー",
        "priority": 1
      },
      {
        "id": "faq_003",
        "question": "ほんだしの1回の使用量の目安は？",
        "answer": "味噌汁の場合、水400ml（2人分）に対して小さじ1（4g）が目安です。",
        "category": "使い方",
        "priority": 1
      }
    ],
    "total": 3
  }
}
```

---

#### 4.3.2 GET /faq/search

FAQを検索する。

**リクエスト**

```http
GET /api/v1/faq/search?q=アレルギー&product_id=prod_001&limit=5
X-API-Key: {api-key}
```

| パラメータ | 型 | 必須 | デフォルト | 説明 |
|-----------|-----|------|-----------|------|
| q | string | ○ | - | 検索クエリ |
| product_id | string | - | - | 製品IDフィルタ |
| category | string | - | - | カテゴリフィルタ |
| limit | integer | - | 5 | 取得件数（1〜20） |
| use_case | string | - | hondashi | ユースケース |

**レスポンス**

```json
{
  "success": true,
  "data": {
    "faqs": [
      {
        "id": "faq_002",
        "product_id": "prod_001",
        "product_name": "ほんだし",
        "question": "ほんだしにアレルギー物質は含まれていますか？",
        "answer": "ほんだしには、小麦・乳成分が含まれています。アレルギーをお持ちの方はご注意ください。詳しくは製品パッケージの表示をご確認ください。",
        "category": "アレルギー",
        "relevance_score": 0.95
      }
    ],
    "total": 1
  }
}
```

---

### 4.4 管理者用API

#### 4.4.1 GET /admin/conversations

会話一覧を取得する（管理者用）。

**リクエスト**

```http
GET /api/v1/admin/conversations?limit=20&offset=0&from=2025-01-01&to=2025-01-31
X-Admin-API-Key: {admin-api-key}
```

| パラメータ | 型 | 必須 | デフォルト | 説明 |
|-----------|-----|------|-----------|------|
| limit | integer | - | 20 | 取得件数（1〜100） |
| offset | integer | - | 0 | オフセット |
| from | date | - | - | 開始日フィルタ |
| to | date | - | - | 終了日フィルタ |
| use_case | string | - | - | ユースケースフィルタ |

**レスポンス**

```json
{
  "success": true,
  "data": {
    "conversations": [
      {
        "id": "conv_123456",
        "session_id": "session_789012",
        "use_case_id": "hondashi",
        "message_count": 5,
        "first_message": "ほんだしの保存方法を教えて",
        "started_at": "2025-01-02T10:00:00Z",
        "ended_at": "2025-01-02T10:15:00Z"
      }
    ],
    "pagination": {
      "total": 150,
      "limit": 20,
      "offset": 0,
      "has_more": true
    }
  }
}
```

---

#### 4.4.2 GET /admin/conversations/{id}

会話詳細（メッセージ履歴含む）を取得する（管理者用）。

**リクエスト**

```http
GET /api/v1/admin/conversations/conv_123456
X-Admin-API-Key: {admin-api-key}
```

**レスポンス**

```json
{
  "success": true,
  "data": {
    "id": "conv_123456",
    "session_id": "session_789012",
    "use_case_id": "hondashi",
    "started_at": "2025-01-02T10:00:00Z",
    "ended_at": "2025-01-02T10:15:00Z",
    "messages": [
      {
        "id": "msg_001",
        "role": "assistant",
        "content": "こんにちは！ほんだしに関するご質問なら何でもお気軽にどうぞ。",
        "created_at": "2025-01-02T10:00:00Z",
        "suggestions": [...]
      },
      {
        "id": "msg_002",
        "role": "user",
        "content": "ほんだしの保存方法を教えてください",
        "created_at": "2025-01-02T10:00:30Z"
      },
      {
        "id": "msg_003",
        "role": "assistant",
        "content": "ほんだしの保存方法についてお答えします...",
        "created_at": "2025-01-02T10:00:33Z",
        "intent": "faq_answered",
        "faqs": [
          {
            "id": "faq_001",
            "question": "ほんだしの保存方法は？",
            "category": "保存方法"
          }
        ],
        "suggestions": [...],
        "usage": {
          "prompt_tokens": 100,
          "completion_tokens": 80,
          "total_tokens": 180
        },
        "response_time_ms": 1200
      }
    ],
    "feedback": [
      {
        "message_id": "msg_003",
        "rating": "positive",
        "created_at": "2025-01-02T10:01:00Z"
      }
    ],
    "total_tokens": 450
  }
}
```

---

### 4.5 レシピAPI

#### 4.5.1 GET /recipes/search

レシピを検索する。

**リクエスト**

```http
GET /api/v1/recipes/search?q=味噌汁&tags=簡単&limit=10
X-API-Key: {api-key}
```

| パラメータ | 型 | 必須 | デフォルト | 説明 |
|-----------|-----|------|-----------|------|
| q | string | ○ | - | 検索クエリ |
| tags | string | - | - | タグフィルタ（カンマ区切り） |
| difficulty | string | - | - | 難易度フィルタ |
| cooking_time_max | integer | - | - | 最大調理時間（分） |
| limit | integer | - | 10 | 取得件数（1〜50） |
| offset | integer | - | 0 | オフセット |
| use_case | string | - | hondashi | ユースケース |

**レスポンス**

```json
{
  "success": true,
  "data": {
    "recipes": [
      {
        "id": "recipe_001",
        "title": "基本の味噌汁",
        "description": "ほんだしを使った定番の味噌汁",
        "image_url": "https://example.com/images/misoshiru.jpg",
        "cooking_time": 10,
        "difficulty": "easy",
        "servings": 2,
        "tags": ["定番", "汁物", "簡単"],
        "rating": 4.5,
        "relevance_score": 0.95
      }
    ],
    "pagination": {
      "total": 25,
      "limit": 10,
      "offset": 0,
      "has_more": true
    }
  }
}
```

---

#### 4.5.2 GET /recipes/{id}

レシピ詳細を取得する。

**リクエスト**

```http
GET /api/v1/recipes/recipe_001
X-API-Key: {api-key}
```

**レスポンス**

```json
{
  "success": true,
  "data": {
    "id": "recipe_001",
    "title": "基本の味噌汁",
    "description": "ほんだしを使った定番の味噌汁。だしの香りが豊かで、毎日食べても飽きない味わいです。",
    "image_url": "https://example.com/images/misoshiru.jpg",
    "cooking_time": 10,
    "difficulty": "easy",
    "servings": 2,
    "tags": ["定番", "汁物", "簡単"],
    "ingredients": [
      {
        "name": "ほんだし",
        "amount": "小さじ1",
        "category": "調味料",
        "notes": null
      },
      {
        "name": "味噌",
        "amount": "大さじ1.5",
        "category": "調味料",
        "notes": "お好みの味噌で"
      },
      {
        "name": "豆腐",
        "amount": "1/4丁",
        "category": "具材",
        "notes": "絹ごしでも木綿でも"
      },
      {
        "name": "わかめ",
        "amount": "適量",
        "category": "具材",
        "notes": "乾燥わかめの場合は戻しておく"
      },
      {
        "name": "水",
        "amount": "400ml",
        "category": "その他",
        "notes": null
      }
    ],
    "steps": [
      {
        "order": 1,
        "instruction": "鍋に水400mlとほんだしを入れ、火にかける",
        "tips": "中火でゆっくり温めると香りが立ちます"
      },
      {
        "order": 2,
        "instruction": "沸騰したら、さいの目に切った豆腐を加える",
        "tips": null
      },
      {
        "order": 3,
        "instruction": "火を弱めて、味噌を溶き入れる",
        "tips": "味噌は沸騰させすぎると風味が飛びます"
      },
      {
        "order": 4,
        "instruction": "わかめを加えて軽く温めたら完成",
        "tips": null
      }
    ],
    "nutrition": {
      "per_serving": {
        "calories": 45,
        "protein": 3.2,
        "fat": 1.5,
        "carbohydrates": 4.8,
        "sodium": 580
      }
    },
    "tips": [
      "具材は季節の野菜に変えても美味しいです",
      "ほんだしの量はお好みで調整してください"
    ],
    "related_products": [
      {
        "id": "prod_001",
        "name": "ほんだし",
        "image_url": "https://example.com/images/hondashi.jpg"
      }
    ],
    "source_url": "https://park.ajinomoto.co.jp/recipe/...",
    "created_at": "2025-01-01T00:00:00Z",
    "updated_at": "2025-01-01T00:00:00Z"
  }
}
```

※ 関連レシピ（related_recipes）はPhase 2で実装

---

### 4.6 製品API

#### 4.6.1 GET /products/search

製品を検索する。

**リクエスト**

```http
GET /api/v1/products/search?q=ほんだし&category=だし調味料
X-API-Key: {api-key}
```

| パラメータ | 型 | 必須 | デフォルト | 説明 |
|-----------|-----|------|-----------|------|
| q | string | - | - | 検索クエリ |
| category | string | - | - | カテゴリフィルタ |
| limit | integer | - | 10 | 取得件数 |
| offset | integer | - | 0 | オフセット |
| use_case | string | - | hondashi | ユースケース |

**レスポンス**

```json
{
  "success": true,
  "data": {
    "products": [
      {
        "id": "prod_001",
        "name": "ほんだし",
        "description": "かつおと昆布の合わせだし",
        "category": "だし調味料",
        "image_url": "https://example.com/images/hondashi.jpg",
        "faq_count": 10,
        "is_active": true
      }
    ],
    "pagination": {
      "total": 5,
      "limit": 10,
      "offset": 0,
      "has_more": false
    }
  }
}
```

---

#### 4.6.2 GET /products/{id}

製品詳細を取得する。

**リクエスト**

```http
GET /api/v1/products/prod_001
X-API-Key: {api-key}
```

**レスポンス**

```json
{
  "success": true,
  "data": {
    "id": "prod_001",
    "name": "ほんだし",
    "description": "厳選したかつお節と昆布から抽出しただしの素です。味噌汁、煮物、炊き込みご飯など幅広い和食にお使いいただけます。",
    "category": "だし調味料",
    "image_url": "https://example.com/images/hondashi.jpg",
    "product_url": "https://www.ajinomoto.co.jp/hondashi/",
    "sizes": [
      {"name": "8g×10本入り", "jan_code": "4901001012345"},
      {"name": "60g袋", "jan_code": "4901001012346"},
      {"name": "120g袋", "jan_code": "4901001012347"}
    ],
    "usage": [
      {"dish": "味噌汁", "amount": "水400mlに小さじ1"},
      {"dish": "煮物", "amount": "水300mlに小さじ1"},
      {"dish": "炊き込みご飯", "amount": "米2合に小さじ1"}
    ],
    "nutrition": {
      "per_serving": "小さじ1（4g）あたり",
      "calories": 10,
      "protein": 0.5,
      "fat": 0,
      "carbohydrates": 2.0,
      "sodium": 400
    },
    "faq_count": 10,
    "is_active": true,
    "created_at": "2025-01-01T00:00:00Z",
    "updated_at": "2025-01-01T00:00:00Z"
  }
}
```

---

### 4.7 フィードバックAPI

#### 4.7.1 POST /feedback

ユーザーフィードバックを送信する。

**リクエスト**

```http
POST /api/v1/feedback
Content-Type: application/json
X-API-Key: {api-key}
```

```json
{
  "session_id": "session_123456",
  "message_id": "msg_789012",
  "rating": "positive",
  "comment": "とても役に立ちました"
}
```

| パラメータ | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| session_id | string | ○ | セッションID |
| message_id | string | ○ | メッセージID |
| rating | string | ○ | 評価（positive/negative） |
| comment | string | - | コメント（500文字以内） |

**重複送信の挙動**
- 同一 `session_id` + `message_id` の組み合わせで再送信した場合、**既存のフィードバックを上書き**（最新の評価が有効）
- 上書き時もレスポンスは成功（`success: true`）を返却
- 1メッセージにつき1フィードバックのみ有効

**レスポンス**

```json
{
  "success": true,
  "data": {
    "feedback_id": "fb_456789",
    "received": true
  }
}
```

---

### 4.8 ヘルスチェックAPI

#### 4.8.1 GET /health

システムの稼働状態を確認する。

**リクエスト**

```http
GET /api/v1/health
```

**レスポンス**

```json
{
  "success": true,
  "data": {
    "status": "healthy",
    "version": "1.0.0",
    "timestamp": "2025-01-02T12:00:00Z",
    "services": {
      "azure_openai": "healthy",
      "cosmos_db": "healthy",
      "ai_search": "healthy"
    }
  }
}
```

---

## 5. レート制限

### 5.1 制限値

| プラン | リクエスト/分 | リクエスト/日 | 備考 |
|--------|-------------|-------------|------|
| **MVP（現行）** | **60** | **10,000** | 本番環境の初期設定 |
| Standard | 120 | 50,000 | 将来プラン（参考） |
| Premium | 300 | 無制限 | 将来プラン（参考） |

> **注記**: MVP期間中は **60リクエスト/分、10,000リクエスト/日** が適用されます。MVPでは大規模な同時アクセスは想定していないため、負荷テストは実施しません。レート制限の動作確認は手動テストで行います。

### 5.2 レート制限の適用単位

| 単位 | 説明 |
|------|------|
| API Key単位 | 同一API Keyからのリクエストを合算 |
| エンドポイント共通 | 全エンドポイントで共通のカウント |

### 5.3 API種別ごとのレート制限

| API種別 | リクエスト/分 | 同時接続数 | 備考 |
|---------|-------------|-----------|------|
| 通常API（Chat, Recipe等） | 60 | - | 標準制限 |
| 管理者API | 30 | - | より厳しい制限 |
| SSEストリーミング | 20 | 10接続 | 同時接続数制限あり |

### 5.4 レート制限ヘッダー

**リクエスト時のレスポンスヘッダー**

```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 55
X-RateLimit-Reset: 1704200000
```

**429エラー時のレスポンス**

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1704200030
Content-Type: application/json

{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "リクエストが多すぎます。しばらく待ってからお試しください。",
    "retry_after": 30
  }
}
```

**クライアント側リトライポリシー**

| 回数 | 待機時間 | 備考 |
|------|---------|------|
| 1回目 | Retry-Afterヘッダーの値 | サーバー指定を優先 |
| 2回目 | 60秒 | 固定待機 |
| 3回目 | 120秒 | 固定待機 |
| 4回目以降 | リトライ中止 | ユーザーに通知 |

---

## 6. SDKサンプルコード

### 6.1 JavaScript/TypeScript

```typescript
// Chat API呼び出し例（製品FAQ対応）
const response = await fetch('https://api.example.com/api/v1/chat/messages', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-API-Key': process.env.API_KEY,
  },
  body: JSON.stringify({
    session_id: 'session_123456',
    message: 'ほんだしの保存方法を教えてください',
  }),
});

const data = await response.json();

// 応答内容
console.log(data.data.response.content);

// FAQが含まれている場合（複数FAQ対応）
if (data.data.response.faqs && data.data.response.faqs.length > 0) {
  data.data.response.faqs.forEach(faq => {
    console.log(`FAQ: ${faq.question}`);
    console.log(`カテゴリ: ${faq.category}`);
  });
}

// サジェスト
data.data.response.suggestions.forEach(suggestion => {
  console.log(`[${suggestion.label}]`);
});
```

### 6.2 Python

```python
import requests
import os

# 製品FAQ取得例
response = requests.get(
    'https://api.example.com/api/v1/products/prod_001/faq',
    headers={
        'X-API-Key': os.environ['API_KEY'],
    },
    params={
        'category': '保存方法',
        'limit': 5
    }
)

data = response.json()

# FAQ一覧
for faq in data['data']['faqs']:
    print(f"Q: {faq['question']}")
    print(f"A: {faq['answer']}")
    print(f"カテゴリ: {faq['category']}")
    print('---')
```

---

## 7. 更新履歴

| 日付 | バージョン | 更新内容 |
|------|-----------|---------|
| 2025-01-02 | 1.0 | 初版作成 |
| 2025-01-02 | 1.1 | MVP版に更新 |
| 2025-01-02 | 1.2 | 製品FAQ API追加（/products/{id}/faq, /faq/search）、関連レシピをPhase2へ移動 |
| 2025-01-02 | 1.3 | faqを配列（faqs）に変更、複数FAQ対応 |
| 2025-01-02 | 1.4 | CORS設定追加、API Key管理追加、SSE再接続ポリシー追加、エラーメッセージ詳細化 |
| 2025-01-02 | 1.5 | 管理者API Key認証方式明確化、session_id使い分け説明追加、サジェストaction詳細追加、フィードバック重複防止追加、レート制限説明強化 |
| 2025-01-03 | 1.6 | 管理者APIサンプルヘッダー修正（X-Admin-API-Key）、SSE再接続パラメータ詳細追加、レート制限詳細（API種別、Retry-After、リトライポリシー）追加 |
| 2025-01-03 | 1.7 | 共通ヘッダーにX-Admin-API-Key追加、CORS許可ヘッダーに管理者ヘッダー追加、FAQ limitデフォルト5件に修正、日次上限10,000に統一、intent一覧表の詳細化 |
| 2025-01-03 | 1.8 | **MVPアクセス認証API（POST /auth/verify）追加**、パスワード検証・ロックアウト仕様追加 |
