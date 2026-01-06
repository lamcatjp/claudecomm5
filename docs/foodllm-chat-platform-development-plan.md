# foodllm 対話型LLMアプリケーション開発プラン

## 📋 プロジェクト概要

### 目的
お客様（一般生活者）向け対話型チャットLLMアプリケーション基盤の開発

### 活用例
1. **ほんだし** - 和食・だし関連の相談・レシピ提案（MVP対象）
2. **たんぱく質関連製品** - 健康・栄養相談、運動提案
3. **料理初心者支援** - 基礎的な料理相談・レシピ提案

### 技術スタック
- **クラウド**: Microsoft Azure（Azureサービスのみで構成）
- **LLM**: Azure OpenAI（将来的にGemini対応）
- **既存資産**: レシピアレンジエンジン（独自RAG + アルゴリズム）

---

## 🎯 全体アーキテクチャ

```
┌─────────────────────────────────────────────────────────────────────┐
│                        フロントエンド                                │
│   React/Next.js Web App  →  (将来) チャットウィジェット/埋め込み      │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                 Azure API Management (Consumption)                   │
│          ・認証(API Key) ・CORS ・レート制限(60 req/min)             │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│              Azure Container Apps Environment                        │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │              Container App: foodllm-api (FastAPI/Python)        │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │ │
│  │  │ Chat API    │  │ Recipe API  │  │ Product API             │ │ │
│  │  │ (SSE対応)   │  │             │  │ (将来: Nutrition API)   │ │ │
│  │  └─────────────┘  └─────────────┘  └─────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                                    │
          ┌─────────────────────────┼─────────────────────────┐
          ▼                         ▼                         ▼
┌─────────────────┐    ┌─────────────────────┐    ┌─────────────────┐
│  Azure OpenAI   │    │  Azure AI Search    │    │   Cosmos DB     │
│  (将来:Gemini)  │    │  (RAG用ベクトル検索) │    │  (会話ログ等)   │
└─────────────────┘    └─────────────────────┘    └─────────────────┘
                                    │
                                    ▼
                       ┌─────────────────────┐
                       │ レシピアレンジ       │
                       │ エンジン (独自RAG)   │
                       └─────────────────────┘
```

### 技術スタック変更理由

| 変更 | 理由 |
|------|------|
| Azure Functions → **Container Apps** | SSEストリーミングの安定動作、ローカル開発の一貫性 |
| API Management追加（MVP） | 正確なレート制限（60 req/min）、APIキー管理の一元化 |

---

## 📅 フェーズ別開発計画

### Phase 0: 基盤構築（Week 1）

#### 前提条件（Week 0完了必須）

| 項目 | 内容 | 担当 | リードタイム |
|------|------|------|-------------|
| Azure OpenAI申請承認 | Azure OpenAI Serviceの利用申請が承認されていること | PM/インフラ | **1〜2週間** |
| Azureサブスクリプション | 開発用サブスクリプションが利用可能であること | インフラ | 1〜3日 |
| データ提供 | ほんだしレシピ・製品・FAQデータが提供されていること | ビジネス | **開発開始前** |
| デザインガイドライン | ブランドカラー・ロゴ使用ルールが確定していること | デザイン | 1週間 |

> ⚠️ **注意**: Azure OpenAI Serviceの申請は承認まで**1〜2週間**かかる場合があります。開発開始の2週間前までに申請を完了してください。

#### 目標
- 開発環境の整備
- Azureリソースの初期構築
- CI/CDパイプラインの構築

#### タスク

**リージョン・命名規則設定（全サービス共通）**

| 項目 | 設定値 | 備考 |
|------|--------|------|
| **プライマリリージョン** | **East US (eastus)** | 全Azureサービスをこのリージョンに統一 |

> **East US選定理由**:
> - Azure OpenAI GPT-4o/GPT-4o-miniが利用可能
> - Azure AI Searchセマンティック検索が利用可能
> - Container Apps、API Management (Consumption) が利用可能
> - 日本リージョンより低コスト（約10-20%安価）
> - 日本からのレイテンシ: 約150-200ms（許容範囲内）

---

#### 命名規則設定

**設定パラメータ（カスタマイズ可能）**

| パラメータ | デフォルト値 | 説明 | 例 |
|-----------|-------------|------|-----|
| `{company}` | `ajinomoto` | 会社名/組織名（小文字） | ajinomoto, contoso |
| `{project}` | `foodllm` | プロジェクト名（小文字） | foodllm, chatbot |
| `{app}` | `hondashi` | アプリケーション名（小文字） | hondashi, protein |
| `{env}` | `dev` | 環境識別子 | dev, stg, prod |
| `{region}` | `eus` | リージョン略称 | eus (East US), jpe (Japan East) |
| `{seq}` | `001` | 連番（必要時） | 001, 002 |

**命名規則パターン**

```
標準パターン: {company}-{project}-{service}-{env}
短縮パターン: {project}-{service}-{env}
```

---

**サブスクリプション命名規則**

| 項目 | パターン | 例 |
|------|---------|-----|
| 開発用 | `{company}-{project}-dev-subscription` | `ajinomoto-foodllm-dev-subscription` |
| ステージング用 | `{company}-{project}-stg-subscription` | `ajinomoto-foodllm-stg-subscription` |
| 本番用 | `{company}-{project}-prod-subscription` | `ajinomoto-foodllm-prod-subscription` |

> **Note**: 既存サブスクリプションを使用する場合は、既存の命名規則に従ってください。

---

**リソースグループ命名規則**

| 項目 | パターン | 例 |
|------|---------|-----|
| メインリソース | `{company}-{project}-{env}-rg` | `ajinomoto-foodllm-dev-rg` |
| 共有リソース | `{company}-{project}-shared-rg` | `ajinomoto-foodllm-shared-rg` |
| ネットワーク | `{company}-{project}-network-{env}-rg` | `ajinomoto-foodllm-network-dev-rg` |

---

**リソース別命名規則**

| リソース種別 | 略称 | パターン | 例 | Azure制約 |
|-------------|------|---------|-----|----------|
| **Azure OpenAI** | `oai` | `{project}-oai-{env}` | `foodllm-oai-dev` | 2-64文字、英数字・ハイフン |
| **Container Apps Environment** | `cae` | `{project}-cae-{env}` | `foodllm-cae-dev` | 1-32文字、小文字英数字・ハイフン |
| **Container App** | `ca` | `{project}-{app}-ca-{env}` | `foodllm-hondashi-ca-dev` | 2-32文字、小文字英数字・ハイフン |
| **API Management** | `apim` | `{project}-apim-{env}` | `foodllm-apim-dev` | 1-50文字、英数字・ハイフン |
| **Cosmos DB Account** | `cosmos` | `{project}-cosmos-{env}` | `foodllm-cosmos-dev` | 3-44文字、小文字英数字・ハイフン |
| **Cosmos DB Database** | - | `{project}-db` | `foodllm-db` | - |
| **AI Search** | `srch` | `{project}-srch-{env}` | `foodllm-srch-dev` | 2-60文字、小文字英数字・ハイフン |
| **Container Registry** | `acr` | `{project}acr{env}` | `foodllmacr` + `dev` → `foodllmacrdev` | 5-50文字、英数字のみ（ハイフン不可） |
| **Key Vault** | `kv` | `{project}-kv-{env}` | `foodllm-kv-dev` | 3-24文字、英数字・ハイフン |
| **Storage Account** | `st` | `{project}st{env}` | `foodllmstdev` | 3-24文字、小文字英数字のみ |
| **Application Insights** | `appi` | `{project}-appi-{env}` | `foodllm-appi-dev` | 1-260文字 |
| **Log Analytics Workspace** | `log` | `{project}-log-{env}` | `foodllm-log-dev` | 4-63文字、英数字・ハイフン |
| **Static Web Apps** | `swa` | `{project}-{app}-swa-{env}` | `foodllm-hondashi-swa-dev` | 1-40文字 |

---

**命名規則設定ファイル（config/naming.yaml）**

```yaml
# config/naming.yaml
# Azure リソース命名規則設定

# === カスタマイズ可能パラメータ ===
naming:
  company: "ajinomoto"      # 会社名/組織名
  project: "foodllm"        # プロジェクト名
  app: "hondashi"           # アプリケーション名（ユースケース）
  
  # 環境識別子
  environments:
    development: "dev"
    staging: "stg"
    production: "prod"
  
  # リージョン略称
  regions:
    eastus: "eus"
    japaneast: "jpe"
    westus2: "wus2"

# === リソース別命名パターン ===
resources:
  subscription:
    pattern: "{company}-{project}-{env}-subscription"
    example: "ajinomoto-foodllm-dev-subscription"
  
  resource_group:
    pattern: "{company}-{project}-{env}-rg"
    example: "ajinomoto-foodllm-dev-rg"
  
  openai:
    abbreviation: "oai"
    pattern: "{project}-oai-{env}"
    example: "foodllm-oai-dev"
    constraints:
      min_length: 2
      max_length: 64
      allowed_chars: "alphanumeric, hyphen"
  
  container_apps_environment:
    abbreviation: "cae"
    pattern: "{project}-cae-{env}"
    example: "foodllm-cae-dev"
    constraints:
      min_length: 1
      max_length: 32
      allowed_chars: "lowercase alphanumeric, hyphen"
  
  container_app:
    abbreviation: "ca"
    pattern: "{project}-{app}-ca-{env}"
    example: "foodllm-hondashi-ca-dev"
    constraints:
      min_length: 2
      max_length: 32
      allowed_chars: "lowercase alphanumeric, hyphen"
  
  api_management:
    abbreviation: "apim"
    pattern: "{project}-apim-{env}"
    example: "foodllm-apim-dev"
    constraints:
      min_length: 1
      max_length: 50
      allowed_chars: "alphanumeric, hyphen"
  
  cosmos_db:
    abbreviation: "cosmos"
    pattern: "{project}-cosmos-{env}"
    example: "foodllm-cosmos-dev"
    constraints:
      min_length: 3
      max_length: 44
      allowed_chars: "lowercase alphanumeric, hyphen"
  
  ai_search:
    abbreviation: "srch"
    pattern: "{project}-srch-{env}"
    example: "foodllm-srch-dev"
    constraints:
      min_length: 2
      max_length: 60
      allowed_chars: "lowercase alphanumeric, hyphen"
  
  container_registry:
    abbreviation: "acr"
    pattern: "{project}acr{env}"
    example: "foodllmacrdev"
    constraints:
      min_length: 5
      max_length: 50
      allowed_chars: "alphanumeric only (no hyphen)"
  
  key_vault:
    abbreviation: "kv"
    pattern: "{project}-kv-{env}"
    example: "foodllm-kv-dev"
    constraints:
      min_length: 3
      max_length: 24
      allowed_chars: "alphanumeric, hyphen"
  
  storage_account:
    abbreviation: "st"
    pattern: "{project}st{env}"
    example: "foodllmstdev"
    constraints:
      min_length: 3
      max_length: 24
      allowed_chars: "lowercase alphanumeric only"
```

---

**環境別リソース名一覧（デフォルト設定時）**

| リソース | 開発 (dev) | ステージング (stg) | 本番 (prod) |
|---------|-----------|-------------------|------------|
| リソースグループ | `ajinomoto-foodllm-dev-rg` | `ajinomoto-foodllm-stg-rg` | `ajinomoto-foodllm-prod-rg` |
| Azure OpenAI | `foodllm-oai-dev` | `foodllm-oai-stg` | `foodllm-oai-prod` |
| Container Apps Env | `foodllm-cae-dev` | `foodllm-cae-stg` | `foodllm-cae-prod` |
| Container App | `foodllm-hondashi-ca-dev` | `foodllm-hondashi-ca-stg` | `foodllm-hondashi-ca-prod` |
| API Management | `foodllm-apim-dev` | `foodllm-apim-stg` | `foodllm-apim-prod` |
| Cosmos DB | `foodllm-cosmos-dev` | `foodllm-cosmos-stg` | `foodllm-cosmos-prod` |
| AI Search | `foodllm-srch-dev` | `foodllm-srch-stg` | `foodllm-srch-prod` |
| Container Registry | `foodllmacrdev` | `foodllmacrstg` | `foodllmacrprod` |
| Key Vault | `foodllm-kv-dev` | `foodllm-kv-stg` | `foodllm-kv-prod` |
| Storage Account | `foodllmstdev` | `foodllmststg` | `foodllmstprod` |
| Application Insights | `foodllm-appi-dev` | `foodllm-appi-stg` | `foodllm-appi-prod` |
| Static Web Apps | `foodllm-hondashi-swa-dev` | `foodllm-hondashi-swa-stg` | `foodllm-hondashi-swa-prod` |

---

> **インフラセットアップ方法**
> 
> 社内ネットワークの制約によりローカルAzure CLIが使用できない場合、以下の方法でセットアップしてください:
> 
> | 方法 | 説明 | 推奨度 |
> |------|------|--------|
> | **Azure Cloud Shell** | ブラウザベースのシェル環境（https://shell.azure.com） | ⭐ 推奨 |
> | **Azure Portal** | Web UIでBicepテンプレートをデプロイ | ○ |
> | **Azure Portal 手動** | Web UIで各リソースを個別作成 | △ 初回のみ |
> 
> 詳細な手順は「推奨Azureアーキテクチャ」セクションの「インフラデプロイ方法」を参照してください。

| カテゴリ | タスク | 使用サービス | リージョン |
|---------|--------|-------------|-----------|
| インフラ | Azureサブスクリプション・リソースグループ設定 | Azure Portal / Cloud Shell | East US |
| インフラ | Azure OpenAI リソース作成・モデルデプロイ | Azure OpenAI (GPT-4o) | **East US** |
| インフラ | **Container Apps Environment作成** | **Azure Container Apps** | East US |
| インフラ | **API Management (Consumption) 作成・API定義** | **Azure API Management** | East US |
| インフラ | Cosmos DB アカウント・データベース作成 | Azure Cosmos DB | East US |
| インフラ | Azure AI Search インスタンス作成 | Azure AI Search | East US |
| インフラ | Azure Key Vault 作成・シークレット管理設定 | Azure Key Vault | East US |
| インフラ | Container Registry 作成 | Azure Container Registry | East US |
| インフラ | Application Insights 設定（基本監視） | Azure Monitor | East US |
| CI/CD | GitHub Actions パイプライン構築（Docker Build → ACR → Container Apps） | GitHub Actions | - |
| データ | ほんだし関連データ収集・構造化 | - | - |
| **データ投入** | **初期データのAzure AI Search投入** | **Azure AI Search** | East US |

#### データ収集詳細

| データソース | 収集方法 | 想定データ量 |
|-------------|---------|-------------|
| ほんだしレシピ | 公式サイトからの構造化抽出 | 100〜200件 |
| 製品情報 | 公式サイト・製品カタログ | 10〜20製品 |
| FAQ | お客様相談室データ | 50〜100件 |
| 調理基礎知識 | foodllm PARK等から抽出 | 30〜50件 |

#### データ投入手順

**1. データ準備（担当: データエンジニア）**
```bash
# JSONフォーマットに変換
python scripts/convert_recipes.py --input raw/recipes.csv --output data/recipes.json
python scripts/convert_products.py --input raw/products.csv --output data/products.json
python scripts/convert_faq.py --input raw/faq.csv --output data/faq.json
```

**2. エンベディング生成（担当: MLエンジニア）**
```bash
# Azure OpenAI Embeddingでベクトル生成
python scripts/generate_embeddings.py --input data/recipes.json --output data/recipes_embedded.json
```

**3. Azure AI Search投入（担当: インフラエンジニア）**
```bash
# インデックス作成・データ投入
python scripts/upload_to_search.py --index hondashi-recipes-index --data data/recipes_embedded.json
python scripts/upload_to_search.py --index hondashi-products-index --data data/products_embedded.json
python scripts/upload_to_search.py --index hondashi-faq-index --data data/faq_embedded.json
```

**4. 動作確認（担当: QA）**
```bash
# 検索テスト
python scripts/test_search.py --query "味噌汁の作り方"
```

**データ更新フロー（運用時）**
1. 新規レシピをCSVで提供
2. 上記手順1〜3を実行
3. 差分のみ投入（`--mode upsert`オプション）
4. 検索精度テスト実施

#### 画像アセットのホスティング

**Azure Blob Storageへのアップロードフロー**

```bash
# 1. 画像ファイルをBlobにアップロード
python scripts/upload_images.py --input raw/images/ --container recipe-images

# 出力例:
# Uploaded: misoshiru.jpg -> https://foodllmstorage.blob.core.windows.net/recipe-images/misoshiru.jpg
# Uploaded: nikujaga.jpg -> https://foodllmstorage.blob.core.windows.net/recipe-images/nikujaga.jpg
```

**2. JSONへのURL埋め込み**

```bash
# 画像URLをJSONに反映
python scripts/update_image_urls.py --input data/recipes.json --mapping image_urls.csv
```

**Blobストレージ構成**

| コンテナ名 | 用途 | アクセスレベル |
|-----------|------|--------------|
| `recipe-images` | レシピ画像 | Blob（匿名読み取り） |
| `product-images` | 製品画像 | Blob（匿名読み取り） |

> **既存CMS利用の場合**: 公式サイト等で既に公開されている画像URLを直接使用可能。その場合は上記スクリプトは不要。

#### データフォーマット例

```json
{
  "recipe_id": "hondashi_001",
  "title": "基本の味噌汁",
  "description": "ほんだしを使った定番の味噌汁",
  "ingredients": [
    {"name": "ほんだし", "amount": "小さじ1", "category": "調味料"},
    {"name": "味噌", "amount": "大さじ1.5", "category": "調味料"},
    {"name": "豆腐", "amount": "1/4丁", "category": "具材"}
  ],
  "steps": ["鍋に水400mlとほんだしを入れる", "..."],
  "cooking_time": 10,
  "difficulty": "easy",
  "tags": ["定番", "汁物", "簡単"]
}
```

#### 成果物
- 基本インフラが稼働状態
- ローカル開発環境でAzure OpenAI接続確認

#### CI/CDパイプライン詳細

**GitHub Actions ワークフロー構成**

| ワークフロー | トリガー | 処理内容 |
|------------|---------|---------|
| `ci.yml` | PR作成/更新 | Lint、単体テスト、Dockerビルド確認 |
| `cd-dev.yml` | developブランチへのマージ | ACRプッシュ → 開発環境へデプロイ |
| `cd-stg.yml` | mainブランチへのマージ | ACRプッシュ → ステージング環境へデプロイ |
| `cd-prod.yml` | リリースタグ作成 | 本番環境へデプロイ（手動承認） |

**パイプライン設定例（ci.yml）**

```yaml
name: CI Pipeline

on:
  pull_request:
    branches: [develop, main]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        working-directory: ./api
        run: |
          pip install -r requirements.txt
          pip install -r requirements-dev.txt
      
      - name: Lint (flake8, black)
        working-directory: ./api
        run: |
          flake8 .
          black --check .
      
      - name: Run tests
        working-directory: ./api
        run: pytest tests/ --cov=. --cov-report=xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v4

  docker-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker image
        run: docker build -t foodllm-api:test ./api
      
      - name: Run container tests
        run: |
          docker run --rm foodllm-api:test pytest tests/ -v

  frontend-check:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./apps/web
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install and lint
        run: |
          npm ci
          npm run lint
          npm run type-check
      
      - name: Run tests
        run: npm test -- --coverage
```

**デプロイパイプライン設定例（cd-dev.yml）**

```yaml
name: CD Development

on:
  push:
    branches: [develop]

env:
  ACR_NAME: foodllmacr
  CONTAINER_APP_NAME: foodllm-api-dev
  RESOURCE_GROUP: foodllm-dev-rg

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Azure Login
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Login to ACR
        run: az acr login --name ${{ env.ACR_NAME }}
      
      - name: Build and push image
        run: |
          IMAGE_TAG=${{ env.ACR_NAME }}.azurecr.io/foodllm-api:${{ github.sha }}
          docker build -t $IMAGE_TAG ./api
          docker push $IMAGE_TAG
      
      - name: Deploy to Container Apps
        run: |
          az containerapp update \
            --name ${{ env.CONTAINER_APP_NAME }} \
            --resource-group ${{ env.RESOURCE_GROUP }} \
            --image ${{ env.ACR_NAME }}.azurecr.io/foodllm-api:${{ github.sha }}
```

**ロールバック手順**

| 環境 | 手順 |
|------|------|
| 開発/ステージング | `az containerapp update --image <前バージョンのタグ>` で前イメージに切り替え |
| 本番 | 1. Container Appsのリビジョンを前バージョンに切り替え<br>2. 問題継続の場合、前リリースタグのイメージを再デプロイ |

#### 技術検証（PoC）項目【Week 1優先】

| 項目 | 検証内容 | リスク | 対策 |
|------|---------|--------|------|
| **Container Apps + SSE** | FastAPI Streaming Responseの動作確認 | ほぼなし（安定動作） | - |
| **API Management連携** | APIM経由でのContainer Apps呼び出し | レイテンシ増加 | キャッシュ設定調整 |
| Azure OpenAI接続 | GPT-4oモデルへの接続・レスポンス確認 | クォータ制限 | East USでクォータ申請済み前提 |
| RAG検索精度 | Azure AI Searchのセマンティック検索精度 | 検索精度不足 | Hybrid Search比率調整 |

> **リージョン**: 全サービスをEast USに統一済み。Azure OpenAI GPT-4oはEast USで利用可能。

**SSE実装（Container Apps + FastAPI）**

```python
# api/routers/chat.py
from fastapi import APIRouter, Request
from fastapi.responses import StreamingResponse
import json

router = APIRouter()

@router.post("/api/v1/chat/messages/stream")
async def stream_chat(request: Request):
    body = await request.json()
    
    async def generate():
        async for chunk in openai_client.chat.completions.create(
            model="gpt-4o",
            messages=build_messages(body),
            stream=True
        ):
            if chunk.choices[0].delta.content:
                yield f"data: {json.dumps({'content': chunk.choices[0].delta.content})}\n\n"
        yield "data: [DONE]\n\n"
    
    return StreamingResponse(
        generate(),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive",
        }
    )
```

**ローカル開発環境（Docker Compose）**

```yaml
# docker-compose.yml
version: '3.8'
services:
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      - AZURE_OPENAI_ENDPOINT=${AZURE_OPENAI_ENDPOINT}
      - AZURE_OPENAI_API_KEY=${AZURE_OPENAI_API_KEY}
      - COSMOS_DB_ENDPOINT=${COSMOS_DB_ENDPOINT}
      - COSMOS_DB_KEY=${COSMOS_DB_KEY}
    volumes:
      - ./api:/app
    command: uvicorn main:app --host 0.0.0.0 --port 8000 --reload

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
```

**Dockerfile（API）**

```dockerfile
# api/Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

> ✅ **Container Apps採用により、SSEのバッファリング問題は解消**。Week 1ではAPI Management連携とAzure OpenAI接続の検証に集中できます。

---

### Phase 1: MVP（Week 2-4）

#### 目標
- ほんだしに特化したシンプルなチャットボット
- 基本的なRAGによるレシピ・製品情報の回答
- 会話ログの保存

#### Week 2: バックエンドコア

- Chat API エンドポイント実装
  - セッション管理
  - Azure OpenAI 呼び出し
  - 会話履歴管理
  - **サジェスト（入力候補）レスポンス構造の実装**
- RAG パイプライン構築
  - ほんだしレシピデータのインデックス作成
  - Azure AI Search へのデータ投入
  - 検索 + 生成の統合
- **クイックリプライ機能（バックエンド）**
  - サジェスト生成ロジック実装
  - ルールベースサジェスト設定（YAML）
  - 意図分類に基づくサジェスト選択
- Cosmos DB スキーマ設計
  - sessions コンテナ（セッション管理）
  - conversations コンテナ（会話管理）
  - messages コンテナ（メッセージ）
  - feedback コンテナ（フィードバック）
  - ※ analytics コンテナは**Phase 2以降**で実装

#### Week 3: フロントエンド + 統合

- React チャットUI 実装
  - メッセージ送受信
  - 会話履歴表示
  - レシピカード表示コンポーネント
  - **QuickReplies（クイックリプライ）コンポーネント**
  - **サジェストボタンのスタイリング（ブランドカラー対応）**
- **クイックリプライ機能（フロントエンド）**
  - サジェストボタン表示・非表示制御
  - ボタン選択時のメッセージ送信処理
  - ローディング中の無効化処理
- API 統合テスト
- プロンプトエンジニアリング
  - ほんだし専用システムプロンプト
  - レシピ提案プロンプト
  - 製品提案プロンプト

#### Week 4: テスト + デプロイ

- テスト実施
  - 単体テスト（API関数、サジェスト生成ロジック）
  - 統合テスト（API + RAG + LLM）
  - E2Eテスト（フロントエンド → バックエンド → 応答表示）
  - LLM応答品質テスト（サンプル質問30件での回答評価）
- Azure Static Web Apps へのデプロイ
- 負荷テスト（基本）
  - 同時10ユーザー、100リクエスト/分
- ドキュメント作成
- MVP リリース

#### テスト戦略詳細

| テスト種別 | 対象 | ツール | カバレッジ目標 |
|-----------|------|--------|--------------|
| 単体テスト | API関数、ユーティリティ | Jest / pytest | 80% |
| 統合テスト | API + 外部サービス連携 | Jest / pytest | 主要フロー |
| E2Eテスト | ユーザーシナリオ | Playwright | 4シナリオ |
| 品質テスト | LLM応答 | 手動評価 | 30サンプル |

#### LLM応答品質評価基準

| 評価項目 | 基準 | 合格ライン |
|---------|------|-----------|
| 関連性 | 質問に対する回答の適切さ | 80%以上が「適切」 |
| 正確性 | 事実誤認がないか | 95%以上が「正確」 |
| 有用性 | ユーザーの役に立つか | 70%以上が「有用」 |
| トーン | ブランドに合った口調か | 90%以上が「適切」 |

#### MVP機能一覧

| 機能 | 詳細 | MVP | Phase 2 |
|------|------|-----|---------|
| 基本対話 | ほんだしに関する質問応答 | ✅ | |
| レシピ検索・提案 | RAGベースのレシピ提案 | ✅ | |
| 製品情報提供 | ほんだし製品ラインナップ説明 | ✅ | |
| **製品FAQ対応** | 保存方法・アレルギー等のよくある質問 | ✅ | |
| **クイックリプライ** | 入力候補ボタンの表示・選択機能 | ✅ | |
| 会話ログ保存 | Cosmos DBへの全会話保存（管理者用） | ✅ | |
| 関連レシピ表示 | 表示中レシピに関連する他のレシピを提案 | - | ✅ |
| レシピアレンジ提案 | 減塩・具材変更などのアレンジ提案 | - | ✅ |
| 分析ダッシュボード | 会話数・トピック傾向の可視化 | - | ✅ |

---

## 💬 クイックリプライ（入力候補）機能

### 概要

ユーザーが次に質問しそうな内容をボタンとして表示し、タップ/クリックで簡単に入力できる機能。

### UI イメージ

```
┌─────────────────────────────────────────────────┐
│  🤖 ほんだしを使った味噌汁のレシピをご紹介します。  │
│     他にもご質問があればお気軽にどうぞ！          │
└─────────────────────────────────────────────────┘

  ┌──────────────┐ ┌──────────────┐ ┌────────────┐
  │ 減塩にしたい  │ │ 具材を変えたい│ │ 他のレシピ │
  └──────────────┘ └──────────────┘ └────────────┘

  ┌─────────────────────────────────────────────┐
  │ メッセージを入力...                     [送信] │
  └─────────────────────────────────────────────┘
```

### APIレスポンス構造

```typescript
// Chat API レスポンスにサジェストを含める
interface ChatResponse {
  message: string;
  suggestions?: Suggestion[];  // 入力候補（0〜4件）
  recipe?: Recipe;             // レシピ情報（該当時）
  products?: Product[];        // 製品情報（該当時）
}

interface Suggestion {
  id: string;
  label: string;           // ボタン表示テキスト（15文字以内推奨）
  value: string;           // 実際に送信されるテキスト
  type: 'text' | 'action'; // テキスト送信 or 特定アクション
  icon?: string;           // オプション: アイコン
}
```

### サジェスト生成ロジック

```python
# packages/api/shared/suggestions/generator.py

class SuggestionGenerator:
    """コンテキストに応じたサジェスト生成"""
    
    def __init__(self, use_case: str):
        self.use_case = use_case
        self.rules = self._load_suggestion_rules()
    
    def generate(
        self,
        context: ConversationContext,
        last_response: str,
        intent: str
    ) -> List[Suggestion]:
        """
        会話コンテキストに基づいてサジェストを生成
        """
        suggestions = []
        
        # 意図に基づくルールベースのサジェスト
        if intent == "recipe_provided":
            suggestions.extend([
                Suggestion(label="減塩にしたい", value="このレシピを減塩にアレンジして"),
                Suggestion(label="具材を変えたい", value="具材を変えたいです"),
                Suggestion(label="他のレシピ", value="他のレシピも見たい"),
            ])
        
        elif intent == "product_inquiry":
            suggestions.extend([
                Suggestion(label="使い方を知りたい", value="この製品の使い方を教えて"),
                Suggestion(label="他の製品を見る", value="他の製品も見たい"),
            ])
        
        return suggestions[:4]  # 最大4つまで
```

### サジェストルール設定（YAML）

```yaml
# config/suggestions/hondashi.yaml

# 初回表示時のサジェスト
welcome:
  - label: "レシピを探す"
    value: "ほんだしを使ったレシピを教えてください"
  - label: "使い方を知りたい"
    value: "ほんだしの基本的な使い方を教えてください"
  - label: "製品について"
    value: "ほんだしの製品ラインナップを教えてください"

# レシピ提案後のサジェスト
after_recipe:
  - label: "減塩にしたい"
    value: "このレシピを減塩にアレンジしてください"
  - label: "具材を変えたい"
    value: "具材を変えたいのですが、おすすめはありますか？"
  - label: "カロリーを知りたい"
    value: "このレシピのカロリーを教えてください"
  - label: "他のレシピを見る"
    value: "他のレシピも見たいです"

# 製品紹介後のサジェスト
after_product:
  - label: "使い方を知りたい"
    value: "この製品の使い方を詳しく教えてください"
  - label: "レシピを見たい"
    value: "この製品を使ったレシピを教えてください"
  - label: "他の製品を見る"
    value: "他の製品も見たいです"

# 質問応答後のサジェスト
after_qa:
  - label: "もっと詳しく"
    value: "もう少し詳しく教えてください"
  - label: "関連レシピ"
    value: "関連するレシピはありますか？"
```

### フロントエンド コンポーネント

```tsx
// apps/web/src/components/Chat/QuickReplies.tsx

import React from 'react';

interface Suggestion {
  id: string;
  label: string;
  value: string;
  type: 'text' | 'action';
  icon?: string;
}

interface QuickRepliesProps {
  suggestions: Suggestion[];
  onSelect: (suggestion: Suggestion) => void;
  disabled?: boolean;
}

export const QuickReplies: React.FC<QuickRepliesProps> = ({
  suggestions,
  onSelect,
  disabled
}) => {
  if (!suggestions?.length) return null;

  return (
    <div className="quick-replies">
      {suggestions.map((suggestion) => (
        <button
          key={suggestion.id}
          className="quick-reply-btn"
          onClick={() => onSelect(suggestion)}
          disabled={disabled}
        >
          {suggestion.icon && <span className="icon">{suggestion.icon}</span>}
          {suggestion.label}
        </button>
      ))}
    </div>
  );
};
```

```css
/* apps/web/src/styles/quick-replies.css */

.quick-replies {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  padding: 12px 0;
  animation: fadeIn 0.3s ease-in;
}

.quick-reply-btn {
  padding: 8px 16px;
  border: 1px solid #E60012;  /* ほんだしブランドカラー（味の素レッド） */
  border-radius: 20px;
  background: white;
  color: #E60012;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s;
}

.quick-reply-btn:hover {
  background: #E60012;
  color: white;
}

.quick-reply-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
```

### チャット画面への統合

```tsx
// apps/web/src/components/Chat/ChatWindow.tsx

export const ChatWindow: React.FC = () => {
  const [messages, setMessages] = useState<Message[]>([]);
  const [suggestions, setSuggestions] = useState<Suggestion[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  
  const handleSendMessage = async (text: string) => {
    setIsLoading(true);
    setSuggestions([]); // 送信時にサジェストをクリア
    
    // ユーザーメッセージ追加
    setMessages(prev => [...prev, { role: 'user', content: text }]);
    
    // API呼び出し
    const response = await chatApi.send(text);
    
    // アシスタントメッセージ追加
    setMessages(prev => [...prev, { 
      role: 'assistant', 
      content: response.message 
    }]);
    
    // サジェスト更新
    setSuggestions(response.suggestions || []);
    setIsLoading(false);
  };
  
  const handleSuggestionSelect = (suggestion: Suggestion) => {
    handleSendMessage(suggestion.value);
  };

  return (
    <div className="chat-window">
      <MessageList messages={messages} />
      
      {/* クイックリプライ表示 */}
      <QuickReplies 
        suggestions={suggestions}
        onSelect={handleSuggestionSelect}
        disabled={isLoading}
      />
      
      <ChatInput onSend={handleSendMessage} disabled={isLoading} />
    </div>
  );
};
```

### Phase 2での拡張予定

| 機能 | Phase 1 (MVP) | Phase 2 |
|------|---------------|---------|
| ルールベースサジェスト | ✅ | ✅ |
| 意図別サジェスト | ✅ | ✅ |
| LLM動的生成サジェスト | - | ✅ |
| ユーザー行動学習 | - | ✅ |
| ユースケース別設定 | ほんだしのみ | 全ユースケース |

---

### Phase 2: 拡張機能（Month 2-3）

#### 2-A: レシピアレンジエンジン統合

```python
class RecipeArrangeEngine:
    """
    独自RAG + アルゴリズムによるレシピアレンジ
    """
    def __init__(self):
        self.ingredient_db = IngredientDatabase()  # 食材変換データ
        self.nutrition_rules = NutritionRules()    # 減塩等のルール
        
    async def arrange_recipe(
        self,
        original_recipe: Recipe,
        constraints: ArrangeConstraints  # 減塩、食材変更など
    ) -> ArrangedRecipe:
        # 独自アルゴリズム適用
        pass
```

#### 2-B: マルチユースケース対応

共通基盤の抽象化により、設定ファイルの切り替えで複数ユースケースに対応:

- `hondashi_config.yaml`
- `protein_config.yaml`
- `cooking_beginner_config.yaml`

共通コンポーネント:
- ChatEngine (LLM呼び出し)
- RAGEngine (検索 + 生成)
- ConversationManager (会話管理)

ユースケース固有:
- プロンプトテンプレート
- データソース設定
- UI カスタマイズ

#### 2-C: 栄養計算機能

- 食材データベース (日本食品標準成分表ベース)
- レシピ栄養計算エンジン
- パーソナライズ推奨 (将来)

#### 2-D: ガードレール機能（Prompt Flow連携）

**概要**: LLMの入出力を監視し、安全性・品質を担保する機能群

| 機能 | 説明 | 実装方式 |
|------|------|---------|
| **入力ガード** | 有害コンテンツ検出 | Azure AI Content Safety |
| **PII検出** | 個人情報のマスキング | Azure AI Language / 正規表現 |
| **Injection防御** | プロンプトインジェクション検出 | ルールベース + LLM判定 |
| **出力ガード** | 不適切応答の検出・修正 | Azure AI Content Safety |
| **Groundedness** | ハルシネーション検出 | Prompt Flow評価ノード |

**アーキテクチャ**:
```
ユーザー入力
    ↓
┌──────────────────────┐
│ 入力ガードレール      │ ← Content Safety, PII, Injection
├──────────────────────┤
│ メイン処理           │ ← 意図分類 → RAG → 応答生成
├──────────────────────┤
│ 出力ガードレール      │ ← Content Safety, Groundedness
└──────────────────────┘
    ↓
安全な応答
```

**導入スケジュール**: Phase 2 Week 1-6（6週間）
**追加コスト**: ¥10,000〜23,000/月

> 詳細は `06_prompt-flow-design.md` を参照

---

### Phase 3: プロダクション強化（Month 4-6）

#### 3-A: 認証基盤

Azure AD B2C / Easy Auth による認証:
- 匿名ユーザー (ほんだし一般)
- 会員ユーザー (たんぱく質製品購入者)
- 将来の会員連携

#### 3-B: チャットウィジェット

```javascript
// 埋め込みスクリプト例
<script src="https://chat.foodllm.co.jp/widget.js"></script>
<script>
  foodllmChat.init({
    useCase: 'hondashi',
    position: 'bottom-right',
    theme: 'hondashi-brand'
  });
</script>
```

#### 3-C: マルチLLMプロバイダー対応

```python
# プロバイダー抽象化
class LLMProvider(ABC):
    @abstractmethod
    async def generate(self, messages: List[Message]) -> Response:
        pass

class AzureOpenAIProvider(LLMProvider):
    # 現在の実装
    pass

class GeminiProvider(LLMProvider):
    # 将来の拡張
    pass

class LLMFactory:
    @staticmethod
    def create(provider_type: str) -> LLMProvider:
        providers = {
            "azure_openai": AzureOpenAIProvider,
            "gemini": GeminiProvider,
        }
        return providers[provider_type]()
```

---

## 🏗️ 推奨Azureアーキテクチャ詳細

### コア サービス

| サービス | 用途 | SKU推奨 |
|---------|------|---------|
| **Azure OpenAI** | LLM (GPT-4o/GPT-4o-mini) | Standard S0 |
| **Azure AI Search** | RAG用ベクトル検索 | Basic (MVP) → Standard (本番) |
| **Azure Cosmos DB** | 会話ログ・ユーザーデータ | Serverless (MVP) → Provisioned (本番) |
| **Azure Container Apps** | バックエンドAPI (FastAPI) | Consumption |
| **Azure API Management** | レート制限・認証・CORS | Consumption (MVP) → Developer (本番) |
| **Azure Container Registry** | コンテナイメージ管理 | Basic |
| **Azure Static Web Apps** | フロントエンドホスティング | Free (MVP) → Standard (本番) |
| **Azure Blob Storage** | レシピ画像・ドキュメント | Standard LRS |

### Container Apps選定理由

| 項目 | Azure Functions | Azure Container Apps | 採用 |
|------|----------------|---------------------|------|
| SSEストリーミング | ⚠️ バッファリング問題 | ✅ 安定動作 | **Container Apps** |
| ローカル開発 | func CLI | Docker Compose | **Container Apps** |
| 常時接続 | Consumptionはコールドスタート | 最小インスタンス設定可 | **Container Apps** |
| 将来のマイクロサービス化 | △ | ✅ 容易 | **Container Apps** |

### 将来追加サービス

| サービス | 用途 | 導入時期 |
|---------|------|---------|
| **Azure AD B2C** | 顧客認証基盤 | Phase 3 |
| **Azure AI Content Safety** | ガードレール（入出力チェック） | Phase 2 |

> **Note**: Azure Key Vault と Application Insights は Phase 0 で導入済み

### IaC テンプレート（Bicep）

**main.bicep（命名規則対応版）**

```bicep
// infra/main.bicep
targetScope = 'subscription'

// =============================================
// 命名規則パラメータ（カスタマイズ可能）
// =============================================
@description('会社名/組織名（小文字）')
param companyName string = 'ajinomoto'

@description('プロジェクト名（小文字）')
param projectName string = 'foodllm'

@description('アプリケーション名（小文字）')
param appName string = 'hondashi'

@description('環境名 (dev/stg/prod)')
@allowed(['dev', 'stg', 'prod'])
param environment string = 'dev'

@description('リージョン')
param location string = 'eastus'

// =============================================
// 命名規則の定義
// =============================================
var naming = {
  // リソースグループ: {company}-{project}-{env}-rg
  resourceGroup: '${companyName}-${projectName}-${environment}-rg'
  
  // Azure OpenAI: {project}-oai-{env}
  openai: '${projectName}-oai-${environment}'
  
  // Container Apps Environment: {project}-cae-{env}
  containerAppsEnv: '${projectName}-cae-${environment}'
  
  // Container App: {project}-{app}-ca-{env}
  containerApp: '${projectName}-${appName}-ca-${environment}'
  
  // API Management: {project}-apim-{env}
  apim: '${projectName}-apim-${environment}'
  
  // Cosmos DB: {project}-cosmos-{env}
  cosmosDb: '${projectName}-cosmos-${environment}'
  
  // AI Search: {project}-srch-{env}
  aiSearch: '${projectName}-srch-${environment}'
  
  // Container Registry: {project}acr{env} (ハイフン不可)
  acr: '${projectName}acr${environment}'
  
  // Key Vault: {project}-kv-{env}
  keyVault: '${projectName}-kv-${environment}'
  
  // Storage Account: {project}st{env} (ハイフン不可)
  storage: '${projectName}st${environment}'
  
  // Application Insights: {project}-appi-{env}
  appInsights: '${projectName}-appi-${environment}'
  
  // Log Analytics: {project}-log-{env}
  logAnalytics: '${projectName}-log-${environment}'
  
  // Static Web Apps: {project}-{app}-swa-{env}
  staticWebApp: '${projectName}-${appName}-swa-${environment}'
}

// =============================================
// リソース定義
// =============================================

// リソースグループ
resource rg 'Microsoft.Resources/resourceGroups@2023-07-01' = {
  name: naming.resourceGroup
  location: location
  tags: {
    Environment: environment
    Project: projectName
    Company: companyName
  }
}

// Log Analytics Workspace（Application Insights用）
module logAnalytics 'modules/log-analytics.bicep' = {
  scope: rg
  name: 'logAnalytics'
  params: {
    name: naming.logAnalytics
    location: location
  }
}

// Application Insights
module appInsights 'modules/app-insights.bicep' = {
  scope: rg
  name: 'appInsights'
  params: {
    name: naming.appInsights
    location: location
    workspaceId: logAnalytics.outputs.workspaceId
  }
}

// Container Apps Environment
module containerAppsEnv 'modules/container-apps-env.bicep' = {
  scope: rg
  name: 'containerAppsEnv'
  params: {
    name: naming.containerAppsEnv
    location: location
    logAnalyticsWorkspaceId: logAnalytics.outputs.workspaceId
  }
}

// API Management (Consumption)
module apim 'modules/api-management.bicep' = {
  scope: rg
  name: 'apim'
  params: {
    name: naming.apim
    location: location
    sku: 'Consumption'
    appInsightsId: appInsights.outputs.appInsightsId
  }
}

// Cosmos DB
module cosmosDb 'modules/cosmos-db.bicep' = {
  scope: rg
  name: 'cosmosDb'
  params: {
    name: naming.cosmosDb
    location: location
    databaseName: '${projectName}-db'
  }
}

// Azure OpenAI
module openai 'modules/openai.bicep' = {
  scope: rg
  name: 'openai'
  params: {
    name: naming.openai
    location: location
    deployments: [
      { name: 'gpt-4o', model: 'gpt-4o', version: '2024-08-06' }
      { name: 'text-embedding-3-small', model: 'text-embedding-3-small', version: '1' }
    ]
  }
}

// Azure AI Search
module aiSearch 'modules/ai-search.bicep' = {
  scope: rg
  name: 'aiSearch'
  params: {
    name: naming.aiSearch
    location: location
    sku: environment == 'prod' ? 'standard' : 'basic'
  }
}

// Container Registry
module acr 'modules/container-registry.bicep' = {
  scope: rg
  name: 'acr'
  params: {
    name: naming.acr
    location: location
    sku: 'Basic'
  }
}

// Key Vault
module keyVault 'modules/key-vault.bicep' = {
  scope: rg
  name: 'keyVault'
  params: {
    name: naming.keyVault
    location: location
  }
}

// Storage Account（画像等）
module storage 'modules/storage-account.bicep' = {
  scope: rg
  name: 'storage'
  params: {
    name: naming.storage
    location: location
    sku: 'Standard_LRS'
  }
}

// =============================================
// 出力
// =============================================
output resourceGroupName string = naming.resourceGroup
output namingConvention object = naming
```

**パラメータファイル（環境別）**

```bicep
// infra/parameters/dev.bicepparam
using '../main.bicep'

param companyName = 'ajinomoto'
param projectName = 'foodllm'
param appName = 'hondashi'
param environment = 'dev'
param location = 'eastus'
```

```bicep
// infra/parameters/prod.bicepparam
using '../main.bicep'

param companyName = 'ajinomoto'
param projectName = 'foodllm'
param appName = 'hondashi'
param environment = 'prod'
param location = 'eastus'
```

---

### インフラデプロイ方法

> **注記**: 社内ネットワークの制約によりローカルAzure CLIが使用できない場合、**Azure Cloud Shell** または **Azure Portal** を使用してください。

---

#### 方法1: Azure Cloud Shell（推奨）

Azure Cloud Shellはブラウザベースのシェル環境で、Azure CLIが事前インストールされています。

**1. Cloud Shellにアクセス**

```
https://shell.azure.com
```

または Azure Portal 右上の Cloud Shell アイコン（ `>_` ）をクリック

**2. 初回セットアップ（ストレージアカウント作成）**

初回アクセス時にストレージアカウントの作成が求められます。「ストレージの作成」をクリックしてください。

**3. Bicepファイルのアップロード**

```bash
# Cloud Shell上でプロジェクトディレクトリを作成
mkdir -p ~/foodllm-infra/infra/modules
mkdir -p ~/foodllm-infra/infra/parameters

# ファイルをアップロード（Cloud Shellのアップロードボタンを使用）
# または、GitHubからクローン
git clone https://github.com/{your-org}/foodllm-chat-platform.git
cd foodllm-chat-platform
```

**4. デプロイ実行**

```bash
# サブスクリプションを確認・設定
az account show
az account set --subscription "{サブスクリプション名またはID}"

# 開発環境デプロイ
az deployment sub create \
  --location eastus \
  --template-file infra/main.bicep \
  --parameters infra/parameters/dev.bicepparam

# ステージング環境デプロイ
az deployment sub create \
  --location eastus \
  --template-file infra/main.bicep \
  --parameters infra/parameters/stg.bicepparam

# 本番環境デプロイ
az deployment sub create \
  --location eastus \
  --template-file infra/main.bicep \
  --parameters infra/parameters/prod.bicepparam
```

**5. デプロイ結果確認**

```bash
# リソースグループ内のリソース一覧
az resource list --resource-group ajinomoto-foodllm-dev-rg --output table
```

---

#### 方法2: Azure Portal（Web UI）でのテンプレートデプロイ

Bicepファイルを使用してAzure Portalからデプロイする方法です。

**1. カスタムテンプレートデプロイにアクセス**

```
https://portal.azure.com/#create/Microsoft.Template
```

または Azure Portal で「カスタムテンプレートのデプロイ」を検索

**2. テンプレートのアップロード**

1. 「エディターで独自のテンプレートを作成する」をクリック
2. 「ファイルの読み込み」をクリック
3. `infra/main.bicep` ファイルをアップロード
4. 「保存」をクリック

**3. パラメータの入力**

| パラメータ | 開発環境の値 | 本番環境の値 |
|-----------|-------------|-------------|
| companyName | ajinomoto | ajinomoto |
| projectName | foodllm | foodllm |
| appName | hondashi | hondashi |
| environment | dev | prod |
| location | eastus | eastus |

**4. デプロイ実行**

1. 「確認と作成」をクリック
2. 検証が成功したら「作成」をクリック
3. デプロイの進行状況を確認

---

#### 方法3: Azure Portal（Web UI）での手動リソース作成

Bicepを使わず、各リソースを個別に作成する方法です。

**作成順序と設定**

| 順序 | リソース | 検索キーワード | 主要設定 |
|------|---------|---------------|---------|
| 1 | リソースグループ | Resource Group | 名前: `ajinomoto-foodllm-dev-rg`, リージョン: East US |
| 2 | Log Analytics ワークスペース | Log Analytics | 名前: `foodllm-log-dev` |
| 3 | Application Insights | Application Insights | 名前: `foodllm-appi-dev`, ワークスペース: 上記を選択 |
| 4 | Azure OpenAI | Azure OpenAI | 名前: `foodllm-oai-dev`, デプロイ: gpt-4o |
| 5 | Azure AI Search | AI Search | 名前: `foodllm-srch-dev`, SKU: Basic |
| 6 | Cosmos DB | Cosmos DB | 名前: `foodllm-cosmos-dev`, API: NoSQL, 容量: Serverless |
| 7 | Key Vault | Key Vault | 名前: `foodllm-kv-dev` |
| 8 | Container Registry | Container Registry | 名前: `foodllmacrdev`, SKU: Basic |
| 9 | Container Apps Environment | Container Apps Environment | 名前: `foodllm-cae-dev` |
| 10 | API Management | API Management | 名前: `foodllm-apim-dev`, SKU: Consumption |
| 11 | Storage Account | Storage Account | 名前: `foodllmstdev`, SKU: Standard LRS |

**各リソースの詳細設定手順**

<details>
<summary>1. Azure OpenAI の作成手順</summary>

1. Azure Portal で「Azure OpenAI」を検索
2. 「作成」をクリック
3. 基本設定:
   - サブスクリプション: 選択
   - リソースグループ: `ajinomoto-foodllm-dev-rg`
   - リージョン: East US
   - 名前: `foodllm-oai-dev`
   - 価格レベル: Standard S0
4. 「確認と作成」→「作成」
5. 作成後、「モデルのデプロイ」からGPT-4oをデプロイ:
   - モデル: gpt-4o
   - デプロイ名: gpt-4o
   - バージョン: 2024-08-06

</details>

<details>
<summary>2. Container Apps Environment の作成手順</summary>

1. Azure Portal で「Container Apps Environment」を検索
2. 「作成」をクリック
3. 基本設定:
   - サブスクリプション: 選択
   - リソースグループ: `ajinomoto-foodllm-dev-rg`
   - 名前: `foodllm-cae-dev`
   - リージョン: East US
4. 監視タブ:
   - Log Analytics ワークスペース: `foodllm-log-dev` を選択
5. 「確認と作成」→「作成」

</details>

<details>
<summary>3. API Management (Consumption) の作成手順</summary>

1. Azure Portal で「API Management」を検索
2. 「作成」をクリック
3. 基本設定:
   - サブスクリプション: 選択
   - リソースグループ: `ajinomoto-foodllm-dev-rg`
   - リージョン: East US
   - リソース名: `foodllm-apim-dev`
   - 組織名: （任意）
   - 管理者メール: （管理者のメールアドレス）
   - 価格レベル: **Consumption**
4. 「確認と作成」→「作成」
5. ※ 作成に20-40分かかる場合があります

</details>

---

#### 方法4: GitHub Actions CI/CD（自動デプロイ）

初回セットアップ完了後は、GitHub ActionsからCI/CDで自動デプロイを行います。

**事前準備（Azure Portal または Cloud Shell で実行）**

```bash
# サービスプリンシパルの作成（Cloud Shellで実行）
az ad sp create-for-rbac \
  --name "foodllm-github-actions" \
  --role contributor \
  --scopes /subscriptions/{subscription-id}/resourceGroups/ajinomoto-foodllm-dev-rg \
  --sdk-auth

# 出力されたJSONをGitHubシークレット「AZURE_CREDENTIALS」に登録
```

**GitHub Secrets設定**

| シークレット名 | 値 |
|--------------|-----|
| `AZURE_CREDENTIALS` | 上記コマンドの出力JSON |
| `ACR_LOGIN_SERVER` | `foodllmacrdev.azurecr.io` |
| `ACR_USERNAME` | Container Registry の ユーザー名 |
| `ACR_PASSWORD` | Container Registry の パスワード |

---

## 📁 推奨プロジェクト構成

```
foodllm-chat-platform/
├── apps/
│   ├── web/                      # フロントエンド (React/Next.js)
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── Chat/
│   │   │   │   │   ├── ChatWindow.tsx
│   │   │   │   │   ├── MessageList.tsx
│   │   │   │   │   ├── ChatInput.tsx
│   │   │   │   │   └── QuickReplies.tsx  # クイックリプライ
│   │   │   │   ├── RecipeCard/
│   │   │   │   └── ProductCard/
│   │   │   ├── hooks/
│   │   │   ├── pages/
│   │   │   └── styles/
│   │   └── package.json
│   │
│   └── widget/                   # 将来: 埋め込みウィジェット
│
├── api/                          # Container Apps バックエンド (FastAPI)
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── main.py                   # FastAPIエントリポイント
│   ├── routers/
│   │   ├── chat.py               # Chat API (SSE対応)
│   │   ├── recipes.py            # レシピ API
│   │   ├── products.py           # 製品・FAQ API
│   │   └── health.py             # ヘルスチェック
│   ├── services/
│   │   ├── llm/                  # LLMプロバイダー抽象化
│   │   ├── rag/                  # RAGエンジン
│   │   ├── suggestions/          # クイックリプライ生成
│   │   ├── database/             # Cosmos DB操作
│   │   └── search/               # AI Search操作
│   └── tests/
│
├── packages/
│   ├── recipe-engine/            # レシピアレンジエンジン
│   │   ├── arrange/
│   │   ├── nutrition/            # 将来: 栄養計算
│   │   └── data/
│   │
│   └── shared/                   # 共通ライブラリ
│       ├── types/
│       ├── utils/
│       └── config/
│
├── infra/                        # IaC (Bicep/Terraform)
│   ├── modules/
│   │   ├── container-apps.bicep
│   │   ├── api-management.bicep
│   │   └── cosmos-db.bicep
│   └── main.bicep
│
├── docker-compose.yml            # ローカル開発環境
├── docker-compose.prod.yml       # 本番用（参考）
│
├── data/
│   ├── hondashi/                 # ほんだしデータ
│   │   ├── recipes/
│   │   ├── products/
│   │   └── faq/
│   ├── protein/                  # 将来: たんぱく質
│   └── cooking-basics/           # 将来: 料理初心者
│
├── prompts/                      # プロンプトテンプレート
│   ├── hondashi/
│   │   ├── system.md
│   │   ├── recipe_suggestion.md
│   │   └── product_recommendation.md
│   └── shared/
│
├── config/                       # 設定ファイル
│   ├── usecases/                 # ユースケース設定
│   │   ├── hondashi.yaml
│   │   ├── protein.yaml          # Phase 2で実装
│   │   └── cooking_beginner.yaml # Phase 2で実装
│   └── suggestions/              # クイックリプライ設定
│       ├── hondashi.yaml
│       ├── protein.yaml          # Phase 2で実装
│       └── cooking_beginner.yaml # Phase 2で実装
│
├── docs/
├── tests/
└── README.md
```

---

## 🔄 マルチユースケース拡張設計

### 核となる考え方：「共通基盤 + ユースケース固有設定」

```
┌─────────────────────────────────────────────────────────────────────┐
│                     共通プラットフォーム基盤                          │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────┐ │
│  │ Chat Engine  │ │ RAG Engine   │ │ Recipe Engine│ │ Analytics  │ │
│  │ (LLM呼び出し)│ │ (検索+生成)   │ │ (アレンジ)   │ │ (ログ分析) │ │
│  └──────────────┘ └──────────────┘ └──────────────┘ └────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                                │
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
            ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
            │  ほんだし   │ │ たんぱく質  │ │ 料理初心者  │
            │  Config     │ │  Config     │ │  Config     │
            └─────────────┘ └─────────────┘ └─────────────┘
```

### ユースケース設定の抽象化

```typescript
// packages/api/shared/usecase/types.ts

interface UseCaseConfig {
  id: 'hondashi' | 'protein' | 'cooking_beginner';
  
  // プロンプト設定
  prompts: {
    system: string;
    recipeRecommendation: string;
    productRecommendation: string;
    custom?: Record<string, string>;
  };
  
  // RAG設定
  rag: {
    indexName: string;
    searchFields: string[];
    semanticConfig: string;
  };
  
  // 機能フラグ
  features: {
    recipeArrange: boolean;
    nutritionCalc: boolean;
    exerciseSuggestion: boolean;
    cookingTerms: boolean;
  };
  
  // UI設定
  ui: {
    theme: 'hondashi' | 'protein' | 'beginner';
    welcomeMessage: string;
    suggestedQuestions: string[];
  };
  
  // 認証設定
  auth: {
    required: boolean;
    allowedUserTypes?: string[];
  };
}
```

### 各ユースケースの特徴

| 拡張項目 | ほんだし (Phase 1) | たんぱく質 (Phase 2) | 料理初心者 (Phase 2) |
|----------|-------------------|---------------------|---------------------|
| **RAGデータ** | レシピ、製品、FAQ | 健康情報、運動、製品 | 基本レシピ、調理基礎 |
| **システムプロンプト** | 和食・だし専門 | 健康・栄養アドバイザー | 優しい料理の先生 |
| **レシピエンジン** | アレンジ機能 | + 栄養計算 | + 簡単度調整 |
| **認証** | 不要 | 購入者限定 | 不要 |
| **追加機能** | - | 運動提案 | 調理用語解説 |

---

## 📄 ユースケース設定例

### たんぱく質ユースケース

```yaml
# config/usecases/protein.yaml
id: protein
displayName: "たんぱく質相談サービス"

prompts:
  system: |
    あなたはfoodllmのたんぱく質・健康アドバイザーです。
    以下の役割を担います：
    - たんぱく質摂取に関する質問への回答
    - 筋肉維持・体づくりのアドバイス
    - 適切な食事・レシピの提案
    - 運動に関する基本的なアドバイス
    - foodllmのたんぱく質関連製品の紹介
    
    回答は科学的根拠に基づき、親しみやすく丁寧に行ってください。
    医療的なアドバイスは控え、必要に応じて専門家への相談を勧めてください。

rag:
  indexName: "protein-knowledge-index"
  searchFields: ["content", "title", "tags"]
  semanticConfig: "protein-semantic"

features:
  recipeArrange: true
  nutritionCalc: true
  exerciseSuggestion: true
  cookingTerms: false

ui:
  theme: protein
  welcomeMessage: "こんにちは！たんぱく質や健康づくりについてお気軽にご相談ください。"
  suggestedQuestions:
    - "1日に必要なたんぱく質量は？"
    - "筋肉を維持するための食事のコツは？"
    - "たんぱく質が摂れる簡単レシピを教えて"

auth:
  required: true
  allowedUserTypes: ["protein_product_purchaser"]
```

### 料理初心者ユースケース

```yaml
# config/usecases/cooking_beginner.yaml
id: cooking_beginner
displayName: "料理初心者サポート"

prompts:
  system: |
    あなたは料理初心者に寄り添う、優しい料理の先生です。
    以下の役割を担います：
    - 料理の基本的な疑問への丁寧な回答
    - 簡単で失敗しにくいレシピの提案
    - 調理用語や手順のわかりやすい解説
    - 初心者が陥りやすいミスの予防アドバイス
    - foodllm製品を活用した簡単調理の提案
    
    専門用語を避け、初心者目線でステップバイステップに説明してください。
    「大丈夫」「簡単にできます」など励ましの言葉も添えてください。

rag:
  indexName: "cooking-basics-index"
  searchFields: ["content", "title", "difficulty_level"]
  semanticConfig: "cooking-basics-semantic"

features:
  recipeArrange: true
  nutritionCalc: false
  exerciseSuggestion: false
  cookingTerms: true

ui:
  theme: beginner
  welcomeMessage: "料理のことなら何でも聞いてください！初めての方も大歓迎です 🍳"
  suggestedQuestions:
    - "包丁の基本的な使い方は？"
    - "失敗しない味噌汁の作り方"
    - "「ひとつまみ」ってどれくらい？"

auth:
  required: false
```

---

## 🔧 ユースケース固有機能

### たんぱく質：栄養計算 + 運動提案

```python
# packages/api/functions/protein/nutrition.py

class NutritionCalculator:
    """たんぱく質・栄養計算エンジン"""
    
    async def calculate_recipe_nutrition(
        self, 
        recipe: Recipe
    ) -> NutritionInfo:
        """レシピの栄養価計算"""
        return NutritionInfo(
            protein=...,
            calories=...,
            fat=...,
            carbs=...,
        )
    
    async def suggest_daily_plan(
        self,
        user_profile: UserProfile,
        goal: str
    ) -> DailyPlan:
        """1日の食事・運動プラン提案"""
        pass


class ExerciseSuggestionEngine:
    """運動提案エンジン"""
    
    async def suggest_exercises(
        self,
        user_profile: UserProfile,
        goal: str,
        available_time: int
    ) -> List[Exercise]:
        """適切な運動を提案"""
        pass
```

### 料理初心者：調理用語解説

```python
# packages/api/functions/cooking_beginner/terms.py

class CookingTermsExplainer:
    """調理用語解説エンジン"""
    
    def __init__(self):
        self.terms_db = load_cooking_terms_database()
    
    async def explain_term(
        self, 
        term: str,
        context: str = None
    ) -> TermExplanation:
        """
        調理用語をわかりやすく解説
        例: "ひとつまみ" → "親指、人差し指、中指の3本でつまんだ量（約1g）"
        """
        pass
    
    async def simplify_recipe_instructions(
        self,
        original_instructions: str
    ) -> str:
        """
        レシピの手順を初心者向けに書き換え
        専門用語を展開し、ステップを細分化
        """
        pass
```

---

## 📊 データ構造

### Azure AI Search: マルチインデックス構成

```
AI Search インスタンス
├── hondashi-recipe-index      # ほんだしレシピ
├── hondashi-product-index     # ほんだし製品
├── hondashi-faq-index         # ほんだしFAQ
│
├── protein-knowledge-index    # 健康・栄養情報
├── protein-recipe-index       # たんぱく質レシピ
├── protein-exercise-index     # 運動情報
│
├── cooking-basics-index       # 料理基礎
├── cooking-terms-index        # 調理用語辞典
└── beginner-recipe-index      # 初心者向けレシピ
```

### Cosmos DB: 会話ログ構造

```json
{
  "id": "conv_xxx",
  "sessionId": "sess_xxx",
  "userId": "anonymous_xxx",
  "useCase": "hondashi",
  "messages": [
    {
      "role": "user",
      "content": "味噌汁を美味しく作るコツは？",
      "timestamp": "2025-01-02T10:00:00Z",
      "metadata": {
        "intent": "cooking_tips",
        "topics": ["味噌汁", "調理法"],
        "inputMethod": "typed"
      }
    },
    {
      "role": "assistant", 
      "content": "...",
      "timestamp": "2025-01-02T10:00:05Z",
      "metadata": {
        "recipesRecommended": ["recipe_001"],
        "productsRecommended": ["hondashi_standard"],
        "suggestionsShown": [
          {"label": "減塩にしたい", "value": "..."},
          {"label": "具材を変えたい", "value": "..."}
        ]
      }
    },
    {
      "role": "user",
      "content": "このレシピを減塩にアレンジして",
      "timestamp": "2025-01-02T10:00:30Z",
      "metadata": {
        "intent": "recipe_arrange",
        "inputMethod": "suggestion_click",
        "suggestionClicked": {"label": "減塩にしたい", "index": 0}
      }
    }
  ],
  "analytics": {
    "duration": 300,
    "messageCount": 6,
    "suggestionClickRate": 0.5,
    "satisfaction": null,
    "primaryIntent": "recipe_help"
  },
  "createdAt": "2025-01-02T10:00:00Z"
}
```

### Cosmos DB: パーティション設計（MVP版）

| コンテナ名 | パーティションキー | 設計理由 |
|-----------|------------------|---------|
| sessions | `/id` | セッション単体での取得が主なアクセスパターン |
| conversations | `/session_id` | 同一セッション内の会話一覧取得を効率化 |
| messages | `/conversation_id` | 同一会話内のメッセージ一覧取得を効率化 |
| feedback | `/conversation_id` | 会話単位でのフィードバック集計を効率化 |
| recipes, products, product_faq | `/use_case_id` | ユースケース別のデータ分離 |

```json
{
  "conversations": {
    "partitionKey": "/session_id",
    "comment": "セッション単位で会話をグループ化",
    "documents": [
      {
        "id": "conv_xxx",
        "session_id": "session_123",
        "use_case_id": "hondashi",
        "status": "active",
        "messages": [...]
      }
    ]
  }
}
```

> **設計方針（MVP版）**: 
> - MVP版ではシンプルな設計を優先し、`session_id`/`conversation_id`ベースのパーティションを採用
> - 想定トラフィック（100ユーザー同時接続）では現設計で十分なパフォーマンス
> - Phase 2以降でトラフィック増加時は `{useCase}_{yearMonth}` 複合キーへの移行を検討
> - 将来的にユーザー数増加時は `/userId` ベースへの移行も視野に入れる

---

## 📅 Phase 2 詳細タイムライン

### Month 2

#### Week 1-2: 共通基盤のリファクタリング
- UseCaseConfig 抽象化
- Chat Engine の設定駆動化
- RAG Engine のマルチインデックス対応

#### Week 3-4: たんぱく質ユースケース
- データ収集・インデックス作成
- システムプロンプト調整
- 栄養計算エンジン統合
- 認証機能（簡易版）

### Month 3

#### Week 1-2: 料理初心者ユースケース
- データ収集・インデックス作成
- システムプロンプト調整
- 調理用語解説機能

#### Week 3-4: 統合テスト・最適化
- 全ユースケース横断テスト
- プロンプト最適化
- UI/UXの統一感調整

---

## 🎯 新ユースケース追加時の作業

| 作業項目 | 作業量 |
|---------|--------|
| 1. 設定ファイル（YAML）作成 | 小 |
| 2. システムプロンプト作成 | 中 |
| 3. RAGデータ収集・インデックス作成 | 中〜大 |
| 4. ユースケース固有機能開発（必要な場合） | 中〜大 |
| 5. UIテーマ調整 | 小 |
| 6. テスト | 中 |

**コア基盤（Chat Engine, RAG Engine, レシピエンジン）は再利用可能**

---

## ✅ 次のステップ

1. **今すぐ**: Azure環境の準備開始（Azure OpenAI申請含む）
2. **Week 1**: Phase 0 の基盤構築
3. **Week 2-4**: MVP開発 → リリース

---

## 📝 備考

- MVPリリース目標: 1ヶ月以内
- 初期ユースケース: ほんだし
- 将来拡張: たんぱく質関連製品、料理初心者支援
- 既存システム連携: 初期では不要、将来的に検討

---

## ⚠️ リスクと対策

### 技術リスク

| リスク | 影響度 | 発生確率 | 対策 |
|--------|-------|---------|------|
| Azure OpenAI 申請承認の遅延 | 高 | 中 | 早期申請（Week 0で即時申請）、承認待ち期間は他タスク並行 |
| RAG検索精度が低い | 高 | 中 | プロンプト調整、チャンク戦略見直し、Hybrid Search活用 |
| LLMハルシネーション | 高 | 高 | RAG強化、システムプロンプトでの制約、回答の出典明示 |
| LLM応答遅延（>5秒） | 中 | 中 | ストリーミング応答、タイムアウト設定、ローディングUI |

### スケジュールリスク

| リスク | 影響度 | 発生確率 | 対策 |
|--------|-------|---------|------|
| 1ヶ月でMVP完成できない | 高 | 中 | 機能の優先順位明確化、スコープ調整の判断基準事前設定 |
| データ収集・整備の遅延 | 高 | 中 | 早期着手、最小限データセットでの開発開始 |
| 開発リソース不足 | 中 | 低 | タスクの優先順位付け、外部リソース活用検討 |

### 対応プロセス

```
リスク発生時の対応フロー:
1. リスク検知 → 影響範囲の特定
2. 対策オプションの検討（スコープ調整 / スケジュール調整 / リソース追加）
3. ステークホルダーへの報告・合意
4. 対策実行・モニタリング
```

---

## 💰 コスト見積もり

### Azure サービス月額概算

#### MVP期間（Phase 0-1）

| サービス | SKU | 月額概算（税抜） | 備考 |
|---------|-----|-----------------|------|
| Azure OpenAI | GPT-4o | ¥30,000〜50,000 | 1日1,000会話想定 |
| Azure AI Search | Basic | ¥10,000 | 1インデックス |
| Azure Cosmos DB | Serverless | ¥3,000〜5,000 | 低トラフィック想定 |
| **Azure Container Apps** | **Consumption** | **¥5,000〜10,000** | **SSE対応、最小0〜1インスタンス** |
| **Azure API Management** | **Consumption** | **¥1,500〜3,000** | **レート制限、APIキー管理** |
| Azure Container Registry | Basic | ¥700 | コンテナイメージ保存 |
| Azure Static Web Apps | Free | ¥0 | MVP期間 |
| Azure Key Vault | Standard | ¥500 | |
| Azure Monitor | - | ¥2,000 | 基本ログ |
| Prompt Flow（評価のみ） | - | ¥1,000〜3,000 | 品質評価 |
| **合計** | | **¥53,700〜84,200** | |

#### コスト内訳詳細

**Azure Container Apps課金**

| リソース | 単価 | MVP想定 |
|---------|------|--------|
| vCPU | ¥6.5/vCPU時間 | 0.5 vCPU × 24h × 30日 = ¥2,340 |
| メモリ | ¥0.65/GiB時間 | 1 GiB × 24h × 30日 = ¥468 |
| リクエスト | ¥0.05/10,000 | 30万/月 = ¥1.5 |
| スケールアウト時（ピーク） | | +¥2,000〜5,000 |
| **小計** | | **¥5,000〜10,000** |

**Azure API Management (Consumption) 課金**

| 項目 | 単価 | MVP想定 |
|------|------|--------|
| 基本料金 | ¥0（従量課金） | ¥0 |
| API呼び出し | ¥0.5/10,000呼び出し | 30万/月 = ¥1,500 |
| **小計** | | **¥1,500〜3,000** |

#### 本番運用期間（Phase 2以降）

| サービス | SKU | 月額概算（税抜） | 備考 |
|---------|-----|-----------------|------|
| Azure OpenAI | GPT-4o | ¥100,000〜200,000 | 1日5,000会話想定 |
| Azure AI Search | Standard S1 | ¥30,000 | 複数インデックス |
| Azure Cosmos DB | Provisioned | ¥15,000〜30,000 | 400-800 RU/s |
| Azure Container Apps | Consumption | ¥15,000〜25,000 | スケールアウト対応 |
| Azure API Management | Developer | ¥7,000 | 高度な機能 |
| Azure Static Web Apps | Standard | ¥1,500 | |
| Azure Container Registry | Basic | ¥700 | |
| ガードレール関連 | - | ¥10,000〜23,000 | Content Safety等 |
| **合計** | | **¥179,200〜317,200** | |

### 開発工数見積もり

| フェーズ | 期間 | 想定工数 | 主な作業 |
|---------|------|---------|---------|
| Phase 0 | 1週間 | 5人日 | インフラ構築、データ収集 |
| Phase 1 | 3週間 | 15人日 | MVP開発 |
| Phase 2 | 2ヶ月 | 40人日 | 機能拡張、マルチユースケース |
| Phase 3 | 2ヶ月 | 30人日 | 認証、ウィジェット、最適化 |

---

## 🔒 セキュリティ・コンプライアンス

### 個人情報の取り扱い

| 項目 | 方針 |
|------|------|
| 収集する情報 | 会話内容、セッションID、タイムスタンプ（個人を特定する情報は収集しない） |
| 利用目的 | サービス改善、製品開発のための分析 |
| 第三者提供 | 行わない（匿名化・統計化したデータの社内利用のみ） |
| 同意取得 | チャット開始時に利用規約・プライバシーポリシーへの同意を取得 |

### データ保持・削除ポリシー

| データ種別 | 保持期間 | 削除方法 | MVP対象 |
|-----------|---------|---------|--------|
| 会話ログ | 90日間 | 自動削除（TTL設定） | ✅ |
| セッション情報 | 90日間 | 自動削除（TTL設定） | ✅ |
| フィードバック | 90日間 | 自動削除（TTL設定） | ✅ |
| サジェスト | 90日間 | 自動削除（TTL設定） | ✅ |
| 分析用集計データ | 5年間 | 手動削除 | **Phase 2以降** |
| エラーログ | 90日間 | 自動削除 | ✅ |

> **注記**: 
> - MVP版では全会話関連データを**90日間保持に統一**
> - 「分析用集計データ」（KPI集計、トレンド分析等）は**Phase 2以降**で実装・運用開始
> - 法務・コンプライアンス要件により変更が必要な場合は、全仕様書を同期して更新すること

### セキュリティ対策

```
実装するセキュリティ対策:
├── 通信暗号化: HTTPS必須（TLS 1.2以上）
├── 認証・認可: Azure AD B2C（Phase 3）、API Key（MVP）
├── シークレット管理: Azure Key Vault
├── 入力検証: XSS・インジェクション対策
├── レート制限: API Management (Consumption)（MVP）→ Developer（本番）
├── 監査ログ: Application Insights
└── 脆弱性対策: 依存パッケージの定期更新、Dependabot
```

### 免責事項（特にたんぱく質ユースケース）

```
表示必須の免責事項:
- 本サービスは一般的な健康情報の提供を目的としており、医療アドバイスではありません
- 健康上の問題がある場合は、必ず医師や専門家にご相談ください
- 提供する栄養情報は参考値であり、正確性を保証するものではありません
- 運動提案は一般的なものであり、個人の健康状態に応じた指導ではありません
```

---

## 📈 KPI・成功指標

### MVP成功の定義

| 指標 | 目標値 | 測定方法 |
|------|-------|---------|
| 基本動作 | エラー率 < 5% | Application Insights |
| 応答速度 | 平均 < 3秒 | Application Insights |
| 会話完了率 | > 70% | Cosmos DB分析 |
| レシピ提案精度 | 関連性 > 80%（社内評価） | 手動サンプリング |

### 本番運用KPI

#### エンゲージメント指標

| 指標 | 目標値 | 測定頻度 |
|------|-------|---------|
| 月間アクティブ会話数 | Phase 2: 5,000件 | 月次 |
| 平均会話ターン数 | > 3ターン | 週次 |
| リピート率 | > 20% | 月次 |
| クイックリプライ利用率 | > 40% | 週次 |

#### 品質指標

| 指標 | 目標値 | 測定頻度 |
|------|-------|---------|
| ユーザー満足度（フィードバック） | > 4.0/5.0 | 月次 |
| ハルシネーション報告率 | < 2% | 週次 |
| エスカレーション率（人間対応必要） | < 5% | 週次 |

#### ビジネス指標

| 指標 | 目標値 | 測定頻度 |
|------|-------|---------|
| レシピページへの誘導数 | 月間 1,000件 | 月次 |
| 製品ページへの誘導数 | 月間 500件 | 月次 |
| 会話からの有用インサイト抽出数 | 月間 10件 | 月次 |

### ダッシュボード構成

```
リアルタイムダッシュボード（Application Insights / Power BI）:
├── 会話数（時間別、日別）
├── 平均応答時間
├── エラー率
├── トピック分布
├── クイックリプライ利用状況
└── ユーザーフィードバック

週次レポート:
├── KPI達成状況
├── 頻出質問・トピック分析
├── 改善機会の特定
└── プロンプト調整提案
```

---

## 🛠️ 運用・監視計画

### 監視体制

#### Phase 1（MVP）

| 項目 | ツール | アラート条件 |
|------|-------|-------------|
| API可用性 | Application Insights | 可用性 < 99% |
| 応答時間 | Application Insights | P95 > 5秒 |
| エラー率 | Application Insights | エラー率 > 5% |
| Azure OpenAI クォータ | Azure Monitor | 使用率 > 80% |

#### Phase 2以降

| 項目 | ツール | アラート条件 |
|------|-------|-------------|
| 上記全て | - | - |
| Cosmos DB RU消費 | Azure Monitor | 使用率 > 80% |
| AI Search クエリ数 | Azure Monitor | 上限の80%到達 |
| 異常な会話パターン | カスタム分析 | 不適切コンテンツ検知 |

### アラート通知先

```yaml
# 通知設定
alerts:
  critical:  # サービス停止レベル
    channels: [email, teams, phone]
    recipients: [開発リード, インフラ担当]
    response_time: 15分以内
    
  warning:  # パフォーマンス低下レベル
    channels: [email, teams]
    recipients: [開発チーム]
    response_time: 1時間以内
    
  info:  # 情報レベル
    channels: [teams]
    recipients: [開発チーム]
    response_time: 翌営業日
```

### エラーハンドリング方針

| エラー種別 | ユーザーへの表示 | システム対応 |
|-----------|----------------|-------------|
| LLMタイムアウト | 「回答の生成に時間がかかっています。もう一度お試しください」 | リトライ1回、ログ記録 |
| LLMエラー | 「一時的なエラーが発生しました。しばらくしてからお試しください」 | アラート発報、ログ記録 |
| RAG検索失敗 | 「関連情報が見つかりませんでした。質問を変えてお試しください」 | フォールバック応答 |
| 不適切入力検知 | 「申し訳ございませんが、その質問にはお答えできません」 | ログ記録、パターン分析 |

### 障害対応フロー

```
障害検知
    │
    ▼
┌─────────────────┐
│ 影響範囲の特定   │ ← 5分以内
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ 一次対応        │ ← 15分以内
│ ・メンテナンス表示
│ ・ログ収集
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ 原因調査・復旧   │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ ポストモーテム   │ ← 48時間以内
│ ・原因分析
│ ・再発防止策
└─────────────────┘
```

### 定期メンテナンス

| 作業 | 頻度 | 内容 |
|------|------|------|
| 依存パッケージ更新 | 週次 | セキュリティパッチ適用 |
| プロンプト見直し | 隔週 | 品質改善、新パターン対応 |
| RAGデータ更新 | 月次 | 新レシピ・製品追加 |
| コスト最適化レビュー | 月次 | リソース使用状況分析 |
| バックアップ検証 | 月次 | リストア手順確認 |

---

## 👥 プロジェクト体制

### チーム構成（MVP）

| 役割 | 人数 | 主な責務 |
|------|------|---------|
| プロジェクトマネージャー | 1名 | 進捗管理、ステークホルダー調整 |
| テックリード | 1名 | アーキテクチャ設計、技術判断、コードレビュー |
| バックエンドエンジニア | 2名 | Container Apps (FastAPI) API開発、RAG実装 |
| フロントエンドエンジニア | 1名 | React/Next.js UI開発 |
| MLエンジニア | 1名（兼務可） | プロンプトエンジニアリング、LLM品質評価 |

**想定工数（MVP: 4週間）**
- 合計: 約80人日（5名 × 16日）
- 内訳: 基盤構築10人日、バックエンド25人日、フロントエンド20人日、テスト15人日、バッファ10人日

### 連絡体制

| シーン | 手段 | 頻度 |
|--------|------|------|
| 日次スタンドアップ | Slack/Teams | 毎日10:00 |
| 週次進捗会議 | オンライン会議 | 週1回 |
| 緊急連絡 | 電話 + Slack | 随時 |
| ドキュメント共有 | Confluence/Notion | 随時 |

---

## 🧩 RAGチャンク設計

### チャンク戦略

| データ種別 | チャンクサイズ | オーバーラップ | 戦略 |
|-----------|--------------|--------------|------|
| レシピ | 512トークン | 128トークン | タイトル+説明+材料で1チャンク、手順は別チャンク |
| 製品情報 | 512トークン | 128トークン | 製品単位で1チャンク |
| FAQ | 256トークン | 64トークン | 1つのQ&Aペアで1チャンク |

### エンベディング設定

| 項目 | 設定値 |
|------|--------|
| モデル | text-embedding-ada-002 |
| 次元数 | 1536 |
| Azure AI Search セマンティック構成 | `hondashi-semantic-config` |

### インデックス構成

| インデックス名 | 用途 | 主要フィールド |
|---------------|------|---------------|
| `hondashi-recipes-index` | レシピ検索 | title, description, ingredients, tags |
| `hondashi-products-index` | 製品検索 | name, description, category, usage |
| `hondashi-faq-index` | FAQ検索 | question, answer, keywords, category |

### Hybrid Search設定

```json
{
  "search_mode": "hybrid",
  "vector_weight": 0.7,
  "keyword_weight": 0.3,
  "top_k": 5,
  "reranker": "semantic"
}
```

> **チューニング方針**: 
> - 初期は上記設定で運用開始
> - ユーザーフィードバック・LLM品質評価を基に重み調整
> - Phase 2以降で機械学習ベースのリランキング導入を検討

---

## 📖 用語集（Glossary）

| 用語 | 説明 |
|------|------|
| **RAG** | Retrieval-Augmented Generation。検索で取得した情報をLLMに与えて回答を生成する手法 |
| **LLM** | Large Language Model。大規模言語モデル（GPT-4o等） |
| **ハルシネーション** | LLMが事実と異なる情報を生成してしまう現象 |
| **プロンプト** | LLMへの指示文。システムプロンプト（役割定義）とユーザープロンプト（質問）がある |
| **ベクトル検索** | テキストを数値ベクトルに変換し、意味的な類似度で検索する手法 |
| **セマンティック検索** | キーワード一致ではなく、意味・文脈を理解した検索 |
| **クイックリプライ** | ユーザーの入力候補をボタンとして表示する機能 |
| **サジェスト** | クイックリプライで表示する候補の内容 |
| **ユースケース** | 本プロジェクトにおける活用シナリオ（ほんだし、たんぱく質、料理初心者） |
| **MVP** | Minimum Viable Product。最小限の機能で動作する製品 |
| **Cosmos DB** | Azure のNoSQLデータベースサービス |
| **AI Search** | Azure の検索サービス（旧 Cognitive Search） |
| **Container Apps** | コンテナ化されたアプリケーションを実行するAzureサービス |
| **API Management** | API管理・認証・レート制限を行うAzureサービス |
| **Static Web Apps** | 静的Webサイト・SPAをホスティングするAzureサービス |
| **Application Insights** | アプリケーションの監視・分析サービス |
| **TTL** | Time To Live。データの自動削除までの期間 |
| **RU** | Request Unit。Cosmos DBの処理能力単位 |
| **FastAPI** | Python製の高速Webフレームワーク。非同期処理・SSE対応 |

---

## 📎 付録

### A. 関連ドキュメントリンク

- [Azure OpenAI Service ドキュメント](https://learn.microsoft.com/azure/ai-services/openai/)
- [Azure AI Search ドキュメント](https://learn.microsoft.com/azure/search/)
- [Azure Cosmos DB ドキュメント](https://learn.microsoft.com/azure/cosmos-db/)
- [Azure Container Apps ドキュメント](https://learn.microsoft.com/azure/container-apps/)
- [Azure API Management ドキュメント](https://learn.microsoft.com/azure/api-management/)
- [FastAPI ドキュメント](https://fastapi.tiangolo.com/)

### B. 参考アーキテクチャ

- [Azure OpenAI + AI Search によるRAG実装パターン](https://learn.microsoft.com/azure/architecture/ai-ml/architecture/rag-openai)
- [会話型AIのベストプラクティス](https://learn.microsoft.com/azure/architecture/ai-ml/architecture/conversational-bot)
- [Container Apps マイクロサービスパターン](https://learn.microsoft.com/azure/container-apps/microservices)

### C. 更新履歴

| 日付 | バージョン | 更新内容 |
|------|-----------|---------|
| 2025-01-02 | 1.0 | 初版作成 |
| 2025-01-02 | 1.1 | クイックリプライ機能追加 |
| 2025-01-02 | 1.2 | リスク、コスト、セキュリティ、KPI、運用計画、用語集を追加 |
| 2025-01-02 | 1.3 | 他仕様書との整合性修正（保持期間90日統一、言語Python統一、E2E4シナリオ統一）、チーム構成追加、RAGチャンク設計追加、CI/CDパイプライン詳細追加、製品FAQ対応追加 |
| 2025-01-03 | 1.4 | Azure OpenAI申請リードタイム明記、データ投入手順追加（スクリプト例含む）、前提条件セクション追加 |
| 2025-01-03 | 1.5 | ブランドカラー#E60012に統一、SSE技術検証セクション追加、画像アセットホスティング追加、分析用集計データPhase2以降と明記 |
| 2025-01-03 | 1.6 | Azure Functions → Container Apps変更、API Management (Consumption) MVP導入、アーキテクチャ図更新、プロジェクト構成更新、コスト見積もり更新 |
| 2025-01-03 | 1.7 | リージョンをEast US (eastus)に統一、IaC (Bicep)テンプレート例追加、環境別設定例追加 |
| 2025-01-03 | 1.8 | Azureリソース命名規則設定セクション追加、命名規則設定ファイル(naming.yaml)追加、Bicepテンプレートを命名規則対応版に更新 |
| 2025-01-03 | 1.9 | Azure Cloud Shell / Azure Portal でのインフラデプロイ手順追加（ローカルAzure CLI不要に対応） |
