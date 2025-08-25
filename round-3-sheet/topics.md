

## 🔐 Core Web3 Security Topics

* **Smart contract vulnerabilities**
  * Reentrancy
  * Integer overflow/underflow
  * Access control flaws
  * Front-running / MEV
  * Logic errors in lending protocols (e.g., price manipulation)
* **DeFi protocol risks**
  * AMM math manipulation
  * Oracle manipulation (flash loans)
  * Stablecoin depeg scenarios
* **Cross-chain bridge security**
  * Lock-and-mint vs burn-and-release
  * Validator compromise risks
  * Replay / fake proof attacks
* **Wallet & key management**
  * Hot vs cold wallets
  * MPC (multi-party computation) and multisig
  * Risks of private key leaks, phishing, SIM-swap

---

## 🏗️ Blockchain / Web3 Fundamentals

* How a **transaction flows** (signing in wallet → mempool → block inclusion in PoS).
* Consensus mechanisms (PoW vs PoS; how validators secure BNB Chain / Ethereum).
* What wrapped assets are (e.g., WBTC, WETH).
* How swaps work (AMM vs order book; oracles vs pool math).
* Lending/borrowing (Aave, Compound) — collateral, LTV, liquidation.
* Role of oracles (Chainlink, TWAP) and risks.

---

## 🛡️ Security Processes & Defense

* **Smart contract auditing basics** (manual review, static analysis, fuzzing).
* Incident response in Web3 (what to do during an exploit).
* Bug bounty programs & responsible disclosure.
* Monitoring on-chain activity for suspicious transactions.
* Security best practices (checks-effects-interactions, OpenZeppelin libraries, unit testing).

---

---

## 🧨 Additional Attack Vectors

* **Wallet Drainer Scams**
  * Fake airdrop or NFT claim sites that trick users into signing `setApprovalForAll` or `permit()`.
  * Malicious transactions where approval looks harmless but grants attacker spending rights.
  * Detection: simulate tx before signing; tools like Fire/ScamSniffer track drainer kits.
  * Defense: educate users, wallet warnings, revoke approvals (`revoke.cash`).

* **NFT-specific Risks**
  * Fake marketplaces with spoofed signatures.
  * Replay attacks on off-chain signed orders.
  * Malicious `onERC721Received` implementations enabling reentrancy.

* **Sandwich / MEV Attacks**
  * Attacker front-runs user swap, then back-runs, extracting value.
  * Especially harmful in low-liquidity pools.
  * Defense: use private relays (Flashbots), slippage protection, MEV-aware routing.

---

## 🛠️ Tools & Automation

* **Static Analysis**: Slither, Mythril — catch unsafe patterns (`tx.origin`, uninitialized storage).
* **Fuzz Testing**: Echidna, Foundry fuzzing — randomized inputs to trigger edge cases.
* **Dynamic Debugging**: Tenderly, HardHat traces, Foundry `vm.recordLogs`.
* **Monitoring / Alerts**: Forta, Phalcon, custom Python scripts watching mempool.
* **Explorers**: Etherscan/BlockScout — follow exploit tx step by step.

---

## 🧾 Exploit Post-Mortem Skills

* **Structure of a write-up**
  * Vulnerability: what bug was exploited.
  * Exploit Flow: how attacker executed it (tx sequence).
  * Impact: funds lost, systemic risks.
  * Fix: what patch was applied.
  * Lessons: what to check in future audits.

* **Recent Incidents to Know**
  * Euler Finance (2023) – flash loan + liquidation logic bug.
  * Mango Markets (2022) – oracle price manipulation.
  * Hundred Finance (2023) – reentrancy on Compound fork.
  * Nomad (2022) – init bug → anyone could withdraw.

---

## 🌉 EVM & Layer 2 Security

* **Optimistic Rollups (Arbitrum, Optimism)**
  * Fraud proofs, 7-day withdrawal delay.
  * Sequencer centralization risk.
* **ZK Rollups (zkSync, StarkNet, Polygon zkEVM)**
  * Validity proofs, faster finality.
  * Risks: proof system bugs, circuit issues.
* **Bridge Risks**
  * Rollup bridges differ from lock-and-mint → message passing + fraud window.
  * Still high-value targets.

---

## 📊 Industry & Binance Context

* Awareness of **major DeFi exploits** (DAO hack, Wormhole, Ronin, Poly Network, Curve stablecoin issue).
* General understanding of Binance ecosystem: BNB Chain, bridges, DeFi products.
* Regulatory context (light touch): why transparency & proof-of-reserves matter.

---

## 🤝 Behavioral & Soft Skills

* Why you want to work in **Web3 security** at Binance.
* Example of a time you solved a **technical challenge** under pressure.
* How you **learn quickly** in a new area (important for interns).
* How you’d work in a **team with incomplete information**.
* Willingness to **admit what you don’t know** and learn fast.


