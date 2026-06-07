# Quickstart

この手順は、開発用の単一 compose 構成を確認するためのものです。

## 事前条件

- Docker と Docker Compose が利用できること
- リポジトリのルートで作業していること

## 起動前確認

```bash
docker compose config
```

このコマンドで、`compose.yaml` の構文、depends_on、healthcheck、ports、profiles が解釈できることを確認します。

## 起動

```bash
docker compose up -d
```

## 確認対象

- `postgres` が 5432 で応答すること
- `temporal` が 7233 で応答すること
- `temporal-ui` が 8080 で応答すること
- `nats` が 4222 と 8222 で応答すること
- `opa` が 8181 で応答すること
- `orchestration-platform` が 4000 で応答すること
- `saas-backend` が 4005 で応答すること
- `saas-frontend` が 5173 で応答すること
- `user-service` が 4002 で応答すること
- `file-storage-service` が 4001 で応答すること
- `routing-file-service` が 4003 で応答すること
- `mail-service-mock` が 4010 で応答すること

## observability 系の起動

observability 系 service は profile `observability` に含めています。

```bash
docker compose --profile observability up -d
```

## 設計表の見方

`compose-design.md` では次を確認できます。

- service ごとの depends_on
- healthcheck の有無
- 環境変数
- 公開ポート
- 初期化ジョブと常駐 service の区分
- 共有 network、volume、profile、restart policy

## 参照ファイル

- [compose.yaml](compose.yaml)
- [compose-design.md](compose-design.md)
- [.env.example](.env.example)
- [README.md](README.md)
