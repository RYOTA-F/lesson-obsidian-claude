# CLI エージェント

## 概要

CLI ベースの AI エージェント(Claude Code, Aider, GitHub Copilot CLI 等)は、ターミナル環境で動作する AI 支援ツール。GUI ベースの AI チャット(ChatGPT, Claude Web 等)とは異なる特性を持ち、エンジニアのワークフローに深く統合される。

## CLI ベース AI エージェントの特徴

### 1. コマンドライン統合

**特徴**:

- ターミナルから直接実行可能
- シェルコマンドとシームレスに組み合わせ可能
- パイプ、リダイレクト等の Unix 哲学を活用

**具体例**:

```bash
# ファイル内容をAIに渡して分析
cat src/auth.ts | claude-code analyze

# Git diffをAIに渡してレビュー依頼
git diff | aider review

# 複数ファイルを一括処理
find . -name "*.ts" | xargs claude-code refactor
```

**メリット**:

- 既存のコマンドと組み合わせて強力なワークフロー構築
- スクリプト化・自動化が容易
- キーボードから手を離さずに操作可能

### 2. ファイルシステム直接操作

**特徴**:

- ローカルファイルシステムへの直接アクセス
- ファイルの読み書き、作成、削除を即座に実行
- ディレクトリ構造の理解とナビゲーション

**具体例**:

```bash
# プロジェクト全体をコンテキストに含める
claude-code --context . "認証システムをリファクタリング"

# 特定ディレクトリ内のファイルを一括編集
aider src/components/**/*.tsx --message "TypeScript strict mode対応"
```

**GUI との違い**:
| 操作 | CLI | GUI |
|------|-----|-----|
| ファイル操作 | 直接実行 | コピペが必要 |
| ディレクトリ移動 | `cd`コマンド | ファイル選択 UI |
| 複数ファイル | ワイルドカード | 個別アップロード |
| 変更適用 | 即座に反映 | 手動でコピー |

### 3. プロジェクトコンテキストの自動理解

**特徴**:

- `.git`ディレクトリからプロジェクト情報を取得
- `package.json`, `requirements.txt`等から依存関係を理解
- `.gitignore`を尊重したファイル走査
- プロジェクト構造の自動マッピング

**具体例**:

```bash
# プロジェクトルートで実行すると自動的に全体を理解
cd ~/projects/my-app
claude-code "テストカバレッジを向上させて"

# Git履歴も活用
git log --oneline | head -10 | claude-code "最近の変更を要約"
```

**GUI との違い**:

- **CLI**: プロジェクト構造を自動認識、コンテキストを自動収集
- **GUI**: 手動でファイルをアップロード、プロジェクト構造を説明する必要

### 4. バージョン管理システム統合

**特徴**:

- Git コマンドと深く統合
- コミット、ブランチ、diff 等を直接操作
- 変更履歴の自動追跡

**具体例**:

```bash
# 未コミットの変更を分析
git status | claude-code "変更内容をレビュー"

# AIが推奨するコミットメッセージを生成
claude-code commit --auto-message

# ブランチを作成してAIに実装依頼
git checkout -b feature/auth
claude-code "JWT認証を実装して"
git add . && git commit -m "Add JWT authentication"
```

**メリット**:

- コード変更とバージョン管理が一体化
- 自動コミットメッセージ生成
- ブランチ戦略との統合

### 5. 高速な入出力処理

**特徴**:

- テキストベースで軽量
- ネットワーク転送量が最小
- ストリーミングレスポンス対応

**パフォーマンス比較**:
| 操作 | CLI | GUI |
|------|-----|-----|
| 起動時間 | <1 秒 | 3-5 秒(ブラウザ) |
| ファイル送信 | 直接アクセス | アップロード待機 |
| レスポンス | ストリーミング | バッファリング |
| メモリ使用量 | 低(50-100MB) | 高(500MB-1GB) |

### 6. スクリプタビリティと自動化

**特徴**:

- シェルスクリプトに組み込み可能
- CI/CD パイプラインに統合可能
- 繰り返しタスクの自動化

**具体例**:

```bash
#!/bin/bash
# 自動コードレビュースクリプト

# 最新のPRをチェックアウト
gh pr checkout $1

# AIによるコードレビュー
git diff main | claude-code review --format markdown > review.md

# レビュー結果をPRにコメント
gh pr comment $1 --body-file review.md

# ローカル環境をクリーンアップ
git checkout main
```

**自動化の例**:

- **Pre-commit hook**: コミット前に自動リント・フォーマット
- **CI/CD**: プルリクエストの自動レビュー
- **定期実行**: 毎日のコード品質チェック

### 7. カスタマイズ性と拡張性

**特徴**:

- 設定ファイル(`.claude/`, `.aider/`)でカスタマイズ
- エイリアス、関数で独自コマンド作成
- プラグイン・拡張機能の追加

**具体例**:

```bash
# .bashrc / .zshrc でエイリアス定義
alias review="git diff | claude-code review"
alias explain="claude-code explain"
alias fix="claude-code fix --auto-commit"

# カスタム関数
ai-refactor() {
  claude-code refactor "$1" --backup --test
}

# プロジェクト固有の設定
cat > .claude/config.yaml << EOF
context:
  - src/
  - tests/
ignore:
  - node_modules/
  - dist/
rules:
  - "TypeScript strict mode"
  - "ESLint準拠"
EOF
```

### 8. プライバシーとセキュリティ

**特徴**:

- ローカル実行オプション(ローカル LLM 対応)
- ネットワーク送信内容の完全制御
- 機密ファイルの明示的除外

**セキュリティ対策**:

```bash
# .gitignoreを尊重
claude-code --respect-gitignore

# 特定ファイルを除外
claude-code --exclude "*.env,*.key,secrets/*"

# ローカルLLMを使用(機密プロジェクト)
export CLAUDE_CODE_MODEL=local
claude-code "内部システムのリファクタリング"
```

**GUI との違い**:

- **CLI**: 送信内容を明示的にコントロール、`.gitignore`自動尊重
- **GUI**: 手動でファイル選択、除外設定が曖昧

## GUI ベース AI チャットとの比較

### GUI の特徴

**利点**:

- 視覚的に分かりやすい
- マウス操作で直感的
- マルチメディア対応(画像、グラフ等)
- 会話履歴の視覚的管理

**欠点**:

- ファイル操作が手動
- プロジェクトコンテキストの手動説明が必要
- コピペ作業が多い
- 自動化が困難

### CLI の特徴

**利点**:

- ワークフローへの深い統合
- 自動化・スクリプト化が容易
- 高速な入出力
- プロジェクトコンテキストの自動理解
- バージョン管理との統合

**欠点**:

- 学習曲線が急
- 視覚的フィードバックが少ない
- マルチメディア非対応(基本はテキストのみ)

### 使い分けの指針

**GUI を使うべき場面**:

- 初めて AI を使う
- 探索的な会話・ブレインストーミング
- 画像や図を含む説明が必要
- プログラミング以外のタスク(文章作成、調査等)

**CLI を使うべき場面**:

- コーディング作業
- プロジェクト全体のリファクタリング
- 複数ファイルの一括処理
- CI/CD 統合
- 繰り返し実行するタスク
- プライバシー重視のプロジェクト

## なぜエンジニアは CLI を使うのか

### 1. キーボード中心のワークフロー

**理由**: エンジニアはコーディング中、キーボードから手を離したくない

**具体例**:

```bash
# コーディング → テスト → AIレビュー → 修正 の流れがキーボードのみで完結
vim src/auth.ts           # コード編集
npm test                  # テスト実行
git diff | claude review  # AIレビュー
vim src/auth.ts           # 修正
git commit -m "Fix auth"  # コミット
```

**メリット**:

- フロー状態(ゾーン)を維持
- マウスとキーボードの切り替えがない
- タイピング速度で作業可能

### 2. 既存ツールチェーンへの統合

**理由**: エンジニアは既に強力な CLI ツール群を使用している

**ツールチェーン例**:

```bash
# Git
git status, git diff, git log

# ビルドツール
npm, cargo, make, gradle

# テスティング
jest, pytest, cargo test

# リンティング
eslint, pylint, rustfmt

# 検索・解析
grep, ripgrep, fd, jq

# これらとAIを組み合わせる
git diff | claude-code review | less
npm test 2>&1 | claude-code "失敗理由を分析"
eslint . --format json | jq | claude-code fix
```

**メリット**:

- 既存ワークフローを壊さない
- 学習コストが低い(既知のパターン)
- 強力な組み合わせが可能

### 3. 自動化とスクリプティング

**理由**: エンジニアは反復作業を自動化したい

**自動化例**:

```bash
# 毎朝のコードレビュー自動化
#!/bin/bash
# morning-review.sh

echo "昨日のコミットをレビュー..."
git log --since="yesterday" --pretty=format:"%H" | while read commit; do
  echo "Commit: $commit"
  git show $commit | claude-code review
done

# Cronで自動実行
0 9 * * * ~/scripts/morning-review.sh | mail -s "Daily Review" dev@example.com
```

**メリット**:

- 一度書けば何度でも実行
- 人間の介入不要
- 一貫性のある処理

### 4. 再現可能性とドキュメント化

**理由**: エンジニアは再現可能な環境を重視

**ドキュメント例**:

````markdown
# プロジェクトセットアップ

## AI によるコードレビュー設定

1. Claude Code インストール
   ```bash
   npm install -g @anthropic/claude-code
   ```
````

2. Git hook セットアップ

   ```bash
   cat > .git/hooks/pre-commit << 'EOF'
   #!/bin/bash
   git diff --cached | claude-code review --strict
   EOF
   chmod +x .git/hooks/pre-commit
   ```

3. CI 設定(.github/workflows/review.yml)
   ```yaml
   - name: AI Code Review
     run: |
       git diff origin/main | claude-code review --format github
   ```

````

**メリット**:
- チーム全体で同じ環境
- 新メンバーのオンボーディングが容易
- バージョン管理可能

### 5. パフォーマンスと効率性

**理由**: エンジニアは待ち時間を嫌う

**パフォーマンス比較**:
```bash
# CLIの速度
$ time echo "hello" | claude-code explain
# real    0m0.8s (レスポンス待機のみ)

# GUIの場合
# 1. ブラウザを開く (2秒)
# 2. ページロード (3秒)
# 3. テキストを入力 (5秒)
# 4. レスポンス待機 (0.8秒)
# 合計: 約10秒
````

**メリット**:

- 起動時間ゼロ
- オーバーヘッドが最小
- フィードバックループが高速

### 6. コンテキストの正確性

**理由**: エンジニアはプロジェクト全体のコンテキストを AI に理解させたい

**コンテキスト収集の比較**:

**GUI の場合**:

```
1. ファイル構造を説明
2. 依存関係を列挙
3. 関連ファイルを個別アップロード
4. プロジェクトの背景を説明
→ 時間がかかり、漏れが発生しやすい
```

**CLI の場合**:

```bash
# プロジェクトルートで実行するだけ
cd ~/projects/my-app
claude-code "認証システムを改善"

# 自動的に収集されるコンテキスト:
# - ディレクトリ構造
# - package.json (依存関係)
# - .gitignore (除外ファイル)
# - Git履歴
# - 関連ファイルの内容
```

**メリット**:

- 手動説明不要
- コンテキスト漏れがない
- 一貫性のある情報提供

### 7. リモート環境での作業

**理由**: エンジニアはサーバーやコンテナ内で作業することが多い

**リモート作業例**:

```bash
# SSH経由でリモートサーバーに接続
ssh user@production-server

# リモート環境でAI支援を受ける
claude-code "このログファイルのエラーを分析"
tail -100 /var/log/app.log | claude-code analyze

# GUIは使用不可(X11フォワーディング等が必要)
```

**メリット**:

- どこからでも同じ操作
- SSH 接続のみで完結
- GUI 不要

### 8. カスタマイズと拡張性

**理由**: エンジニアは自分専用のツールを作りたい

**カスタマイズ例**:

```bash
# ~/.zshrc に独自関数を定義

# AIによる説明 + ドキュメント生成
explain-and-doc() {
  claude-code explain "$1" > docs/$(basename "$1" .ts).md
  echo "Documentation generated at docs/$(basename "$1" .ts).md"
}

# AIレビュー + 自動修正 + テスト
smart-fix() {
  local file="$1"
  claude-code review "$file" --fix --auto-test
  if [ $? -eq 0 ]; then
    git add "$file"
    git commit -m "AI-assisted fix: $file"
  fi
}

# プロジェクト全体の品質チェック
quality-check() {
  echo "Running linter..."
  npm run lint 2>&1 | claude-code "リント問題を要約"

  echo "Running tests..."
  npm test 2>&1 | claude-code "テスト失敗を分析"

  echo "Checking complexity..."
  find src -name "*.ts" -exec wc -l {} \; | sort -rn | head -10 | \
    claude-code "複雑度の高いファイルを特定"
}
```

**メリット**:

- 個人の作業スタイルに最適化
- チーム固有のルールを組み込み可能
- 継続的な改善が容易

## 主要 CLI ベース AI エージェント

### 1. Claude Code

**特徴**:

- Anthropic 公式 CLI
- プロジェクトコンテキストの深い理解
- ファイル操作の直接実行
- Git 統合

**使用例**:

```bash
# インタラクティブモード
claude-code

# ワンショット実行
claude-code "テストを追加"

# 特定ファイルを対象
claude-code --files src/auth.ts "型安全性を向上"
```

### 2. Aider

**特徴**:

- Git 統合に特化
- 複数 LLM 対応(GPT-4, Claude 等)
- 自動コミット機能
- ペアプログラミング的 UI

**使用例**:

```bash
# インタラクティブセッション開始
aider

# 特定ファイルで開始
aider src/components/Button.tsx

# 自動コミット有効化
aider --auto-commit
```

### 3. GitHub Copilot CLI

**特徴**:

- GitHub エコシステム統合
- シェルコマンド生成に特化
- Git 操作の支援

**使用例**:

```bash
# シェルコマンド提案
gh copilot suggest "すべての.logファイルを削除"

# Git操作提案
gh copilot explain "git rebase -i HEAD~3"
```

### 4. ChatGPT CLI (非公式)

**特徴**:

- OpenAI API 利用
- シンプルなチャット
- パイプ処理対応

**使用例**:

```bash
# 標準入力から質問
echo "Rustでバイナリ検索を実装" | chatgpt

# ファイル内容を分析
cat src/main.rs | chatgpt "このコードを最適化"
```

## CLI ベース AI エージェントのベストプラクティス

### 1. コンテキストファイルの活用

**`.claude/CLAUDE.md`パターン**:

```markdown
# プロジェクトコンテキスト

## 技術スタック

- TypeScript 5.0
- React 18
- NestJS 10

## コーディング規約

- ESLint strict mode
- Prettier 適用
- 関数は 100 行以内

## 禁止事項

- `any`型の使用
- `console.log`の残存
- 未テストのコード
```

**効果**: AI が自動的にプロジェクトルールを理解

### 2. エイリアスで高速化

**`.bashrc` / `.zshrc`**:

```bash
# よく使うコマンドをエイリアス化
alias ai="claude-code"
alias review="git diff | claude-code review"
alias explain="claude-code explain"
alias doc="claude-code document"
alias test-fix="npm test 2>&1 | claude-code fix"

# 使用例
ai "認証を追加"
review
explain src/utils.ts
```

### 3. Git Hooks で自動化

**`.git/hooks/pre-commit`**:

```bash
#!/bin/bash

# コミット前にAIレビュー
echo "Running AI review..."
git diff --cached | claude-code review --strict

if [ $? -ne 0 ]; then
  echo "AI review found issues. Fix and try again."
  exit 1
fi

echo "AI review passed!"
```

### 4. 環境変数で設定管理

**`.env` / `.zshrc`**:

```bash
# Claude Code設定
export CLAUDE_CODE_MODEL="claude-3-5-sonnet-20241022"
export CLAUDE_CODE_CONTEXT_SIZE="large"
export CLAUDE_CODE_AUTO_COMMIT="false"

# APIキー(安全に管理)
export ANTHROPIC_API_KEY="sk-ant-xxx"

# プロジェクト固有設定
export CLAUDE_CODE_IGNORE=".git,node_modules,dist,*.log"
```

### 5. シェル関数で複雑なワークフロー

**高度な例**:

```bash
# AI支援付きフィーチャー開発
feature() {
  local feature_name="$1"
  local description="$2"

  # 1. ブランチ作成
  git checkout -b "feature/$feature_name"

  # 2. AIに実装依頼
  claude-code "実装: $description" --auto-test

  # 3. AIレビュー
  git diff main | claude-code review --format markdown > review.md

  # 4. 問題なければコミット
  if grep -q "LGTM" review.md; then
    git add .
    git commit -m "feat: $description"
    echo "Feature implemented successfully!"
  else
    echo "Review found issues. Check review.md"
  fi
}

# 使用例
feature "auth" "JWT認証を実装"
```

## まとめ

### CLI ベース AI エージェントの本質

**統合性**: 既存ワークフローとシームレスに統合
**効率性**: キーボード中心、高速、軽量
**自動化**: スクリプト化・自動化が容易
**正確性**: プロジェクトコンテキストの自動理解
**拡張性**: カスタマイズ・拡張が自由

### エンジニアが CLI を好む理由

1. **フロー状態の維持**: キーボードから手を離さない
2. **既存ツールとの統合**: 学習済みのツールチェーンを活用
3. **自動化への適性**: 反復作業を排除
4. **再現可能性**: 環境を正確に再現・共有
5. **パフォーマンス**: 待ち時間の最小化
6. **コンテキストの正確性**: プロジェクト理解の自動化
7. **リモート作業**: どこでも同じ操作
8. **カスタマイズ性**: 個人・チームに最適化

### GUI と CLI の共存

**理想的なアプローチ**:

- **探索・学習**: GUI(ChatGPT, Claude Web)
- **実装・自動化**: CLI(Claude Code, Aider)
- **コラボレーション**: GUI(共有しやすい)
- **個人作業**: CLI(効率的)

### 次のステップ

- [CLI ツール入門](../tool/cli-basics.md) - ターミナル操作の基礎
- [シェルスクリプティング](../dev/shell-scripting.md) - 自動化の実践
- [Git 統合](../dev/git-integration.md) - バージョン管理との連携
- [開発環境構築](../dev/dev-environment.md) - 最適な開発環境
