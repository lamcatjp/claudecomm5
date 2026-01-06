# [Phase 0-5] Cosmos DB アカウント・データベース作成

## 概要
Azure Cosmos DBアカウントとデータベースを作成し、会話ログ保存の基盤を構築する。

## タスク
- [ ] Cosmos DBアカウント `foodllm-cosmos-dev` を作成
- [ ] データベース `foodllm-db` を作成
- [ ] コンテナ作成:
  - `sessions`（セッション管理）
  - `conversations`（会話管理）
  - `messages`（メッセージ）
  - `feedback`（フィードバック）
- [ ] パーティションキー設定
- [ ] TTL設定（90日）
- [ ] 接続文字列をKey Vaultに保存

## 技術詳細
- **アカウント名**: `foodllm-cosmos-dev`
- **API**: NoSQL
- **リージョン**: East US
- **整合性レベル**: Session

## コンテナ設計
| コンテナ | パーティションキー | TTL |
|---------|-----------------|-----|
| sessions | /session_id | 90日 |
| conversations | /conversation_id | 90日 |
| messages | /conversation_id | 90日 |
| feedback | /session_id | なし |

## 完了条件
- [ ] Cosmos DBアカウントが作成されている
- [ ] データベースとコンテナが作成されている
- [ ] パーティションキーが適切に設定されている
- [ ] 接続文字列がKey Vaultに保存されている

## ラベル
`phase-0`, `infrastructure`, `azure`, `database`
