# [Phase 1-10] Azure Static Web Apps デプロイ・MVP リリース

## 概要
フロントエンドをAzure Static Web Appsにデプロイし、MVPをリリースする。

## タスク
- [ ] Static Web Apps `foodllm-hondashi-swa-dev` 作成
- [ ] GitHub Actionsデプロイワークフロー設定
- [ ] 環境変数設定
- [ ] カスタムドメイン設定（オプション）
- [ ] MVPアクセスパスワード設定
- [ ] 本番前チェックリスト実施
- [ ] リリース

## Static Web Apps設定
- **リソース名**: `foodllm-hondashi-swa-dev`
- **リージョン**: East US
- **SKU**: Free
- **ビルド設定**:
  - App location: `/apps/web`
  - Output location: `.next`
  - API location: （空）

## 環境変数
```
NEXT_PUBLIC_API_BASE_URL=https://foodllm-apim-dev.azure-api.net
NEXT_PUBLIC_USE_CASE=hondashi
```

## 本番前チェックリスト
- [ ] 全テストがパスしている
- [ ] パフォーマンス基準を満たしている
- [ ] セキュリティチェック完了
- [ ] エラーハンドリングが適切
- [ ] ログ出力が適切
- [ ] 監視・アラートが設定されている
- [ ] ロールバック手順が確認されている
- [ ] ドキュメントが更新されている

## リリース手順
1. mainブランチへのマージ
2. GitHub Actions自動デプロイ
3. デプロイ完了確認
4. 動作確認（スモークテスト）
5. MVPアクセス情報の共有

## ロールバック手順
1. Static Web Appsのデプロイ履歴から前バージョンを選択
2. 「Revert」を実行
3. 動作確認

## 完了条件
- [ ] Static Web Appsにデプロイされている
- [ ] MVPアクセスが可能
- [ ] 本番前チェックリスト全項目クリア
- [ ] リリースノートが作成されている

## ラベル
`phase-1`, `deployment`, `release`, `week-4`
