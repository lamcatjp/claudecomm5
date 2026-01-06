# [Phase 0-6] Azure AI Search インスタンス作成

## 概要
Azure AI Searchインスタンスを作成し、RAG用のベクトル検索基盤を構築する。

## タスク
- [ ] AI Search `foodllm-srch-dev` を作成
- [ ] インデックス作成:
  - `hondashi-recipes-index`（レシピ）
  - `hondashi-products-index`（製品）
  - `hondashi-faq-index`（FAQ）
- [ ] セマンティック検索の有効化
- [ ] ベクトルフィールドの設定
- [ ] APIキーをKey Vaultに保存

## 技術詳細
- **リソース名**: `foodllm-srch-dev`
- **SKU**: Basic
- **リージョン**: East US
- **機能**: セマンティック検索、ベクトル検索

## インデックススキーマ（レシピ例）
```json
{
  "name": "hondashi-recipes-index",
  "fields": [
    {"name": "id", "type": "Edm.String", "key": true},
    {"name": "title", "type": "Edm.String", "searchable": true},
    {"name": "description", "type": "Edm.String", "searchable": true},
    {"name": "ingredients", "type": "Collection(Edm.String)", "searchable": true},
    {"name": "steps", "type": "Collection(Edm.String)", "searchable": true},
    {"name": "tags", "type": "Collection(Edm.String)", "filterable": true},
    {"name": "embedding", "type": "Collection(Edm.Single)", "dimensions": 1536, "vectorSearchProfile": "default"}
  ]
}
```

## 完了条件
- [ ] AI Searchインスタンスが作成されている
- [ ] 3つのインデックスが作成されている
- [ ] セマンティック検索が有効化されている
- [ ] APIキーがKey Vaultに保存されている

## ラベル
`phase-0`, `infrastructure`, `azure`, `ai-search`, `rag`
