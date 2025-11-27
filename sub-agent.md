# Claude Code サブエージェント - 専門 AI エージェント

## サブエージェントとは

**定義**: 特定のドメインに特化した専門 AI エージェント

**特徴**:

- 複雑なタスクを自律的に実行
- 専門知識とツールを持つ
- 並列実行可能
- カスタム設定で振る舞いを調整

## Claude Code の標準サブエージェント

| サブエージェント         | 専門分野           | 主な用途                       |
| ------------------------ | ------------------ | ------------------------------ |
| **general-purpose**      | 汎用タスク         | 複雑な調査、多段階タスク       |
| **Explore**              | コードベース探索   | ファイル検索、構造分析         |
| **backend-architect**    | バックエンド設計   | API 設計、DB 設計              |
| **frontend-architect**   | フロントエンド設計 | UI/UX 設計、コンポーネント設計 |
| **security-engineer**    | セキュリティ       | 脆弱性検査、対策提案           |
| **performance-engineer** | パフォーマンス     | ボトルネック分析、最適化       |
| **quality-engineer**     | テスト             | テスト戦略、自動化             |
| **refactoring-expert**   | リファクタリング   | コード改善、技術的負債解消     |

## サブエージェントの使い方

### 基本的な呼び出し

```bash
# 特定のエージェントを指定
claude-code --agent backend-architect "ユーザー管理APIを設計"

# 複数エージェントを並列実行
claude-code --agent backend-architect,security-engineer "認証システム設計"
```

### エージェントの選択基準

**backend-architect**:
- API エンドポイント設計
- データベーススキーマ設計
- マイクロサービス構成

**frontend-architect**:
- UI コンポーネント設計
- 状態管理戦略
- レスポンシブデザイン

**security-engineer**:
- セキュリティ監査
- 脆弱性スキャン
- OWASP Top 10 チェック

**performance-engineer**:
- パフォーマンス分析
- ボトルネック特定
- 最適化提案

**quality-engineer**:
- テスト戦略立案
- カバレッジ分析
- CI/CD パイプライン設計

## カスタムサブエージェントの作成

### 例: チーム専用のアーキテクトエージェント

**`.claude/agents/team-architect.yml`**:

```yaml
name: team-architect
description: チームのアーキテクチャ原則に基づく設計エージェント

# エージェントのベースとなる性格
base: backend-architect

# カスタムプロンプト
system_prompt: |
  あなたはチームのアーキテクチャ原則を深く理解した設計エージェントです。

  ## チームのアーキテクチャ原則

  ### 技術スタック
  - バックエンド: NestJS + TypeScript
  - フロントエンド: Next.js 14 (App Router)
  - データベース: PostgreSQL + TypeORM
  - 認証: JWT (RS256)
  - API: GraphQL + REST混在

  ### 設計原則
  1. **SOLID原則**: すべてのクラス設計で遵守
  2. **DDD**: ドメイン駆動設計を採用
  3. **CQRS**: 読み書き分離パターン
  4. **Event-Driven**: イベントソーシング活用

  ### コーディング規約
  - TypeScript strict mode必須
  - 関数は単一責任、100行以内
  - すべての公開APIはインターフェース定義
  - DTOは必須、バリデーション完備

  ### テスト戦略
  - ユニットテスト: >85%カバレッジ
  - 統合テスト: 全APIエンドポイント
  - E2Eテスト: クリティカルパス

  これらの原則に基づいて設計を行ってください。

# 利用可能なツール
tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash

# 実行時オプション
options:
  auto_test: true # 実装後に自動テスト
  auto_document: true # ドキュメント自動生成
  strict_validation: true # 厳格なバリデーション
```

**使用方法**:

```bash
# カスタムエージェントを呼び出し
claude-code --agent team-architect "ユーザー管理機能を設計"

# 複数エージェントを並列実行
claude-code --agent team-architect,security-engineer "認証システム設計"
```

## エージェントの育成パターン

### パターン 1: プロジェクトメモリ機能

**セッション間での学習**:

```markdown
# .claude/memory/project-context.md

## プロジェクト概要

EC サイトリニューアルプロジェクト

## 技術的決定

### 2025-01-15: GraphQL 採用

- 理由: 複数フロントエンドの柔軟な対応
- 影響: REST API は段階的に移行

### 2025-01-20: Redis 導入

- 理由: セッション管理とキャッシング
- 設定: クラスタモード、6379 ポート

## 頻出パターン

### 認証フロー

1. JWT 検証ミドルウェア
2. ユーザー情報を context に注入
3. 権限チェック(RBAC)

### エラーハンドリング

- グローバルエラーフィルター使用
- カスタム例外クラス継承
- ログは Winston で出力

## チーム慣習

- PR 前に必ず lint + test
- コミットメッセージ: Conventional Commits
- デプロイ: main → staging → production
```

**効果**: エージェントがプロジェクトの歴史と文脈を理解

### パターン 2: 個人スタイル学習

**`.claude/style/coding-preferences.md`**:

```markdown
# 個人的なコーディングスタイル

## 命名規則

- 変数: `camelCase`, 意味が明確な長い名前を好む
- 定数: `UPPER_SNAKE_CASE`
- プライベートメソッド: `_prefixWithUnderscore`

## コード構造

- 早期リターン好み(ネストを浅く)
- 三項演算子は 1 行のみ、複雑なら if 文
- async/await 優先、.then()は使わない

## コメント

- 「なぜ」を書く、「何を」は書かない
- TODO コメントは禁止、Issue を作成
- 複雑なロジックには必ずコメント

## テスト

- AAA(Arrange-Act-Assert)パターン
- 1 テスト 1 アサーション原則
- モックは最小限、実装に近いテスト

## 例

\`\`\`typescript
// ❌ 好まないスタイル
function getUserData(id) {
  if (id) {
    return db.query('SELECT * FROM users WHERE id = ?', [id])
      .then(result => {
        if (result) {
          return result;
        } else {
          return null;
        }
      });
  }
}

// ✅ 好むスタイル
async function getUserData(userId: string): Promise<User | null> {
  // 早期リターン: 無効な入力をすぐに拒否
  if (!userId) {
    return null;
  }

  const user = await db.user.findUnique({
    where: { id: userId }
  });

  return user ?? null;
}
\`\`\`
```

**効果**: エージェントがあなたのスタイルでコードを生成

### パターン 3: チーム知識ベース

**`.claude/knowledge/team-patterns.md`**:

```markdown
# チーム共通パターン集

## 認証パターン

### JWT 検証ミドルウェア

\`\`\`typescript
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  canActivate(context: ExecutionContext) {
    // チーム標準: GraphQL コンテキストから取得
    return super.canActivate(context);
  }

  handleRequest(err, user, info) {
    if (err || !user) {
      throw new UnauthorizedException('Invalid token');
    }
    return user;
  }
}
\`\`\`

## エラーハンドリングパターン

### カスタム例外階層

\`\`\`typescript
export class ApplicationException extends Error {
  constructor(
    public readonly code: string,
    public readonly message: string,
    public readonly statusCode: number = 500
  ) {
    super(message);
  }
}

export class ValidationException extends ApplicationException {
  constructor(message: string) {
    super('VALIDATION_ERROR', message, 400);
  }
}
\`\`\`

## データベースパターン

### リポジトリパターン

\`\`\`typescript
@Injectable()
export class UserRepository {
  constructor(
    @InjectRepository(User)
    private readonly userRepo: Repository<User>,
  ) {}

  async findByEmail(email: string): Promise<User | null> {
    return this.userRepo.findOne({ where: { email } });
  }
}
\`\`\`

## テストパターン

### 統合テスト setup

\`\`\`typescript
describe('UserController (e2e)', () => {
  let app: INestApplication;
  let userService: UserService;

  beforeAll(async () => {
    const moduleFixture = await Test.createTestingModule({
      imports: [AppModule],
    })
      .overrideProvider(UserService)
      .useValue(mockUserService)
      .compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });
});
\`\`\`
```

**効果**: チーム全体で一貫したコード生成

## エージェントの最適化

### 1. フィードバックループ

```markdown
# .claude/feedback/improvements.md

## エージェント改善履歴

### 2025-01-20

- 問題: テストコードが冗長
- 改善: テストスキルに AAA 原則を明記
- 結果: テスト可読性が向上

### 2025-01-25

- 問題: エラーハンドリングが統一されていない
- 改善: エラーパターンを knowledge に追加
- 結果: 一貫したエラー処理
```

### 2. パフォーマンス測定

```markdown
# .claude/metrics/agent-performance.md

## エージェント効率指標

### レビュー品質
- 検出された問題数: 増加傾向
- 誤検出率: 10% → 3%に改善

### コード生成品質
- 初回コンパイル成功率: 70% → 95%
- スタイル一貫性: 手動修正が50%減少

### 生産性向上
- PR作成時間: 60分 → 30分
- レビュー時間: 30分 → 10分
```

## 高度なエージェント設定

### 1. コンテキスト管理戦略

**動的コンテキスト読み込み**:

```yaml
# .claude/context-config.yml

# プロジェクト開始時に自動読み込み
auto_load:
  - .claude/CLAUDE.md
  - .claude/memory/architecture-decisions.md
  - package.json
  - README.md

# タスク別の追加コンテキスト
tasks:
  review:
    - .claude/knowledge/team-patterns.md
    - .eslintrc.js
    - tsconfig.json

  implement:
    - .claude/knowledge/team-patterns.md
    - .claude/style/coding-preferences.md

  refactor:
    - .claude/knowledge/refactoring-patterns.md
    - .claude/metrics/code-quality.md
```

### 2. ワークフロー自動化

**カスタムワークフロー定義**:

```yaml
# .claude/workflows/feature-complete.yml

name: Feature Complete Workflow
description: 新機能の完全な実装フロー

steps:
  - name: Design
    agent: team-architect
    prompt: "機能を設計"
    output: docs/design.md

  - name: Implement
    agent: general-purpose
    prompt: "設計に基づいて実装"
    context:
      - docs/design.md

  - name: Test
    agent: quality-engineer
    prompt: "包括的なテストを作成"
    validation:
      - coverage > 85%

  - name: Review
    skills:
      - /review-pr
    validation:
      - no critical issues

  - name: Document
    skills:
      - /doc-gen
    output: docs/features/
```

**使用方法**:

```bash
# ワークフロー実行
claude-code workflow feature-complete "ユーザー通知機能"
```

### 3. エージェント間の協調

**複数エージェントの連携例**:

```yaml
# .claude/agent-collaboration/secure-api.yml

name: Secure API Implementation
description: セキュリティを重視したAPI実装

agents:
  - name: backend-architect
    role: 設計
    tasks:
      - API エンドポイント設計
      - データモデル定義
    output: api-design.md

  - name: security-engineer
    role: セキュリティレビュー
    input: api-design.md
    tasks:
      - 脆弱性チェック
      - セキュリティ要件定義
    output: security-requirements.md

  - name: general-purpose
    role: 実装
    input: [api-design.md, security-requirements.md]
    tasks:
      - セキュアな実装
      - 入力バリデーション
      - 認証・認可の実装
    output: implementation

  - name: quality-engineer
    role: テスト
    input: implementation
    tasks:
      - セキュリティテスト作成
      - 侵入テストシナリオ
    validation:
      - all security tests pass
```

## エージェントの育成ロードマップ

### フェーズ 1: 基礎設定(1 週間)

**ステップ 1: エージェント設定ファイル作成**

```bash
# エージェント設定ディレクトリ作成
mkdir -p .claude/agents

# 最初のカスタムエージェント作成
cat > .claude/agents/team-architect.yml << 'EOF'
name: team-architect
description: チーム専用アーキテクトエージェント
base: backend-architect
system_prompt: |
  [チーム固有の原則を記述]
EOF
```

**ステップ 2: メモリファイル準備**

```bash
# メモリディレクトリ作成
mkdir -p .claude/memory

# プロジェクトコンテキストを記録
cat > .claude/memory/project-context.md << 'EOF'
# プロジェクト概要
[プロジェクトの説明]

# 技術スタック
[使用技術のリスト]
EOF
```

### フェーズ 2: 知識蓄積(2-4 週間)

**よく使うパターンを記録**:

```bash
# 知識ベースディレクトリ作成
mkdir -p .claude/knowledge

# チームパターンを記録
cat > .claude/knowledge/team-patterns.md << 'EOF'
# 認証パターン
[コード例]

# エラーハンドリング
[コード例]
EOF
```

### フェーズ 3: 継続的改善(継続的)

**フィードバックと改善**:

1. **使用状況の記録**:
   - どのエージェントをいつ使ったか
   - 成功率と問題点

2. **改善の実施**:
   - system_prompt の調整
   - knowledge の追加
   - tools の最適化

3. **効果測定**:
   - 生産性の向上度合い
   - コード品質の改善
   - エラー率の減少

## まとめ

### サブエージェントの価値

**専門性**:
- 各ドメインの深い知識
- 適切なツールセット
- 最適化された振る舞い

**効率性**:
- 並列実行による高速化
- タスク特化による精度向上
- 自律的な問題解決

**拡張性**:
- カスタムエージェント作成
- チーム知識の蓄積
- 継続的な学習と改善

### 次のステップ

1. **標準エージェントを試す**
   - backend-architect で設計
   - security-engineer でレビュー
   - quality-engineer でテスト

2. **カスタムエージェント作成**
   - チーム固有の原則を定義
   - プロジェクトパターンを記録
   - フィードバックで改善

3. **ワークフロー構築**
   - 複数エージェントの連携
   - 自動化パイプライン
   - 品質ゲートの設定

## 関連ドキュメント

- [スキル](./skills.md) - タスク固有の専門知識
- [カスタマイズ](./customize.md) - 全体的なカスタマイズ戦略
- [ワークフロー自動化](./workflow-automation.md) - エージェントを組み合わせた自動化
