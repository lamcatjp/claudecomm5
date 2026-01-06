# [Phase 0-9] Application Insights 設定

## 概要
Azure Application Insightsを設定し、アプリケーションの監視・トレーシング基盤を構築する。

## タスク
- [ ] Application Insights `foodllm-appi-dev` を作成
- [ ] Log Analytics Workspaceとの連携
- [ ] Container Appsへの接続設定
- [ ] 基本アラート設定（エラー率、レスポンスタイム）
- [ ] カスタムメトリクス設定（トークン使用量等）

## 技術詳細
- **リソース名**: `foodllm-appi-dev`
- **リージョン**: East US
- **連携**: Log Analytics Workspace `foodllm-log-dev`

## 監視項目
| メトリクス | 閾値 | アラート |
|-----------|------|---------|
| エラー率 | > 5% | 警告 |
| 応答時間 P95 | > 5秒 | 警告 |
| 可用性 | < 99% | 重大 |

## 環境変数
```bash
APPLICATIONINSIGHTS_CONNECTION_STRING=<接続文字列>
```

## 完了条件
- [ ] Application Insightsが作成されている
- [ ] Container Appsと連携されている
- [ ] 基本アラートが設定されている

## ラベル
`phase-0`, `infrastructure`, `azure`, `monitoring`
