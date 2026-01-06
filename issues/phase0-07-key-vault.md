# [Phase 0-7] Azure Key Vault 作成・シークレット管理設定

## 概要
Azure Key Vaultを作成し、各種APIキーと接続文字列を安全に管理する。

## タスク
- [ ] Key Vault `foodllm-kv-dev` を作成
- [ ] アクセスポリシー設定
- [ ] シークレット登録:
  - `azure-openai-api-key`
  - `cosmos-db-key`
  - `ai-search-api-key`
  - `mvp-access-password-hash`
- [ ] Container Appsからのマネージドアイデンティティアクセス設定

## 技術詳細
- **リソース名**: `foodllm-kv-dev`
- **リージョン**: East US
- **SKU**: Standard

## シークレット一覧
| シークレット名 | 説明 |
|--------------|------|
| `azure-openai-api-key` | Azure OpenAI APIキー |
| `cosmos-db-key` | Cosmos DB接続キー |
| `ai-search-api-key` | AI Search APIキー |
| `mvp-access-password-hash` | MVPアクセスパスワードのSHA-256ハッシュ |

## コマンド例
```bash
az keyvault create \
  --name foodllm-kv-dev \
  --resource-group ajinomoto-foodllm-dev-rg \
  --location eastus

az keyvault secret set \
  --vault-name foodllm-kv-dev \
  --name azure-openai-api-key \
  --value "<APIキー>"
```

## 完了条件
- [ ] Key Vaultが作成されている
- [ ] 全シークレットが登録されている
- [ ] Container Appsからアクセス可能

## ラベル
`phase-0`, `infrastructure`, `azure`, `security`
