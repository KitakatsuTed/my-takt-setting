# .takt/ 要件定義ワークフロー設定

## 概要

このディレクトリは、[takt](https://github.com/nrslib/takt)（Claude Code用ワークフローライブラリ）をカスタマイズした設定群です。

Webアプリ開発プロジェクト（開発者3〜6名、クライアント協業体制）において、**要件定義ドキュメントの作成・整合性チェック・品質レビューをAIで効率化する**ことを目的としています。

具体的には、プロジェクト概要を入力するだけで、IPA共通フレーム2013やBABOK v3に準拠した11種類の要件定義ドキュメントを自動生成し、ドキュメント間の整合性チェックや品質レビューまでをClaude Codeのワークフローとして実行できます。

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
│   │   ├── requirements-writer.md    ← 要件定義書作成者（ドキュメントを生成）
│   │   └── requirements-checker.md   ← 整合性チェック専門家（矛盾・漏れを検出）
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
    ├── requirements-generator.yaml     ← テンプレート自動生成
    ├── requirements-consistency.yaml   ← 整合性チェック
    └── requirements-review.yaml        ← 品質レビュー
```

### facets/ の各ファイルの役割

| ファイル | 種別 | 役割 |
|---------|------|------|
| `personas/requirements-planner.md` | Persona | プロジェクト概要を分析し、11種のドキュメント生成計画とID体系を立案する。質問せず、不足情報には仮定を置いて自律的に進める |
| `personas/requirements-writer.md` | Persona | 生成計画に基づいてドキュメントを作成する。IPA共通フレーム2013・BABOK v3に準拠し、Draw.ioで図表も作成する |
| `personas/requirements-checker.md` | Persona | ドキュメント間のID参照整合性、網羅性、用語一貫性、意味的矛盾を検出する。重大度（High/Medium/Low）付きで報告する |
| `policies/requirements.md` | Policy | ID体系の統一、自律行動、ドキュメント品質、図表品質に関するREJECT/OK判定基準を定義 |
| `knowledge/requirements-documents.md` | Knowledge | コアドキュメント11種の構成（セクション・カラム）とドキュメント間の依存関係 |
| `knowledge/consistency-patterns.md` | Knowledge | ID参照整合性・名称一貫性・網羅性・意味的矛盾・図表整合性のチェックパターンとトレーサビリティマトリクス |
| `output-contracts/requirements-template.md` | Template | 11種のドキュメント + 生成計画のMarkdownテンプレート（ファイル名: 00〜11） |

## 使い方

### requirements-generator: テンプレート自動生成

プロジェクト概要から11種のコアドキュメントを一括生成します。

```bash
takt run requirements-generator
```

**処理の流れ（5ステップ、最大20ムーブメント）:**

1. **gather-info** (Planner) -- プロジェクト概要を分析し、仮定を設定、ID体系を定義、生成計画を立案
2. **generate-core** (Writer) -- BRD、業務フロー図（As-Is/To-Be）、要求一覧を生成
3. **generate-screens** (Writer) -- 機能一覧、画面一覧、画面遷移図、ワイヤーフレームを生成
4. **generate-nfr** (Writer) -- 非機能要件定義書、セキュリティ要件定義書、要件合意書、スコープ記述書+変更管理台帳を生成
5. **validate** (Supervisor) -- 全11種のドキュメントを検証し、問題があればgather-infoに差し戻し

**入力:** プロジェクト概要（自由テキスト）
**出力:** 12ファイル（生成計画 + 11種のドキュメント）

### requirements-consistency: 整合性チェック

生成済みの要件定義ドキュメント間の矛盾・漏れを検出します。

```bash
takt run requirements-consistency
```

**処理の流れ（3ステップ、最大10ムーブメント）:**

1. **gather** (Planner) -- ドキュメントを収集し、ID体系を特定、チェック準備
2. **checkers** (Checker x4 並列実行) -- 以下の4観点を同時にチェック
   - **ID参照整合性**: ダングリング参照、孤立IDの検出
   - **網羅性**: BRDゴール→要求→機能のトレーサビリティ検証
   - **用語一貫性**: 名称揺れ、未定義用語の検出
   - **意味的矛盾**: BRDとスコープの矛盾、非機能要件との不整合の検出
3. **synthesize** (Supervisor) -- 4つのチェック結果を重大度順に統合し、修正優先順位を提示

**入力:** 要件定義ドキュメント群（ディレクトリ指定）
**出力:** 6ファイル（チェック対象情報 + 4観点チェック結果 + 統合レポート）

### requirements-review: 品質レビュー

要件定義ドキュメントの品質をビジネス・UX・技術の3観点からレビューします。

```bash
takt run requirements-review
```

**処理の流れ（3ステップ、最大10ムーブメント）:**

1. **gather** (Planner) -- ドキュメントを把握し、プロジェクト特性を理解、レビュー準備
2. **reviewers** (Checker x3 並列実行) -- 以下の3観点を同時にレビュー
   - **ビジネス観点**: ゴールの明確さ、KPIの定量性、スコープの適切さ、ROIの現実性
   - **UX観点**: 業務フローの効率性、画面遷移の自然さ、UIの使いやすさ、エラーケースの考慮
   - **技術観点**: 技術的実現可能性、非機能要件の現実性、セキュリティ要件の十分性、開発規模の妥当性
3. **synthesize** (Supervisor) -- 3つのレビュー結果を統合し、総合評価（A〜D）と改善提案を提示

**入力:** 要件定義ドキュメント群（ディレクトリ指定）
**出力:** 5ファイル（レビュー対象情報 + 3観点レビュー結果 + 統合レポート）

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

`pieces/` 内のYAMLファイルを編集します。ムーブメントの追加・削除・順序変更が可能です。例えば、新しいレビュー観点を追加したい場合は、`requirements-review.yaml` の `parallel` セクションにムーブメントを追加します。
