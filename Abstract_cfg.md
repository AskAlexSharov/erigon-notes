
## Configuration (do by many small steps - easy to review PR)

- 3 steps: 1. create configs, 2. create objects and stages, 3. start goroutines, httpListeners, connect to remote
  servers
- step1:
    - 1 ethconfig.Config object is the result of step 1 and input of step 2.
    - Each component (sentry, txpool, erigon, rpc, downloader) provide own config (for example torrentcfg.Cfg)
    - ethconfig package must not depend on huge packages (like “core”), but it can depend on other small config
      packages (like “torrentcfg“) or utilities (like “common” or “types”).
    - Move configuration logic out of `erigon/eth/backend.go:New` to higher-level funcs
    - [done] Create 1 DataDir object with pre-filled paths to all sub-dirs: dir.tmp, dir.chaindata, dir.snap, etc…
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
- reduce nil-ness:
    - less rely on nil-ness of objects: “if txpool == nil {“ - better use cfg.TxPool.Enabled. It means we can create
      internal objects: grpcServers, txpool, etc… But start goroutines and http listeners only if component is
      enabled.
- Code Deduplication
    - Several “StartGrpc” funcs are exists
    - `cmd/integration/commands/root.go:openKV`, `node/node.go:OpenDatabase`,`consensus/db/db.go`
      and `p2p/enode/nodedb.go` - must use same code to open db. And this code must separate configuration from db
      opening (steps 1 and 3).
    - 1 ethconfig.Config object is the result of step 1 and input to step 2.
    - 1 Ethereum object to hold other objects. No DI, no inversion of control. But use interfaces. It’s result of
      step 2. ethereum.Start() and ethereum.Stop() for goroutines creation

