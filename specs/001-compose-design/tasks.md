# Tasks: compose.yaml 構成案

**Input**: `/specs/001-compose-design/` 配下の設計文書
**Prerequisites**: plan.md, spec.md

**Tests**: このタスク一覧では、compose 設計の妥当性確認として `docker compose config`、healthcheck/port/depends_on の整合確認、profile/共有設定の確認を含める。

**Organization**: タスクはユーザーストーリー単位で整理し、各ストーリーを独立して実装・検証できるようにする。

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 並列実行可能なタスク
- **[Story]**: 対応するユーザーストーリー (US1, US2, US3 など)
- 説明には対象ファイルパスを含める

## Phase 1: Setup (Shared Infrastructure)

**目的**: compose 設計の土台と、成果物の置き場所を先に作る

- [X] T001 `compose.yaml` の構成骨子と共通 anchor 方針を定義する
- [X] T002 `compose-design.md` に service 設計表の見出しと列定義を作成する
- [X] T003 [P] `.env.example` に共通環境変数の置き場と命名規則を作成する

---

## Phase 2: Foundational (Blocking Prerequisites)

**目的**: すべてのユーザーストーリーに先行する基盤要件を整える

- [X] T004 [P] `compose.yaml` に postgres、temporal、temporal-ui、nats、opa、redis の基盤 service を定義する
- [X] T005 [P] `compose.yaml` に prometheus、vector、jaeger の観測系 service を定義する
- [X] T006 `compose.yaml` に opa-policy-loader と各 Prisma migration job を初期化 service として定義する
- [X] T007 [P] `compose.yaml` に network、volume、共有 anchor、restart policy の共通設定を定義する
- [X] T008 `compose-design.md` に基盤必須・観測系・初期化必須の区分表を作成する

**Checkpoint**: 基盤と共通設定が揃ったら、各ユーザーストーリーの service 群へ進む

---

## Phase 3: User Story 1 - 1 つの compose で全体を起動したい (Priority: P1)

**Goal**: 開発者が 1 つの `compose.yaml` で基盤、プラットフォーム本体、SaaS 層、下流サービス層を起動できるようにする

**Independent Test**: `docker compose config` で、全体の service 群と起動関係が 1 つの定義として解釈できることを確認する

### Tests for User Story 1

- [X] T009 [P] [US1] `docker compose config` を想定した構成整合チェック手順を `quickstart.md` に追加する
- [X] T010 [P] [US1] 全体 service グラフの smoke check 手順を `quickstart.md` に追加する

### Implementation for User Story 1

- [X] T011 [P] [US1] `compose.yaml` に orchestration-platform service を定義する
- [X] T012 [P] [US1] `compose.yaml` に saas-backend と saas-frontend service を定義する
- [X] T013 [P] [US1] `compose.yaml` に user-service、file-storage-service、routing-file-service、mail-service-mock service を定義する
- [X] T014 [US1] `compose.yaml` で全体の depends_on を接続して起動順を確定する
- [X] T015 [US1] `compose-design.md` に全体 service 一覧と起動対象/除外対象を整理する

---

## Phase 4: User Story 2 - 各 service の依存と稼働条件を把握したい (Priority: P2)

**Goal**: 各 service の depends_on、healthcheck、環境変数、公開ポートを一覧で確認できるようにする

**Independent Test**: `compose-design.md` と `compose.yaml` を見れば、各 service の依存先と公開インターフェースを追跡できることを確認する

### Tests for User Story 2

- [X] T016 [P] [US2] すべての service に healthcheck があるかを `compose-design.md` で確認する手順を追加する
- [X] T017 [P] [US2] すべての service に公開ポートと環境変数があるかを `compose-design.md` で確認する手順を追加する

### Implementation for User Story 2

- [X] T018 [P] [US2] `compose.yaml` に各 service の healthcheck を追加する
- [X] T019 [P] [US2] `compose.yaml` に各 service の環境変数と公開ポートを追加する
- [X] T020 [US2] `compose-design.md` に service ごとの depends_on、healthcheck、env、port の設計表を作成する
- [X] T021 [US2] `compose-design.md` に migration job と policy loader の一時実行 service 区分を明記する

---

## Phase 5: User Story 3 - 共通設定と再利用単位を揃えたい (Priority: P3)

**Goal**: compose 全体で共有する network、volume、profile、restart policy を明記し、起動分離と永続化方針を揃える

**Independent Test**: 共有設定だけで、どの service がどの network、volume、profile を使うかが把握できることを確認する

### Tests for User Story 3

- [X] T022 [P] [US3] `compose.yaml` に network、volume、profile、restart policy が揃っているかを確認する手順を追加する
- [X] T023 [P] [US3] observability 系 service と補助系 service の profile 分離を確認する手順を追加する

### Implementation for User Story 3

- [X] T024 [P] [US3] `compose.yaml` に shared network、volume、profile、restart policy の共通定義を抽出する
- [X] T025 [US3] `compose.yaml` に observability 系 service の profile と公開方針を適用する
- [X] T026 [US3] `compose.yaml` に migration job と mail-service-mock の起動方針を適用する
- [X] T027 [US3] `README.md` と `quickstart.md` に compose 起動方法と共通設定の扱いを追記する

---

## Phase 6: Polish & Cross-Cutting Concerns

**目的**: 全体の整合性、運用前提、文書品質を最終確認する

- [X] T028 [P] `compose.yaml` と `compose-design.md` の整合を最終レビューする
- [X] T029 `quickstart.md` に `docker compose up` と `docker compose config` の手順をまとめる
- [X] T030 [P] `README.md` に compose 構成案の位置付けと利用前提を追記する
- [X] T031 `compose.yaml`、`compose-design.md`、`quickstart.md` の命名と表記を統一する

---

## Dependencies & Execution Order

### Phase Dependencies

- Setup は即時開始できる
- Foundational は Setup 完了後に実施し、全ユーザーストーリーをブロックする
- User Story フェーズは Foundational 完了後に開始する
- Polish は対象とする全ユーザーストーリー完了後に実施する

### Constitution-Driven Checks

- 各ストーリーで `compose.yaml` の `depends_on`、`healthcheck`、環境変数、公開ポートの整合を確認する
- テナント境界、認可、監査、ランタイム反映境界は compose 設計上の前提として文書に残す
- 互換性変更がある場合は migration test と downgrade consideration を必ず含める
- Temporal DSL は compose 対象外であることを spec と設計表の両方で明記する
- mail-service は mail-service-mock を標準とし、実サービス前提にしない
- Prisma migration job は compose up 時に自動実行する前提を維持する
- observability 系 service は固定公開ポートを持つ開発用構成として扱う

### Parallel Opportunities

- [P] が付いたテストタスクは同一ファイルを競合しない限り並列実行できる
- [P] が付いたモデル、契約、観測項目の追加は並列実行できる
- 基盤 service と観測系 service は Foundational フェーズで並列進行できる
- US1 の service 追加と US2 の設計表整備は、基盤完了後に並列進行できる
- US3 の共有設定抽出は、US1/US2 の service 定義確定後に並列進行できる

---

## Implementation Strategy

### MVP First

1. Setup を完了する
2. Foundational を完了する
3. User Story 1 を実装し、全体 compose の起動骨子を完成させる
4. 単独で検証してから次のストーリーへ進む

### Incremental Delivery

1. 基盤整備後、優先度順にユーザーストーリーを追加する
2. 各ストーリーで service 定義、設計表、検証手順を同時に完成させる
3. 各ストーリー完了時に独立して compose の差分をレビューできる状態を作る

---

## Notes

- 曖昧な service 名や省略語は禁止する
- 同じファイルを更新するタスクは順序を明示する
- compose 設計表と `compose.yaml` の記述を切り離しすぎてはならない
- 実装タスクは compose 設定、設計表、quickstart の三点を一貫して更新する
