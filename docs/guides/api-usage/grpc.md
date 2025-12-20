# gRPC API Usage
description: "Connecting to Paxeer gRPC surfaces (where available) for protocol engineers"

:::scalar-callout{type="info"}
Audience: engineers integrating with Paxeer gRPC services or building gRPC gateways.
:::

## 1. Availability
- Paxeer core ships JSON-RPC as primary. If gRPC services are enabled (custom builds or sidecars), they typically expose:
  - **Public data**: blocks, txs, receipts, logs.
  - **Subscriptions**: streamed new heads / logs.
  - **Node/admin**: keep on private network only.

## 2. Endpoint & TLS
- Default (example): `grpc://<host>:<port>` or `grpcs://` with TLS.
- Configure max message size, keepalive, and auth (mTLS or token) on both client/server.

## 3. Example Proto (illustrative)
```proto
service Chain {
  rpc GetBlockByNumber(GetBlockByNumberRequest) returns (Block);
  rpc GetTransactionReceipt(GetTxReceiptRequest) returns (Receipt);
  rpc SubscribeHeads(SubscribeHeadsRequest) returns (stream Head);
}
```

## 4. Client Snippet (Go, illustrative)
```go
conn, err := grpc.Dial(addr,
  grpc.WithTransportCredentials(insecure.NewCredentials()), // replace with TLS in prod
  grpc.WithDefaultCallOptions(grpc.MaxCallRecvMsgSize(8<<20)),
)
defer conn.Close()
cli := pb.NewChainClient(conn)
resp, err := cli.GetBlockByNumber(ctx, &pb.GetBlockByNumberRequest{Number: 12345})
```

## 5. Streaming (Heads/Logs)
- Use server-side streaming RPCs to avoid polling.
- Implement backpressure by handling `Recv()` promptly; consider client-side buffering caps.

## 6. AuthZ / Exposure
- Keep admin RPCs on private networks; prefer mTLS.
- Enforce per-namespace/service ACLs if fronted by Envoy/Ingress.

## 7. Interop with REST
- Option 1: run a **gRPC-Gateway** to expose REST/JSON from gRPC definitions.
- Option 2: consume JSON-RPC directly for Ethereum-compatible methods, reserve gRPC for bulk/streaming.

## 8. Performance Tips
- Reuse channels; avoid per-request dials.
- Tune max inflight streams; set deadlines on all calls.
- Enable gzip only if CPU headroom is available; otherwise prefer no compression for low latency.
