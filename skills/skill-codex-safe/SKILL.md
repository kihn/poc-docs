---
name: skill-codex-safe
version: 2.1.0
description: "Codex CLI に安全に相談する（read-only モード、非対話型実行）"
allowed-tools:
  - Bash(codex:*)
---

# Codex Safe Skill

Claude Code から Codex CLI に安全に相談するためのスキルです。
`codex exec` コマンドを使用して非対話型で実行します。

## モデル設定

`--model` オプションでモデルを指定できます。**デフォルトは `gpt-5.2-codex`（最高性能）** を推奨します。

### 利用可能なモデル

| モデル | 説明 | 推奨用途 |
|--------|------|----------|
| `gpt-5.2-codex` | 最高性能コーディングモデル（デフォルト推奨） | 複雑な推論、アーキテクチャ設計、リファクタリング |
| `gpt-5.1-codex-mini` | 軽量・コスト効率モデル | 簡単な質問、コード補完 |
| `gpt-5.1-codex-max` | 長時間タスク向け最適化モデル | プロジェクト規模の大きな変更 |

**モデル一覧の確認方法**: [Codex Models ドキュメント](https://developers.openai.com/codex/models/)

### モデル指定例

```bash
# デフォルト（gpt-5.2-codex）で実行
codex exec --model gpt-5.2-codex --sandbox read-only "質問内容"

# 軽量モデルで高速に実行
codex exec --model gpt-5.1-codex-mini --sandbox read-only "簡単な質問"
```

## Quick Start

```bash
# 基本的な使い方（read-only モード、o3モデル）
codex exec --model gpt-5.2-codex --sandbox read-only "質問や依頼内容"

# 例: コードレビューを依頼
codex exec --model gpt-5.2-codex --sandbox read-only "このファイルの問題点を指摘して: $(cat src/index.ts)"

# 例: 実装方針を相談
codex exec --model gpt-5.2-codex --sandbox read-only "認証機能の実装方針についてアドバイスください"
```

## 注意事項

**重要**: Codex CLI は Git リポジトリ内でのみ動作します。このモノレポでは、ルートディレクトリではなく各サブディレクトリ（`poc-backend/`, `poc-flutter-app/`, `poc-docs/`）に移動してから実行してください。

```bash
# 正しい例
cd /path/to/poc-backend && codex exec --sandbox read-only "質問"

# エラーになる例（ルートディレクトリから実行）
codex exec --sandbox read-only "質問"
# → "Not inside a trusted directory" エラー
```

## 使い方

### 1) 読み取り専用で相談（推奨）

```bash
cd /path/to/poc-backend && codex exec --model gpt-5.2-codex --sandbox read-only "質問内容"
```

### 2) ファイル変更が必要な場合（ユーザー確認後）

```bash
cd /path/to/poc-backend && codex exec --model gpt-5.2-codex --sandbox workspace-write "変更依頼内容"
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
codex exec --model gpt-5.2-codex --sandbox read-only "以下のコードをレビューして、問題点があれば指摘してください:

$(cat src/auth/handler.ts)"
```

### 実装方針の相談

```bash
codex exec --model gpt-5.2-codex --sandbox read-only "WebAuthn認証のチャレンジ生成について、セキュリティ上の注意点を教えてください"
```

### エラー解決の相談

```bash
codex exec --model gpt-5.2-codex --sandbox read-only "以下のエラーの原因と解決方法を教えてください:

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
