# Galois

A Layer 1 blockchain with deterministic execution through **Nix**-based reproducible builds and pure functional smart contracts.

## Motivation

Existing blockchains face a fundamental tradeoff between deterministic consensus and implementation resilience. Ethereum achieves fault tolerance through client diversity (Geth, Erigon, Nethermind) but sacrifices determinism—different implementations occasionally produce divergent results. Single-implementation chains guarantee determinism but introduce monoculture risk where a single bug affects all validators.

Galois resolves this through **verification diversity**: all validators execute identical binaries but validate correctness using different mathematical methods.

## Architecture

### Reproducible Execution Layer

- **Nix builds**: Every validator runs bit-identical binaries, eliminating version drift and upgrade inconsistencies
- **Pure functional contracts**: Smart contracts are mathematical functions with no side effects, enabling parallel execution and formal verification
- **Deterministic state transitions**: Given identical inputs, all validators produce identical outputs without interpretation ambiguity

### Verification Diversity

While execution is uniform, validators employ distinct verification strategies:

- zkSNARK-based proofs
- Optimistic verification with fraud proofs  
- Symbolic execution
- Interactive verification protocols

Economic incentives reward validators for using diverse verification methods. If one verification approach fails to detect a bug, others provide redundancy.

## Smart Contracts

Contracts are expressed as pure functions: `f(state, input) → (new_state, output)`. This paradigm provides:

| Property | Benefit |
|----------|---------|
| **No global state mutation** | Transactions are order-independent and parallelizable |
| **Mathematical provability** | Contract behavior can be formally verified |
| **Deterministic reconstruction** | State can be recomputed from minimal transaction data |

Example signature:
```
transfer(from: Address, to: Address, amount: u64) → Result<State, Error>
```

## Consensus Mechanism

Galois implements **Proof of Stake** with reproducibility verification. When a validator proposes a block, they include both the block content and the Nix expression that generated it. Other validators execute this expression; if the output matches bit-for-bit, the block is valid.

Finality is immediate upon reproduction verification. Malicious or faulty validators are identified when their outputs cannot be reproduced and are slashed accordingly.

### Security Model

- Cryptographic agility through multiple hash functions
- Fraud proof propagation for incorrect erasure coding or execution
- Slashing for non-reproducible block proposals
- Defense in depth: monoculture at execution layer, diversity at verification layer

## Data Availability

Pure functional paradigm makes state reconstruction deterministic. Light clients need only transaction inputs and function definitions rather than full state. This eliminates the need for separate DA layers like Celestia—the protocol provides inherent DA guarantees through reproducibility.

## Comparison

| Feature | Ethereum | Modular Chains | Galois |
|---------|----------|----------------|--------|
| Client diversity | Multiple implementations | Varies by layer | Single execution, diverse verification |
| Smart contracts | Imperative, stateful | Varies | Pure functional |
| Throughput | Requires L2s | High via modularity | High via parallelization |
| DA guarantees | On-chain | External (e.g., Celestia) | Native reproducibility |
| Consensus | PoS + client diversity | Various | PoS + reproducibility |

## Implementation Status

Galois is under active development. Core components include validator software in Nix, functional contract runtime, and consensus protocol specification.

---

*Galois: Named after Évariste Galois, whose work on group theory and field equations established foundations for deterministic mathematical structures.*
