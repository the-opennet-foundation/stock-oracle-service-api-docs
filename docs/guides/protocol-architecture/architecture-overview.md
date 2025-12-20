# Paxeer Architecture Overview
description: "High-level architecture of the Paxeer execution layer for protocol engineers"

:::scalar-callout{type="info"}
Audience: protocol engineers designing, extending, or auditing Paxeer core.
:::

## 1. System Diagram
:::scalar-embed
src="https://pbs.twimg.com/profile_banners/1599772885857538051/1740351609/1500x500"
caption="High-level Paxeer runtime layers"
:::

- **Process boundary:** single `paxeer` binary hosts all subsystems.
- **Service container (`node.Node`):** lifecycle-managed services (p2p, RPC, execution backends, metrics).
- **Execution:** dual-VM (EVM + AVM) with router + precompile bridge.
- **Consensus:** Clique PoA (5s); signer set maintained in chain state.
- **Storage:** unified state trie + block/receipt DB in `rawdb/ethdb`.
- **Interfaces:** HTTP/WS/IPC/inproc RPC; GraphQL optional; metrics endpoints.

## 2. Design Goals
- **Dual-VM flexibility:** choose optimal VM per workload; bridge via precompiles.
- **Deterministic execution:** fixed gas costing, bounded memory, syscall whitelist.
- **Operator safety:** RPC module allowlisting, auth/non-auth API sets, size limits.
- **Extensibility:** pluggable services via lifecycle + API/protocol registration.
- **Observability-first:** metrics exporters (Influx v1/v2, expvar), structured logs.

## 3. Service Container (`network/node`)
- `Node` owns: `p2p.Server`, `rpc.Server` (inproc), HTTP/WS/IPC servers, key store, DB handles.
- Lifecycle pattern:
  1. Construct node with `Config`.
  2. Register protocols/APIs/handlers while state = initializing.
  3. Register lifecycles (execution, metrics, stats).
  4. `Start()` opens endpoints → starts lifecycles; `Stop()` halts goroutines and closes DBs.
- HTTP/WS prefixes validated; batch limits set on RPC server.

### Key Interfaces
```go
type Lifecycle interface { Start() error; Stop() error }
type Node struct {
  server *p2p.Server
  inprocHandler *rpc.Server
  lifecycles []Lifecycle
  rpcAPIs []rpc.API
}
```

## 4. Execution Routing
- **Router** decides target VM per transaction/contract type.
- **Precompile bridge:** EVM-facing precompiles (e.g., VMBridge `0x101`, RiskEngine `0x102`) call into AVM logic.
- **Unified state:** both VMs commit to the same state DB; receipts/logs follow Ethereum shape for tooling compatibility.

## 5. Data Flow (Block Import)
1. Block arrives via p2p or local sealing.
2. Header validated (Clique rules); body tx list passed to execution.
3. Router dispatches each tx to EVM or AVM.
4. Execution yields receipts/logs/state root.
5. Consensus finalizes; RPC surfaces via `eth_*`/AVM namespaces.

## 6. Security Considerations
- **RPC exposure:** module allowlist; auth vs non-auth sets; body/read limits.
- **Determinism:** AVM forbids non-deterministic sources; EVM inherited Ethereum rules.
- **Signer keys:** store in keystore; datadir lock prevents concurrent use.
- **Resource caps:** gas metering, batch limits, WS read limits protect from DoS.

## 7. Extensibility Map
- Add p2p protocol → `RegisterProtocols`.
- Add RPC namespace → `RegisterAPIs` with allowlist entry.
- Add long-running service → implement `Lifecycle` and register on node.
- Add observability → mount handler via `RegisterHandler`, export metrics.

## 8. References
- Node lifecycle: `network/node/node.go`
- Lifecycle interface: `network/node/lifecycle.go`
- RPC server/registry: `network/rpc/server.go`, `network/rpc/service.go`
- Module allowlist: `network/node/rpcstack.go`
