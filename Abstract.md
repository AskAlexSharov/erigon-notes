## Big things

- AuRa Gnosis DAO: https://github.com/ledgerwatch/erigon/issues/3643
    - Also https://github.com/ledgerwatch/erigon/issues/2659 and https://github.com/ledgerwatch/erigon/issues/2658

- Observer
    - Observer to Otterscan
- Erigon2 Upgrade2:
    - run all /tests on new commitment code
    - Addapt new History and Commitment code behind experimental flag
    - RPC KV - add higher level interface than kv: where we will operate by Erigon's Domains and Inverted indices (no
      matter where they stored in DB or in Snapshots).
        - Alex added type Inverted index. Take a look.
        - Alex will write proposal
        - Maybe we can read historical things by: get domain where blockNumb between A and B.

- After Erigon2:
    - mining must use new commitment code
    - mining: eth_getBlockByNumber returns null for pending block https://github.com/ledgerwatch/erigon/issues/2771
- Snapshots/Bittorrent:
    - Automate git push to snapshots repo
    - I think sometime it does ban all peers - and stuck
    - BitTorrent has feature: fallback to http urls if no peers alive. Investigate if we can use public S3 or GCS as
      such fallback. How much it will cost? CDN can make it cheaper? Cheaper than maintain Downloader
      node + backups? Can community do it (add own urls)?
    - Some code to migrate from v1 to v2 of snapshots
    - Will we introduce new stages for snapshots? Now code is in "before stage header" and "before stage senders prune".
      To make decision and create task for it.
    - erigon-lib/compress: slow and CPU-greedy
    - bittorent lib does produce much goroutines
- Rollup:
    - We already have multi-protocol support: Node -> Ethereum. Can add one more Node -> Rollup (If need run Rollup
      inside Erigon).
- TxPool:
    - erigon-lib/txpool/pool.go:MainLoop producing much goroutines
    - Investigate nil in LRU (likely it's race, not enough locks) https://github.com/ledgerwatch/erigon/issues/3799
      and https://github.com/hashicorp/golang-lru/issues/100 (or run with -race, or add more locks, or change LRU to
      thread-safe version)
    - move tx.rlp field to separated map, to make tx immutable
    - Implement txpool_pendingTransactions (similar to
      parity_pendingTransactions) https://github.com/ledgerwatch/erigon/issues/2917 . txpool_content - doesn’t support
      filters (TxPool can have 500K transactions). we need new method or extend this
      one - to support basic filters.
    - after Erigon2 need mining move to TxPool
- RPCDaemon:
    - remote_kv server - change architecture to reduce amount of read txs (or just somehow increase amount of read txs).
    - remote_kv.TxLookup() - doesn’t use existing read transaction
    - remote_kv.Block() - doesn’t use existing read transaction
    - Under heavy load to getBlockByNumber - does race (or panic)

# Erigon support (from Github)

- Merkle Trie root miss-match: big domain which need an owner
- Tracing:
    - Custom Tracer Method Handler Crash https://github.com/ledgerwatch/erigon/issues/4340
    - we need more auto-tests for Tracing - otherwise easy to break (maybe can copy them from OpenEthereum, maybe can
      run ./cmd/rpctest)
    - bsc trace rpc return bnb burning inner transaction on every https://github.com/ledgerwatch/erigon/issues/3968
    - Different results found for rpctest benchTraceCallMany (Erigon vs
      OpenEthereum)  https://github.com/ledgerwatch/erigon/issues/2490
    - Different results for trace_filter between Erigon and OE
      #2138 https://github.com/ledgerwatch/erigon/issues/2138
    - Tracing has incorrect gasUsed for value transfers #1436 https://github.com/ledgerwatch/erigon/issues/1436
    - Please add support for debug_traceBlockByNumber and debug_traceBlockByHash
      methods: https://github.com/ledgerwatch/erigon/issues/3080
- RPC:
    - eth_getBlockByNumber returns null for pending block: https://github.com/ledgerwatch/erigon/issues/2771
    - some eth_getTransactionReceipt calls return null for some early
    - Wrong results for eth_call when using a future block number https://github.com/ledgerwatch/erigon/issues/3136
    - EIP-1898 differences in responses between Erigon and GETH https://github.com/ledgerwatch/erigon/issues/3434
    - Adopt tests in api_test.go for RPCDaemon https://github.com/ledgerwatch/erigon/issues/939
    - eth_subscriptionLogs with topics filter problem https://github.com/ledgerwatch/erigon/issues/4030
      block https://github.com/ledgerwatch/erigon/issues/3968
- Address user's feedback:
    - Address flags usability feedback: https://github.com/ledgerwatch/erigon/issues/3932
    - Address something from user feedback https://github.com/ledgerwatch/erigon/issues/3121
- some cross-compile issue: https://github.com/ledgerwatch/erigon/issues/3676
  blocks https://github.com/ledgerwatch/erigon/issues/3243
- High number of concurrent network connections https://github.com/ledgerwatch/erigon/issues/3126
- Support (like geth) .toml config file

## Low Prio

- can’t run erigon with -race flag
- allow --metrics and --pprof use same port (they conflict now)
- Erigon:
    - compress.Getter - iterator on stack
    - like tables.go - but for snapshots
    - like rawdb - but for snapshots (pure funcs)
    - Add more test for blocks snapshots
- Docker:
    - Multi-platform images on
      DockerHub: https://dev.to/cloudx/multi-arch-docker-images-the-easy-way-with-github-actions-4k54 https://github.com/ledgerwatch/erigon/issues/3878
- Binance:
    - make —db.pageSize=8kb default
- Grafana:
    - Add torrent metrics
- Compressor:
    - predict posmap size (or move it to the end of file). predict posmap size. Or move .seg header to footer.
- Cloud:
    - Add some flag to enable kv.ReadAhead and more parallelism on disk reads (—disk=throughput ?)
- Please provide official binaries: https://github.com/ledgerwatch/erigon/issues/4153
- TxPool:
    - History of txPool (by option): timestamp when we received it by p2p, timestamp of rejection, tx rlp, senderAddr
      senderNonce, senderBalance, rejection reason, reason of moving to pool X.
    - Default history of rejection by txHash for 1 day only for local txs.
