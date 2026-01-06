# [Phase 1-4] Cosmos DB スキーマ設計・実装

## 概要
Cosmos DBのスキーマを設計し、データアクセス層を実装する。

## タスク
- [ ] スキーマ設計の確定
- [ ] Cosmos DB クライアント実装
- [ ] sessions コンテナ CRUD実装
- [ ] conversations コンテナ CRUD実装
- [ ] messages コンテナ CRUD実装
- [ ] feedback コンテナ CRUD実装
- [ ] TTL設定（90日）

## スキーマ設計

### sessions コンテナ
```json
{
  "id": "session_123456",
  "session_id": "session_123456",
  "use_case_id": "hondashi",
  "created_at": "2025-01-02T12:00:00Z",
  "last_activity_at": "2025-01-02T12:30:00Z",
  "status": "active",
  "ttl": 7776000
}
```

### conversations コンテナ
```json
{
  "id": "conv_789012",
  "conversation_id": "conv_789012",
  "session_id": "session_123456",
  "use_case_id": "hondashi",
  "started_at": "2025-01-02T12:00:00Z",
  "ended_at": null,
  "message_count": 5,
  "total_tokens": 450,
  "ttl": 7776000
}
```

### messages コンテナ
```json
{
  "id": "msg_345678",
  "message_id": "msg_345678",
  "conversation_id": "conv_789012",
  "role": "assistant",
  "content": "ほんだしの保存方法についてお答えします...",
  "intent": "faq_answered",
  "created_at": "2025-01-02T12:00:33Z",
  "usage": {
    "prompt_tokens": 100,
    "completion_tokens": 80
  },
  "ttl": 7776000
}
```

### feedback コンテナ
```json
{
  "id": "fb_456789",
  "feedback_id": "fb_456789",
  "session_id": "session_123456",
  "message_id": "msg_345678",
  "rating": "positive",
  "comment": "とても役に立ちました",
  "created_at": "2025-01-02T12:01:00Z"
}
```

## データアクセス層
```python
class CosmosDBRepository:
    async def create_session(self, session: Session) -> Session
    async def get_session(self, session_id: str) -> Session
    async def create_message(self, message: Message) -> Message
    async def get_messages(self, conversation_id: str) -> List[Message]
    async def create_feedback(self, feedback: Feedback) -> Feedback
```

## 完了条件
- [ ] 全コンテナのスキーマが確定している
- [ ] CRUD操作が実装されている
- [ ] TTLが正しく設定されている
- [ ] 単体テストが通過する

## ラベル
`phase-1`, `backend`, `database`, `week-2`
