<!--
Sync Impact Report
- Version change: template -> 1.0.0
- Modified principles: placeholders -> I. Workflow Determinism; placeholders -> II. Idempotency by Default; placeholders -> III. Saga over Distributed Transaction; placeholders -> IV. Single Source of Truth; placeholders -> V. Contract-First Evolution; added VI. Tenant Isolation and Security; added VII. Policy Enforcement by Default Deny; added VIII. Explicit Failure Semantics; added IX. Observability Is a Feature; added X. Operational Readiness and Fail Fast; added XI. Quality Gates; added XII. Extensibility with Reviewable Definitions
- Added sections: Platform Scope; Feature Specification Requirements
- Removed sections: none
- Templates requiring updates: ✅ updated .specify/templates/constitution-template.md; ✅ updated .specify/templates/checklist-template.md; ✅ updated .specify/templates/spec-template.md; ✅ updated .specify/templates/plan-template.md; ✅ updated .specify/templates/tasks-template.md
- Follow-up TODOs: none
-->

# Orchestration Platform Constitution

## Core Principles

### I. Workflow Determinism

すべてのワークフローは決定的でなければならない。ワークフロー本体で外部
I/O を行ってはならず、ネットワーク、DB、時刻、乱数、環境依存値の取得は
Activity または明示された connector 層経由でのみ行う。ワークフローは制御、
分岐、状態遷移、補償判断に専念し、実装は replay 可能でなければならない。
これにより回復、再実行、監査時の挙動が再現可能になる。

### II. Idempotency by Default

すべての要求と副作用は冪等でなければならない。すべての要求はグローバルに
一意な requestId を持ち、再送、重複投入、回復時の識別キーとして扱う。
ワークフロー起動、永続化、下流 API 呼び出し、補償は requestId ベースで
重複安全でなければならず、同じ requestId の再投入で副作用が重複しては
ならない。これは部分失敗や再配信を前提とした基盤の安全性を担保するための
必須条件である。

### III. Saga over Distributed Transaction

クロスサービス整合性は Saga と補償で担保し、分散トランザクションに依存しては
ならない。サービス境界をまたぐ DB トランザクションは禁止し、各サービスは
自サービス所有データのみを直接更新できる。複数サービスにまたがる処理は
workflow と compensation で表現し、副作用を伴う step には補償戦略または
補償不要である根拠を明記しなければならない。これによりサービス境界を保った
まま失敗回復を可能にする。

### IV. Single Source of Truth

状態の正源は一箇所に限定しなければならない。プラットフォーム処理状態の正源は
オーケストレーション基盤が所有する状態ストアとし、外部表示用状態や同期先の
状態は派生データとして扱う。feature spec は正源、派生先、同期方式、整合性
レベルを明記しなければならず、正源が曖昧な設計は禁止する。これにより回復と
監査の判断基準を一意に保つ。

### V. Contract-First Evolution

イベント、API、オーケストレーション定義は contract-first で管理しなければ
ならない。すべての外部契約は明示的なスキーマ、バージョン、互換性方針を持つ。
後方互換を壊す変更は契約改訂として扱い、移行計画を伴わなければならない。
暗黙のフィールド追加や解釈変更は禁止する。これは複数 SaaS と下流サービスの
連携を段階的に進化させるための前提である。

### VI. Tenant Isolation and Security

テナント分離は全レイヤで強制しなければならない。すべての要求は tenantId を
持ち、tenantId はイベント、ワークフロー、永続化、ログ、メトリクス、トレース、
補償に伝播しなければならない。他テナントのデータ参照、状態更新、補償実行が
発生しないことを設計で保証し、認証、認可、監査可能性を伴わない管理操作は
禁止する。これは基盤の信頼性とセキュリティ境界を維持するためである。

### VII. Policy Enforcement by Default Deny

認可は policy-based かつ default deny を原則としなければならない。実行可否の
判断は外部化可能なポリシーとして定義し、不許可は明示的に説明可能でなければ
ならない。業務コードへ認可条件を散在させてはならず、feature spec は認可入力、
評価タイミング、deny 時の挙動、監査記録内容を定義しなければならない。これに
より認可の変更可能性と監査性を両立する。

### VIII. Explicit Failure Semantics

失敗は分類され、回復方針が明示されなければならない。すべての失敗は少なくとも
retryable、non-retryable、compensatable、requires-manual-intervention、
dead-letter に分類する。リトライ回数、バックオフ、タイムアウト、打ち切り条件、
補償失敗時の扱い、運用介入条件を spec に明記しなければならない。不正メッセージ
や契約違反メッセージを通常リトライへ流してはならず、隔離経路へ送る。これは
障害時の挙動を運用可能な形で固定するためである。

### IX. Observability Is a Feature

可観測性は必須機能であり、実装後付けにしてはならない。すべての要求で
requestId、tenantId、workflowId、orchestrationId を追跡可能にし、すべての
feature は最低限のログ、メトリクス、トレース、監査イベントを持たなければ
ならない。成功率、失敗率、リトライ率、DLQ 件数、補償実行率、遅延を計測可能に
し、監査要件を満たさない認可、状態遷移、管理操作は禁止する。運用で見えない
機能は完成とはみなさない。

### X. Operational Readiness and Fail Fast

運用準備が完了していない機能は完成扱いにしてはならない。設定値は起動時に
検証し、危険な設定や不正値は fail fast で停止する。liveness、readiness、
shutdown 手順を持たない runtime component は採用せず、DLQ 再処理、キャンセル、
再開、ポリシー更新、障害時切り分けの手順を用意しなければならない。手動運用は
最後の手段であり、恒常運用の前提にしてはならない。

### XI. Quality Gates

品質ゲートを満たさない変更は採用してはならない。すべての feature は unit test
を持ち、契約を持つ feature は contract test、workflow を持つ feature は replay
test または同等の決定性検証を持たなければならない。副作用を持つ feature は
idempotency test と failure-path test、補償を持つ feature は compensation test
を持たなければならない。互換性に影響する変更は migration test と downgrade
consideration を伴わなければならない。

### XII. Extensibility with Reviewable Definitions

拡張は定義駆動を優先し、レビュー可能でなければならない。新しい
オーケストレーションは可能な限り DSL または定義ファイルで追加できる形を
優先し、step は service、input、timeout、retry、compensation、expected outputs
を明示しなければならない。実装にしか存在しない暗黙ルールは禁止し、定義の変更
だけで振る舞いが変わる場合、その差分はレビュー可能でなければならない。

## Platform Scope

### Purpose

このプラットフォームは、複数 SaaS からの要求を安全に受け取り、認可し、状態を
保持し、複数サービスを横断して実行し、失敗時にも一貫した形で回復可能にする
オーケストレーション基盤である。本憲法は、すべての feature spec、設計、実装、
運用判断に優先する。

本プラットフォームに関する feature spec、設計文書、運用文書、レビュー記録は、
原則として日本語を用いる。Spec Kit のテンプレートも日本語で維持しなければ
ならない。

### Scope

本憲法の対象は、イベント受信、ワークフロー実行、ポリシー評価、状態永続化、
補償処理、下流サービス連携、可観測性、運用設計、テスト戦略とする。

### Non-Goals

本プラットフォームは、各業務サービス固有のビジネスルールの実装、サービス境界を
またぐ共有 DB トランザクション、下流サービス内部の状態管理の代行、暗黙的な
人手運用を前提にした整合性回復を責務としない。

## Feature Specification Requirements

すべての feature spec は最低限、目的と非目的、関与する契約、状態モデルと正源、
冪等性戦略、認可とテナント境界、失敗分類と回復戦略、補償の要否、可観測性要件、
テスト計画、運用影響と移行計画を含まなければならない。これらが欠ける spec は
計画や実装へ進めてはならない。

## Governance

本憲法は本プラットフォームの他の開発慣行より優先される。すべてのレビュー、
計画、実装判断は本憲法への適合を確認しなければならない。憲法変更は通常の
feature 変更より重く扱い、既存原則を破る提案は例外が必要な理由を明示し、
将来の運用負債と移行計画を伴って審査しなければならない。憲法変更時は、影響を
受ける feature spec、テンプレート、runtime rule を同時更新する。

本憲法のバージョンは semantic versioning に従う。原則の削除や後方非互換な
再定義は MAJOR、新しい原則や必須セクションの追加は MINOR、意味を変えない
明確化や文言修正は PATCH とする。準拠確認は pull request と設計レビューで
実施し、Constitution Check を通過できない変更は採用してはならない。

**Version**: 1.0.0 | **Ratified**: 2026-05-11 | **Last Amended**: 2026-05-11
