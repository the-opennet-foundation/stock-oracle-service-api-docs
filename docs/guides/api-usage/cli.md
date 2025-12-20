# CLI Usage
description: "Using the Paxeer CLI to query, send transactions, and manage nodes"

:::scalar-callout{type="info"}
Audience: protocol engineers and operators using the `paxeer` client via CLI.
:::

## 1. Binaries
- Build: `make paxeer` (see repo README).
- Run: `./build/bin/paxeer`.
- Common flags: `--paxeer` (mainnet), `--paxeer-testnet`, `--datadir`, `--http`, `--ws`, `--ipcpath`.

## 2. Node Lifecycle
### Initialize with genesis
```bash
./build/bin/paxeer init genesis/paxeer_mainnet.json --datadir ~/.paxeer
```

### Start node (HTTP + WS)
```bash
./build/bin/paxeer --paxeer --http --http.port 8229 --ws --ws.port 8230
```

### Console (attach in-proc)
```bash
./build/bin/paxeer --paxeer console
```
or attach to running node:
```bash
./build/bin/paxeer attach ~/.paxeer/geth.ipc
```

## 3. Account & Keystore
```bash
./build/bin/paxeer account new --datadir ~/.paxeer
./build/bin/paxeer account list --datadir ~/.paxeer
```
Store keystore securely; do not share `~/.paxeer/keystore`.

## 4. Transactions
### Send raw transaction (via console JS)
```javascript
eth.sendRawTransaction("0x<signed_tx>")
```

### Gas estimation
```javascript
eth.estimateGas({from:eth.accounts[0], to:"0x...", data:"0x..."})
```

## 5. Logs & Filters (console)
```javascript
var f = eth.filter({fromBlock: "latest", topics: []});
f.watch(function(err, log){ if(!err) console.log(log); });
```

## 6. AVM Precompiles (EVM side)
Call precompiles like:
```javascript
eth.call({
  to: "0x0000000000000000000000000000000000000102", // RiskEngine
  data: "0x<abi_encoded_args>"
})
```

## 7. Admin/Debug (keep on IPC)
```javascript
admin.peers
admin.nodeInfo
debug.setHead("0x...")
```
:::scalar-callout{type="warning"}
Do not expose admin/debug over HTTP/WS; use IPC only.
:::

## 8. Metrics
Enable metrics flags from CLI (InfluxDB, expvar) as needed; prefer dedicated handlers over public exposure.

## 9. Useful Flags (sampling)
- `--http.api eth,net,web3,avm`
- `--http.corsdomain *` (only for local dev)
- `--ipcdisable` (if IPC unwanted)
- `--ws.api eth,net,web3,avm`
- `--maxpeers N` (p2p cap)

## 10. Troubleshooting
- Check logs for RPC allowlist failures.
- Verify datadir lock not held by another process.
- Use `--nodiscover` for isolated dev nets.
