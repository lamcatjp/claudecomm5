# [Phase 0-4] API Management (Consumption) 作成・API定義

## 概要
Azure API Management (Consumption tier) を作成し、レート制限とAPIキー管理を設定する。

## タスク
- [ ] API Management `foodllm-apim-dev` を作成（Consumption tier）
- [ ] API定義のインポート（OpenAPI仕様）
- [ ] レート制限ポリシー設定（60 req/min）
- [ ] CORS設定
- [ ] APIキー（サブスクリプション）の発行
- [ ] Container Appsとの連携設定

## 技術詳細
- **リソース名**: `foodllm-apim-dev`
- **SKU**: Consumption
- **リージョン**: East US
- **レート制限**: 60リクエスト/分

## ポリシー設定
```xml
<rate-limit calls="60" renewal-period="60" />
<cors>
    <allowed-origins>
        <origin>https://foodllm-hondashi-swa-dev.azurestaticapps.net</origin>
        <origin>http://localhost:3000</origin>
    </allowed-origins>
    <allowed-methods>
        <method>GET</method>
        <method>POST</method>
        <method>DELETE</method>
        <method>OPTIONS</method>
    </allowed-methods>
    <allowed-headers>
        <header>Content-Type</header>
        <header>X-API-Key</header>
        <header>X-Admin-API-Key</header>
    </allowed-headers>
</cors>
```

## 完了条件
- [ ] API Managementが作成されている
- [ ] レート制限が設定されている
- [ ] CORSが設定されている
- [ ] APIキーが発行されている

## ラベル
`phase-0`, `infrastructure`, `azure`, `api-management`
