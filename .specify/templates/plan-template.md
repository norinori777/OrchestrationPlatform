# 実装計画: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]
**入力**: `/specs/[###-feature-name]/spec.md` の feature specification

**注記**: このテンプレートは `/speckit.plan` コマンドが埋める。詳細な実行手順は `.specify/templates/plan-template.md` を参照する。

## Summary

[feature spec から抽出した要約と、採用する技術方針]

## Technical Context

**Language/Version**: [例: Java 21 / NEEDS CLARIFICATION]  
**Primary Dependencies**: [例: Temporal, Spring Boot / NEEDS CLARIFICATION]  
**Storage**: [例: PostgreSQL, Redis, N/A]  
**Testing**: [例: JUnit, pytest / NEEDS CLARIFICATION]  
**Target Platform**: [例: Linux, Kubernetes / NEEDS CLARIFICATION]  
**Project Type**: [例: web-service, worker, library / NEEDS CLARIFICATION]  
**Performance Goals**: [測定目標]  
**Constraints**: [運用、セキュリティ、互換性制約]  
**Scale/Scope**: [対象テナント数、トラフィック規模など]

## Constitution Check

*GATE: Phase 0 の調査開始前に必ず通過し、Phase 1 の設計完了後に再確認する。*

- Workflow Determinism: ワークフロー本体から外部 I/O を排除し、replay 可能性を維持しているか
- Idempotency by Default: requestId の採番元、重複検知、再送時挙動が設計されているか
- Saga over Distributed Transaction: サービス境界をまたぐ分散トランザクションを導入していないか
- Single Source of Truth: 正源、派生先、同期方式、整合性レベルが明記されているか
- Contract-First Evolution: 契約、スキーマ、バージョン、移行方針が定義されているか
- Tenant Isolation and Security: tenantId 伝播、認証、認可、監査境界が設計されているか
- Explicit Failure Semantics: 失敗分類、リトライ、補償、DLQ、手動介入条件が明記されているか
- Observability Is a Feature: 必須ログ、メトリクス、トレース、監査イベントが定義されているか
- Operational Readiness and Fail Fast: 設定検証、ヘルスチェック、停止手順、運用手順が準備されているか
- Quality Gates: unit、contract、replay、idempotency、failure-path、compensation、migration の各テスト方針が揃っているか

## Research and Design Outputs

### Phase 0: 調査

- [未確定技術要素の調査項目]
- [契約、互換性、回復戦略の比較検討]

### Phase 1: 設計成果物

- research.md: [採用判断と根拠]
- data-model.md: [状態モデルと正源]
- contracts/: [イベント、API、メッセージ契約]
- quickstart.md: [主要シナリオの検証手順]

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
└── tasks.md
```

### Source Code (repository root)

```text
[実際のリポジトリ構成をここに記載]
```

**Structure Decision**: [採用した構成と理由]

## Delivery and Migration View

- デプロイ単位: [記述]
- 互換性維持方針: [記述]
- 移行順序: [記述]
- ロールバック条件: [記述]

## Complexity Tracking

> **Constitution Check に違反があり、例外として審査する場合のみ記載する**

| 逸脱項目 | 必要性 | より単純な代替を採用しない理由 |
|----------|--------|--------------------------------|
| [例] | [理由] | [理由] |
