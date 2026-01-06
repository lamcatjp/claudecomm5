# [Phase 1-3] クイックリプライ機能（バックエンド）

## 概要
会話コンテキストに基づいて動的にサジェストを生成するバックエンド機能を実装する。

## タスク
- [ ] SuggestionGeneratorクラス実装
- [ ] YAMLルールベースサジェスト設定
- [ ] 意図分類に基づくサジェスト選択
- [ ] APIレスポンスへのサジェスト追加
- [ ] ユースケース別設定対応

## サジェスト生成ロジック
```python
class SuggestionGenerator:
    def generate(
        self,
        context: ConversationContext,
        last_response: str,
        intent: str
    ) -> List[Suggestion]:
        suggestions = []

        if intent == "recipe_provided":
            suggestions.extend([
                Suggestion(label="減塩にしたい", value="このレシピを減塩にアレンジして"),
                Suggestion(label="具材を変えたい", value="具材を変えたいです"),
                Suggestion(label="他のレシピ", value="他のレシピも見たい"),
            ])
        elif intent == "faq_answered":
            suggestions.extend([
                Suggestion(label="もっと詳しく", value="もう少し詳しく教えてください"),
                Suggestion(label="関連レシピ", value="関連するレシピはありますか？"),
            ])

        return suggestions[:4]  # 最大4つ
```

## YAML設定ファイル
```yaml
# config/suggestions/hondashi.yaml
welcome:
  - label: "レシピを探す"
    value: "ほんだしを使ったレシピを教えてください"
  - label: "使い方を知りたい"
    value: "ほんだしの基本的な使い方を教えてください"

after_recipe:
  - label: "減塩にしたい"
    value: "このレシピを減塩にアレンジしてください"
  - label: "他のレシピを見る"
    value: "他のレシピも見たいです"
```

## APIレスポンス構造
```json
{
  "response": {
    "content": "...",
    "intent": "recipe_provided",
    "suggestions": [
      {"id": "sug_001", "label": "減塩にしたい", "value": "...", "type": "text"},
      {"id": "sug_002", "label": "具材を変えたい", "value": "...", "type": "text"}
    ]
  }
}
```

## 完了条件
- [ ] SuggestionGeneratorが実装されている
- [ ] YAML設定が読み込める
- [ ] 意図に応じたサジェストが生成される
- [ ] APIレスポンスにサジェストが含まれる

## ラベル
`phase-1`, `backend`, `feature`, `week-2`
