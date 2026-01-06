# [Phase 1-1] Chat API エンドポイント実装

## 概要
FastAPIでChat APIエンドポイントを実装し、Azure OpenAIとの連携を構築する。

## タスク
- [ ] FastAPIプロジェクト初期構造作成
- [ ] `POST /api/v1/chat/messages` 実装
- [ ] `POST /api/v1/chat/messages/stream` 実装（SSE）
- [ ] `POST /api/v1/conversations` 実装
- [ ] セッション管理機能実装
- [ ] 会話履歴管理（最新10メッセージ、8000トークン制限）
- [ ] Azure OpenAI呼び出しロジック実装
- [ ] エラーハンドリング

## API仕様
### POST /api/v1/chat/messages
```json
// リクエスト
{
  "session_id": "session_123456",
  "message": "ほんだしの保存方法を教えてください"
}

// レスポンス
{
  "success": true,
  "data": {
    "session_id": "session_123456",
    "conversation_id": "conv_789012",
    "message_id": "msg_345678",
    "response": {
      "content": "ほんだしの保存方法についてお答えします...",
      "intent": "faq_answered",
      "suggestions": [...],
      "faqs": [...]
    }
  }
}
```

## ディレクトリ構造
```
api/
├── main.py
├── routers/
│   ├── chat.py
│   ├── conversations.py
│   └── health.py
├── services/
│   ├── openai_service.py
│   ├── session_service.py
│   └── conversation_service.py
├── models/
│   └── schemas.py
└── config/
    └── settings.py
```

## 完了条件
- [ ] 全エンドポイントが実装されている
- [ ] SSEストリーミングが動作する
- [ ] Azure OpenAIへの呼び出しが成功する
- [ ] 単体テストカバレッジ80%以上

## ラベル
`phase-1`, `backend`, `api`, `week-2`
