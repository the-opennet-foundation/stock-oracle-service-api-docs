# Consensus (Clique PoA)
description: "Consensus architecture and block lifecycle for Paxeer"

:::scalar-callout{type="info"}
Audience: protocol engineers working on sealing, validation, or consensus safety.
:::

## 1. Diagram: Block Lifecycle
:::scalar-embed
src="https://pbs.twimg.com/profile_banners/1599772885857538051/1740351609/1500x500"
caption="Clique PoA block sealing and import flow"
:::

Flow:
1) **Tx pool** assembles pending txs.
2) **Proposer (authorized signer)** builds header, selects txs, seals with signer key.
3) **Gossip** sealed block to peers.
4) **Validators** verify header + signer + difficulty rules, execute txs, check state root/receipts.
5) **Commit** if valid; otherwise drop/penalize peer.

## 2. Clique Essentials
- **Signer set**: stored in chain state; updated via special votes.
- **Period**: 5s block time (mainnet/testnet).
- **Difficulty rule**: encodes in-turn/out-of-turn proposer to discourage racing.
- **Sealing**: no VRF/VRF randomness; deterministic signer rotation.
- **Finality**: practical finality after > half+1 signers build on a block; short reorg window.

## 3. Header Validation (summary)
- Parent existence; timestamp ≥ parent + period.
- ExtraData contains signer list on checkpoints; signature present and valid.
- Difficulty matches in-turn rule.
- Gas limit, size, base fee (if enabled) within bounds.

## 4. Block Import & Execution
- On receive: header pre-check → signature verify → execute txs via router (EVM/AVM) → compare state root/receipts → mark imported.
- Invalid blocks: peer penalty/ban; never re-propose.

## 5. Signer Management
- **Add/remove signer**: voting tx modifies signer set (majority).
- **Key storage**: use keystore; avoid sharing keystore across processes (datadir flock).
- **Rotation**: encourage operators to rotate keys and maintain HSM/secure enclave.

## 6. Liveness & Safety Considerations
- Minimum **N ≥ 3** signers; best with odd N.
- **In-turn** sealing reduces fork rate; out-of-turn allowed with higher difficulty.
- **Clock sync**: nodes should maintain NTP discipline to avoid timestamp drift.

## 7. Instrumentation
- Track sealing success, missed slots, fork rate, block propagation time.
- Alert on signer divergence (different signer sets), high uncle/fork counts.

## 8. References
- Clique rules (Ethereum heritage).
- Paxeer headers/validation in `consensus/` (implementation-specific).
