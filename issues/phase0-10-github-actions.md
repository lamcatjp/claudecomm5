# [Phase 0-10] GitHub Actions CI/CDパイプライン構築

## 概要
GitHub Actionsを使用したCI/CDパイプラインを構築し、自動テスト・デプロイを実現する。

## タスク
- [ ] CIワークフロー（ci.yml）作成
  - Lint (flake8, black, ESLint)
  - 単体テスト (pytest, Jest)
  - Dockerビルド確認
- [ ] CD開発環境ワークフロー（cd-dev.yml）作成
  - ACRへのイメージプッシュ
  - Container Appsへのデプロイ
- [ ] GitHub Secretsの設定
- [ ] ブランチ保護ルールの設定

## ワークフロー構成
| ワークフロー | トリガー | 処理内容 |
|------------|---------|---------|
| `ci.yml` | PR作成/更新 | Lint、単体テスト、Dockerビルド確認 |
| `cd-dev.yml` | developブランチへのマージ | ACRプッシュ → 開発環境デプロイ |
| `cd-stg.yml` | mainブランチへのマージ | ACRプッシュ → ステージング環境デプロイ |
| `cd-prod.yml` | リリースタグ作成 | 本番環境デプロイ（手動承認） |

## GitHub Secrets
```
AZURE_CREDENTIALS - Azure認証情報（JSON）
ACR_LOGIN_SERVER - foodllmacrdev.azurecr.io
ACR_USERNAME - レジストリユーザー名
ACR_PASSWORD - レジストリパスワード
```

## 完了条件
- [ ] CIワークフローがPR時に実行される
- [ ] CDワークフローがマージ時に実行される
- [ ] Secretsが設定されている
- [ ] デプロイが自動化されている

## ラベル
`phase-0`, `ci-cd`, `github-actions`, `devops`
