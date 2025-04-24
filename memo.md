# logパッケージについて

- posは、長さ+バイトのセットで何セット目？を示す。
- offsetは、何バイト目？を示す。
- segmentは、一定のバイトごとに分割されるstoreとindexを管理する構造体。
  - baseOffsetで何バイト目を扱うのか？がわかる。

# serverパッケージについて

- testでは、ダミクラを建ててリクエストを送り、レスポンスが期待した結果と一致しているか？を見ている。

# discovery, replicator, agentについて

- discoveryのmembershipでは、serfがmembership(サーバ間ディスカバリ)を管理・操作している
- replicatorの実体はgrpcクライアントで、リクエスト結果のログをローカルサーバーにさらにリクエストすることで、複製を行う。
  - replicatorは、membershipのhandlerのinterfaceを実装している。
  - これにより、membershipのNewの第一引数にreplicatorを入れれば、自動でJoin時に複製が行われるようになる。
  - 尚、membershipのNewで、Joinは行われるようになっている。
  - 一度Joinしたら、for文により、ずーーっと複製を試みるようになる。
- agentが分散サービスの実体。と思われる。
  - 実際にgrpcサーバーとして動く
  - replicator、membershipを持つので、他agentとの協働、複製が行える。
  - testでは、各agentにダミクラを用意し、期待したレスポンスを返すか？というシナリオをチェックしている。