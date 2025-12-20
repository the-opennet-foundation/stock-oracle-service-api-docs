---
title: "Paxeer Network Documentation"
description: "Build on a high-performance, developer-friendly blockchain platform for the next generation of decentralized applications"
---

> **Note:** Chain ID: 229 | Currency: PAX | Version: v1.0.0

## Welcome to Paxeer Network

A high-performance, developer-friendly blockchain platform for the next generation of decentralized applications. Paxeer Network is fully EVM-compatible, making it easy to deploy your existing Ethereum smart contracts and dApps.

- **Quick Start:** Get up and running in minutes with our step-by-step guide — [/quickstart](/quickstart)
- **Smart Contracts:** Deploy and interact with contracts using Hardhat, Foundry, or Remix — [/contracts](/contracts)
- **API Reference:** Explore our comprehensive JSON-RPC API documentation — [/api-reference](/api-reference)
- **Network Configuration:** Configure Paxeer Network with wagmi, viem, ethers.js, or web3.js — [/configuration](/configuration)

## Why Build on Paxeer?

- **High Performance:** Low-latency transactions with high throughput for your dApps
- **Secure:** Built with security in mind from the ground up
- **Developer Friendly:** Comprehensive documentation and developer tools

## Revolutionary Core Infrastructure

> ⚠️ **Pioneering Independence:** Paxeer Network is building the **first EVM L2 to transition into EVM-compatibility without EVM-dependency**. We maintain full compatibility while creating a fully self-sustainable ecosystem.

Paxeer Network is built on our own native infrastructure, designed for independence and performance:

- **ArgusVM (AVM):** Register-based virtual machine with 256-bit native arithmetic. EVM-compatible but not EVM-dependent—the foundation of our self-sustainable ecosystem. — [/argus-vm](/argus-vm)
- **OPAX-28 Token Standard:** Native fungible token standard optimized for ArgusVM and OpenLang. Built-in safety, better performance, and Rust-inspired syntax. — [/opax-28](/opax-28)

### EVM-Compatible but Not EVM-Dependent

We maintain full compatibility with existing Ethereum tooling and contracts while building our own execution environment:

- ✅ Execute existing Solidity contracts (via translation layer)
- ✅ Support all Ethereum wallets and tools
- ✅ Maintain bridge compatibility
- ✅ **No dependency on EVM infrastructure**
- ✅ Full control over VM evolution

### Self-Sustainable Ecosystem

Build a complete ecosystem without external dependencies:

- **ArgusVM:** Native register-based execution environment
- **OpenLang:** Rust-inspired smart contract language
- **OPAX-28:** Native token standard
- **Custom gas model:** Optimized for our architecture
- **Independent evolution:** No reliance on Ethereum upgrades

### Performance Benefits

Our native infrastructure provides significant advantages:

- **10,000+ TPS** per core (vs ~15 TPS for EVM)
- **<1ms latency** for simple contract calls
- **Register-based architecture** eliminates stack overhead
- **Optimized gas costs** for common operations

## Network Details

Connect to Paxeer Network with these details:

```json Network Configuration
{
  "networkName": "Paxeer Network",
  "rpcUrl": "https://public-rpc.paxeer.app/rpc",
  "chainId": 229,
  "currencySymbol": "PAX",
  "blockExplorer": "https://paxscan.paxeer.app"
}
```

- **Add to MetaMask:** Follow our guide to add Paxeer Network to your wallet — [/quickstart#add-paxeer-network](/quickstart#add-paxeer-network)

## Ecosystem Protocols

Explore the powerful protocols built on Paxeer Network:

- **PaxDex Protocol:** Decentralized exchange with 0.3% swap fees and real-time price feeds — [/paxdex](/paxdex)
- **Lending Protocol:** Lend and borrow assets with dynamic APY and credit scoring — [/lending](/lending)
- **ChainFlow:** Decentralized prop trading platform with on-chain evaluation and automated profit distribution — [/chainflow](/chainflow)
- **Computable Token Machine (CTM):** Revolutionary token standard where tokens are execution environments — [/ctm](/ctm)
- **BlockScout API:** Query transactions, blocks, addresses, and tokens — [/blockscout-api](/blockscout-api)

## Developer Resources

- **RPC Tester:** Test Paxeer Network RPC methods in real-time — [/rpc](/rpc)
- **Code Examples:** Integration guides and code samples — [/examples](/examples)
- **SDKs & Tools:** Recommended tools and libraries for development — [/tools](/tools)
- **API Explorer:** Interactive API testing interface — [/api-explorer](/api-explorer)

## Need Help?

- **Join Our Community:** Get support and connect with other developers building on Paxeer Network — <https://paxeer.app/community>
