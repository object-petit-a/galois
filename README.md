# Galois

A Layer 1 blockchain with deterministic execution through Nix-based reproducible builds and pure functional smart contracts.

## Motivation

Existing blockchains face a fundamental tradeoff between deterministic consensus and implementation resilience. Ethereum achieves fault tolerance through client diversity (Geth, Erigon, Nethermind) but sacrifices determinism—different implementations occasionally produce divergent results. Single-implementation chains guarantee determinism but introduce monoculture risk where a single bug affects all validators.

Galois resolves this through **verification diversity**: all validators execute identical binaries but validate correctness using different mathematical methods.

## Architecture

### Reproducible Execution Layer

- **Nix builds**: Every validator runs bit-identical binaries, eliminating version drift and upgrade inconsistencies
- **Pure functional contracts**: Smart contracts are mathematical functions with no side effects, enabling parallel execution and formal verification
- **Deterministic state transitions**: Given identical inputs, all validators produce identical outputs without interpretation ambiguity

### State Commitment

Global state is organized in a **Merkle tree** structure. Each block commits to a state root, allowing:

- Light clients to verify specific state elements via Merkle proofs
- State snapshots for efficient synchronization
- Incremental state updates without full replay

### Verification Diversity

All validators execute the same Nix binary and produce identical outputs. However, they generate **different mathematical proofs** that the execution is correct:

- **zkSNARK proofs**: Succinct cryptographic proof of correct execution
- **Optimistic verification**: Assume correctness unless fraud proof submitted
- **Interactive proofs**: Challenge-response protocol for execution verification
- **Symbolic execution traces**: Step-by-step execution path validation

Validators using diverse proof methods receive bonus rewards. If a bug exists in the execution binary, different proof systems have different failure modes—increasing the probability of detection.

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

## Consensus Mechanism: Proof of Reproducibility

### Block Production

1. **Proposer selection**: Validators stake minimal collateral for Sybil resistance. Block proposer is selected via VRF (Verifiable Random Function) based on stake weight
2. **Block proposal**: Proposer broadcasts block containing transactions, resulting state root, and Nix expression for reproducibility
3. **Reproduction phase**: Validator subset (quorum of 2f+1 where f is Byzantine fault tolerance threshold) independently executes the Nix expression
4. **Verification**: Each validator generates a proof using their chosen verification method
5. **Finality**: Block achieves finality when quorum reproduces identical state root and submits valid proofs

### Security Model

- **Byzantine fault tolerance**: System tolerates up to f malicious validators in quorum of 3f+1
- **Immediate finality**: No probabilistic finality—blocks are final upon quorum reproduction
- **Slashing conditions**: Validators are slashed for proposing non-reproducible blocks or submitting invalid proofs
- **Cryptographic agility**: Multiple hash functions prevent single-point cryptographic failure

### Economic Model

- **Staking**: Minimal stake required for validator participation (Sybil prevention)
- **Rewards**: Block proposers receive transaction fees; all validators in quorum receive inflationary rewards
- **Verification diversity bonus**: Validators using statistically underrepresented proof methods receive 20% bonus
- **Slashing**: 5% stake for invalid reproduction, 10% for provably malicious block proposal

## Data Availability

Pure functional paradigm enables deterministic state reconstruction. Light clients maintain only:

- Current state root (32 bytes)
- Block headers (consensus metadata)

For specific state queries:
1. Light client requests state element and Merkle proof from full node
2. Full node provides data + proof path to state root
3. Light client verifies proof against trusted state root

Full synchronization uses incremental snapshots: light client downloads latest state snapshot (cryptographically verified) then replays blocks from snapshot height. No separate DA layer required—reproducibility guarantees data availability.

## Comparison

| Feature | Ethereum | Modular Chains | Galois |
|---------|----------|----------------|--------|
| Client diversity | Multiple implementations | Varies by layer | Single execution, diverse verification |
| Smart contracts | Imperative, stateful | Varies | Pure functional |
| Consensus | PoS + LMD GHOST | Various | Proof of Reproducibility |
| Finality | Probabilistic (~15min) | Varies | Immediate upon quorum |
| Throughput | ~15 TPS (L1) | High via modularity | High via parallelization |
| DA guarantees | On-chain | External (e.g., Celestia) | Native via reproducibility |
| State verification | Full execution | Varies | Merkle proofs + reproduction |

## Implementation Status

Galois is under active development. Core components include validator software in Nix, functional contract runtime, and consensus protocol specification.

---

*Galois: Named after Évariste Galois, whose work on group theory and field equations established foundations for deterministic mathematical structures.*

