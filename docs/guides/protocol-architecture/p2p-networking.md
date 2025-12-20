# P2P Networking
description: "Peer discovery, protocols, and message flow for Paxeer"

:::scalar-callout{type="info"}
Audience: protocol engineers extending wire protocols or tuning networking.
:::

## 1. Diagram: P2P Stack
:::scalar-embed
src="https://pbs.twimg.com/profile_banners/1599772885857538051/1740351609/1500x500"
caption="Paxeer p2p layers: transport → discovery → protocols → execution"
:::

- **Transport:** TCP with RLPx-style session establishment (encryption, auth).
- **Discovery:** Node table + discovery DB; uses node key; advertises capabilities.
- **Protocols:** Pluggable; registered via `RegisterProtocols`.
- **Services:** Execution sync, tx propagation, block propagation.

## 2. Node Identity
- Node key from `Config.NodeKey()`; persisted in datadir; used for handshake and ENR.
- Node name from `Config.NodeName()`; included in peer logs.

## 3. Protocol Registration
- Services add protocol definitions (message handlers + version) before node start.
- P2P server holds aggregate protocol list; peers negotiate overlapping capabilities.

## 4. Message Flow
1) **Handshake**: exchange status/version; capability negotiation.
2) **Tx propagation**: announce hashes → request bodies on demand.
3) **Block propagation**: announce/new block → request bodies → import pipeline.
4) **State sync** (if applicable): request/response tries, headers, bodies.

## 5. Peer Management
- **Limits**: max peers enforced; inbound/outbound mix.
- **Penalties**: invalid block/tx → drop/ban peer; rate-limit noisy peers.
- **Backoff**: dial cooldown on failures.

## 6. Performance Tuning
- Keep-alives and timeouts set in `p2p.Server` config.
- Prefer smaller announcements (hashes) vs full payloads; request-on-demand.
- Monitor peer churn, duplicate deliveries, latency to first byte for blocks.

## 7. Security
- Encrypted sessions; node key confidentiality is critical.
- Validate all payload sizes; drop malformed messages.
- Only register trusted protocols; avoid enabling debug admin over public interfaces.

## 8. Observability
- Metrics: peer counts (in/out), protocol errors, message rates, drops.
- Logs: protocol negotiation, disconnect reasons, throttling events.

## 9. References
- Node container/server creation: `network/node/node.go`
- Protocol registration hook: `RegisterProtocols`
- P2P server struct and config: `p2p/` (implementation-specific)
