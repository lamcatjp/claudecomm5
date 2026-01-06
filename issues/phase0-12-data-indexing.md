# [Phase 0-12] 初期データのAzure AI Search投入

## 概要
収集したデータにエンベディングを生成し、Azure AI Searchに投入する。

## 前提条件
- [ ] Phase 0-6: AI Search インスタンスが作成済み
- [ ] Phase 0-11: データ収集・構造化が完了済み

## タスク
- [ ] エンベディング生成スクリプト作成
- [ ] レシピデータのエンベディング生成
- [ ] 製品データのエンベディング生成
- [ ] FAQデータのエンベディング生成
- [ ] AI Searchへのデータ投入
- [ ] 検索精度テスト

## スクリプト実行順序
```bash
# 1. エンベディング生成
python scripts/generate_embeddings.py --input data/recipes.json --output data/recipes_embedded.json
python scripts/generate_embeddings.py --input data/products.json --output data/products_embedded.json
python scripts/generate_embeddings.py --input data/faq.json --output data/faq_embedded.json

# 2. AI Searchへ投入
python scripts/upload_to_search.py --index hondashi-recipes-index --data data/recipes_embedded.json
python scripts/upload_to_search.py --index hondashi-products-index --data data/products_embedded.json
python scripts/upload_to_search.py --index hondashi-faq-index --data data/faq_embedded.json

# 3. 検索テスト
python scripts/test_search.py --query "味噌汁の作り方"
```

## 検索精度テスト
| クエリ | 期待結果 | 実際結果 |
|-------|---------|---------|
| 味噌汁の作り方 | 味噌汁レシピがトップ | |
| ほんだしの保存方法 | 保存方法FAQがトップ | |
| アレルギー | アレルギーFAQがトップ | |

## 完了条件
- [ ] 全データにエンベディングが生成されている
- [ ] 3つのインデックスにデータが投入されている
- [ ] 検索精度テストが合格している

## ラベル
`phase-0`, `data`, `ai-search`, `rag`
