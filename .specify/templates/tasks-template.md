---

description: "機能実装用のタスクリストテンプレート"
---

# Tasks: [FEATURE NAME]

**Input**: `/specs/[###-feature-name]/` 配下の設計文書
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: このテンプレートではテストタスクを明示する。憲法上必須の品質ゲートは省略してはならない。

**Organization**: タスクはユーザーストーリー単位で整理し、各ストーリーを独立して実装・検証できるようにする。

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 並列実行可能なタスク
- **[Story]**: 対応するユーザーストーリー (US1, US2, US3 など)
- 説明には対象ファイルパスを含める

## Phase 1: Setup (Shared Infrastructure)

**目的**: プロジェクト初期化と共通設定

- [ ] T001 実装計画に従ってプロジェクト構成を作成する
- [ ] T002 実行基盤、依存関係、CI 設定を初期化する
- [ ] T003 [P] lint、format、静的解析、共通テスト実行設定を追加する

---

## Phase 2: Foundational (Blocking Prerequisites)

**目的**: すべてのストーリーに先行する基盤要件を整える

- [ ] T004 契約スキーマ、バージョニング方針、互換性ルールを整備する
- [ ] T005 [P] requestId、tenantId、trace 識別子の伝播基盤を実装する
- [ ] T006 [P] 認証、認可、policy evaluation、監査記録の共通基盤を整備する
- [ ] T007 状態の正源、永続化モデル、派生データ同期の基盤を定義する
- [ ] T008 失敗分類、リトライ、タイムアウト、DLQ、補償の共通制御を整備する
- [ ] T009 可観測性と運用 readiness の共通部品を整備する
- [ ] T010 [P] Temporal DSL の定義、アクティビティ登録、ランタイムルーティングの共通基盤を整備する

**Checkpoint**: Foundation 完了後にユーザーストーリーへ進む

---

## Phase 3: User Story 1 - [Title] (Priority: P1)

**Goal**: [このストーリーが提供する価値]

**Independent Test**: [単独での検証方法]

### Tests for User Story 1

- [ ] T011 [P] [US1] unit test を tests/unit/ に追加する
- [ ] T012 [P] [US1] contract test を tests/contract/ に追加する
- [ ] T013 [P] [US1] replay test または決定性検証を tests/integration/ に追加する
- [ ] T014 [P] [US1] idempotency test と failure-path test を tests/integration/ に追加する
- [ ] T015 [P] [US1] compensation test を必要に応じて tests/integration/ に追加する

### Implementation for User Story 1

- [ ] T016 [P] [US1] ドメインモデルと状態遷移を src/models/ に実装する
- [ ] T017 [US1] ワークフローまたはアプリケーションサービスを src/services/ に実装する
- [ ] T018 [US1] 契約準拠の入出力ハンドラを src/ または api/ に実装する
- [ ] T019 [US1] 可観測性、監査、メトリクスを対象処理へ組み込む
- [ ] T020 [US1] 運用手順と quickstart の検証手順を更新する

---

## Phase 4: User Story 2 - [Title] (Priority: P2)

**Goal**: [このストーリーが提供する価値]

**Independent Test**: [単独での検証方法]

### Tests for User Story 2

- [ ] T021 [P] [US2] unit test を追加する
- [ ] T022 [P] [US2] contract test を追加する
- [ ] T023 [P] [US2] replay test または決定性検証を追加する
- [ ] T024 [P] [US2] idempotency test、failure-path test、必要な compensation test を追加する

### Implementation for User Story 2

- [ ] T025 [P] [US2] ドメインモデルまたは契約差分を実装する
- [ ] T026 [US2] ワークフローまたはサービスを実装する
- [ ] T027 [US2] 認可、テナント境界、監査を組み込む
- [ ] T028 [US2] 観測項目、運用手順、移行手順を更新する

---

## Phase 5: User Story 3 - [Title] (Priority: P3)

**Goal**: [このストーリーが提供する価値]

**Independent Test**: [単独での検証方法]

### Tests for User Story 3

- [ ] T029 [P] [US3] 必須テスト群を追加する
- [ ] T030 [P] [US3] 契約または移行互換性の検証を追加する

### Implementation for User Story 3

- [ ] T031 [P] [US3] モデルとサービスを実装する
- [ ] T032 [US3] 補償、DLQ、運用回復経路を実装する
- [ ] T033 [US3] 可観測性、監査、運用文書を更新する

---

## Phase N: Polish & Cross-Cutting Concerns

**目的**: 全体品質と運用準備を完了させる

- [ ] TXXX [P] ドキュメントと移行計画を更新する
- [ ] TXXX 互換性、ロールバック、設定 fail fast を再確認する
- [ ] TXXX [P] 監視、アラート、監査イベントを最終確認する
- [ ] TXXX quickstart.md の手順を通しで検証する

---

## Dependencies & Execution Order

### Phase Dependencies

- Setup は即時開始できる
- Foundational は Setup 完了後に実施し、全ユーザーストーリーをブロックする
- User Story フェーズは Foundational 完了後に開始する
- Polish は対象とする全ユーザーストーリー完了後に実施する

### Constitution-Driven Checks

- 各ストーリーで contract、replay、idempotency、failure-path、compensation、observability を確認する
- テナント境界、認可、監査、運用 readiness を最後まで残課題にしてはならない
- 互換性変更がある場合は migration test と downgrade consideration を必ず含める
- Temporal DSL を採用する場合は、定義のバージョン、アクティビティ登録、ワーカー再登録、ランタイム反映の検証を含める

### Parallel Opportunities

- [P] が付いたテストタスクは同一ファイルを競合しない限り並列実行できる
- [P] が付いたモデル、契約、観測項目の追加は並列実行できる
- 異なるユーザーストーリーは基盤完了後に並列進行できる

---

## Implementation Strategy

### MVP First

1. Setup を完了する
2. Foundational を完了する
3. User Story 1 を実装し、必須品質ゲートを満たす
4. 単独で検証してから次のストーリーへ進む

### Incremental Delivery

1. 基盤整備後、優先度順にユーザーストーリーを追加する
2. 各ストーリーで契約、失敗回復、可観測性、運用手順を同時に完成させる
3. 各ストーリー完了時に独立してデモ可能な状態を作る

---

## Notes

- 曖昧なタスク名は禁止する
- 同じファイルを更新するタスクは順序を明示する
- 実装タスクから契約、観測、運用、テストを切り離しすぎてはならない
