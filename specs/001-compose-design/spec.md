# フィーチャー仕様: compose.yaml 構成案

**Feature Branch**: `[001-compose-design]`  
**作成日**: `2026-06-07`  
**ステータス**: Draft  
**入力**: `環境定義に記載された環境のdock-compose.ymlを作成する`

## 目的と非目的 *(必須)*

### 目的

- 環境定義に記載された構成を、開発用の単一 `compose.yaml` に落とし込める仕様を確定する。
- 各 service の `depends_on`、`healthcheck`、環境変数、公開ポート、初期化ジョブを含む compose 設計表を確定する。
- 単一ファイルで起動順、依存関係、観測系、永続化、任意起動の方針を判断できる状態にする。

### 非目的

- 本番クラスタやマルチノード運用を設計すること。
- 各 service の実装、Dockerfile、アプリケーションコードを変更すること。
- 既存の個別起動スクリプトを直ちに削除すること。
- dev 以外の環境差分をこの spec で固定すること。

## ユーザーシナリオと受け入れ確認 *(必須)*

### ユーザーストーリー 1 - 1 つの compose で全体を起動したい (優先度: P1)

開発者は、基盤ミドルウェア、プラットフォーム本体、SaaS 層、下流マイクロサービス層を 1 つの `compose.yaml` で起動したい。これにより、開発環境の立ち上げ手順を一本化できる。

**この優先度の理由**: compose 化の主目的が、全体を 1 つの起動単位にまとめることだから。

**独立確認方法**: `compose.yaml` だけで、対象 service の構成と起動関係が一意に読み取れることを確認する。

**受け入れシナリオ**:

1. **Given** 必要な service がすべて定義されている, **When** compose 構成案を読む, **Then** 起動対象と除外対象が明確にわかる。
2. **Given** platform 側と SaaS 側と下流サービス側がある, **When** compose 構成案を読む, **Then** それらが同一ファイルに整理されている。

---

### ユーザーストーリー 2 - 各 service の依存と稼働条件を把握したい (優先度: P2)

開発者は、各 service の `depends_on`、`healthcheck`、環境変数、公開ポートを一覧で確認したい。これにより、起動順と障害時の切り分けがしやすくなる。

**この優先度の理由**: compose 設計の実用性は、依存関係と稼働条件が見えるかどうかで決まるから。

**独立確認方法**: 設計表だけで、各 service の起動条件と公開インターフェースが追跡できることを確認する。

**受け入れシナリオ**:

1. **Given** service 一覧がある, **When** 設計表を見る, **Then** 各 service の依存先と公開ポートを確認できる。
2. **Given** 初期化ジョブと常駐 service が混在する, **When** 設計表を見る, **Then** 種別ごとの扱いが区別されている。

---

### ユーザーストーリー 3 - 共通設定と再利用単位を揃えたい (優先度: P3)

開発者は、compose 全体で共有する network、volume、profile、restart policy を明記したい。これにより、単一ファイルでも運用前提がぶれない。

**この優先度の理由**: service 個別設定だけでは、compose 全体の起動・永続化・任意起動の方針が欠けるから。

**独立確認方法**: 共有設定の記述だけで、起動分離と永続化の方針が把握できることを確認する。

**受け入れシナリオ**:

1. **Given** 共有 network と volume が必要である, **When** 仕様を読む, **Then** どの service がそれを使うかが明確である。
2. **Given** 任意起動の観測系がある, **When** 仕様を読む, **Then** profile の扱いが明確である。

## 契約と境界 *(必須)*

### 関与する契約

- `compose.yaml` の service 定義
- 各 service の起動順と依存関係
- 各 service の healthcheck と可用性判定
- 各 service の環境変数
- 各 service の公開ポート
- 初期化ジョブと常駐 service の区分
- 共有 network、volume、profile、restart policy

### Temporal DSL アーティファクト

- ワークフロー定義: 対象外
- アクティビティ参照: 対象外
- ランタイム登録手順: 対象外
- 移行手順: compose 構成の移行手順として扱う

### 認可とテナント境界

- tenantId の伝播方法: 各 service の環境変数とアプリケーション間通信で伝播可能にする
- 認可入力: 該当なし。compose 自体は認可判定を行わない
- 評価タイミング: 該当なし
- deny 時の挙動: 該当なし

### ランタイム反映とデプロイ境界

- プラットフォーム側デプロイの要否: dev 用 compose の構成案として、アプリケーション再デプロイではなく compose 設定の更新で反映できることを前提とする
- 反映単位: service 追加、依存順、環境変数、公開ポート、profile、volume の変更
- 検証方法: compose 設計表のレビュー、起動順レビュー、healthcheck と公開ポートの整合確認

## 状態と整合性 *(必須)*

### 状態モデルと正源

- 正源となる状態: `compose.yaml` と compose 設計表
- 派生先: 起動手順書、README、実行時のオペレーションメモ
- 同期方式: 手動レビューと文書反映
- 整合性レベル: 文書間での一貫性を保つ

### 冪等性戦略

- requestId の採番元: 該当なし
- 重複検知方式: 同一 service の重複定義を避ける
- 再送時の期待挙動: 同じ定義は同じ構成案として扱う

## 失敗と回復 *(必須)*

### 失敗分類

- retryable: service の起動順不足や一時的な依存待ち
- non-retryable: service 名やポートの設計不整合
- compensatable: 設計変更で補正できる依存関係の見落とし
- requires-manual-intervention: 実装が存在しない service の追加が必要な場合
- dead-letter: 該当なし

### 回復戦略

- リトライ回数、バックオフ、タイムアウト: compose 設計表で起動待ちを明記する
- 打ち切り条件: 必須 service が揃わない場合は構成案を未完了とする
- 補償の要否と根拠: 仕様文書の修正で対応するため補償は不要
- 補償失敗時の扱い: 該当なし

## 可観測性と運用 *(必須)*

### 可観測性要件

- 必須ログ項目: service 名、依存先、公開ポート、初期化ジョブの順序
- 必須メトリクス: 起動成功率、起動失敗率、healthcheck 成功率、初期化ジョブ成功率
- 必須トレースと監査イベント: compose 構成変更の履歴とレビュー記録

### 運用影響と移行計画

- 設定追加や変更: `compose.yaml` と compose 設計表の更新
- liveness/readiness/shutdown への影響: healthcheck と起動順の整合に反映する
- 移行計画または後方互換方針: 既存の `docker-compose.yaml` と `start-all.bat` から段階的に整理できる構成とする

## Requirements *(必須)*

### Functional Requirements

- **FR-001**: システムは、環境定義に記載された service を 1 つの `compose.yaml` の構成案として表現しなければならない。
- **FR-002**: システムは、各 service の `depends_on`、`healthcheck`、環境変数、公開ポートを一覧化しなければならない。
- **FR-003**: 利用者は、初期化ジョブと常駐 service の違いを設計表で確認できなければならない。
- **FR-004**: システムは、基盤、プラットフォーム本体、SaaS 層、下流マイクロサービス層を同一 compose 案で扱わなければならない。
- **FR-005**: システムは、network、volume、profile、restart policy の共通設定を明記しなければならない。
- **FR-006**: システムは、mail-service を実サービスではなく mail-service-mock として扱わなければならない。
- **FR-007**: システムは、Prisma migration job を compose up 時に自動実行する前提で設計しなければならない。
- **FR-008**: システムは、observability 系 service を固定公開ポートを持つ開発用構成として扱わなければならない。

### Key Entities

- **Service**: compose に含める実行単位。名前、役割、依存関係、公開ポートを持つ。
- **Compose Design Table**: service ごとの起動条件と設定をまとめた一覧。
- **Shared Compose Setting**: network、volume、profile、restart policy の共通方針。

### Edge Cases

- mail-service は使わず mail-service-mock を標準にする。
- migrate job が常駐 service と誤認される場合の扱い。
- 既存の個別起動スクリプトと compose 案の重複がある場合の扱い。

## テスト計画 *(必須)*

- unit test: 文書中の service 一覧と設計表の整合
- contract test: `compose.yaml` に必要な項目が揃っているかの確認
- replay test または決定性検証: 該当なし
- idempotency test: 該当なし
- failure-path test: 必須 service 欠落時の扱い確認
- compensation test: 該当なし
- migration test / downgrade consideration: 既存 `docker-compose.yaml` からの移行観点

## Success Criteria *(必須)*

### Measurable Outcomes

- **SC-001**: 主要 service の一覧が 1 つの `compose.yaml` 構成案として 100% 記載されている。
- **SC-002**: 各 service の `depends_on`、`healthcheck`、環境変数、公開ポートが 100% 記載されている。
- **SC-003**: 共通設定として network、volume、profile、restart policy が 100% 記載されている。
- **SC-004**: compose 案だけで、初期化ジョブと常駐 service の区別を 1 回で判別できる。
- **SC-005**: mail-service の扱いが mail-service-mock として一意に決まっている。
- **SC-006**: Prisma migration job が自動実行前提として明記されている。

## Assumptions

- 既存の `docker-compose.yaml` と `start-all.bat` は、compose 統合のための参照資料として扱う。
- mail-service は未実装のため、compose 案では mail-service-mock を標準とする。
- migration job は開発用 compose で自動起動する。
- ここで作るのは実装ファイルではなく、compose 設計を確定するための仕様である。
