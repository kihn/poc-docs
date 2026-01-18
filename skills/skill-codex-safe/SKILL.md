---
name: skill-codex-safe
version: 2.0.0
description: "Codex CLI に安全に相談する（read-only モード、非対話型実行）"
allowed-tools:
  - Bash(codex:*)
---

# Codex Safe Skill

Claude Code から Codex CLI に安全に相談するためのスキルです。
`codex exec` コマンドを使用して非対話型で実行します。

## Quick Start

```bash
# 基本的な使い方（read-only モード）
codex exec --sandbox read-only "質問や依頼内容"

# 例: コードレビューを依頼
codex exec --sandbox read-only "このファイルの問題点を指摘して: $(cat src/index.ts)"

# 例: 実装方針を相談
codex exec --sandbox read-only "認証機能の実装方針についてアドバイスください"
```

## 使い方

### 1) 読み取り専用で相談（推奨）

```bash
codex exec --sandbox read-only "質問内容"
```

### 2) ファイル変更が必要な場合（ユーザー確認後）

```bash
codex exec --sandbox workspace-write "変更依頼内容"
```

## 安全方針

- **既定は read-only**: まず状況把握から
- **自動実行を避ける**: `--full-auto` は使わない
- **変更前に確認**: 破壊的操作は事前に明示確認
- **stderr を抑制しない**: 警告を隠さない
- **最小変更の原則**: 影響範囲を最小限に

## 禁止・非推奨

- `--full-auto` の使用（自動実行は避ける）
- `--skip-git-repo-check` の常用（安全チェックを回避しない）
- `2>/dev/null` による stderr の抑制（警告が消える）

## 実行例

### コードレビュー依頼

```bash
codex exec --sandbox read-only "以下のコードをレビューして、問題点があれば指摘してください:

$(cat src/auth/handler.ts)"
```

### 実装方針の相談

```bash
codex exec --sandbox read-only "WebAuthn認証のチャレンジ生成について、セキュリティ上の注意点を教えてください"
```

### エラー解決の相談

```bash
codex exec --sandbox read-only "以下のエラーの原因と解決方法を教えてください:

$(cat error.log)"
```

## Claude Code での使い方

このスキルが有効な場合、以下のように Codex に相談できます：

1. `/skill-codex-safe` でスキルを呼び出す
2. 相談内容を伝える
3. Claude Code が `codex exec` コマンドを実行
4. Codex の回答を確認

## 参考

このスキルは安全性を優先した運用例です。
運用環境の権限ポリシーに合わせて調整してください。
