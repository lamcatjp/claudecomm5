# [Phase 1-2] RAG パイプライン構築

## 概要
Azure AI Searchを使用したRAG（Retrieval-Augmented Generation）パイプラインを構築する。

## タスク
- [ ] Azure AI Search クライアント実装
- [ ] ベクトル検索機能実装
- [ ] ハイブリッド検索（キーワード＋ベクトル）実装
- [ ] セマンティックランキング実装
- [ ] 検索結果のコンテキスト生成
- [ ] 意図分類によるインデックス選択ロジック
- [ ] 検索精度チューニング

## RAGフロー
```
ユーザー入力
    ↓
意図分類
    ↓
┌─────────────────────────────────┐
│ recipe_search → recipes-index   │
│ product_info → products-index   │
│ faq_question → faq-index        │
└─────────────────────────────────┘
    ↓
ハイブリッド検索 + セマンティックランキング
    ↓
上位3件をコンテキストとして取得
    ↓
LLMに送信して回答生成
```

## 検索設定
```python
search_config = {
    "vector_weight": 0.5,
    "keyword_weight": 0.5,
    "top_k": 5,
    "semantic_reranking": True,
    "min_score": 0.7
}
```

## 意図分類
| intent | 検索対象インデックス |
|--------|-------------------|
| recipe_search | hondashi-recipes-index |
| product_info | hondashi-products-index |
| faq_question | hondashi-faq-index |
| general_chat | 検索なし |
| out_of_scope | 検索なし |

## 完了条件
- [ ] ベクトル検索が動作する
- [ ] ハイブリッド検索が動作する
- [ ] 意図に応じたインデックス選択ができる
- [ ] 検索精度テストが合格（適合率80%以上）

## ラベル
`phase-1`, `backend`, `rag`, `ai-search`, `week-2`
