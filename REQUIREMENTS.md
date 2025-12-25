# A.K.A.T.S.U.K.I. システム要件定義書

**System:** A.K.A.T.S.U.K.I. (暁 / アカツキ)  
**Full:** Autonomous Knowledge Agent Team for Strategic Update & Knowledge Integration  
**直訳:** 戦略的更新と知識統合のための自律的ナレッジエージェントチーム

> **注意:** このドキュメントは、実装時の設計図（ブループリント）として機能するレベルで記述しています。

---

## 1. プロジェクト基本情報

### 1.1 目的
事業の立ち上げおよび運営業務の自律的遂行。

### 1.2 基本概念
階層型マルチエージェントシステムにより、以下の機能を自律的に実現する：
- 業務実行
- 品質管理
- 組織拡張
- ナレッジ蓄積

人間（User）は戦略的意思決定のみに注力する。

---

## 2. システムアーキテクチャ

### 2.1 技術スタック

| カテゴリ | 技術 | 用途 |
|---------|------|------|
| **Orchestration** | **LangGraph** (Python) | 状態遷移図に基づくエージェント制御、循環ループ、条件分岐の実装 |
| **Foundation Model** | **Claude 4.5 Sonnet** (ベース、via Anthropic API) / **任意のAIモデル** (各種API経由) | 推論、計画、コード生成、レビュー。各エージェントごとにAIモデルを選択可能 |
| **Database & Interface** | **Obsidian** (Local Markdown Files) | データベースとして機能し、ユーザーインターフェースでもある |
| **Data Structure** | **YAML Frontmatter** | エージェント属性、リレーション、ルール属性の管理 |
| **Tools** | **Browser-use** | Web操作 |
| **Tools** | **Zapier** | アプリ連携 |
| **Tools** | **Local File System** | ファイル操作 |

### 2.2 ディレクトリ構造 (Database Schema)

システムは以下のディレクトリ構造を厳格に維持し、各フォルダを特定のデータベース領域として扱う。

```
/AI_Manager_Root
  ├── 00_Strategy_Input/       # [User I/O] 戦略指示書格納（Trigger）
  ├── 10_Decision_Board/       # [User I/O] 承認・決裁・相談案件の提示
  ├── 20_Approved_Outputs/     # [Storage] 承認済み成果物のアーカイブ
  ├── 30_Work_Space/           # [Runtime] 作業用一時領域（ステート管理対象）
  │   ├── 01_Drafts/           # 作成中の成果物（エージェントIDでサブディレクトリ分離）
  │   └── 02_Review_Logs/      # 修正指示・レビュー記録（エージェントIDでサブディレクトリ分離）
  ├── 80_Team_Profiles/        # [DB: HR] エージェント定義ファイル（YAML+MD）
  ├── 90_Rules/                # [DB: Legal] 業務ルール・ガイドライン（階層管理）
  │   ├── 00_Global/           # 全社共通ルール（全エージェントに適用）
  │   ├── 10_Dept/             # 部門ルール（特定部門に適用）
  │   │   ├── [dept_id_1]/     # 部門ごとのルールディレクトリ
  │   │   └── [dept_id_2]/
  │   └── 20_Cross_Dept/       # 複数部門共通ルール
  ├── 95_Library/              # [DB: Knowledge] 共有ナレッジ・外部情報・学習履歴
  │   ├── 00_Global/           # 全社共有ナレッジ
  │   └── 10_Dept/             # 部門共有ナレッジ
  ├── 96_Agents/               # [DB: Agent-Specific] エージェント個別領域
  │   └── [agent_id]/          # エージェントごとのディレクトリ
  │       ├── Rules/           # 個人ルール（このエージェント専用）
  │       ├── Library/         # 個人ナレッジ（このエージェントが収集した知識）
  │       └── Work_Space/      # 個人作業領域（必要に応じて）
  └── 99_Logs/                 # [System] 実行ログ・コスト管理ログ
```

#### ディレクトリの役割詳細

| ディレクトリ | 用途 | アクセス権限 | 備考 |
|------------|------|------------|------|
| `00_Strategy_Input/` | 戦略指示書の格納 | User → System | システムのトリガーとして機能 |
| `10_Decision_Board/` | 承認・決裁・相談案件の提示 | System → User | Userの意思決定を待つ領域 |
| `20_Approved_Outputs/` | 承認済み成果物のアーカイブ | System | 最終的な成果物を永続保存 |
| `30_Work_Space/` | 作業用一時領域 | System (Runtime) | LangGraphのステート管理対象 |
| `30_Work_Space/01_Drafts/[agent_id]/` | 作成中の成果物 | System | エージェントごとに分離された作業中のドラフト |
| `30_Work_Space/02_Review_Logs/[agent_id]/` | 修正指示・レビュー記録 | System | エージェントごとに分離されたレビュー履歴 |
| `80_Team_Profiles/` | エージェント定義ファイル | System | YAML+MD形式でエージェント情報を管理 |
| `90_Rules/00_Global/` | 全社共通ルール | System | 全エージェントに適用されるルール |
| `90_Rules/10_Dept/[dept_id]/` | 部門ルール | System | 特定部門に適用されるルール |
| `90_Rules/20_Cross_Dept/` | 複数部門共通ルール | System | 複数の部門に共通して適用されるルール |
| `95_Library/00_Global/` | 全社共有ナレッジ | System | 全エージェントが参照可能な知識 |
| `95_Library/10_Dept/[dept_id]/` | 部門共有ナレッジ | System | 特定部門内で共有される知識 |
| `96_Agents/[agent_id]/Rules/` | 個人ルール | System | 特定エージェント専用のルール |
| `96_Agents/[agent_id]/Library/` | 個人ナレッジ | System | エージェントが個別に収集・蓄積した知識 |
| `96_Agents/[agent_id]/Work_Space/` | 個人作業領域 | System | エージェント専用の作業領域（必要に応じて） |
| `99_Logs/` | 実行ログ・コスト管理ログ | System | システムの動作記録とコスト追跡 |

---

## 3. 組織設計とエージェント仕様

### 3.1 階層構造と役割 (Hierarchy)

| 階層 | 役職ID | 役割定義 | 権限・責任 |
| :--- | :--- | :--- | :--- |
| **L0** | **User** | 会長・オーナー | 戦略決定、最終成果物の承認、予算承認、増員（定員超）の許可 |
| **L1** | **General Commander** | 総司令官 | 全体戦略のタスク分解、L2の生成・管理、L2間の調整、Userへの報告 |
| **L2** | **Dept. Manager** | 部門責任者 | 専門領域の指揮、**成果物の承認（Gatekeeper）**、L3の生成・管理、**担当領域ルールの策定・更新** |
| **L3** | **Worker** | 実務担当者 | 実務実行（調査・制作）、**上位方針への批判的意見具申（Bottom-up Critique）** |

### 3.2 エージェントデータ構造 (YAML Schema)

`80_Team_Profiles/*.md` のヘッダー仕様。LangGraphはこの情報を読み取り、ノードを動的に生成する。

#### YAML Frontmatter スキーマ

```yaml
---
id: "agent_marketing_lead"      # 一意のID（必須）
type: "Manager"                 # Manager | Worker（必須）
name: "Marketing Director"      # 表示名（必須）
supervisor_id: "agent_commander" # 直属の上司ID（必須）
# --- 組織管理パラメータ ---
current_headcount: 2            # 現在の部下数（必須）
max_subordinates: 3             # 部下数の上限（コスト管理用）（必須）
can_hire: true                  # 部下作成権限の有無（必須）
# --- スキル・ナレッジ ---
tools:                          # 使用可能ツール（必須）
  - "file_read"
  - "file_write"
  - "agent_creation"
  - "browser_use"
subscription_rules:             # 遵守すべきルールID（必須、階層的）
  - "00_Global/rule_constitution"      # 全社ルール
  - "10_Dept/marketing/rule_branding"  # 部門ルール（部門ID/ルール名）
  - "20_Cross_Dept/rule_collaboration"  # 複数部門共通ルール
  # 個人ルールは自動的に 96_Agents/[agent_id]/Rules/ から読み込まれる
read_access_library:            # 参照すべきナレッジカテゴリ（任意）
  - "00_Global/Trends"          # 全社共有ナレッジ
  - "10_Dept/marketing/Best_Practices" # 部門共有ナレッジ
  # 個人ナレッジは自動的に 96_Agents/[agent_id]/Library/ から読み込まれる
# --- AIモデル設定 ---
foundation_model:                # 使用するAIモデル（必須）
  provider: "anthropic"         # APIプロバイダー（anthropic, openai, google, etc.）
  model: "claude-4-5-sonnet"    # モデル名（例: claude-4-5-sonnet, gpt-4, gemini-pro）
  api_key_env: "ANTHROPIC_API_KEY"  # 環境変数名（.envファイルから読み込み）
# --- 自己進化パラメータ ---
learned_lessons:                # タスク完了ごとに追記される学習履歴（任意）
  - "2025-01-01: Xの投稿は朝7時が最適である"
---
```

#### フィールド詳細

| フィールド | 型 | 必須 | 説明 |
|----------|-----|------|------|
| `id` | string | ✓ | エージェントの一意識別子。システム内で重複不可 |
| `type` | enum | ✓ | `Manager` または `Worker`。階層構造を決定 |
| `name` | string | ✓ | エージェントの表示名 |
| `supervisor_id` | string | ✓ | 直属の上司エージェントのID。Userの場合は空文字列 |
| `current_headcount` | integer | ✓ | 現在の直属部下の数 |
| `max_subordinates` | integer | ✓ | 部下数の上限。コスト管理とリソース制限に使用 |
| `can_hire` | boolean | ✓ | 部下を生成する権限の有無 |
| `tools` | array[string] | ✓ | エージェントが使用可能なツールのリスト |
| `subscription_rules` | array[string] | ✓ | 遵守すべきルールファイルのIDリスト |
| `read_access_library` | array[string] | 任意 | 参照可能なナレッジライブラリのカテゴリ |
| `foundation_model` | object | ✓ | エージェントが使用するAIモデルの設定（provider, model, api_key_env） |
| `learned_lessons` | array[string] | 任意 | タスク完了時に追記される学習履歴 |

---

## 4. 業務プロセス仕様 (LangGraph Workflow)

システムは以下のステートマシンとして実装される。

### 4.1 計画と批判 (Planning & Critique Phase)

#### 4.1.1 プロセスフロー

1. **Instruction（指示）**
   - 上位階層（User or Commander）が指示を発行
   - 指示は `00_Strategy_Input/` にMarkdownファイルとして保存
   - システムは指示を解析し、担当Managerを特定

2. **Assignment（割り当て）**
   - 担当Managerへタスクを割り当て
   - タスク情報を `30_Work_Space/01_Drafts/[agent_id]/` に記録

3. **Critique（批判的検証 - Bottom-up）**
   - 指示を受けたエージェントは即座に実行せず、以下を実施：
     - 自身のナレッジ（`learned_lessons`、`96_Agents/[agent_id]/Library/`）を参照
     - 共有ライブラリ（`95_Library/00_Global/`、`95_Library/10_Dept/[dept_id]/`）を検索
     - 関連ルールを階層的に確認：
       - 全社ルール（`90_Rules/00_Global/`）
       - 部門ルール（`90_Rules/10_Dept/[dept_id]/`）
       - 複数部門共通ルール（`90_Rules/20_Cross_Dept/`）
       - 個人ルール（`96_Agents/[agent_id]/Rules/`）
   - **指示の妥当性を検証**
   - 矛盾やリスクがある場合：
     - 代替案を提示
     - 上司と合意形成を行う（`10_Decision_Board/` に相談チケットを発行）

#### 4.1.2 実装要件

- Critique機能は必須機能として実装
- エージェントは常に「指示の妥当性」を検証するプロンプトを持つ
- 検証結果は `30_Work_Space/02_Review_Logs/` に記録

### 4.2 実行と承認 (Execution & Review Phase)

#### 4.2.1 プロセスフロー

1. **Drafting（ドラフト作成）**
   - Workerがツールを使用してドラフトを作成
   - ドラフトは `30_Work_Space/01_Drafts/[agent_id]/` に保存
   - ドラフトにはメタデータ（作成者、作成日時、関連タスクID等）を付与

2. **Review（Manager審査）**
   - 直属のManagerが成果物を審査
   - **審査基準:**
     - 全社ルール（`90_Rules/00_Global/`）との整合性
     - 部門ルール（`90_Rules/10_Dept/[dept_id]/`）との整合性
     - 複数部門共通ルール（`90_Rules/20_Cross_Dept/`）との整合性
     - 個人ルール（`96_Agents/[agent_id]/Rules/`）との整合性
     - 指示内容との整合性
     - 品質基準（明確性、完全性、実現可能性）

3. **Review結果の処理**

   **NG（差し戻し）の場合:**
   - 具体的な修正理由を添えて差し戻す
   - 修正理由は `30_Work_Space/02_Review_Logs/[agent_id]/` に記録
   - **ルール振り分け（Rule Routing）:** Managerは指摘内容を分析し、以下のいずれかに分類してルール化する：
     - **全社ルール** (`90_Rules/00_Global/`): 全エージェントに適用すべき指摘
     - **部門ルール** (`90_Rules/10_Dept/[dept_id]/`): 特定部門に適用すべき指摘
     - **複数部門共通ルール** (`90_Rules/20_Cross_Dept/`): 複数部門に共通して適用すべき指摘
     - **個人ルール** (`96_Agents/[agent_id]/Rules/`): 特定エージェントのみに適用すべき指摘
   - ルール化により、同じ指摘を繰り返さない仕組みを実現
   - **Loop Limit:** 修正ループは**最大5回**まで
   - 5回超過時は自動的にエスカレーション（上位階層へ相談チケット発行）

   **OK（承認）の場合:**
   - 成果物を承認
   - 上位階層へ統合（または `20_Approved_Outputs/` へ移動）
   - 承認記録を `30_Work_Space/02_Review_Logs/[agent_id]/` に保存

#### 4.2.2 実装要件

- レビューループのカウンターをステートに保持
- 5回超過時のエスカレーション処理を実装
- レビュー結果は必ずログに記録

### 4.3 ナレッジ共有と自己進化 (Evolution Phase)

タスク完了ステータスへの遷移時、システムは強制的に以下の処理を実行する。

#### 4.3.1 プロセスフロー

1. **Self-Update（自己更新）**
   - エージェントは自身のYAML（`80_Team_Profiles/[agent_id].md`）の `learned_lessons` に今回の教訓を追記
   - 追記形式: `"YYYY-MM-DD: [教訓の内容]"`
   - 教訓は具体的で再利用可能な形式で記述

2. **Knowledge Sharing（ナレッジ共有）**
   - 汎用性が高い知見（成功事例、市場トレンド等）を判定
   - **ナレッジの振り分け:**
     - **全社共有ナレッジ** (`95_Library/00_Global/`): 全エージェントが参照すべき知識
     - **部門共有ナレッジ** (`95_Library/10_Dept/[dept_id]/`): 特定部門内で共有すべき知識
     - **個人ナレッジ** (`96_Agents/[agent_id]/Library/`): エージェントが個別に収集・蓄積した知識
   - 該当する知見は適切なディレクトリに新規Markdownファイルとして体系化・保存
   - ファイル名は `[カテゴリ]_[日付]_[タイトル].md` 形式
   - YAML Frontmatterでメタデータ（作成者、関連タスク、カテゴリ等）を付与

#### 4.3.2 実装要件

- タスク完了時に必ずEvolution Phaseを実行
- `learned_lessons` の更新は自動化
- ナレッジ共有の判定ロジックを実装（汎用性スコアリング等）

### 4.4 例外処理と人事 (Exception & HR Phase)

#### 4.4.1 エスカレーション（Escalation）

- **発生条件:**
  - 現場で解決できないルール矛盾が発生
  - レビュー修正ループが5回超過
  - 技術的制約により実現不可能と判断
- **処理:**
  - 上位階層へ「相談チケット」を発行
  - チケットは `10_Decision_Board/` に保存
  - チケットには以下を含む:
    - 問題の詳細
    - 試行した解決策
    - 必要な意思決定事項
    - 推奨される対応

#### 4.4.2 増員申請（Hiring Request）

- **発生条件:**
  - Managerがリソース不足と判断
  - かつ `current_headcount >= max_subordinates` の場合
- **処理:**
  - Userへ「増員申請」を行う
  - 申請は `10_Decision_Board/` に保存
  - 申請には以下を含む:
    - 必要なスキル・役割
    - 増員の理由（業務量、専門性の必要性等）
    - 予想されるコスト影響
- **承認後:**
  - Userの承認後、新しいエージェントを `80_Team_Profiles/` に作成
  - `current_headcount` を更新

#### 4.4.3 ルール更新（Rule Update）

- **発生条件:**
  - Managerがレビューを通じて頻出する指摘事項を発見
  - 既存ルールの不備や矛盾を発見
- **処理:**
  - Managerは指摘内容を分析し、適切なルール階層に振り分けて記録
  - **ルール振り分けの判断基準:**
    - 全社ルール: 複数の部門で発生する可能性がある指摘、組織全体の方針に関わる指摘
    - 部門ルール: 特定部門の業務特性に特化した指摘
    - 複数部門共通ルール: 2つ以上の部門に共通して適用すべき指摘
    - 個人ルール: 特定エージェントのスキルや特性に特化した指摘
  - ルールファイルを直接編集・更新
  - 更新履歴をルールファイル内に記録（YAML Frontmatterに `updated_at`, `updated_by`, `related_review_log` を記録）
  - 重大な変更の場合は `10_Decision_Board/` に通知
  - **重複防止:** 既存ルールと類似の指摘がないか検索し、重複を避ける

---

## 5. ガバナンス・非機能要件

### 5.1 セキュリティ (Security)

#### 5.1.1 機密情報の隔離

- **要件:**
  - APIキー、パスワード、ログイン情報は Obsidian 内（Markdown）には一切記述しない
  - 全てローカル環境の `.env` ファイルで管理
  - Pythonコード経由でのみアクセスする
- **実装:**
  - `.env` ファイルを `.gitignore` に追加
  - 環境変数の読み込みは `python-dotenv` を使用
  - Markdownファイル内に機密情報が含まれていないか、定期的に検証

#### 5.1.2 外部アクション制限

- **要件:**
  - SNS投稿、メール送信、Webフォーム送信などの対外的アクションは、原則として「下書き作成」または「テスト送信」までとする
  - 本番実行（Commit）には User の明示的な承認アクションを必須とする
- **実装:**
  - 外部アクション用のツールは「Dry-runモード」をデフォルトとする
  - 本番実行は `10_Decision_Board/` に承認リクエストを発行
  - Userが承認した場合のみ実行

### 5.2 コスト・リソース管理 (Resource Management)

#### 5.2.1 Token Optimization

- **要件:**
  - 各エージェントに渡すコンテキスト（過去ログ）は、関連性の高いものだけに制限
  - RAG的アプローチでトークン消費を抑制
- **実装:**
  - 過去ログの検索機能を実装（セマンティック検索）
  - 関連性スコアに基づいて上位N件のみをコンテキストに含める
  - トークン使用量を `99_Logs/cost_tracking.log` に記録

#### 5.2.2 Execution Limit

- **要件:**
  - 承認プロセスのリトライ回数上限（5回）をハードコードで設定
  - 無限ループによる課金を防止
- **実装:**
  - LangGraphのステートに `review_loop_count` を保持
  - 5回超過時は自動的にエスカレーション
  - ループ検出ロジックを実装

#### 5.2.3 コスト追跡

- **要件:**
  - API呼び出しのコストを追跡
  - 日次・月次のコストレポートを生成
- **実装:**
  - 各API呼び出し後にトークン数とコストを記録
  - `99_Logs/cost_tracking.log` にCSV形式で保存
  - 定期的にレポートを生成（`99_Logs/cost_report_[date].md`）

### 5.3 拡張性 (Scalability)

#### 5.3.1 エージェントの動的生成

- **要件:**
  - 各エージェントは共通のPythonクラス（`AgentNode`）のインスタンスとして生成
  - YAMLファイルの追加のみで組織をスケール可能とする
- **実装:**
  - `AgentNode` クラスを実装
  - `80_Team_Profiles/` 内のYAMLファイルをスキャンしてエージェントを動的に生成
  - LangGraphのノードを動的に構築

#### 5.3.2 ルールの階層管理

- **要件:**
  - 全社ルール、部門ルール、複数部門共通ルール、個人ルールを階層的に管理
  - 新しい部門やルールカテゴリを容易に追加可能
  - エージェントごとに個別のルール領域を提供
- **実装:**
  - `90_Rules/` 内で階層別にディレクトリを分割（00_Global, 10_Dept, 20_Cross_Dept）
  - `96_Agents/[agent_id]/Rules/` で個人ルールを管理
  - ルールの継承メカニズムを実装（全社ルール → 部門ルール → 個人ルール）
  - エージェントは自身が適用される全階層のルールを自動的に読み込む

#### 5.3.3 ルール振り分け機能（Rule Routing）

- **要件:**
  - レビュー時の指摘を適切なルール階層に自動振り分け
  - 同じ指摘の重複を防止
  - 指摘の適用範囲を明確化
- **実装:**
  - Managerがレビュー時に指摘を分類する機能を実装
  - 指摘内容を分析し、適用範囲を判定：
    - 全社適用: 複数部門で発生する可能性がある指摘
    - 部門適用: 特定部門の業務特性に特化した指摘
    - 複数部門共通: 2つ以上の部門に共通する指摘
    - 個人適用: 特定エージェントのスキルや特性に特化した指摘
  - 既存ルールとの類似度検索機能を実装（重複防止）
  - ルールファイルに `related_review_log` を記録し、指摘の出所を追跡可能にする

---

## 6. 実装チェックリスト

### 6.1 ディレクトリ構造
- [ ] `AI_Manager_Root` ディレクトリと全サブディレクトリを作成
- [ ] システム全体の共有ディレクトリ（00_Strategy_Input, 10_Decision_Board等）を作成
- [ ] ルール階層ディレクトリ（90_Rules/00_Global, 10_Dept, 20_Cross_Dept）を作成
- [ ] ナレッジ階層ディレクトリ（95_Library/00_Global, 10_Dept）を作成
- [ ] エージェント個別領域ディレクトリ（96_Agents/[agent_id]/）の動的生成機能を実装
- [ ] エージェントごとの作業領域分離（30_Work_Space/01_Drafts/[agent_id]/等）を実装

### 6.2 エージェントシステム
- [ ] `AgentNode` クラスを実装
- [ ] YAML Frontmatterパーサーを実装
- [ ] エージェントの動的生成機能を実装
- [ ] 階層構造（supervisor_id）の管理機能を実装
- [ ] Foundation Model選択機能を実装（各エージェントごとに異なるAIモデルを使用可能）
- [ ] 複数APIプロバイダー（Anthropic, OpenAI, Google等）の統合

### 6.3 LangGraph実装
- [ ] ステートマシンの定義
- [ ] Planning & Critique Phaseの実装
- [ ] Execution & Review Phaseの実装
- [ ] Evolution Phaseの実装
- [ ] Exception & HR Phaseの実装
- [ ] ループカウンターの実装（最大5回）

### 6.4 ツール統合
- [ ] Browser-use統合
- [ ] Zapier統合
- [ ] Local File System操作
- [ ] Obsidianファイル操作

### 6.5 セキュリティ
- [ ] `.env` ファイル管理
- [ ] 機密情報の検証機能
- [ ] 外部アクションのDry-runモード
- [ ] 承認フローの実装

### 6.6 コスト管理
- [ ] トークン使用量の追跡
- [ ] コストログの記録
- [ ] RAG的コンテキスト最適化
- [ ] コストレポート生成

### 6.7 ナレッジ管理
- [ ] `learned_lessons` の自動更新
- [ ] `95_Library/` へのナレッジ保存機能
- [ ] ナレッジ検索機能（セマンティック検索）

### 6.8 ルール管理
- [ ] ルールファイルの読み込み機能（階層的読み込み）
- [ ] ルールの階層管理（全社、部門、複数部門共通、個人）
- [ ] ルール更新機能
- [ ] ルール振り分け機能（Rule Routing）の実装
- [ ] 既存ルールとの類似度検索機能（重複防止）
- [ ] エージェントごとの個人ルール領域管理（96_Agents/[agent_id]/Rules/）
- [ ] ルールの適用範囲判定ロジック

### 6.9 ナレッジ管理（拡張）
- [ ] ナレッジの階層管理（全社、部門、個人）
- [ ] ナレッジ振り分け機能（全社/部門/個人の判定）
- [ ] エージェントごとの個人ナレッジ領域管理（96_Agents/[agent_id]/Library/）

---

## 7. 用語集

| 用語 | 説明 |
|------|------|
| **AgentNode** | エージェントを表現するPythonクラス。LangGraphのノードとして機能 |
| **Bottom-up Critique** | 下位階層のエージェントが上位の指示に対して批判的検証を行う機能 |
| **Gatekeeper** | Managerが成果物を承認する役割。品質管理の要 |
| **Dry-runモード** | 実際には実行せず、実行内容をシミュレーションするモード |
| **RAG** | Retrieval-Augmented Generation。関連情報を検索してコンテキストに含める手法 |
| **YAML Frontmatter** | Markdownファイルの先頭に記述されるYAML形式のメタデータ |

---

## 8. Foundation Model設定仕様

### 8.1 対応プロバイダー

システムは以下のAI APIプロバイダーに対応する：

| プロバイダー | 環境変数名 | 対応モデル例 |
|------------|----------|------------|
| **Anthropic** | `ANTHROPIC_API_KEY` | claude-4-5-sonnet, claude-3-5-sonnet, claude-3-7-sonnet |
| **OpenAI** | `OPENAI_API_KEY` | gpt-4, gpt-4-turbo, gpt-3.5-turbo |
| **Google** | `GOOGLE_API_KEY` | gemini-pro, gemini-ultra |
| **その他** | カスタム | カスタムAPIエンドポイントに対応 |

### 8.2 モデル選択の原則

- **デフォルト:** Claude 4.5 Sonnet（ベースモデル）
- **個別設定:** 各エージェントのYAMLで `foundation_model` を指定
- **コスト最適化:** タスクの重要度や複雑度に応じてモデルを選択可能
- **フォールバック:** 指定モデルが利用不可の場合、デフォルトモデルに自動切り替え

### 8.3 実装要件

- 各エージェントは独立したAIモデルインスタンスを持つ
- APIキーは `.env` ファイルから環境変数経由で読み込み
- モデル切り替えはランタイムで動的に実行可能
- コスト追跡はモデルごとに個別に記録

---

## 9. 参考資料

- LangGraph公式ドキュメント: https://langchain-ai.github.io/langgraph/
- Anthropic API ドキュメント: https://docs.anthropic.com/
- OpenAI API ドキュメント: https://platform.openai.com/docs/
- Google AI API ドキュメント: https://ai.google.dev/docs
- Obsidian公式サイト: https://obsidian.md/
- YAML Frontmatter仕様: https://jekyllrb.com/docs/front-matter/

---

**ドキュメントバージョン:** 1.0  
**最終更新日:** 2025-01-XX  
**作成者:** A.K.A.T.S.U.K.I. プロジェクトチーム

