Abstract

- Erigon support:
    - BSC to support Euler hard-fork https://github.com/ledgerwatch/erigon/issues/4097
    - bsc trace rpc return bnb burning inner transaction on every
    - eth_subscriptionLogs with topics filter problem https://github.com/ledgerwatch/erigon/issues/4030
      block https://github.com/ledgerwatch/erigon/issues/3968
    - Address flags usability feedback: https://github.com/ledgerwatch/erigon/issues/3932
    - Trace: Bad address balance during SUICIDE https://github.com/ledgerwatch/erigon/issues/3953
    - can’t run erigon with -race flag
    - Can docker hub build images for other architectures? https://github.com/ledgerwatch/erigon/issues/3878
    - some cross-compile issue: https://github.com/ledgerwatch/erigon/issues/3676
    - eth_sendRawTransaction error https://github.com/ledgerwatch/erigon/issues/3625
    - some eth_getTransactionReceipt calls return null for some early
      blocks https://github.com/ledgerwatch/erigon/issues/3243
    - eth_call "method handler crashed" depending on block argument https://github.com/ledgerwatch/erigon/issues/3152
    - Wrong results for eth_call when using a future block number https://github.com/ledgerwatch/erigon/issues/3136
    - EIP-1898 differences in responses between Erigon and GETH https://github.com/ledgerwatch/erigon/issues/3434
    - High number of concurrent network connections https://github.com/ledgerwatch/erigon/issues/3126
    - Address something from user feedback https://github.com/ledgerwatch/erigon/issues/3121
    - Please add support for debug_traceBlockByNumber and debug_traceBlockByHash
      methods: https://github.com/ledgerwatch/erigon/issues/3080
    - Different results found for rpctest benchTraceCallMany (Erigon vs OpenEthereum)  https://github.com/ledgerwatch/erigon/issues/2490
    - Different results for trace_filter between Erigon and OE #2138 https://github.com/ledgerwatch/erigon/issues/2138
    - close this tasks? https://github.com/ledgerwatch/erigon/issues/2659 and https://github.com/ledgerwatch/erigon/issues/2658


- Configuration (do by many small steps - easy to review PR):
    - 3 steps: 1. create configs, 2. create objects and stages, 3. start goroutines, httpListeners, connect to remote
      servers
    - step1:
        - 1 ethconfig.Config object is the result of step 1 and input of step 2.
        - Each component (sentry, txpool, erigon, rpc, downloader) provide own config (for example torrentcfg.Cfg)
        - ethconfig package must not depend on huge packages (like “core”), but it can depend on other small config
          packages (like “torrentcfg“) or utilities (like “common” or “types”).
        - Move configuration logic out of `erigon/eth/backend.go:New` to higher-level funcs
        - Create 1 DataDir object with pre-filled paths to all sub-dirs: dir.tmp, dir.chaindata, dir.snap, etc…
        - Each sub-config must have defaults. Example: txpool.DefaultConfig, torrentcfg.Default()
        - ethconfig.Config.Genesis must be filled on this step
    - step2:
        - 1 turbo/services.All object to hold all services. It’s output of step 2.
        - Each component (sentry, txpool, erigon, rpc, downloader) provide own object (for example rpcservices.All).
        - It’s ok to create gRPC connections here (for example remote.NewKVClient(conn)), because grpc connection is
          lazy (not fail-fast) and has retry logic inside. It’s important because when run components (sentry, txpool,
          erigon, rpc, downloader) as individual processes (outside of Erigon) - they must start independently - even if
          all other components are down. It helps to prevent cascade outage and simplify cluster operations.
        - SetEthConfig() - must return error
        - turbo/services package must not depend on huge packages
        - “cmd/rpcdaemon/rpcservices/eth_txpool.go” must not depend on “erigon-lib/txpool“ (it needs only 1 constant)
    - step3:
        - ethereum.Start or txpool.MainLoop
        - Apply same ideas to cmd/txpool/main.go, cmd/sentry/main.go, cmd/downloader/main.go, cmd/rpcdaemon/main.go
    - Code Deduplication
        - Several “StartGrpc” funcs are exists
        - `cmd/integration/commands/root.go:openKV`, `node/node.go:OpenDatabase`,`consensus/db/db.go`
          and `p2p/enode/nodedb.go` - must use same code to open db. And this code must separate configuration from db
          opening (steps 1 and 3).
        - 1 ethconfig.Config object is the result of step 1 and input to step 2.
        - 1 Ethereum object to hold other objects. No DI, no inversion of control. But use interfaces. It’s result of
          step 2. ethereum.Start() and ethereum.Stop() for goroutines creation
    - reduce nil-ness:
        - less rely on nil-ness of objects: “if txpool == nil {“ - better use cfg.TxPool.Enabled. It means we can create
          internal objects: grpcServers, txpool, etc… But start goroutines and http listeners only if component is
          enabled.
- Rollup:
    - We already have multi-protocol support: Node -> Ethereum. Can add one more Node -> Rollup (If need run Rollup
      inside Erigon).
- DockerCompose: add external txpool and 2 sentries
- TxPool - less goroutines (1 channel and 1 goroutine per subscriber)
    - txpool autostart on netID > 10. Mining has hacks. Mining need
    - rlp and transaction receipts
    - after Erigon2 need mining move to TxPool
    - Investigate nil in LRU https://github.com/ledgerwatch/erigon/issues/3799
      and https://github.com/hashicorp/golang-lru/issues/100
    - move tx.rlp field to separated map, to make tx immutable
    - Implement txpool_pendingTransactions (similar to
      parity_pendingTransactions) https://github.com/ledgerwatch/erigon/issues/2917 . txpool_content - doesn’t support
      filters (TxPool can have 500K transactions). we need new method or extend this
      one - to support basic filters.
- Pool Far:
    - History of txPool (by option): timestamp when we received it by p2p, timestamp of rejection, tx rlp, senderAddr
      senderNonce, senderBalance, rejection reason, reason of moving to pool X.
    - Default history of rejection by txHash for 1 day only for local txs.
- BitTorrent:
    - after restart it may Ban good peers https://github.com/anacrolix/torrent/issues/746
    - I think sometime it does ban all peers - and stuck
    - BitTorrent has feature: fallback to http urls if no peers alive. Investigate if we can use public S3 or GCS as
      such fallback. How much it will cost? Cheaper than maintain Downloader node + backups?
    - Instead of datadir/snapshots_tmp need use datadir/snapshots/tmp - because user must be able to mount
      datadir/snapshots to another disk
- Snapshots:
    - Automate git push to snapshots repo
- Erigon2:
    - run all /tests on new commitment code
    - getter - iterator on stack
    - like tables.go - but for snapshots
    - like rawdb - but for snapshots
    - Test for snapshots
- Erigon2 Upgrade2:
    - Addapt new History and Commitment code behind experimental flag
    -
- After Erigon2:
    - mining must use new commitment code
    - mining: eth_getBlockByNumber returns null for pending block https://github.com/ledgerwatch/erigon/issues/2771
- RPCDaemon:
    - remote_kv server - change architecture to reduce amount of read txs (or just somehow increase amount of read txs).
    - remote_kv.TxLookup() - doesn’t use existing read transaction
    - remote_kv.Block() - doesn’t use existing read transaction
    - Under heavy load to getBlockByNumber - does race (or panic)

## In progress

- AuRa Gnosis DAO: https://github.com/ledgerwatch/erigon/issues/3643
- Observer
-

## Low Prio

- Binance:
    - make —db.pageSize=8kb default
- Grafana:
    - Add torrent metrics
- Compressor:
    - predict posmap size (or move it to the end of file). predict posmap size. Or move .seg header to footer.
- Cloud:
    - Add some flag to enable kv.ReadAhead and more parallelism on disk reads (—disk=throughput ?)
