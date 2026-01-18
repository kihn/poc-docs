---
name: inutomo-git
version: 1.0.0
description: "モノレポ対応のGitコミット・プッシュ・PR作成コマンド"
skills:
  - name: commit
    description: "Create a git commit"
  - name: commit-push-pr
    description: "Commit, push, and open a PR"
  - name: clean_gone
    description: "Cleans up all git branches marked as [gone]"
allowed-tools:
  - Bash(git:*)
  - Bash(gh:*)
---

# inutomo-git Skill（モノレポ対応版）

このスキルは、ルートディレクトリがGit管理されていないモノレポ構造に対応したGit操作を提供します。

## プロジェクト構造

このモノレポは3つの独立したGitリポジトリで構成されています：

```
inutomo-now-poc/           # ルート（Git管理なし）
├── poc-backend/           # 独立リポジトリ: Cloudflare Workers API
├── poc-flutter-app/       # 独立リポジトリ: Flutterモバイルアプリ
└── poc-docs/              # 独立リポジトリ: 設計ドキュメント
```

## スキル一覧

### 1. commit

現在の変更をコミットします。

**使い方**: `/inutomo-git:commit`

**動作**:
1. 各サブリポジトリの変更状態を確認
2. 変更があるリポジトリを検出
3. 変更内容を分析してコミットメッセージを生成
4. ユーザー確認後にコミット実行

### 2. commit-push-pr

コミット、プッシュ、PR作成を一括実行します。

**使い方**: `/inutomo-git:commit-push-pr`

**動作**:
1. 各サブリポジトリの変更状態を確認
2. 変更があるリポジトリでコミット作成
3. リモートにプッシュ
4. GitHub PRを作成
5. 各リポジトリのPR URLを報告

### 3. clean_gone

リモートで削除されたブランチをローカルからクリーンアップします。

**使い方**: `/inutomo-git:clean_gone`

**動作**:
1. 各サブリポジトリで `git fetch --prune` を実行
2. `[gone]` マークのブランチを検出
3. ローカルブランチを削除

---

## 実装手順

### commit スキル実行時

```bash
# 1. 各リポジトリの状態確認
for repo in poc-backend poc-flutter-app poc-docs; do
  echo "=== $repo ==="
  cd /Users/vonsalza/dev-private/inutomo-now-poc/$repo
  git status --short
done

# 2. 変更があるリポジトリで詳細確認
cd /path/to/repo-with-changes
git status
git diff
git diff --staged
git log --oneline -5

# 3. コミット実行
git add -A
git commit -m "$(cat <<'EOF'
<type>: <subject>

<body>

Co-Authored-By: Claude <modelname> <noreply@anthropic.com>
EOF
)"
```

### commit-push-pr スキル実行時

```bash
# 1. 各リポジトリの状態確認（上記と同様）

# 2. コミット実行（上記と同様）

# 3. プッシュ
git push -u origin <branch-name>

# 4. PR作成
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points>

## Test plan
- [ ] テスト項目

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"

# 5. PR URLを報告
```

### clean_gone スキル実行時

```bash
# 各リポジトリで実行
for repo in poc-backend poc-flutter-app poc-docs; do
  echo "=== $repo ==="
  cd /Users/vonsalza/dev-private/inutomo-now-poc/$repo
  git fetch --prune
  git branch -vv | grep ': gone]' | awk '{print $1}' | xargs -r git branch -D
done
```

---

## コミットメッセージ規約

```
<type>: <subject>

<body>

Co-Authored-By: Claude <modelname> <noreply@anthropic.com>
```

### Types
- `feat`: 新機能
- `fix`: バグ修正
- `refactor`: リファクタリング
- `test`: テスト追加・修正
- `docs`: ドキュメント
- `chore`: その他（ビルド、CI等）
- `style`: コードスタイル（フォーマット等）

### 注意事項

- **NEVER** use `git commit --amend` unless explicitly requested
- **NEVER** use `git push --force` to main/master
- **NEVER** skip hooks (`--no-verify`)
- **ALWAYS** use HEREDOC for commit messages to ensure proper formatting
- **ALWAYS** report all PR URLs when complete

---

## 複数リポジトリにまたがる変更

複数のリポジトリに変更がある場合：

1. 各リポジトリで個別にコミット・プッシュ・PR作成
2. 関連するPRは相互にリンク（PR本文に記載）
3. すべてのPR URLを最後にまとめて報告

例:
```
## 作成されたPR

- **poc-backend**: https://github.com/owner/poc-backend/pull/123
- **poc-flutter-app**: https://github.com/owner/poc-flutter-app/pull/456
```

---

## トラブルシューティング

### "Not a git repository" エラー
ルートディレクトリではなく、サブディレクトリ（poc-backend, poc-flutter-app, poc-docs）に移動して実行してください。

### リモートが設定されていない
```bash
git remote add origin https://github.com/owner/repo.git
```

### 認証エラー
GitHub CLIでログインしてください：
```bash
gh auth login
```
