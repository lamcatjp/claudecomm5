# [Phase 0-2] Azure OpenAI リソース作成・モデルデプロイ

## 概要
Azure OpenAI Serviceのリソースを作成し、GPT-4oモデルをデプロイする。

## 前提条件
- [ ] Azure OpenAI Serviceの利用申請が承認済み（1〜2週間必要）

## タスク
- [ ] Azure OpenAI リソース `foodllm-oai-dev` を作成
- [ ] GPT-4o モデルをデプロイ（デプロイ名: `gpt-4o`）
- [ ] text-embedding-ada-002 モデルをデプロイ（RAG用）
- [ ] APIキーを取得しKey Vaultに保存
- [ ] クォータ設定の確認・調整

## 技術詳細
- **リソース名**: `foodllm-oai-dev`
- **リージョン**: East US
- **モデル**:
  - GPT-4o（チャット用）
  - text-embedding-ada-002（ベクトル検索用）

## 環境変数
```bash
AZURE_OPENAI_ENDPOINT=https://foodllm-oai-dev.openai.azure.com/
AZURE_OPENAI_API_KEY=<Key Vault管理>
AZURE_OPENAI_DEPLOYMENT_NAME=gpt-4o
```

## 完了条件
- [ ] Azure OpenAIリソースが作成されている
- [ ] GPT-4oモデルがデプロイされ、API呼び出し可能
- [ ] Embeddingモデルがデプロイされている
- [ ] APIキーがKey Vaultに保存されている

## ラベル
`phase-0`, `infrastructure`, `azure`, `llm`
