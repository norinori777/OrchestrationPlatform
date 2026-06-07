# OrchestrationPlatform

## Compose Development Set

開発用の単一 compose 構成案は次のファイルにまとまっています。

- [compose.yaml](compose.yaml)
- [compose-design.md](compose-design.md)
- [.env.example](.env.example)
- [quickstart.md](quickstart.md)

compose の目的は、基盤ミドルウェア、プラットフォーム本体、SaaS 層、下流マイクロサービス層を 1 つの `compose.yaml` で扱い、各 service の `depends_on`、`healthcheck`、環境変数、公開ポート、共有設定を明示することです。
