# Shielded Yield Vault

**Shielded Yield Vault** is a privacy-first, cross-chain yield aggregator built on Horizen that enables users to earn optimized yields while keeping individual positions completely private.

It combines:

* Confidential computation
* Cross-chain liquidity
* Automated yield optimization

# Technical Architecture

The protocol uses a **hybrid architecture** that balances:

* **Strong individual privacy**
* **Aggregate on-chain transparency**

---

## System Overview

```mermaid
flowchart TD

    U["User (Horizen L3)"]
    D["dApp Frontend"]

    subgraph Horizen_L3
        V["Shielded Vault<br/>Commitments + TVL"]
        S["Settlement Contract<br/>Verify + Execute"]
    end

    R["Relayer / Backend"]

    subgraph Private_Core
        TEE["Vela TEE<br/>Private balances<br/>Strategy + Withdraw batching<br/>Attestation"]
    end

    B["Stargate Bridge<br/>Aggregated transfers"]

    subgraph Base_L2
        Y["Yield Protocols<br/>Morpho / Aave / Pendle"]
    end

    U --> D --> V --> R --> TEE
    TEE --> S
    S --> B
    B --> Y

    %% Withdraw reverse flow
    Y --> B --> S --> U
```
---

## How It Works

### 1. Private Deposits (Horizen)

Users deposit assets into a **Shielded Vault Contract** on Horizen L3.

* Deposits are **not stored as public balances**
* Instead, they are represented as **cryptographic commitments (private notes)**
* Public chain only sees:

  * Total Value Locked (TVL)
  * Aggregate pool movements

---

### 2. Confidential Strategy Engine (Vela TEE)

At the core of the system is a **Vela Trusted Execution Environment (TEE)** powered by AWS Nitro Enclaves.

Inside the TEE:

* Private user balances are reconstructed from notes
* Market conditions are analyzed
* Funds are **dynamically allocated** across:

  * Morpho Blue
  * Aave V3
  * Pendle

Key guarantees:

* No raw user data leaves the enclave
* Strategy logic runs in **hardware-isolated execution**
* Outputs are **cryptographically attested**

---

### 3. Attested On-Chain Settlement

Only a **verified instruction** exits the TEE.

* The **Settlement Contract**:

  * Verifies the TEE attestation
  * Executes the requested action
  * Updates the public state root

This ensures:

* No unauthorized fund movement
* Full trust minimization of off-chain compute

---

### 4. Cross-Chain Liquidity Movement

Funds are bridged using **Stargate V2**:

* Only **aggregated pool funds** are moved
* Individual user allocations are never exposed
* Assets are transferred from:

  * Horizen L3 → Base L2 (while deposit)
  * Base L2 -> Horizen L3 (while withdraw)

---

### 5. Yield Generation on Base

On Base, funds are deployed into **audited DeFi protocols**:

* Morpho (efficient lending markets)
* Aave V3 (battle-tested liquidity)
* Pendle (yield tokenization strategies)

At the current stage of the ecosystem, Horizen offers very limited opportunities for sustainable on-chain yield generation.

* The DeFi landscape is still early and underdeveloped
* Capital efficiency, limited tokens support and liquidity depth are not yet competitive

#### Current Approach

To ensure users still access best-in-class yields, the protocol

Bridges aggregated liquidity to Base, where:
* Deep liquidity exists
* Proven yield strategies are available
* Generates yield externally
* Recalls funds back to Horizen during withdrawals
* Distributes funds privately to users

#### Forward-Looking Design

* More protocols launch
* Liquidity deepens
* Native yield opportunities emerge

We will
* Gradually introduce Horizen-native strategies

### 6. Withdrawals
* User submits withdraw request
* Request is sent to TEE via relayer

TEE:
- Verifies private note
- Queues request

Multiple requests are:
- Batched together
System:

- Unwinds liquidity on Base
- Bridges funds back to Horizen
- Settlement contract:
- Verifies TEE attestations
- Distributes funds to users


#### Full Lifecycle (Deposit + Yield + Withdraw)

```mermaid
flowchart LR

    U["User"]

    V["Vault (Horizen)<br/>Private deposits"]

    B1["Bridge → Base"]

    Y["Yield Strategies<br/>Morpho / Aave / Pendle"]

    B2["Bridge → Horizen"]

    S["Settlement<br/>Distribute funds"]

    U --> V --> B1 --> Y --> B2 --> S --> U
```

---

## Withdraw Flow

```mermaid
sequenceDiagram
    participant U as User
    participant D as dApp
    participant R as Relayer
    participant T as Vela TEE
    participant Y as Base Protocols
    participant B as Bridge
    participant S as Settlement

    U->>D: Request withdraw
    D->>R: Send withdraw intent
    R->>T: Forward request

    T->>T: Verify private balance
    T->>T: Queue withdrawal
    T->>T: Batch multiple users

    T->>Y: Trigger liquidity unwind

    Y-->>B: Send funds to bridge
    B-->>S: Bridge to Horizen

    T->>T: Generate attestations

    R->>S: Submit batch proof

    S->>S: Verify
    S-->>U: Transfer funds (stealth)
```

---

# Privacy Model

Shielded Yield Vault ensures:

### What is Private

* Individual deposits
* User balances
* Yield earned per user
* Withdrawal amounts
* Transaction linkability

### What is Public

* Total pool TVL
* Strategy allocations (aggregate)
* Bridge transactions
* Performance metrics

This creates a **privacy-preserving yet auditable system**.

---

# Transparency for Compliance & Trust

The protocol is designed to be **compliance-friendly without sacrificing privacy**.

Anyone can verify on-chain:

* Funds are only deployed into **reputable, audited protocols**
* No exposure to:

  * High-risk strategies
  * Unauthorized destinations

This enables:

* Independent auditing
* Community verification
* Institutional confidence