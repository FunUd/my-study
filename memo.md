Implementation Workflow Skill 設計書

1. 目的

ユーザーから実装要求を受けた際、要求を明確化し、コードベースへの影響とリスクを分析したうえで、リスクに応じた開発フローを選択し、必要な承認を経て実装・レビュー・検証まで一貫して実施する。

軽微な変更に過剰な工程を要求せず、重要な変更には必要な品質保証を適用する。

2. 基本方針

- AIがユーザー要求を勝手に補完せず、実装結果に影響する曖昧点を先に解消する。
- 変更行数ではなく、挙動・状態・データ・I/F・製品への影響、故障時の影響度、検証難度等を基準にリスクを判定する。
- リスクに応じて実施する工程、承認、検証レベルを変更する。
- 各工程は「必須」「条件付き」「不要」を判断する。
- 承認が必要な工程では、承認前に次工程へ進まない。
- 既存の仕様・設計・プロジェクト規約・テスト方針を優先する。
- 不明な事項を推測で確定しない。
- 詳細な実行フローは "references/" の該当文書を参照する。

3. 基本フロー

Clarify
  ↓
Analyze
  ↓
Risk Assessment
  ↓
Plan
  ↓
User Approval（必要時）
  ↓
Test Design（必要時）
  ↓
User Approval（必要時）
  ↓
Implement / TDD（必要時）
  ↓
Self Code Review（必要時）
  ↓
Verify（必要時）
  ↓
Finalize

Verify Failure
  ↓
Failure Analysis
  ↓
適切な工程へ戻る

Risk Assessment後は、該当する "workflow-rX.md" のみを参照して具体的なフローを実行する。

4. 工程の責務

4.1 Clarify

ユーザー要求の曖昧点・不足情報・複数の解釈が存在する事項を特定し、必要な場合のみユーザーへ質問する。

詳細は "references/clarify-guidelines.md" を参照する。

4.2 Analyze

要求が明確になった後、コードベースを調査する。

- 変更対象
- 関連コード・呼び出し関係
- 関連モジュール・データ
- 影響範囲
- 既存テスト
- 製品・構成への影響
- 既存仕様・制約
- 変更カテゴリ
- リスク

4.3 Risk Assessment

Analyze結果に基づき、変更をR0〜R3へ分類する。

詳細な判定基準は "references/risk-assessment.md" を参照する。

4.4 Plan

AnalyzeおよびRisk Assessmentの結果から、以下を決定する。

- 実装方針
- 変更範囲
- 影響範囲
- リスク
- 実施する工程
- スキップする工程と理由
- 必要な承認

必要な場合、ユーザーに提示して承認を得る。

4.5 Test Design

挙動が変わる変更について、実装前にテストケースを設計する。

必要に応じて以下を考慮する。

- 正常系
- 異常系
- 境界値
- 状態遷移
- 既存機能への影響

テストが不要な変更ではスキップする。

4.6 Implement

承認されたPlanおよびTest Designに従って実装する。

自動テストで検証可能な新規・変更ロジックには、必要に応じてTDDを適用する。

Red → Green → Refactor

4.7 Self Code Review

実装後のdiffを確認する。

主な確認対象：

- 要求との整合性
- Planとの整合性
- 設計・実装の妥当性
- 不要な変更の有無
- エッジケース
- テスト内容
- コーディング規約
- コメント

レビューで問題が見つかった場合は修正し、再度Self Code Reviewを行う。

4.8 Verify

実装結果を客観的・機械的に検証する。

必要に応じて以下を実施する。

- Build
- Unit Test
- Coverage
- Static Analysis
- その他必要な検証
- Scope Check

Verifyは要求・設計そのものの妥当性を判断する工程ではない。

4.9 Failure Analysis

Verifyに失敗した場合、原因を特定し、適切な工程へ戻る。

実装・テスト・静的解析の問題
  → Implement

テストケース不足
  → Test Design

要求・設計上の問題が判明
  → Clarify / Analyze / Plan

修正後はSelf Code Reviewを再実施し、Verifyを再実施する。

要求・設計への遡及は、前工程の前提誤りが判明した場合の例外的なフローとする。

4.10 Finalize

以下を簡潔に報告する。

- 実施内容
- 変更ファイル
- 実施したテスト・検証
- Coverage結果（実施した場合）
- Review結果
- 残課題・注意事項

5. リスクレベル

R0: 非機能・機械的変更

例：

- コメント変更
- 明らかな変数名変更
- Formatting
- typo修正

R1: 低リスク変更

例：

- 限定的なリファクタリング
- 挙動を変えない内部構造変更

R2: 通常の機能変更

例：

- 既存ロジック変更
- 新規機能追加
- 条件分岐変更
- 状態遷移変更

R3: 高リスク変更

例：

- NVM・永続データ
- 通信プロトコル
- 状態管理
- 並行処理
- 基盤機能
- 複数モジュール・製品系列に大きく影響する変更

具体的な判定基準は "references/risk-assessment.md"、各レベルの実行フローは対応する "references/workflow-rX.md" を参照する。

6. ユーザー承認

承認ゲートはリスクに応じて設定する。

- R0/R1：原則として承認不要
- R2：Plan承認を基本とする
- R3：Planに加えてTest Design等を必要に応じて承認対象とする

承認対象の工程は承認前に実施しない。

7. 他Skillへの委譲

各工程は、対応する専用Skillが存在する場合、そのSkillへ委譲する。

委譲先および呼び出し条件は "references/skill-delegation.md" を参照する。

専用Skillが存在しない場合：

1. Workflow Skill自身で実施可能なら実施する。
2. 実施できない場合はユーザーに通知する。

委譲先Skillの結果を受け取り、その結果に応じてWorkflowを継続・分岐する。

例：

Implementation Workflow
        ↓
Self Code Review Skill
        ↓
Review Result
   ┌────┴────┐
問題あり    問題なし
   ↓           ↓
Implement    Verify

8. References構成

implementation-workflow/
├─ SKILL.md
└─ references/
   ├─ risk-assessment.md
   ├─ clarify-guidelines.md
   ├─ workflow-r0.md
   ├─ workflow-r1.md
   ├─ workflow-r2.md
   ├─ workflow-r3.md
   └─ skill-delegation.md

risk-assessment.md

- R0〜R3の詳細な判定基準
- 判定時の考慮事項
- 境界ケース
- 判定例

Risk Assessment時に参照する。

clarify-guidelines.md

- 質問すべき条件
- 質問すべきでない条件
- 既存コード・仕様から判断してよい事項
- ユーザー判断を必要とする事項
- 質問方法

Clarify時に参照する。

workflow-r0.md ～ workflow-r3.md

各リスクレベル専用の実行フローを定義する。

- 工程順序
- 適用する工程
- スキップする工程
- 承認ゲート
- TDD適用条件
- Coverage適用条件
- Review適用条件
- Verify Failure時の戻り先
- 具体例

Risk Assessment後、該当するファイルのみを参照する。

skill-delegation.md

- 工程と専用Skillの対応関係
- Skill呼び出し条件
- 委譲時に渡すべき情報
- Skill実行結果の扱い
- Skillが存在しない場合のFallback

9. Skillの責務

本Skillは、各作業の詳細をすべて内包するのではなく、以下を担当する。

- 要求の明確化
- 影響分析
- リスク判定
- 適用フローの選択
- 承認管理
- 他Skillへの委譲
- 工程間のオーケストレーション
- 完了条件の管理

詳細な実装・テスト・レビュー方法は、必要に応じて専用Skillへ委譲する。

本Skillは固定手順を常に実行するのではなく、変更内容とリスクに応じて必要な工程だけを選択するオーケストレーターとして設計する。