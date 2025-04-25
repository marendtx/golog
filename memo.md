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

# raftとdistributedについて

- raftは、主に以下の構成からなる。
  - StreamLayer。raftでの通信を可能にするレイヤー。基本的に生のTCPのクラサバ。
  - logStore。ログ。DBを例にすると、コミットログ。
  - FSM。状態。DBを例にすると、DBそのもの。
    - このリポジトリでは、ログの複製を目的としていて、logStoreと混合されてしまうところがややこしい。
- distributedは、以下の役割を持つ。
  - membershipでJoin、Leaveした時のハンドラ。Raftにいろいろと任せている。
  - gRPC serverのハンドラ。ログのcommit、appendの実体。
- 構成は、agent -> server・membership -> distributed -> segment, store, index, log

# loadbalanceについて

- resolverで名前解決を行っている。
- pickerで接続先を選ぶようにしている。
  - gRPCのメソッド名を見て、書き込みはリーダーへ、読み取りはフォロワーへリクエストを転送する。
- 上記をサーバー側ではなく、クライアント側で行うようにしている。サーバーはただの接続待ち受け。

# まとめ

- NewSQL(Raft)であっても、リーダーに書き込むのは変わらない。これはすべてのステートフルなサーバーに当てはまる。
- 大切なのは、Raftにより、リーダー障害時に自動で復旧できること。
  - MySQLは、マスターを事前に決める。マスターに障害が起こると、もうどうしようもなくなる。
  - NewSQLはリーダー選出が可能。これにより、障害復旧を自動で行える。

# 補足

- Raftは、基本ログから状態を複製するためのコンセンサスアルゴリズム。つまりステートフルなサーバー用。
- 単純にリーダー・フォロワーを決めたいだけだったら、serfだけを使うといい。
- そんな訳で、simple_agentにserfによるリーダー・フォロワーのシステムも書いた。
- manger・workerで、managerの選出に使えそう。