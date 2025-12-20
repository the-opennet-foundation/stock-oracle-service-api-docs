# REST API Usage
description: "Using Paxeer REST/JSON-RPC over HTTP for protocol engineers and integrators"

:::scalar-callout{type="info"}
Audience: engineers building services against Paxeer node HTTP endpoints (JSON-RPC over HTTP).
:::

## 1. Base Endpoint
- Default: `http://<host>:8229` (if `--http` enabled; default port per README).
- Prefix/path: respect configured `--http.api` allowlist and `--http.pathprefix` if set.
- Content-Type: `application/json`.
- **Spec sources:** OpenRPC/YAML under `docs/paxeer/` (e.g., `block.yaml`, `state.yaml`, `transaction.yaml`, `fee_market.yaml`, `execute.yaml`, `submit.yaml`, `filter.yaml`, `sign.yaml`) and OpenRPC assets in `docs/openrpc/`.

## 2. Authentication & Allowlisting
- Modules exposed are governed by the node allowlist; ensure `eth`, `avm`, and required namespaces are enabled.
- For privileged namespaces (`admin`, `debug`), use IPC instead of public HTTP.

:::scalar-callout{type="warning"}
Never expose `admin` / `debug` / signer-related namespaces on public interfaces. Use IPC for those.
:::

## 3. Request Shape (JSON-RPC 2.0)
```json Request
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_blockNumber",
  "params": []
}
```

## 4. Common Calls
- **Network info:** `net_version`, `eth_chainId`, `eth_blockNumber`
- **Tx submission:** `eth_sendRawTransaction`
- **State queries:** `eth_getBalance`, `eth_getLogs`, `eth_call`
- **Filters:** `eth_newFilter`, `eth_getFilterChanges`, `eth_uninstallFilter`
- **AVM (expected):** `avm_call`, `avm_estimateGas`, `avm_getTransactionReceipt`
- **Paxeer OpenRPC set:** see method definitions in `docs/paxeer/*.yaml` (e.g., `eth_getBlockByNumber` in `block.yaml`; fee markets in `fee_market.yaml`; signing in `sign.yaml`).

## 5. Examples
### Get latest block number
```bash
curl -X POST http://localhost:8229 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}'
```

### Send raw transaction
```bash
curl -X POST http://localhost:8229 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_sendRawTransaction","params":["0x<signed_tx>"]}'
```

### Query logs (example)
```bash
curl -X POST http://localhost:8229 \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc":"2.0","id":1,"method":"eth_getLogs",
    "params":[{"fromBlock":"0x0","toBlock":"latest","topics":[]}]
  }'
```

## 6. Pagination and Limits
- Respect node-side limits (HTTP body limit, batch size); avoid very large log ranges—page by block ranges.
- For high-volume queries, prefer WebSocket subscriptions to avoid polling.

## 7. Error Handling
- JSON-RPC error codes with message; do not rely on transport-level status beyond 200/4xx.
- Treat `-32000` range as server-side execution errors; bubble to callers with context.

## 8. Performance Tips
- Batch requests where reasonable but stay under batch item/size limits.
- Co-locate clients near nodes to reduce latency; enable HTTP keep-alive.
- Use filters with selective topics to minimize payload size.

## 9. References
- Node HTTP config flags: `--http`, `--http.port`, `--http.api`, `--http.corsdomain`, `--http.vhosts`.
- Module allowlist logic: `network/node/rpcstack.go`.
