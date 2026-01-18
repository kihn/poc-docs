# ログ出力方針

本ドキュメントでは、イヌトモNow PoCバックエンドにおけるログ出力の方針を定義します。
セキュリティと運用性のバランスを考慮し、機密情報の漏洩を防止しながらデバッグ可能性を確保します。

## 基本原則

1. **機密情報は絶対にログに出力しない**
2. **エラー調査に必要な最小限の情報を記録する**
3. **本番環境と開発環境でログレベルを適切に制御する**

## 機密情報（ログ出力禁止）

以下の情報は**絶対にログに出力してはならない**：

| カテゴリ | 具体例 |
|----------|--------|
| 認証情報 | JWT秘密鍵、Firebase秘密鍵、APIキー |
| ユーザー識別子 | userId、credentialId（ハッシュ化していないもの） |
| 認証データ | JWTトークン、パスキー公開鍵、チャレンジ値 |
| 環境変数 | `FIREBASE_SERVICE_ACCOUNT`、`JWT_PRIVATE_KEY`の内容 |
| エラー詳細 | `error.message`（機密情報を含む可能性があるため） |
| リクエストボディ | 認証関連のリクエストペイロード全体 |

## 非機密メタ情報（ログ出力可）

以下の情報はログに出力して**問題ない**：

| カテゴリ | 具体例 | 用途 |
|----------|--------|------|
| **エラー種別** | `error.constructor.name`（例: `SyntaxError`, `TypeError`） | エラー分類 |
| **リクエストパス** | `/v1/api/auth/passkey/register/options` | 問題箇所特定 |
| **HTTPメソッド** | `GET`, `POST` | リクエスト識別 |
| **HTTPステータス** | `200`, `401`, `503` | レスポンス結果 |
| **タイムスタンプ** | ISO 8601形式（例: `2026-01-18T10:30:00.000Z`） | 時系列分析 |
| **環境識別子** | `development`, `staging`, `production` | 環境区別 |
| **リクエストID** | UUID v7（時系列ソート可能、リクエスト追跡用） | ログ相関 |
| **処理時間** | ミリ秒単位（例: `150ms`） | パフォーマンス分析 |
| **User-Agent** | ブラウザ/アプリ識別子 | クライアント分析 |
| **IPアドレス** | クライアントIP（プライバシー考慮が必要） | 不正アクセス調査 |

### IPアドレスに関する注意

- 開発環境: 出力可
- 本番環境: GDPRなどプライバシー規制を考慮し、必要に応じてマスキングまたは非出力

## ログフォーマット

### 推奨フォーマット（構造化ログ）

```typescript
console.error("[Firestore] Initialization failed", {
  errorType: "SyntaxError",
  path: c.req.path,
  method: c.req.method,
  environment: c.env.ENVIRONMENT,
  timestamp: new Date().toISOString(),
});
```

### 出力例

```json
{
  "level": "error",
  "message": "[Firestore] Initialization failed",
  "errorType": "SyntaxError",
  "path": "/v1/api/auth/passkey/register/options",
  "method": "POST",
  "environment": "production",
  "timestamp": "2026-01-18T10:30:00.000Z"
}
```

## エラーログの実装例

### 現在の実装（最小限・安全優先）

```typescript
const errorType = error instanceof Error ? error.constructor.name : "UnknownError";
console.error(`[Firestore] Initialization failed: ${errorType}`);
```

### 推奨実装（運用性向上）

```typescript
const errorType = error instanceof Error ? error.constructor.name : "UnknownError";
console.error("[Firestore] Initialization failed", {
  errorType,
  path: c.req.path,
  method: c.req.method,
  environment: c.env.ENVIRONMENT ?? "unknown",
  timestamp: new Date().toISOString(),
});
```

## ログレベル

| レベル | 用途 | 本番環境 |
|--------|------|----------|
| `error` | システムエラー、初期化失敗、認証失敗 | ✅ 出力 |
| `warn` | 非推奨機能の使用、レート制限接近 | ✅ 出力 |
| `info` | リクエスト開始/終了、重要なビジネスイベント | ⚠️ 要検討 |
| `debug` | 詳細なデバッグ情報 | ❌ 出力しない |

## Cloudflare Workers固有の考慮事項

1. **console.log/error**はCloudflare Logsに送信される
2. **Workers Analytics Engine**を使用する場合は追加のメトリクス出力が可能
3. **Tail Workers**でログを外部サービスに転送可能

## 関連ファイル

- `poc-backend/src/index.ts` - Firestore初期化エラーログ
- `poc-backend/src/middleware/error-handler.ts` - グローバルエラーハンドラー

## 更新履歴

| 日付 | 変更内容 |
|------|----------|
| 2026-01-18 | 初版作成 |
