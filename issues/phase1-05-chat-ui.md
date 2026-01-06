# [Phase 1-5] React チャットUI 実装

## 概要
Next.js/Reactでチャットインターフェースを実装する。

## タスク
- [ ] Next.jsプロジェクト初期構造作成
- [ ] チャットウィンドウコンポーネント実装
- [ ] メッセージリストコンポーネント実装
- [ ] 入力フォームコンポーネント実装
- [ ] SSEストリーミング対応
- [ ] レシピカード表示コンポーネント実装
- [ ] ローディング状態表示
- [ ] エラーハンドリング
- [ ] レスポンシブデザイン対応

## コンポーネント構成
```
apps/web/src/
├── components/
│   ├── Chat/
│   │   ├── ChatWindow.tsx
│   │   ├── MessageList.tsx
│   │   ├── Message.tsx
│   │   ├── ChatInput.tsx
│   │   └── QuickReplies.tsx
│   ├── Recipe/
│   │   ├── RecipeCard.tsx
│   │   └── RecipeModal.tsx
│   └── common/
│       ├── Loading.tsx
│       └── ErrorBoundary.tsx
├── hooks/
│   ├── useChat.ts
│   └── useSSE.ts
├── services/
│   └── chatApi.ts
└── styles/
    └── chat.css
```

## デザインガイドライン
| 要素 | カラー |
|------|--------|
| プライマリ（味の素レッド） | #E60012 |
| セカンダリ（ほんだしブラウン） | #8B4513 |
| アクセント（だしゴールド） | #D4A853 |
| ベース（ウォームホワイト） | #FFFAF5 |
| AIメッセージ背景 | #FFF8F0 |
| ユーザーメッセージ背景 | #E60012 |

## SSE実装
```typescript
const useSSE = (url: string) => {
  const [content, setContent] = useState('');

  const startStream = async (message: string) => {
    const response = await fetch(url, {
      method: 'POST',
      body: JSON.stringify({ message }),
    });

    const reader = response.body?.getReader();
    // ストリーミング処理...
  };

  return { content, startStream };
};
```

## 完了条件
- [ ] チャットUIが動作する
- [ ] SSEストリーミングが動作する
- [ ] レシピカードが表示される
- [ ] レスポンシブデザインが適用されている
- [ ] ブランドカラーが適用されている

## ラベル
`phase-1`, `frontend`, `ui`, `week-3`
