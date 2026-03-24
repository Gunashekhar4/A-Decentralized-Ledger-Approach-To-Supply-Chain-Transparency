# A Decentralized Ledger Approach to Supply Chain Transparency

A fully functional Ethereum-based decentralized application (DApp) that replaces centralized supply chain record-keeping with an immutable, tamper-proof blockchain ledger. Every product movement — from raw material supply through to final sale — is recorded as an on-chain transaction, giving all verified participants cryptographically secured, independently verifiable provenance data.

---

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [System Architecture](#system-architecture)
- [Features](#features)
- [Supply Chain Flow](#supply-chain-flow)
- [Smart Contract Algorithms](#smart-contract-algorithms)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [1. Clone & Install](#1-clone--install)
  - [2. Configure Ganache](#2-configure-ganache)
  - [3. Compile & Deploy Contracts](#3-compile--deploy-contracts)
  - [4. Configure MetaMask](#4-configure-metamask)
  - [5. Register Participants](#5-register-participants)
  - [6. Run the Frontend](#6-run-the-frontend)
- [Running Tests](#running-tests)
- [Account Mapping](#account-mapping)
- [Future Roadmap](#future-roadmap)

---

## Overview

Traditional supply chains rely on centralized databases, paper records, and intermediaries — making them vulnerable to data tampering, disputes, and slow reconciliation. This project addresses those gaps by anchoring every supply chain event onto the Ethereum blockchain via Solidity smart contracts.

**Key properties guaranteed by the system:**
- No single party can alter historical records
- Role-based access control is enforced at the protocol level — not the application layer
- Every transaction is time-stamped, linked to a verified Ethereum address, and permanently visible to all registered participants
- Product history is queryable by any registered stakeholder at any time, at zero gas cost

---

## Tech Stack

| Layer | Technology |
|---|---|
| Smart Contracts | Solidity v0.8.x |
| Blockchain (local) | Ganache |
| Development Framework | Truffle Suite |
| Frontend | React.js |
| Blockchain Communication | Web3.js |
| Wallet & Auth | MetaMask |
| Runtime | Node.js v14+ |

---

## System Architecture

The system is organized into four distinct layers:

```
┌─────────────────────────────────────────────┐
│              ACTORS                         │
│  Owner | Supplier | Manufacturer | ...      │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│         WEB APPLICATION LAYER               │
│  React.js / HTML / JavaScript               │
│  (Role-aware UI — shows only permitted ops) │
└────────────────┬────────────────────────────┘
                 │ API Calls
┌────────────────▼────────────────────────────┐
│          INTEGRATION LAYER                  │
│  Web3.js — Blockchain Interaction &         │
│            Transaction Management           │
└──────────┬──────────────────────────────────┘
           │
┌──────────▼──────────────────────────────────┐
│           WALLET LAYER                      │
│  MetaMask — User Authentication &           │
│             Transaction Signing             │
└──────────┬──────────────────────────────────┘
           │
┌──────────▼──────────────────────────────────┐
│         BLOCKCHAIN LAYER                    │
│  Ethereum / Ganache Network                 │
│  SupplyChain.sol (Solidity Smart Contract)  │
│  • Participant Registration                 │
│  • Goods Registration                       │
│  • Ownership Transfer                       │
│  • Supply Chain Tracking                    │
└─────────────────────────────────────────────┘
```

---

## Features

### Participant Registration with Ethereum Identity
All six supply chain roles register using their MetaMask-connected Ethereum wallet address. The contract Owner verifies and assigns each address to exactly one role before that participant can perform any on-chain operation. Identity is secured by Ethereum's public key infrastructure — no passwords, no external identity provider.

### Raw Material Supply on Chain
The Raw Material Supplier records the supply of raw materials as the very first on-chain transaction. The smart contract enforces that only a verified Raw Material Supplier can trigger this stage-one transition, creating a tamper-proof record of goods origin before manufacturing begins.

### Immutable Goods Registration
A verified Manufacturer creates new product entries on-chain, each assigned a unique sequential integer ID. The registration transaction is permanently stored on Ethereum, forming the anchor for all subsequent custody events.

### Ownership Transfer on Chain
Every custody handoff between participants is recorded as a separate blockchain transaction. The contract enforces that only the current custodian of the correct role can transfer to the next authorized party — skipping stages or unauthorized transfers are rejected at the EVM level.

### End-to-End Supply Chain Tracking
Any registered participant can query the complete, ordered chain of custody for any product at any time. The smart contract returns every stage transition — including the Ethereum address, role, and block timestamp of each handler.

### React.js Dashboard with Live Blockchain Data
The frontend reads live on-chain state via Web3.js and renders only the operations permitted for the connected MetaMask account's role. All displayed data reflects true on-chain state in real time via event-driven updates.

### MetaMask Wallet Integration
Transaction signing happens entirely within MetaMask. Private keys never leave the browser extension — the application layer only receives the signed transaction.

### Local Development with Ganache & Truffle
Ganache provides a simulated Ethereum network with pre-funded accounts for full local testing. Truffle manages contract compilation, migration, and the Mocha/Chai test suite.

---

## Supply Chain Flow

```
Goods Order
    │
    ▼
Raw Material Supplier  ──→  Manufacturer  ──→  Distributor  ──→  Retailer  ──→  Consumer
(supplyRawMaterial)       (manufactureGoods)  (distributeGoods)  (retailGoods)   (purchase)
```

Each arrow represents a distinct on-chain transaction. No stage can be skipped, and no stage can be reversed once committed.

---

## Smart Contract Algorithms

### Algorithm 1 — Role-Based Access Control (RBAC)
Every Ethereum address maps to exactly one role: `DEFAULT`, `RAW_MATERIAL_SUPPLIER`, `MANUFACTURER`, `DISTRIBUTOR`, `RETAILER`, or `CUSTOMER`. Solidity modifiers check the caller's role at the top of every state-changing function — any mismatch reverts the transaction immediately, consuming only the gas used by the check.

### Algorithm 2 — Product Lifecycle State Machine
Each product moves through five sequential states: `RAW_MATERIAL_SUPPLIED → MANUFACTURE → DISTRIBUTE → RETAIL → SOLD`. The contract enforces strict ordering — out-of-order transitions are rejected by a `require()` check. The state and ownership update happen atomically in a single transaction.

### Algorithm 3 — Ownership Transfer Logic
`transferOwnership()` performs four checks before executing: (a) product exists, (b) caller is the current owner, (c) caller's role matches the current product state, (d) recipient is a registered participant of the correct receiving role. All four must pass or the entire transaction reverts.

### Algorithm 4 — Goods Registration
`addItem()` checks the `MANUFACTURER` role, increments `productCount`, creates the `Product` struct with state `MANUFACTURE`, and emits a `SupplyChainStep` event. IDs are sequential integers starting from 1 — gas-efficient and collision-free without requiring hashes or UUIDs.

### Algorithm 5 — On-Chain Event Logging & History Query
Every state transition emits `SupplyChainStep(uint indexed productId, address indexed owner, State state)`. Events are stored in transaction logs (not contract storage), making them dramatically cheaper than mappings and queryable off-chain via Web3.js `getPastEvents()` at zero gas cost.

---

## Project Structure

```
A-Decentralized-Ledger-Approach-To-Supply-Chain-Transparency/
│
├── contracts/
│   ├── Migrations.sol          # Truffle migration tracking contract
│   └── SupplyChain.sol         # Main supply chain smart contract
│
├── migrations/
│   ├── 1_initial_migration.js  # Deploys Migrations.sol
│   └── 2_deploy_contracts.js   # Deploys SupplyChain.sol
│
├── test/
│   └── supplyChain.test.js     # Mocha/Chai test suite (24 test cases)
│
├── src/
│   ├── abis/
│   │   └── SupplyChain.json    # ABI + deployed address (auto-generated)
│   │
│   └── components/
│       ├── App.js              # Root React component
│       ├── Navbar.js           # Navigation bar
│       └── Main.js             # Role-aware main view
│   ├── index.js                # React entry point
│   └── index.css               # Global styles
│
├── truffle-config.js           # Truffle network + compiler config
└── package.json                # npm dependencies
```

---

## Getting Started

### Prerequisites

- **Node.js** v14.x or higher → [nodejs.org](https://nodejs.org)
- **Truffle** (global install)
- **Ganache Desktop** → [trufflesuite.com/ganache](https://trufflesuite.com/ganache/)
- **MetaMask** browser extension → [metamask.io](https://metamask.io) (Chrome or Firefox)

### 1. Clone & Install

```bash
git clone https://github.com/Gunashekhar4/A-Decentralized-Ledger-Approach-To-Supply-Chain-Transparency.git
cd A-Decentralized-Ledger-Approach-To-Supply-Chain-Transparency
npm install
npm install -g truffle
```

### 2. Configure Ganache

1. Open **Ganache Desktop** and click **New Workspace**
2. Ganache provisions 10 accounts, each pre-loaded with 100 ETH
3. Default RPC endpoint: `http://127.0.0.1:7545` — Network ID: `5777`
4. Keep Ganache running throughout development

The 10 accounts map to roles as follows (see [Account Mapping](#account-mapping)).

### 3. Compile & Deploy Contracts

```bash
# Compile all Solidity contracts
truffle compile

# Deploy to the local Ganache network
truffle migrate --reset --network development
```

After migration, Truffle writes the contract ABI and deployed address to `src/abis/SupplyChain.json` — the frontend reads from this file automatically.

### 4. Configure MetaMask

1. Open MetaMask → **Add Network** → fill in:
   - Network Name: `Ganache Local`
   - RPC URL: `http://127.0.0.1:7545`
   - Chain ID: `1337`
   - Currency Symbol: `ETH`

2. In Ganache, copy each account's **private key** (click the key icon)

3. In MetaMask → **Import Account** → paste the private key, then label each account:

| MetaMask Label | Ganache Account | Role |
|---|---|---|
| Owner | account[0] | Contract deployer / admin |
| Supplier | account[1] | Raw Material Supplier |
| Manufacturer | account[2] | Manufacturer |
| Distributor | account[3] | Distributor |
| Retailer | account[4] | Retailer |
| Consumer | account[5] | Consumer |

4. When performing role-specific operations, **switch to the corresponding MetaMask account** before submitting the transaction.

### 5. Register Participants

Before any supply chain operations can occur, the Owner must register each participant on-chain:

1. Open the DApp with MetaMask connected to the **Owner** account
2. Navigate to **Register** (Step 1 on the home page)
3. For each participant, enter their Ethereum address (from Ganache) and select their role
4. Submit — MetaMask will prompt for gas approval
5. Repeat for all five stakeholder roles

The contract calls `addRawMaterialSupplier()`, `addManufacturer()`, `addDistributor()`, `addRetailer()`, and `addConsumer()` — each stores the address-to-role mapping on-chain.

### 6. Run the Frontend

```bash
npm start
# Opens at http://localhost:3000
# Ensure MetaMask is connected to Ganache (RPC: http://127.0.0.1:7545)
```

---

## Running Tests

```bash
truffle test
```

The test suite contains **24 test cases** across three categories:

| Category | Tests | What is verified |
|---|---|---|
| Registration & Access Control | TC-01 to TC-08 | Owner-only role assignment, DEFAULT role for unregistered addresses, re-registration overwrites |
| Product Lifecycle & Stage Transitions | TC-09 to TC-16 | Goods registration, stage enforcement, ownership transfer, skip-stage rejection, post-sale lock |
| Integration & End-to-End Workflow | TC-17 to TC-24 | Full six-stage lifecycle, 5-event history, multiple products, MetaMask rejection handling, product tracker ordering, Ganache log verification |

All 24 tests pass on a clean Ganache state.

---

## Account Mapping

```
account[0]  →  Owner              (deploys contract, registers all participants)
account[1]  →  Raw Material Supplier
account[2]  →  Manufacturer
account[3]  →  Distributor
account[4]  →  Retailer
account[5]  →  Consumer
account[6–9]   available for additional testing
```

---

## Future Roadmap

- **Ethereum Testnet Deployment** — Deploy to Sepolia or Goerli by updating the RPC endpoint in `truffle-config.js`. No contract changes required.
- **IoT Sensor Integration** — Connect temperature loggers, GPS trackers, and tamper-detection seals via a Chainlink oracle to write environmental data on-chain alongside ownership transfers.
- **IPFS Document Storage** — Store certificates of origin, bills of lading, and quality inspection reports on IPFS with content hashes anchored on-chain for tamper-proof document verification.
- **ERC-721 NFT Product Tokenization** — Represent each product as a non-fungible token for standardized, marketplace-compatible on-chain product identity.
- **Layer 2 Scaling** — Migrate to Polygon, Optimism, or Arbitrum to reduce gas costs while retaining Ethereum's security guarantees. Current Solidity code is fully compatible without modification.
- **Multi-Signature Custody** — Require approval signatures from both sender and receiver before a transfer is committed, more accurately modelling real-world delivery confirmation protocols.
- **Hybrid AI-Blockchain Analytics** — Add an analytics layer that monitors on-chain event patterns to flag anomalies, unusual stage timing, or potential fraud in real time.
