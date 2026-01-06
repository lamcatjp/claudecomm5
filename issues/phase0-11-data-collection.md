# [Phase 0-11] ほんだし関連データ収集・構造化

## 概要
ほんだしに関するレシピ、製品情報、FAQデータを収集し、JSON形式に構造化する。

## タスク
- [ ] レシピデータ収集（100〜200件）
- [ ] 製品情報収集（10〜20製品）
- [ ] FAQデータ収集（50〜100件）
- [ ] 調理基礎知識収集（30〜50件）
- [ ] JSONフォーマットへの変換
- [ ] データ品質チェック

## データソース
| データ種別 | ソース | 想定件数 |
|-----------|-------|---------|
| レシピ | 公式サイト、AJINOMOTO PARK | 100〜200件 |
| 製品情報 | 公式サイト、製品カタログ | 10〜20製品 |
| FAQ | お客様相談室データ | 50〜100件 |
| 調理基礎知識 | AJINOMOTO PARK | 30〜50件 |

## データフォーマット（レシピ例）
```json
{
  "recipe_id": "hondashi_001",
  "title": "基本の味噌汁",
  "description": "ほんだしを使った定番の味噌汁",
  "ingredients": [
    {"name": "ほんだし", "amount": "小さじ1", "category": "調味料"},
    {"name": "味噌", "amount": "大さじ1.5", "category": "調味料"}
  ],
  "steps": ["鍋に水400mlとほんだしを入れる", "..."],
  "cooking_time": 10,
  "difficulty": "easy",
  "tags": ["定番", "汁物", "簡単"]
}
```

## スクリプト
```bash
python scripts/convert_recipes.py --input raw/recipes.csv --output data/recipes.json
python scripts/convert_products.py --input raw/products.csv --output data/products.json
python scripts/convert_faq.py --input raw/faq.csv --output data/faq.json
```

## 完了条件
- [ ] 全データが収集されている
- [ ] JSONフォーマットに変換されている
- [ ] データ品質チェックが完了している
- [ ] data/ ディレクトリにファイルが配置されている

## ラベル
`phase-0`, `data`, `content`
