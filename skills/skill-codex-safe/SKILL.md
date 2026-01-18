---
name: codex-safe
version: 1.0.0
description: "Codex CLI を安全寄りに運用するためのガイド（自動実行を避け、確認と可視性を重視）"
---

# Codex Skill Guide (Safe)

このガイドは、Codex CLI を**安全寄り**に運用するための最小ルールです。  
**自動実行を避け、確認と可視性を優先**します。

## 基本方針

- 既定は**read-only**で開始する
- 変更が必要な場合のみ、ユーザーに確認して workspace-write へ移行する
- 重要操作（削除・広範な変更・外部アクセス）は事前に明示確認する
- 出力は隠さず、**stderr を抑制しない**
- 自動実行フラグ（`--full-auto`）は使わない
- 内部思考は英語で行い、最終出力は日本語で簡潔に記述する

## 推奨の起動方法

### 1) 読み取り専用で開始

```bash
codex --sandbox read-only
```

### 2) 変更が必要になったら確認して移行

```bash
codex --sandbox workspace-write
```

## 禁止・非推奨

- `--full-auto` の使用（自動実行は避ける）
- `--skip-git-repo-check` の常用（安全チェックを回避しない）
- `2>/dev/null` による stderr の抑制（警告が消える）

## 使い方の原則

- **まず状況把握**: ファイル読み取りや検索のみで開始
- **変更前に説明**: 何をどこに、なぜ変更するか短く伝える
- **最小変更**: 影響範囲を最小限に
- **安全確認**: 破壊的操作や広範な編集は必ず事前確認

## 例: 安全な進め方

1. read-only で調査
2. 変更案の提示
3. ユーザーの了承を得て workspace-write へ
4. 必要最小限の編集
5. 変更内容の要約と次のステップ提示

## 例: 簡易プロンプト

```
You are a cautious coding assistant.
- Start in read-only mode.
- Ask before any file edits or destructive commands.
- Never use --full-auto.
- Do not suppress stderr.
- Keep changes minimal and explain intent.
```

## 参考

このガイドは安全性を優先した運用例です。  
運用環境の権限ポリシーに合わせて調整してください。
