# compose.yaml 設計表

この文書は、開発用の単一 `compose.yaml` に含める service、依存関係、healthcheck、環境変数、公開ポート、共有設定を一覧化したものです。

## 1. 役割区分

### 基盤必須

- postgres
- temporal
- temporal-ui
- nats
- opa
- redis

### 観測系

- prometheus
- vector
- jaeger

### 初期化必須

- opa-policy-loader
- platform-migrate
- saas-backend-migrate
- user-service-migrate
- file-storage-service-migrate
- routing-file-service-migrate

### 業務実行必須

- orchestration-platform
- saas-backend
- saas-frontend
- user-service
- file-storage-service
- routing-file-service
- mail-service-mock

## 2. Service 設計表

| Service | 役割 | depends_on | healthcheck | 環境変数 | 公開ポート | 備考 |
| --- | --- | --- | --- | --- | --- | --- |
| postgres | 永続化基盤 | なし | pg_isready | POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD | 5432 | PostgreSQL 16 |
| temporal | ワークフロー基盤 | postgres | temporal operator cluster health | DB, POSTGRES_SEEDS, POSTGRES_USER, POSTGRES_PWD, POSTGRES_DB | 7233 | Temporal server |
| temporal-ui | 可視化 UI | temporal | /health | TEMPORAL_ADDRESS | 8080 | 開発用 UI |
| nats | JetStream メッセージング | なし | /healthz | なし | 4222, 8222 | JetStream 有効 |
| opa | 認可ポリシー | なし | /health | なし | 8181 | policies を読み込む |
| redis | クォータ/共有フラグ | なし | redis-cli ping | なし | 6379 | AOF 有効 |
| prometheus | メトリクス収集 | orchestration-platform | /-/healthy | なし | 9090 | profile: observability |
| vector | ログ集約 | なし | /health | なし | 8686 | profile: observability |
| jaeger | トレース可視化 | なし | / | COLLECTOR_OTLP_ENABLED | 16686, 4317, 4318 | profile: observability |
| opa-policy-loader | 初期化ジョブ | opa | なし | なし | なし | 1 回実行 |
| platform-migrate | 初期化ジョブ | postgres | なし | なし | なし | Prisma migrate |
| saas-backend-migrate | 初期化ジョブ | postgres | なし | なし | なし | Prisma migrate |
| user-service-migrate | 初期化ジョブ | postgres | なし | なし | なし | Prisma migrate |
| file-storage-service-migrate | 初期化ジョブ | postgres | なし | なし | なし | Prisma migrate |
| routing-file-service-migrate | 初期化ジョブ | postgres | なし | なし | なし | Prisma migrate |
| orchestration-platform | 業務実行 | temporal, temporal-ui, nats, opa, redis, 各 migrate, opa-policy-loader | /health | TEMPORAL_ADDRESS, NATS_URL, OPA_URL, REDIS_URL, POSTGRES_URL | 4000 | 中核 Node サービス |
| saas-backend | 業務実行 | orchestration-platform, saas-backend-migrate, nats | /health | NATS_URL, ORCHESTRATION_CALLBACK_URL, POSTGRES_URL | 4005 | SaaS API |
| saas-frontend | 業務実行 | saas-backend | / | VITE_API_BASE_URL | 5173 | Vite/React |
| user-service | 業務実行 | user-service-migrate, postgres | /health | POSTGRES_URL, PORT | 4002 | 下流マイクロサービス |
| file-storage-service | 業務実行 | file-storage-service-migrate, postgres | /health | POSTGRES_URL, PORT | 4001 | 下流マイクロサービス |
| routing-file-service | 業務実行 | routing-file-service-migrate, postgres | /health | POSTGRES_URL, PORT | 4003 | 下流マイクロサービス |
| mail-service-mock | 業務実行 | なし | /health | PORT | 4010 | mail-service 代替 |

## 3. 起動順

1. postgres
2. temporal
3. temporal-ui
4. nats
5. opa
6. redis
7. opa-policy-loader
8. 各 migrate job
9. orchestration-platform
10. saas-backend
11. saas-frontend
12. user-service
13. file-storage-service
14. routing-file-service
15. mail-service-mock

## 4. 共有設定

- network: `orchestration-net`
- volume: `postgres-data`, `temporal-data`, `nats-data`, `redis-data`, `prometheus-data`, `vector-data`
- profile: `observability`
- restart policy: `unless-stopped` を基本とし、初期化ジョブは `no`

## 5. 検証観点

- `docker compose config` で YAML と依存関係が解釈できること
- 各 service に必要な公開ポートが揃っていること
- healthcheck が service ごとに定義されていること
- migration job が常駐 service と分離されていること
- mail-service が mail-service-mock で代替されていること
