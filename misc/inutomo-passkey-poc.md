# イヌトモNow Passkey PoC 実施メモ

## 目的
本PoCの目的は、以下を最小構成で検証すること。

- パスキー（WebAuthn）でユーザー登録ができる
- パスキーでログインできる
- Cloudflare Workers（Hono）で発行したJWTでAPI認可ができる
- Flutterアプリから一連の流れが成立する

※ UI完成度や本番セキュリティは対象外。**通るかどうかだけ**を見る。

---

## 全体構成
- フロントエンド：Flutter（iOS or Android どちらか1つでOK）
- バックエンド：Cloudflare Workers + Hono
- ストレージ：Firestore（またはKV / DOでも可）

---

## バックエンド（Workers）でやること

### 1. Passkey登録（Registration）
#### エンドポイント
- `POST /v1/api/auth/passkey/register/options`
- `POST /v1/api/auth/passkey/register/verify`

#### 処理内容
- チャレンジを生成
- チャレンジを一時保存（TTLあり）
- WebAuthn options を返す
- attestation を検証
- credential（公開鍵等）を保存

---

### 2. Passkeyログイン（Authentication）
#### エンドポイント
- `POST /v1/api/auth/passkey/login/options`
- `POST /v1/api/auth/passkey/login/verify`

#### 処理内容
- チャレンジを生成
- assertion を検証
- ユーザー特定
- JWTを発行

---

### 3. 認可確認用API
#### エンドポイント
- `GET /v1/api/me`

#### 処理内容
- AuthorizationヘッダのJWTを検証
- userIdを返す

---

## Firestore（または代替）に保存するもの

### users
- userId
- createdAt

### credentials
- credentialId
- publicKey
- counter
- userId

### challenges（短命）
- challenge
- userId（optional）
- expiresAt

---

## Flutterアプリでやること

### 画面構成（最小）
- Passkey登録ボタン
- Passkeyログインボタン
- ログイン成功結果表示

---

### Flutter側処理フロー

#### 登録
1. `/register/options` を呼ぶ
2. ネイティブAPIでパスキー作成
3. `/register/verify` に結果送信

#### ログイン
1. `/login/options` を呼ぶ
2. ネイティブAPIで署名
3. `/login/verify` に結果送信
4. JWTを保存

#### 認可確認
- JWT付きで `/me` を呼ぶ

---

## PoCの合格条件
- 登録が成功する
- ログインが成功する
- JWTで保護APIが叩ける

これが通れば **Passkey採用可** と判断する。

---

## PoCでやらないこと
- UIデザイン
- 複数端末対応
- アカウント復旧
- 位置共有ロジック
- セキュリティチューニング

---

## 次のステップ
- 位置共有API設計
- イヌとも関係モデル
- 位置情報公開レベル設計
- 本番向けセキュリティ設計
