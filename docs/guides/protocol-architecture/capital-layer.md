# Capital Layer
description: "Capital orchestration, risk, and cross-VM integration"

:::scalar-callout{type="info"}
Audience: protocol engineers extending capital/risk primitives and their precompiles.
:::

## 1. Diagram: Capital & Bridge
:::scalar-embed
src="https://pbs.twimg.com/profile_banners/1599772885857538051/1740351609/1500x500"
caption="Capital engine on AVM, bridged to EVM via precompiles"
:::

Components:
- **Capital engine** (`capital/engine`): orchestrates flows, liquidity, accounting.
- **Pools** (`capital/pools`): liquidity and funding pools.
- **Risk** (`capital/risk`): scoring/limits; AVM-native heavy compute.
- **Precompile bridge**: EVM calls into AVM logic at fixed addresses.

## 2. Precompile Map (EVM-facing)
- `0x100` Capital — capital operations routed to AVM engine.
- `0x101` VMBridge — cross-VM call routing helper.
- `0x102` RiskEngine — risk assessment compute.

Pattern:
1) EVM contract → call precompile address with ABI args.
2) Bridge shims → AVM syscall/handler.
3) AVM executes capital/risk logic → returns ABI-packed result → EVM resumes.

## 3. Design Goals
- Keep **heavy compute** in AVM (register-based, faster arithmetic).
- Maintain **EVM compatibility** via precompile ABIs.
- Ensure **gas parity**: charge EVM caller appropriately; enforce AVM gas internally.
- Preserve **determinism**: fixed gas costs, bounded memory, no host I/O.

## 4. State & Accounting
- Unified state backend: balances/positions stored in common trie.
- Pool accounting must be identical across nodes → design deterministic math; avoid float; prefer fixed-point.
- Consider replay tests to ensure identical receipts/state roots.

## 5. Risk Engine Notes
- Inputs: user/account keys, exposure, collateral, pricing feeds (on-chain).
- Outputs: risk scores/limits; may gate capital engine actions.
- AVM implementation can exploit 256-bit registers for vectorized math patterns.

## 6. Extending the Layer
- Add new operation → extend AVM handler + precompile ABI.
- Add new precompile address → register shim + cost schedule.
- Add oracle inputs → ensure data availability on-chain; avoid off-chain lookups.

## 7. Testing Guidance
- ABI conformance tests: EVM caller ↔ AVM bridge roundtrips.
- Gas tests: verify cost schedule matches intended compute.
- Differential: same tx on multiple nodes → identical receipts/state root.

## 8. Operational Considerations
- Monitor capital/risk call rates; ensure bridge precompiles not starved by gas pricing.
- Observe latency: AVM calls should remain sub-ms for simple paths.
- Backpressure: if AVM queueing appears, profile hotspots; consider opcode cost tuning.
