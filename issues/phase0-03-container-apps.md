# [Phase 0-3] Container Apps Environment 作成

## 概要
Azure Container Apps Environmentを作成し、FastAPI バックエンドをホストする基盤を構築する。

## タスク
- [ ] Log Analytics Workspace `foodllm-log-dev` を作成
- [ ] Container Apps Environment `foodllm-cae-dev` を作成
- [ ] Container App `foodllm-hondashi-ca-dev` を作成（初期設定）
- [ ] スケーリング設定（min: 0, max: 10）
- [ ] ネットワーク設定（Ingress有効化）

## 技術詳細
- **Environment名**: `foodllm-cae-dev`
- **Container App名**: `foodllm-hondashi-ca-dev`
- **リージョン**: East US
- **スケーリング**: 最小0、最大10インスタンス

## 環境変数設定
Container Appに以下の環境変数を設定：
- `AZURE_OPENAI_ENDPOINT`
- `AZURE_OPENAI_API_KEY`（Key Vault参照）
- `COSMOS_DB_ENDPOINT`
- `AI_SEARCH_ENDPOINT`

## 完了条件
- [ ] Container Apps Environmentが作成されている
- [ ] Container Appが作成され、Ingressが有効
- [ ] Log Analyticsと連携されている
- [ ] スケーリング設定が完了している

## ラベル
`phase-0`, `infrastructure`, `azure`, `container-apps`
