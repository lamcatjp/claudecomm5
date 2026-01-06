# [Phase 1-6] クイックリプライ機能（フロントエンド）

## 概要
バックエンドから受け取ったサジェストをボタンとして表示し、タップで送信できる機能を実装する。

## タスク
- [ ] QuickRepliesコンポーネント実装
- [ ] サジェストボタンスタイリング
- [ ] ボタン選択時のメッセージ送信処理
- [ ] ローディング中の無効化処理
- [ ] アニメーション追加
- [ ] アクセシビリティ対応

## コンポーネント実装
```tsx
interface Suggestion {
  id: string;
  label: string;
  value: string;
  type: 'text' | 'action';
  icon?: string;
}

export const QuickReplies: React.FC<{
  suggestions: Suggestion[];
  onSelect: (suggestion: Suggestion) => void;
  disabled?: boolean;
}> = ({ suggestions, onSelect, disabled }) => {
  if (!suggestions?.length) return null;

  return (
    <div className="quick-replies">
      {suggestions.map((suggestion) => (
        <button
          key={suggestion.id}
          className="quick-reply-btn"
          onClick={() => onSelect(suggestion)}
          disabled={disabled}
        >
          {suggestion.label}
        </button>
      ))}
    </div>
  );
};
```

## スタイリング
```css
.quick-replies {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  padding: 12px 0;
  animation: fadeIn 0.3s ease-in;
}

.quick-reply-btn {
  padding: 8px 16px;
  border: 1px solid #E60012;
  border-radius: 20px;
  background: white;
  color: #E60012;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s;
}

.quick-reply-btn:hover {
  background: #E60012;
  color: white;
}

.quick-reply-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

## UIフロー
1. AIメッセージ表示後、サジェストボタンが表示される
2. ユーザーがボタンをクリック
3. ボタンのvalueがメッセージとして送信される
4. サジェストボタンが非表示になる
5. 新しいAIメッセージ後、新しいサジェストが表示される

## 完了条件
- [ ] QuickRepliesコンポーネントが実装されている
- [ ] ブランドカラーでスタイリングされている
- [ ] ボタンクリックでメッセージ送信される
- [ ] ローディング中は無効化される
- [ ] アニメーションが動作する

## ラベル
`phase-1`, `frontend`, `feature`, `week-3`
