# CLAUDE.md - foodllm プロジェクトガイド

## プロジェクト概要

**foodllm** は、お客様（一般生活者）向けの対話型チャットLLMアプリケーション基盤です。
MVP では「ほんだし」をユースケースとして、和食・だし関連の相談・レシピ提案・製品FAQ対応を提供します。

## アーキテクチャ

```
フロントエンド (Next.js/React)
        ↓
Azure API Management (認証・レート制限 60req/min)
        ↓
Azure Container Apps (FastAPI/Python)
        ↓
┌───────────────┬───────────────┬───────────────┐
│ Azure OpenAI  │ Azure AI      │ Cosmos DB     │
│ (GPT-4o)      │ Search (RAG)  │ (会話ログ)    │
└───────────────┴───────────────┴───────────────┘
```

**リージョン**: East US (全サービス統一)

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js (React), TypeScript |
| バックエンド | FastAPI, Python 3.11 |
| LLM | Azure OpenAI (GPT-4o) |
| 検索 | Azure AI Search (RAG ベクトル検索) |
| データベース | Azure Cosmos DB |
| API Gateway | Azure API Management (Consumption) |
| ホスティング | Azure Container Apps, Azure Static Web Apps |
| CI/CD | GitHub Actions |

## ディレクトリ構成

```
/
├── .claude/                 # Claude Code 設定
│   ├── agents/              # カスタムエージェント定義
│   ├── skills/              # 開発スキル定義
│   └── settings.local.json  # 許可コマンド設定
├── docs/                    # 設計ドキュメント
│   ├── 01_system-requirements-mvp.md    # システム要件定義
│   ├── 03_screen-definition-mvp.md      # 画面定義
│   ├── 04_api-specification-mvp.md      # API仕様
│   ├── 05_test-specification-mvp.md     # テスト仕様
│   ├── 06_prompt-flow-design.md         # Prompt Flow設計
│   └── foodllm-chat-platform-development-plan.md  # 開発プラン
├── api/                     # バックエンド (FastAPI)
├── apps/web/               # フロントエンド (Next.js)
├── prompts/                # LLMプロンプト管理
├── scripts/                # データ投入・運用スクリプト
└── tests/                  # テストコード
```

## 主要機能 (MVP)

### チャット機能
- メッセージ送受信、ストリーミング応答 (SSE)
- セッション管理、会話ログ保存
- コンテキスト制限: 8000トークン (最新10メッセージ保持)

### RAG 検索
- **レシピ**: `hondashi-recipes-index`
- **製品**: `hondashi-products-index`
- **FAQ**: `hondashi-faq-index`
- チャンクサイズ: 512トークン、オーバーラップ: 128トークン

### クイックリプライ
- 会話コンテキストに基づく動的サジェスト生成
- 最大4つまで表示
- YAMLルールベース + LLM生成のハイブリッド

### 意図分類 (Intent)
| intent | 説明 |
|--------|------|
| `recipe_search` / `recipe_provided` | レシピ関連 |
| `product_info` | 製品情報 |
| `faq_question` / `faq_answered` | FAQ |
| `general_chat` | 一般会話 |
| `out_of_scope` | 対応範囲外 |

## API エンドポイント

| メソッド | エンドポイント | 説明 |
|---------|---------------|------|
| POST | `/api/v1/auth/verify` | MVPアクセスパスワード検証 |
| POST | `/api/v1/chat/messages` | メッセージ送信・応答取得 |
| POST | `/api/v1/chat/messages/stream` | ストリーミング応答 (SSE) |
| POST | `/api/v1/conversations` | 新規会話作成 |
| GET | `/api/v1/recipes/search` | レシピ検索 |
| GET | `/api/v1/recipes/{id}` | レシピ詳細 |
| GET | `/api/v1/products/search` | 製品検索 |
| GET | `/api/v1/products/{id}` | 製品詳細 |
| GET | `/api/v1/products/{id}/faq` | 製品FAQ取得 |
| GET | `/api/v1/faq/search` | FAQ検索 |
| POST | `/api/v1/feedback` | フィードバック送信 |
| GET | `/api/v1/health` | ヘルスチェック |
| GET | `/api/v1/admin/conversations` | 会話一覧 (管理者用) |

**認証ヘッダー**:
- 通常API: `X-API-Key`
- 管理者API: `X-Admin-API-Key`

## 環境変数

```bash
# Azure OpenAI
AZURE_REGION=eastus
AZURE_OPENAI_ENDPOINT=https://foodllm-oai-{env}.openai.azure.com/
AZURE_OPENAI_API_KEY=         # Key Vault管理
AZURE_OPENAI_DEPLOYMENT_NAME=gpt-4o

# Cosmos DB
COSMOS_DB_ENDPOINT=https://foodllm-cosmos-{env}.documents.azure.com:443/
COSMOS_DB_KEY=                # Key Vault管理
COSMOS_DB_DATABASE_NAME=foodllm-db

# Azure AI Search
AI_SEARCH_ENDPOINT=https://foodllm-srch-{env}.search.windows.net
AI_SEARCH_API_KEY=            # Key Vault管理
AI_SEARCH_INDEX_RECIPES=hondashi-recipes-index
AI_SEARCH_INDEX_PRODUCTS=hondashi-products-index
AI_SEARCH_INDEX_FAQ=hondashi-faq-index

# MVP認証
MVP_ACCESS_PASSWORD_HASH=     # Key Vault管理
```

## 命名規則

| リソース | パターン | 例 |
|---------|---------|-----|
| リソースグループ | `{company}-{project}-{env}-rg` | `ajinomoto-foodllm-dev-rg` |
| Azure OpenAI | `{project}-oai-{env}` | `foodllm-oai-dev` |
| Container Apps | `{project}-{app}-ca-{env}` | `foodllm-hondashi-ca-dev` |
| Cosmos DB | `{project}-cosmos-{env}` | `foodllm-cosmos-dev` |
| AI Search | `{project}-srch-{env}` | `foodllm-srch-dev` |

## デザインガイドライン

### カラー
| 用途 | カラーコード |
|------|-------------|
| プライマリ (味の素レッド) | `#E60012` |
| セカンダリ (ほんだしブラウン) | `#8B4513` |
| アクセント (だしゴールド) | `#D4A853` |
| ベース (ウォームホワイト) | `#FFFAF5` |
| AIメッセージ背景 | `#FFF8F0` |
| ユーザーメッセージ背景 | `#E60012` |

### フォント
- Noto Sans JP
- 本文: 14px Regular
- 見出し: 16-24px Bold

## 開発コマンド

```bash
# バックエンド
cd api
pip install -r requirements.txt
uvicorn main:app --reload

# フロントエンド
cd apps/web
npm install
npm run dev

# テスト
pytest tests/ --cov=.
npm run test

# Lint
flake8 .
black --check .
npm run lint
```

## テスト

| 種別 | ツール | カバレッジ目標 |
|------|--------|--------------|
| 単体テスト | pytest / Jest | 80% |
| 統合テスト | pytest / Jest | 主要フロー100% |
| E2Eテスト | Playwright | 4シナリオ |
| LLM品質 | 手動評価 | 30サンプル |

## 非機能要件

| 項目 | 目標値 |
|------|--------|
| 応答時間 | 平均3秒以内、P95 5秒以内 |
| レート制限 | 60 req/min |
| 可用性 | 99%以上 |
| セッションタイムアウト | 30分 |
| 会話ログ保持 | 90日 |

## セキュリティ

- HTTPS必須 (TLS 1.2以上)
- Azure Key Vault でシークレット管理
- XSS・SQLインジェクション対策
- PIIマスキング (メール・電話番号)
- MVPアクセス制限: パスワード認証 (5回失敗で1分ロック)

## Claude Code 設定

### エージェント (.claude/agents/)

プロジェクト専用のカスタムエージェントを定義しています。`/agent <name>` で呼び出し可能。

| エージェント | 説明 |
|-------------|------|
| `azure-infra-engineer` | Azure インフラ構築、Bicep/ARM、PowerShell 自動化 |
| `backend-developer` | FastAPI/Python バックエンド開発、API設計 |
| `api-designer` | RESTful API 設計、OpenAPI 仕様作成 |
| `nextjs-developer` | Next.js フロントエンド開発 |
| `fullstack-developer` | フルスタック開発全般 |
| `llm-architect` | LLM アーキテクチャ設計、プロンプトエンジニアリング |
| `prompt-engineer` | プロンプト最適化、RAG パイプライン設計 |
| `devops-engineer` | CI/CD パイプライン、コンテナ化、デプロイ |
| `qa-expert` | テスト戦略、品質保証 |
| `test-automator` | テスト自動化、E2E テスト |
| `security-auditor` | セキュリティ監査、脆弱性診断 |
| `performance-engineer` | パフォーマンス最適化 |
| `code-reviewer` | コードレビュー、ベストプラクティス |
| `python-pro` | Python 開発のエキスパート |
| `typescript-pro` | TypeScript 開発のエキスパート |
| `ai-engineer` | AI/ML エンジニアリング |

### スキル (.claude/skills/)

開発プラクティスを定義したスキルセットです。

| スキル | 説明 |
|--------|------|
| `test-driven-development` | TDD（テスト駆動開発）の実践ガイド。Red-Green-Refactor サイクル必須 |
| `verification-before-completion` | 完了報告前の検証必須。証拠なき主張は禁止 |
| `systematic-debugging` | 体系的なデバッグ手法、根本原因分析 |
| `varlock` | 変数ロック、状態管理のベストプラクティス |
| `webapp-testing` | Web アプリケーションテスト自動化 |

### 許可コマンド (.claude/settings.local.json)

以下のコマンドが事前許可されています：

- **Git**: `git add`, `git commit`, `git push`, `git checkout`, `git branch`, `git pull`
- **GitHub CLI**: `gh pr create`, `gh pr view`, `gh issue create`, `gh repo create`
- **npm**: `npm install`, `npm run build`, `npm run lint`, `npm run test`
- **Python**: `pip`, `python -m venv`
- **Azure CLI**: `az group create`, `az deployment group create`, `az acr build`, `az containerapp update` など
- **その他**: `tree`, `find`, `curl`

## 参考ドキュメント

- [システム要件定義](docs/01_system-requirements-mvp.md)
- [画面定義](docs/03_screen-definition-mvp.md)
- [API仕様](docs/04_api-specification-mvp.md)
- [テスト仕様](docs/05_test-specification-mvp.md)
- [Prompt Flow設計](docs/06_prompt-flow-design.md)
- [開発プラン](docs/foodllm-chat-platform-development-plan.md)
