# Claude Skills

Claude Skills は、Claude Code で Claude の能力を拡張するモジュール化された機能拡張システムです。

## 概要

### Claude Skill とは

**Claude Skills** は、特定のタスクを自動的に実行するためのモジュール化されたフォルダです。スラッシュコマンドとは異なり、**ユーザーの明示的な指示がなくても**、Claude がリクエストの文脈に基づいて**自動的に発動**します。

**主な特徴**

- **自動起動**: リクエスト内容に基づいて Claude が自動判断して発動
- **モデル駆動型**: Skill の `description` を Claude が分析し、適切性を判断
- **構造化された能力**: 複数ファイル、複雑なワークフロー、チーム標準化に最適
- **Tool アクセス制御**: 各 Skill が使用できる Tool を制限可能

### Skill vs スラッシュコマンド

| 特性         | Skill                                        | スラッシュコマンド                 |
| ------------ | -------------------------------------------- | ---------------------------------- |
| **起動方式** | 自動（context ベース）                       | 明示的（`/` 指定）                 |
| **発見**     | Claude が認識して自動起動                    | ユーザーが明示的に呼び出し         |
| **構造**     | 複数ファイル可、複雑な能力向け               | シンプル、単一ファイル             |
| **用途**     | 複雑なワークフロー、チーム標準化             | よく使う簡単なプロンプト           |
| **例**       | PDF 処理、セキュリティ分析、ドキュメント生成 | `/review`, `/explain`, `/optimize` |

**選択基準**

- **スラッシュコマンドを選ぶ**: 簡単で頻繁に使うプロンプト
- **Skill を選ぶ**: Claude が自動的に認識すべき複雑な能力、複数ファイル、チーム共有

---

## Skill の種類

Claude Code には **3 種類の Skill** があります。

### 1. Personal Skills (`~/.claude/skills/`)

**所有**: ユーザー個人

**スコープ**: ローカルマシン上のすべてのプロジェクト

**用途**

- 個人的なワークフロー
- 実験的なツール
- 個人向けカスタマイズ

**共有**: チーム共有されない（ローカル限定）

**配置場所**

```
~/.claude/skills/
├── my-personal-skill/
│   └── SKILL.md
└── experimental-tool/
    └── SKILL.md
```

### 2. Project Skills (`.claude/skills/`)

**所有**: プロジェクト

**スコープ**: 特定のプロジェクトのみ

**用途**

- チーム共有の専門知識
- プロジェクト固有のワークフロー
- 標準化されたチームプラクティス

**共有**: Git で管理され、チーム全体が利用可能

**配置場所**

```
my-project/
├── .claude/
│   └── skills/
│       ├── security-reviewer/
│       │   └── SKILL.md
│       └── doc-generator/
│           └── SKILL.md
└── src/
```

**推奨される用途**

- コードレビュー標準
- セキュリティチェック
- ドキュメント生成
- テスト戦略

### 3. Plugin Skills

**所有**: Plugin 開発者

**スコープ**: Plugin がインストールされた環境

**構造**

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
├── commands/
├── agents/
├── skills/          ← Plugin skills ここに配置
│   └── plugin-skill/
│       └── SKILL.md
└── hooks/
```

### Skill の優先順位

同じ名前の Skill が複数存在する場合、以下の優先順位で適用されます。

1. **Project Skill** (`.claude/skills/`) - 最も高い優先度
2. **Personal Skill** (`~/.claude/skills/`)
3. **Plugin Skill**

---

## Skill の作成方法

### ファイル構造

各 Skill は独立したディレクトリに配置し、必須ファイル `SKILL.md` と任意のサポートファイルを含めます。

```
.claude/skills/my-skill/
├── SKILL.md              # メイン skill ファイル（必須）
├── script.sh             # 支援スクリプト（オプション）
├── template.txt          # テンプレート（オプション）
└── additional-file.md    # その他のサポートファイル（オプション）
```

### SKILL.md の構造

`SKILL.md` は **YAML フロントマター** + **Markdown Instructions** の形式で記述します。

```markdown
---
name: my-skill
description: "Brief description of what this Skill does and when to use it"
allowed-tools: [Read, Grep, WebSearch] # オプション：tool アクセス制限
---

# Skill Instructions

[Markdown 形式の詳細な instructions...]
```

### YAML フロントマターの必須フィールド

| フィールド      | 型     | 説明                                   | 制限                                       |
| --------------- | ------ | -------------------------------------- | ------------------------------------------ |
| `name`          | 文字列 | Skill の一意な名前                     | 小文字、英数字、ハイフンのみ、最大 64 文字 |
| `description`   | 文字列 | **最重要**: Skill の機能と使用トリガー | 最大 1,024 文字                            |
| `allowed-tools` | 配列   | アクセス可能な Tool を制限             | オプション                                 |

### description フィールドの重要性

`description` は Claude が Skill を認識し、発動するかどうかを判断する**最も重要な要素**です。

**良い description の書き方**

- ✅ **具体的なトリガーシナリオを含める**: "When analyzing security vulnerabilities..."
- ✅ **機能と用途を明確に**: "Finds and classifies SQL injection, XSS, CSRF risks"
- ✅ **ユースケースを示す**: "Use when reviewing authentication systems, API endpoints, data validation"

**悪い description の例**

- ❌ "helps with documents" → あまりにも曖昧
- ❌ "processes files" → トリガーが不明確
- ❌ "useful tool" → 何に使うのか不明

**良い description の例**

- ✅ "Analyzes PDF documents for security vulnerabilities and compliance issues. Use when reviewing PDF files for sensitive data, validating document structure, or extracting metadata."
- ✅ "Performs comprehensive security code review, identifying SQL injection, XSS, CSRF, authentication flaws, data exposure, and access control vulnerabilities in source code. Activate when reviewing authentication systems, API endpoints, user input handlers, or database queries."

### allowed-tools による Tool アクセス制限

セキュリティやパフォーマンスのため、Skill がアクセスできる Tool を制限できます。

```yaml
---
name: security-analyzer
description: "Analyzes code for security vulnerabilities"
allowed-tools: [Read, Grep, WebSearch] # Bash や Edit は許可しない
---
```

**使用例**

- **読み取り専用 Skill**: `[Read, Grep]`
- **分析専用 Skill**: `[Read, Grep, WebSearch]`
- **実装 Skill**: `[Read, Grep, Edit, Write]`

---

## Skill の実行方法

### 発動メカニズム

Skill は **自動的に発動**されます（ユーザーによる明示的な呼び出しは不要）。

```
ユーザーのリクエスト
    ↓
Claude がリクエストを分析
    ↓
登録されている Skill の descriptions と比較
    ↓
マッチング → Skill を自動的に起動
```

**例**

```
ユーザー: "このコードのセキュリティ脆弱性をチェックして"
    ↓
Claude が "security-code-reviewer" Skill の description と照合
    ↓
description に "security vulnerabilities" や "authentication systems" が含まれている
    ↓
"security-code-reviewer" Skill が自動起動
```

### 発動の条件

Skill が起動されるには以下の条件を満たす必要があります。

1. **YAML フロントマターが有効** (構文が正しい)
2. **description がリクエストと関連している**
3. **Skill がアクセス可能なディレクトリに存在する**

### Subagent での使用

Subagent の設定で Skill を明示的に指定することも可能です。

```yaml
---
name: code-reviewer
skills: security-checker,code-quality-analyzer
---
# Subagent instructions...
```

これにより、特定の Subagent が特定の Skill を使用することを保証できます。

---

## ベストプラクティス

### 設計原則

**単一責任の原則**

- ✅ 各 Skill は **1 つの能力に焦点** を当てる
- ✅ 複数の関連機能は、複数の Skill に分割する
- ❌ 1 つの Skill に複数の無関係な機能を含める

**例**

```
❌ 悪い設計
  general-helper/
    └── SKILL.md  (PDF処理、セキュリティ分析、ドキュメント生成を全て含む)

✅ 良い設計
  pdf-analyzer/
    └── SKILL.md  (PDF処理のみ)
  security-reviewer/
    └── SKILL.md  (セキュリティ分析のみ)
  doc-generator/
    └── SKILL.md  (ドキュメント生成のみ)
```

### description の書き方

**具体的なトリガーシナリオを含める**

```yaml
# ❌ 悪い例
description: "helps with code review"

# ✅ 良い例
description: "Performs comprehensive code review focusing on readability, maintainability, and best practices. Activates when reviewing pull requests, analyzing code quality, or checking adherence to coding standards. Provides actionable feedback with examples and references to best practices."
```

**機能とユースケースを明確に**

```yaml
# ❌ 悪い例
description: "PDF tool"

# ✅ 良い例
description: "Analyzes PDF documents for content extraction, structure analysis, security review, and data extraction. Activates when you need to process PDF files, extract data, review document structure, or audit PDF security properties. Supports multi-page documents and table extraction."
```

### テスト手法

**発動テスト**

Skill を作成したら、発動すべき質問を投げかけてテストします。

```
1. Skill "security-reviewer" を作成
2. テスト質問: "このコードにセキュリティ問題はある?"
3. Claude が "security-reviewer" を自動起動するか確認
4. 起動しない場合は description を修正
```

**Tool アクセステスト**

`allowed-tools` を設定した場合、制限が正しく機能するかテストします。

```yaml
---
name: read-only-analyzer
allowed-tools: [Read, Grep]
---
```

テスト: Skill が Edit や Bash を使おうとしないか確認

### Git 管理

**Project Skill の管理**

Project Skill は Git で管理し、チーム全体が参照可能にします。

```bash
# .gitignore に追加しない
# .claude/skills/ をコミット対象に含める

git add .claude/skills/
git commit -m "Add security-reviewer skill"
git push
```

**バージョン管理**

Skill の更新履歴を追跡することで、変更の影響を把握できます。

```bash
# Skill の更新履歴を確認
git log -- .claude/skills/security-reviewer/

# 特定バージョンの Skill に戻す
git checkout <commit-hash> -- .claude/skills/security-reviewer/
```

---

## 具体例

### 例 1: セキュリティコードレビュー Skill

````yaml
---
name: security-code-reviewer
description: "Performs comprehensive security code review, identifying SQL injection, XSS, CSRF, authentication flaws, data exposure, and access control vulnerabilities in source code. Activate when reviewing authentication systems, API endpoints, user input handlers, or database queries."
allowed-tools: [Read, Grep, WebSearch]
---

# Security Code Review Skill

## Purpose
Analyze code for common security vulnerabilities and suggest remediation.

## Activation Triggers
This skill activates automatically when you ask to:
- Review code for security issues
- Find vulnerabilities in authentication systems
- Analyze user input handling
- Check database query safety
- Review API endpoint security

## Analysis Framework

### 1. Authentication & Authorization
- Check for hardcoded credentials
- Verify proper password hashing (bcrypt, argon2)
- Validate JWT implementation
- Check role-based access control

**Example Check:**
```javascript
// ❌ Bad: Hardcoded credentials
const password = "admin123";

// ✅ Good: Environment variables
const password = process.env.DB_PASSWORD;
````

### 2. Input Validation

- SQL injection risks
- XSS vulnerabilities
- CSRF protection
- Command injection

**Example Check:**

```javascript
// ❌ Bad: Direct string interpolation
const query = `SELECT * FROM users WHERE id = ${userId}`;

// ✅ Good: Parameterized query
const query = "SELECT * FROM users WHERE id = ?";
db.query(query, [userId]);
```

### 3. Data Protection

- Sensitive data exposure
- Encryption at rest/in transit
- Proper secrets management

**Example Check:**

```javascript
// ❌ Bad: Logging sensitive data
console.log("User password:", password);

// ✅ Good: Redacted logging
console.log("User authenticated:", userId);
```

## Output Format

Provide findings in priority order:

**Critical**: Immediate security risks (SQL injection, auth bypass)
**High**: Significant vulnerabilities (XSS, CSRF)
**Medium**: Security best practice violations
**Low**: Minor security improvements

## Remediation Guidance

For each finding:

1. Explain the vulnerability
2. Show vulnerable code snippet
3. Provide secure alternative
4. Reference OWASP or security standards

```

**使用例**

```

ユーザー: "この認証システムにセキュリティ問題はありますか?"
↓
Claude が "security-code-reviewer" を自動起動
↓
コードをスキャンして脆弱性を検出
↓
優先度順に結果を報告 (Critical → High → Medium → Low)

````

### 例 2: PDF 分析 Skill

```yaml
---
name: pdf-analyzer
description: "Analyzes PDF documents for content extraction, structure analysis, security review, and data extraction. Activates when you need to process PDF files, extract data, review document structure, or audit PDF security properties. Supports multi-page documents and table extraction."
allowed-tools: [Read, Bash]
---

# PDF Analysis Skill

## Purpose
Comprehensive PDF document analysis and data extraction.

## Capabilities

### 1. Document Structure Analysis
- Page count and layout
- Embedded fonts and images
- Metadata extraction
- Bookmarks and navigation

### 2. Content Extraction
- Text extraction from all pages
- Table detection and extraction
- Image extraction
- Form field identification

### 3. Security Review
- Encryption status
- Password protection
- Digital signatures
- Permissions and restrictions

### 4. Data Extraction
- Extract structured data from forms
- Parse tables into CSV/JSON
- Identify and extract email addresses, URLs
- Custom pattern extraction

## Analysis Workflow

1. **Initial Assessment**
   ```bash
   pdfinfo document.pdf
````

2. **Content Extraction**

   ```bash
   pdftotext document.pdf output.txt
   ```

3. **Table Extraction**

   ```bash
   tabula-py --format csv document.pdf
   ```

4. **Security Analysis**
   ```bash
   qpdf --show-encryption document.pdf
   ```

## Output Format

### Summary

- File: `document.pdf`
- Pages: 25
- File size: 2.5 MB
- Encryption: None
- Forms: Yes (12 fields)

### Extracted Data

- Text content: See `output.txt`
- Tables: 3 tables extracted to CSV
- Images: 8 images extracted

### Security Status

- ✅ No encryption (publicly accessible)
- ⚠️ Contains form fields (review for PII)
- ✅ No password required

```

**使用例**

```

ユーザー: "この PDF ファイルから表データを抽出して"
↓
Claude が "pdf-analyzer" を自動起動
↓
PDF を分析してテーブルを検出
↓
CSV 形式でデータを抽出して返す

````

### 例 3: テクニカルドキュメント生成 Skill

```yaml
---
name: technical-documentation-generator
description: "Generates comprehensive technical documentation for APIs, functions, and system architecture. Creates well-structured markdown with examples, diagrams, and usage patterns. Use when documenting new APIs, creating architecture guides, or writing SDK documentation."
allowed-tools: [Read, Grep, Write]
---

# Technical Documentation Generator

## Purpose
Automate creation of high-quality technical documentation.

## Activation Scenarios
- Creating API documentation
- Writing architecture guides
- Documenting new functions/classes
- Generating SDK documentation
- Creating integration guides

## Documentation Templates

### API Documentation Template

```markdown
# API Name

## Overview
Brief description of the API purpose and capabilities.

## Base URL
`https://api.example.com/v1`

## Authentication
[Authentication method and requirements]

## Endpoints

### GET /endpoint

**Description**: What this endpoint does

**Parameters**:
| Name | Type | Required | Description |
|------|------|----------|-------------|
| id   | string | Yes    | Resource ID |

**Request Example**:
```bash
curl -X GET https://api.example.com/v1/endpoint?id=123
````

**Response Example**:

```json
{
  "id": "123",
  "status": "success"
}
```

**Error Codes**:
| Code | Description |
|------|-------------|
| 400 | Bad Request |
| 404 | Not Found |

````

### Function Documentation Template

```markdown
## functionName(param1, param2)

**Description**: What this function does

**Parameters**:
- `param1` (Type): Description
- `param2` (Type): Description

**Returns**: Return type and description

**Example**:
```javascript
const result = functionName('value1', 'value2');
console.log(result); // Expected output
````

**Throws**:

- `TypeError`: When invalid type is provided
- `RangeError`: When value out of range

```

## Documentation Best Practices

1. **Clear Structure**: Logical hierarchy and navigation
2. **Code Examples**: Real, working examples for all features
3. **Error Handling**: Document all error cases
4. **Performance**: Note performance characteristics
5. **Version Info**: Specify version compatibility

## Output Format

Generate documentation in the following structure:

1. **Overview**: Purpose and scope
2. **Quick Start**: Minimal example to get started
3. **Detailed API Reference**: Complete endpoint/function documentation
4. **Examples**: Real-world usage scenarios
5. **Error Reference**: Common errors and solutions
6. **Changelog**: Version history (if applicable)
```

**使用例**

```
ユーザー: "この API の完全なドキュメントを生成して"
    ↓
Claude が "technical-documentation-generator" を自動起動
    ↓
コードを分析してエンドポイント、パラメータ、レスポンスを抽出
    ↓
構造化されたマークダウンドキュメントを生成
```

---

## トラブルシューティング

### Skill が起動しない

**原因 1: YAML 構文エラー**

```yaml
# ❌ エラー: description の引用符が閉じていない
---
name: my-skill
description: "This is a skill
---

# ✅ 修正: 正しい引用符
---
name: my-skill
description: "This is a skill"
---
```

**解決方法**: YAML 構文チェッカーで検証

**原因 2: description が不明瞭**

```yaml
# ❌ 不明瞭
description: "helps with code"

# ✅ 明確
description: "Analyzes code for performance issues, identifying slow algorithms, memory leaks, and inefficient database queries. Use when optimizing performance or debugging slow operations."
```

**解決方法**: 具体的なトリガーシナリオを追加

**原因 3: Skill ディレクトリの配置ミス**

```
❌ 間違った配置
my-project/
└── skills/  # ここではない
    └── SKILL.md

✅ 正しい配置
my-project/
└── .claude/
    └── skills/
        └── my-skill/
            └── SKILL.md
```

### allowed-tools の制限が機能しない

**確認事項**

1. YAML 配列の構文が正しいか
2. Tool 名のスペルが正確か
3. Skill が正しく読み込まれているか

```yaml
# ❌ 間違った構文
allowed-tools: Read, Grep  # カンマ区切りではダメ

# ✅ 正しい構文
allowed-tools: [Read, Grep]  # 配列形式
```

### 複数の Skill が競合する

**原因**: 複数の Skill が同じトリガーに反応する

**解決方法**

1. description をより具体的にする
2. Skill の責任範囲を明確に分ける
3. 優先度の高い Skill を Project Skill に配置

```yaml
# Skill A: セキュリティ専門
description: "Security-focused code review for authentication, authorization, and data protection vulnerabilities."

# Skill B: パフォーマンス専門
description: "Performance analysis focusing on algorithm complexity, database optimization, and resource usage."
```

---

## 高度なテクニック

### 複数ファイルを使った Skill

複雑な Skill では、複数のサポートファイルを使用できます。

```
security-reviewer/
├── SKILL.md              # メイン instructions
├── owasp-checklist.md    # OWASP チェックリスト
├── cwe-database.json     # CWE (Common Weakness Enumeration) データ
└── security-rules.yaml   # カスタムセキュリティルール
```

**SKILL.md での参照**

```markdown
---
name: security-reviewer
description: "Comprehensive security review using OWASP Top 10 and CWE database"
---

# Security Review Skill

## Checklist

Refer to [OWASP Checklist](owasp-checklist.md) for detailed checks.

## CWE Database

Use `cwe-database.json` to classify vulnerabilities by CWE ID.

## Custom Rules

Project-specific security rules are defined in `security-rules.yaml`.
```

### 動的スクリプトの使用

Bash スクリプトを使って動的な分析を実行できます。

```
pdf-analyzer/
├── SKILL.md
└── analyze.sh  # PDF 分析スクリプト
```

**analyze.sh**

```bash
#!/bin/bash

PDF_FILE="$1"

echo "=== PDF Analysis ==="
echo "File: $PDF_FILE"
echo ""

# メタデータ抽出
echo "=== Metadata ==="
pdfinfo "$PDF_FILE"

# テキスト抽出
echo "=== Text Content ==="
pdftotext "$PDF_FILE" -

# セキュリティ情報
echo "=== Security ==="
qpdf --show-encryption "$PDF_FILE"
```

**SKILL.md での使用**

````markdown
To analyze a PDF:

```bash
bash analyze.sh document.pdf
```
````

````

### チーム標準の強制

Project Skill を使ってチーム標準を強制できます。

```yaml
---
name: team-code-standards
description: "Enforces team coding standards including naming conventions, file structure, import ordering, and comment styles. Activate when reviewing code for standards compliance or setting up new files."
allowed-tools: [Read, Grep, Edit]
---

# Team Code Standards

## Naming Conventions
- Files: `kebab-case.ts`
- Classes: `PascalCase`
- Functions: `camelCase`
- Constants: `UPPER_SNAKE_CASE`

## File Structure
````

src/
├── components/ # React components
├── services/ # Business logic
├── utils/ # Helper functions
└── types/ # TypeScript types

```

## Import Ordering
1. External libraries
2. Internal modules
3. Types
4. Styles

## Enforcement
When creating or reviewing files, verify:
- [ ] Naming convention compliance
- [ ] Correct directory placement
- [ ] Import ordering
- [ ] Comment style (JSDoc)
```

---

## まとめ

### Skill 開発のステップ

1. **目的を定義**: Skill が解決する問題を明確にする
2. **description を作成**: 具体的なトリガーシナリオを含める
3. **Instructions を書く**: 詳細な手順とテンプレートを提供
4. **テスト**: 発動テストと機能テストを実施
5. **反復改善**: description と instructions を最適化

### 重要なポイント

- ✅ **description が最重要**: Claude の発動判断の鍵
- ✅ **単一責任**: 各 Skill は 1 つの能力に焦点
- ✅ **具体的なトリガー**: 曖昧な説明を避ける
- ✅ **Tool アクセス制限**: セキュリティのため適切に制限
- ✅ **Git 管理**: Project Skill をバージョン管理

### 次のステップ

1. [Slash Commands](command.md) - スラッシュコマンドでよく使うプロンプトを効率化
2. [Sub-agents](sub-agent.md) - 複雑なタスクをサブエージェントに委任

## 関連

[Claude Code](README.md) | [Slash Commands](command.md) | [Sub-agents](sub-agent.md)
