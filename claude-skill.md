# Claude Skill - タスク固有の専門知識

## スキルとは

**定義**: 特定のタスクに特化した、再利用可能な AI 命令セット

**特徴**:

- プロジェクト固有の専門知識を定義
- `/skill-name` コマンドで即座に呼び出し
- YAML フロントマター + Markdown 形式
- チーム全体で共有可能

## スキルの構造

**基本フォーマット**:

```markdown
---
description: スキルの説明(50文字以内)
tags: [category, domain, type]
version: 1.0.0
author: あなたの名前
---

# スキル名

## 概要

このスキルが何をするか

## 前提条件

- 必要な知識
- 前提となる設定

## 実行内容

具体的な処理ステップ

## 成果物

期待される出力

## 例

使用例
```

## 実践例: カスタムスキル集

### 1. プロジェクト固有のコードレビュースキル

**`.claude/skills/review-pr.md`**:

```markdown
---
description: プロジェクト規約に基づくプルリクエストレビュー
tags: [review, quality, team]
version: 1.0.0
---

# PR Review - プロジェクト規約準拠

## レビュー観点

### 1. コーディング規約

- **TypeScript**: strict mode 必須、any 禁止
- **命名規則**: camelCase(変数)、PascalCase(クラス)
- **ファイル構成**: feature-based directory structure
- **最大行数**: 関数 100 行、ファイル 300 行以内

### 2. テスト要件

- **カバレッジ**: >85% 必須
- **テストタイプ**: ユニット + 統合テスト
- **命名**: `describe('ClassName')` / `it('should ...')`

### 3. セキュリティ

- **入力検証**: 全 API 入力はバリデーション必須
- **認証**: JWT 検証ミドルウェア使用
- **機密情報**: 環境変数のみ、ハードコード禁止

### 4. パフォーマンス

- **DB**: N+1 クエリ禁止、必ず eager loading
- **API**: レスポンスタイム <200ms
- **バンドル**: チャンクサイズ <500KB

## レビュープロセス

1. **構造チェック**: ディレクトリ構成が規約に準拠
2. **コード品質**: ESLint、Prettier が通過
3. **テスト**: 全テストが成功、カバレッジ 85%以上
4. **セキュリティ**: OWASP Top 10 チェック
5. **パフォーマンス**: 遅いクエリ、大きなバンドルの検出

## 出力フォーマット

\`\`\`markdown

## レビューサマリー

✅ 合格 / ⚠️ 要改善 / ❌ 却下

## 指摘事項

### 🔴 Critical (マージブロッカー)

- [ファイル名:行番号] 問題の説明

### 🟡 Warning (推奨修正)

- [ファイル名:行番号] 改善提案

### 🟢 Good Points

- 良い実装のハイライト

## 次のアクション

- [ ] 修正すべき項目
      \`\`\`

## 使用例

\`\`\`bash

# プルリクエストをレビュー

git diff main | claude-code /review-pr
\`\`\`
```

**使用方法**:

```bash
# プロジェクトルートで実行
cd ~/projects/my-app

# スキルを呼び出し
claude-code /review-pr

# または Git diffと組み合わせ
git diff main | claude-code /review-pr
```

### 2. データベースマイグレーションスキル

**`.claude/skills/db-migration.md`**:

```markdown
---
description: 安全なデータベースマイグレーション生成
tags: [database, migration, safety]
version: 1.0.0
---

# Database Migration - 安全性重視

## マイグレーション原則

### 安全性ルール

1. **後方互換性**: 既存データを破壊しない
2. **ロールバック可能**: 必ず down migration を定義
3. **インデックス**: 大規模テーブルは段階的に追加
4. **デフォルト値**: NOT NULL 追加時は必須

### 命名規約

- ファイル名: `YYYYMMDDHHMMSS_descriptive_name.ts`
- テーブル名: `snake_case`, 複数形
- カラム名: `snake_case`, 単数形

## 生成プロセス

1. **スキーマ分析**: 既存テーブル構造を確認
2. **変更計画**: 安全な変更順序を決定
3. **マイグレーション生成**: up/down 両方を作成
4. **テスト SQL**: ロールバックテスト用 SQL 生成

## テンプレート

\`\`\`typescript
import { MigrationInterface, QueryRunner, Table } from "typeorm";

export class DescriptiveName1234567890123 implements MigrationInterface {
public async up(queryRunner: QueryRunner): Promise<void> {
// 1. 新テーブル作成
// 2. カラム追加(NULL 許可)
// 3. データ移行
// 4. NOT NULL 制約追加
// 5. インデックス作成
}

public async down(queryRunner: QueryRunner): Promise<void> {
// up の逆順で実行
}
}
\`\`\`

## チェックリスト

- [ ] `up`と`down`両方が実装されている
- [ ] 既存データを破壊しない
- [ ] インデックスは適切に配置
- [ ] 外部キー制約は正しく設定
- [ ] デフォルト値が定義されている
- [ ] ロールバックテストが成功
```

### 3. API 設計スキル

**`.claude/skills/api-design.md`**:

```markdown
---
description: RESTful API設計ガイドライン準拠
tags: [api, rest, design]
version: 1.0.0
---

# API Design - RESTful ベストプラクティス

## API 設計原則

### エンドポイント設計

- **リソース指向**: 名詞を使用(動詞は避ける)
- **階層構造**: `/api/v1/users/:userId/posts/:postId`
- **複数形**: コレクションは複数形(`/users`, `/posts`)

### HTTP メソッド

| メソッド | 用途     | 例                  |
| -------- | -------- | ------------------- |
| GET      | 取得     | `GET /users/:id`    |
| POST     | 作成     | `POST /users`       |
| PUT      | 全体更新 | `PUT /users/:id`    |
| PATCH    | 部分更新 | `PATCH /users/:id`  |
| DELETE   | 削除     | `DELETE /users/:id` |

### ステータスコード

- **200 OK**: 成功
- **201 Created**: 作成成功
- **400 Bad Request**: バリデーションエラー
- **401 Unauthorized**: 認証エラー
- **403 Forbidden**: 権限エラー
- **404 Not Found**: リソース未発見
- **500 Internal Server Error**: サーバーエラー

## レスポンスフォーマット

### 成功時

\`\`\`typescript
{
"data": {
"id": "123",
"name": "John Doe",
"email": "john@example.com"
},
"meta": {
"timestamp": "2025-01-15T10:00:00Z"
}
}
\`\`\`

### エラー時

\`\`\`typescript
{
"error": {
"code": "VALIDATION_ERROR",
"message": "入力値が不正です",
"details": [
{
"field": "email",
"message": "有効なメールアドレスを入力してください"
}
]
}
}
\`\`\`

## バリデーション

- **入力**: 全入力を DTO でバリデーション
- **出力**: レスポンスを DTO で型安全に
- **エラーハンドリング**: グローバルエラーハンドラー使用

## ドキュメント生成

- OpenAPI (Swagger) 形式
- エンドポイントごとに JSDoc コメント
- 例を含む完全な仕様
```

## スキルの組み合わせ

**複数スキルの連携**:

```bash
# API設計 → 実装 → マイグレーション → レビュー
claude-code /api-design "ユーザー管理API"
claude-code implement
claude-code /db-migration
claude-code /review-pr
```

## スキル開発のベストプラクティス

### 1. 単一責任の原則

**良い例**: 1 つのスキル = 1 つの明確なタスク

```
✅ /review-pr - PRレビュー専用
✅ /test-gen - テスト生成専用
✅ /api-design - API設計専用

❌ /everything - すべてをやる汎用スキル
```

### 2. 再利用性を重視

**パラメータ化で汎用性を持たせる**:

```markdown
# /deploy スキル例

## 使用方法

\`\`\`bash
/deploy <environment> <version>

# 例

/deploy staging v1.2.3
/deploy production v1.2.3
\`\`\`
```

### 3. チェックリストを含める

**実行後の確認事項を明記**:

```markdown
## 完了チェックリスト

- [ ] すべてのテストが成功
- [ ] リンターエラーなし
- [ ] ドキュメントが更新されている
- [ ] 環境変数が設定されている
```

## スキルの作成ワークフロー

### ステップ 1: よく使うタスクを特定

```bash
# 繰り返し実行するタスクをリストアップ
- コードレビュー (週5回)
- テスト生成 (週3回)
- API設計 (週1回)
- リファクタリング (週2回)
```

### ステップ 2: スキルテンプレートを作成

```bash
# スキルディレクトリ作成
mkdir -p .claude/skills

# 最初のスキルを作成
cat > .claude/skills/review.md << 'EOF'
---
description: プロジェクト規約に基づくコードレビュー
tags: [review, quality]
version: 1.0.0
---

# Code Review

## レビュー観点
[具体的な観点を記述]

## チェックリスト
- [ ] 項目1
- [ ] 項目2
EOF
```

### ステップ 3: 実際に使って改善

```bash
# スキルを実行
claude-code /review

# フィードバックを記録
cat >> .claude/feedback/skill-improvements.md << 'EOF'
## /review スキル改善点

### 2025-01-20
- 追加: セキュリティチェック項目
- 改善: 出力フォーマットをより読みやすく
EOF
```

### ステップ 4: チームと共有

```bash
# Git にコミット
git add .claude/skills/review.md
git commit -m "Add code review skill"
git push

# チームメンバーが同じスキルを使用可能
```

## 高度なスキルテクニック

### 1. スキル内でのコンテキスト参照

```markdown
# /implement-feature スキル

## 実装時の参照ドキュメント

- アーキテクチャ: [.claude/knowledge/architecture.md](..knowledge/architecture.md)
- コーディング規約: [.claude/style/coding-preferences.md](../style/coding-preferences.md)
- テストパターン: [.claude/knowledge/test-patterns.md](../knowledge/test-patterns.md)

これらのドキュメントに従って実装してください。
```

### 2. スキルチェーン

```yaml
# .claude/skill-chains/full-feature.yml

name: Full Feature Implementation
description: 設計から実装、テスト、デプロイまでの完全フロー

chain:
  - skill: /api-design
    output: design-doc

  - skill: /implement
    input: design-doc
    output: implementation

  - skill: /test-gen
    input: implementation
    output: tests

  - skill: /review-pr
    input: [implementation, tests]
    validation: must-pass

  - skill: /deploy
    input: all
    condition: review-passed
```

### 3. 条件付きスキル実行

```markdown
# /smart-deploy スキル

## デプロイ前チェック

1. 環境を確認:

   - staging → 自動デプロイ
   - production → 承認待ち

2. テストカバレッジを確認:

   - > 85% → デプロイ実行
   - <85% → エラーで停止

3. セキュリティスキャン:
   - Critical 0 件 → デプロイ実行
   - Critical あり → エラーで停止
```

## まとめ

### スキルの価値

**効率化**:

- 繰り返しタスクを数秒で実行
- チーム全体で知識を共有
- 一貫性のある品質を維持

**学習**:

- ベストプラクティスをスキルに蓄積
- 新メンバーのオンボーディング加速
- 組織の知的資産として蓄積

**進化**:

- 使いながら改善
- フィードバックループで最適化
- チームの成長に合わせて拡張

### 次のステップ

1. **最初の 3 つのスキルを作成**

   - よく使うタスクから始める
   - シンプルに保つ
   - 実際に使って改善

2. **チームと共有**

   - スキルをリポジトリにコミット
   - 使い方をドキュメント化
   - フィードバックを収集

3. **継続的改善**
   - 使用頻度を測定
   - 改善点を記録
   - 定期的にアップデート

## 関連ドキュメント

- [サブエージェント](./sub-agents.md) - 複雑なタスクの自律実行
- [カスタマイズ](./customize.md) - 全体的なカスタマイズ戦略
- [ワークフロー自動化](./workflow-automation.md) - スキルを組み合わせた自動化
