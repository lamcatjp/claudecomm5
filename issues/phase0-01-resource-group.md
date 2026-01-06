# [Phase 0-1] Azureサブスクリプション・リソースグループ設定

## 概要
Azure開発環境の基盤となるサブスクリプションとリソースグループを設定する。

## タスク
- [ ] Azureサブスクリプションの確認・設定
- [ ] リソースグループ `ajinomoto-foodllm-dev-rg` を East US リージョンに作成
- [ ] タグ設定（Environment: dev, Project: foodllm）
- [ ] RBAC設定（開発チームへの適切な権限付与）

## 技術詳細
- **リージョン**: East US (eastus)
- **リソースグループ名**: `ajinomoto-foodllm-dev-rg`
- **命名規則**: `{company}-{project}-{env}-rg`

## コマンド例
```bash
az group create \
  --name ajinomoto-foodllm-dev-rg \
  --location eastus \
  --tags Environment=dev Project=foodllm
```

## 完了条件
- [ ] リソースグループが作成されている
- [ ] タグが正しく設定されている
- [ ] 開発チームがアクセス可能

## ラベル
`phase-0`, `infrastructure`, `azure`
