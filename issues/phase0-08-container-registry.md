# [Phase 0-8] Container Registry 作成

## 概要
Azure Container Registryを作成し、Dockerイメージの保存・管理基盤を構築する。

## タスク
- [ ] Container Registry `foodllmacrdev` を作成
- [ ] 管理者ユーザーの有効化
- [ ] Container Appsとの連携設定
- [ ] イメージプル権限の設定

## 技術詳細
- **リソース名**: `foodllmacrdev`（ハイフン使用不可）
- **SKU**: Basic
- **リージョン**: East US

## コマンド例
```bash
az acr create \
  --name foodllmacrdev \
  --resource-group ajinomoto-foodllm-dev-rg \
  --location eastus \
  --sku Basic \
  --admin-enabled true
```

## イメージ命名規則
```
foodllmacrdev.azurecr.io/foodllm-api:<tag>
```

## 完了条件
- [ ] Container Registryが作成されている
- [ ] 管理者ユーザーが有効化されている
- [ ] Container Appsからイメージをプル可能

## ラベル
`phase-0`, `infrastructure`, `azure`, `container`
