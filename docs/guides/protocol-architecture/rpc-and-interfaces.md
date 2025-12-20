# RPC and Interfaces
description: "RPC architecture, namespaces, and transport surfaces for Paxeer"

:::scalar-callout{type="info"}
Audience: protocol engineers and infra owners exposing/consuming Paxeer RPC.
:::

## 1. Transports
- **HTTP**: JSON-RPC 2.0; optional auth; prefix validation.
- **WebSocket**: subscriptions and bidirectional streaming; read size limits.
- **IPC**: Unix domain socket; highest trust; same service registry.
- **In-process**: `node.Attach()` for embedded clients.

## 2. Service Registry
- Backed by `rpc.Server`; reflective registration of methods/subscriptions.
- Namespaced via `RegisterName(namespace, receiver)`.
- Methods must be exported; subscription callbacks supported.

### Batch Controls
- `SetBatchLimits(itemLimit, maxResponseSize)` before serving.
- Protects from oversized batch DoS.

## 3. Module Allowlisting
- `RegisterApis(apis, modules, srv)` builds an allowlist; only namespaces in list are exposed.
- If allowlist empty → expose all registered APIs (not recommended on public endpoints).
- Use separate allowlists for auth vs non-auth servers.

## 4. Canonical Namespaces (illustrative)
- `eth` — Ethereum-compatible JSON-RPC; filters/logs; tx submit.
- `avm` — AVM-specific tx/inspection (expected).
- `net` — networking info.
- `debug`/`admin` — node management (restrict to IPC).
- `graphql` — optional GraphQL endpoint (configured via flags).
- `ethstats` — stats daemon integration.

## 5. HTTP Handlers
- Custom handlers mountable via `RegisterHandler(name, path, handler)`.
- Use for metrics, health, profiling (ensure auth/protection).

## 6. Limits & Hardening
- **HTTP body limit** and **WS read limit** set on server.
- **Batch item/size limits** set on RPC server.
- Prefer **IPC** for privileged namespaces (`admin`, `debug`); avoid exposing them over public HTTP/WS.

## 7. Error Surfaces
- Standard JSON-RPC error codes; include meaningful messages without leaking internals.
- Subscription drop on WS close; ensure clients handle reconnects/backfill.

## 8. Observability
- Log RPC handler registration and transport start.
- Emit metrics per method (latency, error rate); consider rate-limiting at edge.

## 9. References
- Service registry: `network/rpc/service.go`
- Server limits/registration: `network/rpc/server.go`
- Module allowlist: `network/node/rpcstack.go`
- Handler mounting: `network/node/node.go` (RegisterHandler)
