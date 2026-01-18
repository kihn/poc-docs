# 統合テスト改善タスク

## 概要

タスク16（統合テスト作成）のCodexレビューで指摘された改善点をまとめる。
現時点で42件の統合テストは全てパスしているが、テストの信頼性向上のために以下の改善を検討する。

## 指摘事項と改善タスク

### 1. [重大] MockTransactionのコミット処理が未実装

**問題**
`MockTransaction`が`set/update/delete`をキューするだけで、`runTransaction`終了時にコミットされない。
実際のFirestoreと挙動が異なり、データ整合性テストが機能していない可能性がある。

**影響箇所**
- `tests/integration/test-helpers.ts:134-159` (MockTransaction)
- `tests/integration/test-helpers.ts:34-57` (MockFirestore.runTransaction)

**改善案**
```typescript
async runTransaction<T>(
  fn: (transaction: MockTransaction) => Promise<T>,
): Promise<T> {
  const transaction = new MockTransaction(this);
  const result = await fn(transaction);
  // トランザクション終了時に全操作をコミット
  await transaction.commit();
  return result;
}

// MockTransactionにcommitメソッドを追加
async commit(): Promise<void> {
  for (const op of this.operations) {
    await op();
  }
  this.operations = [];
}
```

**優先度**: 高

---

### 2. [中] モックDBの状態がテスト間でリセットされていない

**問題**
`mockDb`が`vi.mock`のスコープで1回生成され、`beforeEach`は`vi.clearAllMocks()`のみ。
状態が残ると順序依存やフレーキーテストの原因になる。

**影響箇所**
- `tests/integration/passkey-registration.test.ts:23-83`
- `tests/integration/passkey-login.test.ts:18-62`
- `tests/integration/jwt-auth.test.ts:30-43`

**改善案**
各テストファイルで`beforeEach`内でmockDbの状態をリセットする仕組みを追加。
ただし、現在のモック構造では`mockDb`にアクセスできないため、以下のいずれかの対応が必要：

1. **グローバルリセット関数のエクスポート**
   ```typescript
   // test-helpers.ts
   let globalMockDb: MockFirestore | null = null;
   export function getOrCreateMockDb(): MockFirestore {
     if (!globalMockDb) {
       globalMockDb = new MockFirestore();
     }
     return globalMockDb;
   }
   export function resetMockDb(): void {
     globalMockDb?.clear();
   }
   ```

2. **テストファイルごとに独立したmockDbインスタンス**
   - `vi.mock`内で新しいインスタンスを生成し、各テストで完全に分離

**優先度**: 中

---

### 3. [中] MISSING_TOKEN分岐が実質未カバー

**問題**
`Authorization: "Bearer "`を送っても`app.request`がヘッダー値をトリムし、
`MISSING_TOKEN`ではなく`INVALID_AUTHORIZATION_FORMAT`が返される。
実運用（Workers/Fetch）でもトリムされる可能性があり、`MISSING_TOKEN`分岐が到達不能かもしれない。

**影響箇所**
- `tests/integration/jwt-auth.test.ts:88-147`
- `src/middleware/jwt-auth.ts` のMISSING_TOKEN分岐

**確認事項**
- Cloudflare Workers実環境でヘッダー値がトリムされるか確認
- トリムされる場合、`MISSING_TOKEN`分岐は不要（デッドコード）

**改善案**
1. **実環境テスト**: `pnpm dev`でローカルサーバーを起動し、curlで`Authorization: "Bearer "`を送信して確認
2. **分岐が到達不能なら削除**: コードの簡素化
3. **分岐が必要なら**: ミドルウェア単体テストで直接コンテキストを構築してテスト

**優先度**: 中

---

### 4. [中] パスキー系の重要分岐が未カバー

**問題**
以下のエラーパスがテストされていない：

**登録（passkey-register.ts）**
- `INVALID_CHALLENGE`（challenge/userId不整合）
- `USER_ALREADY_EXISTS`
- `REGISTRATION_VERIFICATION_FAILED`（publicKey欠落）
- `credential_exists`の409/422

**ログイン（passkey-login.ts）**
- `INVALID_CHALLENGE`
- `updateCredentialCounter`の非カウンターエラーで500
- `userHandle`未設定を許容する経路

**改善案**
各エラーパスに対応するテストケースを追加：

```typescript
// passkey-registration.test.ts に追加
it("異常系: チャレンジのuserIdが不正 → INVALID_CHALLENGE", async () => {
  // challengeRecord.userIdがundefinedの場合をモック
});

it("異常系: publicKeyが欠落 → REGISTRATION_VERIFICATION_FAILED", async () => {
  // verifyRegistrationResponseがpublicKeyなしで返すモック
});
```

**優先度**: 中

---

### 5. [低] JWT検証がセキュリティ観点で弱い

**問題**
パスキー成功系で返却されたJWTについて：
- `sub`がuserIdと一致するか検証していない
- JWTが実際に検証可能か確認していない（形式チェックのみ）

**影響箇所**
- `tests/integration/passkey-registration.test.ts`
- `tests/integration/passkey-login.test.ts`

**改善案**
```typescript
it("正常系: 登録完了でJWTを返す", async () => {
  // ... existing code ...

  // JWTの形式確認
  const jwtParts = verifyJson.token.split(".");
  expect(jwtParts).toHaveLength(3);

  // JWTのペイロードを検証
  const payload = JSON.parse(atob(jwtParts[1]));
  expect(payload.sub).toBe(verifyJson.userId);

  // JWTが実際に検証可能か確認
  const meRes = await app.request(
    "/v1/api/auth/me",
    {
      method: "GET",
      headers: { Authorization: `Bearer ${verifyJson.token}` },
    },
    testEnv,
  );
  expect(meRes.status).toBe(200);
});
```

**優先度**: 低

---

## 実装優先度まとめ

| 優先度 | タスク | 工数目安 |
|--------|--------|----------|
| 高 | MockTransactionのコミット処理実装 | 小 |
| 中 | モックDBの状態リセット | 中 |
| 中 | MISSING_TOKEN分岐の検証・対応 | 小 |
| 中 | パスキー異常系テスト追加 | 中 |
| 低 | JWT検証強化 | 小 |

## 参考

- レビュー実施日: 2026-01-18
- レビュアー: Codex CLI (gpt-5.2-codex)
- 対象コミット: feat/integration-tests-passkey-auth ブランチ
