# .takt/ 要件定義ワークフロー設定

## 概要

このディレクトリは、[takt](https://github.com/nrslib/takt)（Claude Code用ワークフローライブラリ）をカスタマイズした設定群です。

Webアプリ開発プロジェクト（開発者3〜6名、クライアント協業体制）において、**要件定義ドキュメントの作成・品質レビュー・整合性チェック・修正までをAIで一貫実行する**ことを目的としています。

具体的には、プロジェクト概要を入力するだけで、IPA共通フレーム2013やBABOK v3に準拠した11種類の要件定義ドキュメントを自動生成し、品質レビュー・整合性チェック・修正までをClaude Codeのワークフローとして実行できます。High問題が0件になるまで自動でループし、最大10回まで繰り返します。

## 前提条件

### takt のインストール

taktはnpmパッケージとして提供されています。プロジェクトにインストールしてください。

```bash
npm install -g @nrslib/takt
```

詳細は [takt公式リポジトリ](https://github.com/nrslib/takt) を参照してください。

### Draw.io MCPサーバーの設定

業務フロー図・画面遷移図・ワイヤーフレームの作成にDraw.ioのMCPサーバーを使用します。Claude Codeの MCP設定で Draw.io MCPサーバーを有効にしておく必要があります。

使用するMCPツール: `drawio_create_diagram`

### Claude Code

taktはClaude Code上で動作します。Claude Codeがインストールされ、利用可能な状態であることを確認してください。

## ディレクトリ構成

```
.takt/
├── README.md                 ← このファイル
├── .gitignore                ← ランタイムファイル（runs/, persona_sessions.json等）を除外
├── persona_sessions.json     ← taktのセッション管理（自動生成、Git管理対象外）
│
├── facets/                   ← Persona・Policy・Knowledge・テンプレートの定義
│   ├── personas/             ← AIエージェントの役割定義
│   │   ├── requirements-planner.md   ← 要件定義計画者（生成計画を立案）
│   │   ├── requirements-writer.md    ← 要件定義書作成者（ドキュメントを生成・修正）
│   │   ├── requirements-checker.md   ← 品質レビュー・整合性チェック専門家
│   │   └── requirements-supervisor.md ← 統合判定・品質管理（レビュー＋チェック結果を統合判定）
│   │
│   ├── policies/             ← 品質・行動規範
│   │   └── requirements.md           ← 要件定義ポリシー（ID体系、品質基準、REJECT/OK判定）
│   │
│   ├── knowledge/            ← ドメイン知識
│   │   ├── requirements-documents.md ← コアドキュメント11種の構成と依存関係
│   │   └── consistency-patterns.md   ← 整合性チェックのパターンと手法
│   │
│   └── output-contracts/     ← 出力テンプレート
│       └── requirements-template.md  ← 11種のドキュメントのMarkdownテンプレート
│
└── pieces/                   ← 実行可能なワークフロー定義
    └── requirements-generator.yaml     ← ドキュメント生成・レビュー・チェック・修正の一貫実行
```

### facets/ の各ファイルの役割

| ファイル | 種別 | 役割 |
|---------|------|------|
| `personas/requirements-planner.md` | Persona | プロジェクト概要を分析し、11種のドキュメント生成計画とID体系を立案する。質問せず、不足情報には仮定を置いて自律的に進める |
| `personas/requirements-writer.md` | Persona | 生成計画に基づいてドキュメントを作成・修正する。IPA共通フレーム2013・BABOK v3に準拠し、Draw.ioで図表も作成する |
| `personas/requirements-checker.md` | Persona | ドキュメントの品質レビュー（ビジネス・UX・技術）と整合性チェック（ID参照・網羅性・用語・意味的矛盾）を実施する。重大度（High/Medium/Low）付きで報告する |
| `personas/requirements-supervisor.md` | Persona | 品質レビューと整合性チェックの結果を統合し、High問題の有無を判定する。修正指示を具体的に提示する |
| `policies/requirements.md` | Policy | ID体系の統一、自律行動、ドキュメント品質、図表品質に関するREJECT/OK判定基準を定義 |
| `knowledge/requirements-documents.md` | Knowledge | コアドキュメント11種の構成（セクション・カラム）とドキュメント間の依存関係 |
| `knowledge/consistency-patterns.md` | Knowledge | ID参照整合性・名称一貫性・網羅性・意味的矛盾・図表整合性のチェックパターンとトレーサビリティマトリクス |
| `output-contracts/requirements-template.md` | Template | 11種のドキュメント + 生成計画のMarkdownテンプレート（ファイル名: 00〜11） |

## 使い方

### requirements-generator: ドキュメント生成・レビュー・チェック・修正

プロジェクト概要から11種のコアドキュメントを一括生成し、品質レビュー・整合性チェック・修正までを自動実行します。

```bash
takt run requirements-generator
```

**処理の流れ（8ステップ、最大50ムーブメント）:**

1. **gather-info** (Planner) — プロジェクト概要を分析し、仮定を設定、ID体系を定義、生成計画を立案
2. **generate-core** (Writer) — BRD、業務フロー図（As-Is/To-Be）、要求一覧を生成
3. **generate-screens** (Writer) — 機能一覧、画面一覧、画面遷移図、ワイヤーフレームを生成
4. **generate-nfr** (Writer) — 非機能要件定義書、セキュリティ要件定義書、要件合意書、スコープ記述書+変更管理台帳を生成
5. **reviewers** (Checker x3 並列実行) — ビジネス・UX・技術の3観点から品質レビュー
6. **checkers** (Checker x4 並列実行) — ID参照整合性・網羅性・用語一貫性・意味的整合性の4観点をチェック
7. **synthesize** (Supervisor) — レビュー＋チェック結果を統合し、High問題の有無を判定
8. **fix** (Writer) — High問題に基づいてドキュメントを修正

**ループ:** ステップ5〜8をHigh問題が0件になるまで繰り返す（最大約10回）。上限到達時は残存するHigh問題を一覧で報告して終了。

**入力:** プロジェクト概要（自由テキスト）
**出力:** 生成計画 + 11種のドキュメント + 品質判定レポート

## コアドキュメント11種

要件定義で管理する11種のドキュメントは、3つのフェーズで段階的に生成されます。

### Phase 1: Core（基盤ドキュメント）

| # | ドキュメント | ファイル名 | 概要 |
|---|------------|-----------|------|
| 1 | ビジネス要件定義書（BRD） | `01-brd.md` | 全ドキュメントの起点。プロジェクトの目的・背景・ゴール・KPI・スコープを定義 |
| 2 | 業務フロー図（As-Is / To-Be） | `02-business-flow.md` | 現状と改善後の業務プロセスをDraw.ioでフローチャート化 |
| 3 | 要求一覧 | `03-requirements-list.md` | ステークホルダーからの要求をREQ-IDで一元管理 |

### Phase 2: Screens（画面系ドキュメント）

| # | ドキュメント | ファイル名 | 概要 |
|---|------------|-----------|------|
| 4 | 機能一覧 | `04-function-list.md` | 要求をFUNC-IDで機能に分解 |
| 5 | 画面一覧 | `05-screen-list.md` | 機能をSCR-IDで画面に対応付け |
| 6 | 画面遷移図 | `06-screen-transition.md` | 画面間の遷移をDraw.ioで状態遷移図化 |
| 7 | ワイヤーフレーム | `07-wireframes.md` | 主要画面のUIレイアウトをDraw.ioで作成 |

### Phase 3: NFR（非機能要件・管理系ドキュメント）

| # | ドキュメント | ファイル名 | 概要 |
|---|------------|-----------|------|
| 8 | 非機能要件定義書 | `08-nfr.md` | IPA非機能要求グレード6大項目（可用性・性能・運用・移行・セキュリティ・環境）に準拠 |
| 9 | セキュリティ要件定義書 | `09-security.md` | OWASP Top 10を参考にした認証・暗号化・アクセス制御・監査ログ・脆弱性対策 |
| 10 | 要件合意書 | `10-agreement.md` | ステークホルダー間の合意を文書化 |
| 11 | プロジェクトスコープ記述書 + 変更管理台帳 | `11-scope-and-change.md` | スコープ定義と変更追跡を一体管理 |

### ID体系

ドキュメント横断で以下のID体系を使用します。

| 対象 | プレフィックス | 例 |
|------|--------------|-----|
| ビジネス要件 | BRD- | BRD-001 |
| 要求 | REQ- | REQ-001 |
| 機能 | FUNC- | FUNC-001 |
| 画面 | SCR- | SCR-001 |
| 非機能要件 | NFR- | NFR-001 |
| セキュリティ要件 | SEC- | SEC-001 |
| 変更要求 | CHG- | CHG-001 |

### ドキュメント間の依存関係

```
BRD
 ├── 業務フロー図（As-Is/To-Be）
 │     └── 要求一覧
 │           ├── 機能一覧
 │           │     ├── 画面一覧
 │           │     │     ├── 画面遷移図
 │           │     │     └── ワイヤーフレーム
 │           │     └── 非機能要件定義書
 │           │           └── セキュリティ要件定義書
 │           └── プロジェクトスコープ記述書
 │                 └── 変更管理台帳
 └── 要件合意書
```

上流ドキュメントの内容が下流に影響するため、生成順序はこの依存関係に従います。

## カスタマイズ方法

### Personaの変更

`facets/personas/` 内のMarkdownファイルを編集します。各Personaには「役割の境界」「行動姿勢」「ドメイン知識」が定義されています。例えば、新しい業界特有の知識を追加したい場合は、該当Personaの「ドメイン知識」セクションに追記してください。

### Policyの変更

`facets/policies/requirements.md` を編集します。REJECT/OK判定基準を変更することで、ドキュメント品質のしきい値を調整できます。

### テンプレートの変更

`facets/output-contracts/requirements-template.md` を編集します。各ドキュメントのセクション構成やカラムを変更できます。変更した場合、対応するPieceのYAMLファイル内の `output_contracts` セクションも合わせて更新してください。

### Knowledgeの追加

`facets/knowledge/` にMarkdownファイルを追加し、Pieceの `knowledge` セクションで参照を追加します。例えば、特定業界の規制要件などを追加できます。

### ワークフローの変更

`pieces/requirements-generator.yaml` を編集します。ムーブメントの追加・削除・順序変更が可能です。例えば、新しいレビュー観点を追加したい場合は、`reviewers` の `parallel` セクションにサブムーブメントを追加します。
