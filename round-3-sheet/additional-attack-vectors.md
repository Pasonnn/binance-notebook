

# 🧨 Additional Attack Vectors (Deep Dive)

---

## 🔑 1. Wallet Drainer Scams

### 🕵️ What it is

* Attackers trick users into signing malicious approvals, giving away control of tokens or NFTs.
* Common disguise: fake airdrop sites, fake NFT mints, phishing DApps.

---

### ⚡ Exploit Flow

1. Victim connects wallet to scam DApp.
2. Site presents a transaction that looks harmless (“Claim reward,” “Mint NFT”).
3. Underneath, it’s a **`setApprovalForAll`** (for NFTs) or **ERC20 `permit()`** (EIP-2612).

   * Grants attacker **infinite spending approval** for victim’s assets.
4. Attacker then calls `transferFrom()` directly to drain tokens/NFTs.

---

### 📦 Real Proof

* **Monkey Drainer (2022)**: fake NFT mint sites tricked users, drained \~\$1M+ in ETH + NFTs.
* **Inferno Drainer (2023)**: “scam-as-a-service” kit, clients paid 20% fee. Stole \$40M+ across chains.

---

### 🛡️ Defense

* **Simulation before signing**: Wallets like Rabby show “This tx will give unlimited approval of USDT to attacker.”
* **Education**: Teach users to double-check tx data.
* **Revocation**: If compromised, revoke token approvals via **revoke.cash** or Etherscan’s “token approvals” page.
* **Wallet warnings**: MetaMask now flags suspicious approvals.

---

### ✅ Interview phrasing

> “Wallet drainers don’t steal funds directly — they trick users into granting malicious approvals. Once granted, attackers use `transferFrom` to drain tokens. Defenses include simulated tx previews, wallet warnings, and tools like revoke.cash to revoke dangerous approvals.”

---

## 🔑 2. NFT-Specific Risks

### 🕵️ What it is

NFTs introduce unique attack vectors due to **signatures, marketplaces, and special callbacks**.

---

### ⚡ Exploit Flows

**1. Fake marketplaces (spoofed signatures)**

* User signs an **off-chain order** (e.g., “Sell NFT for 10 ETH”).
* Fake site modifies terms → attacker replays signature on real marketplace (OpenSea, Blur).
* Victim unintentionally sells for 0.1 ETH instead of 10 ETH.

**2. Replay attacks on off-chain orders**

* Off-chain signed listings can be replayed if not canceled.
* Example: Victim lists NFT for 1 ETH → cancels on one marketplace.
* Signature still valid elsewhere → attacker executes on another platform.

**3. Malicious `onERC721Received` (reentrancy)**

* Contracts that receive NFTs can execute **reentrant callbacks**.
* Example: Victim deposits NFT into vault contract. Malicious receiver contract re-enters withdraw logic to steal multiple NFTs.

---

### 📦 Real Proof

* **OpenSea 2022 replay attack**: old signed orders reused, \~\$1.7M in NFTs stolen.
* **Fake marketplaces**: phishing sites imitating Blur/LooksRare, draining BAYC, Azuki, etc.

---

### 🛡️ Defense

* Sign only on trusted marketplaces.
* Marketplaces should use **nonces** + **EIP-712 typed data** to prevent replay.
* Validate ERC721 receiver contracts.
* Wallets should flag risky signatures (e.g., OpenSea order signing).

---

### ✅ Interview phrasing

> “NFTs add new risks: replay attacks on off-chain orders, fake marketplace signatures, and malicious receiver contracts that exploit reentrancy. Strong nonce management, typed data signing, and user education are key defenses.”

---

## 🔑 3. Sandwich / MEV Attacks

### 🕵️ What it is

* Attacker manipulates a user’s swap by **placing tx before and after** it in the same block.
* Common in AMM-based DEXs (Uniswap, Sushi).

---

### ⚡ Exploit Flow (Sandwich Attack)

1. Victim submits swap: 100 ETH → DAI on Uniswap.
2. Attacker sees it in mempool.
3. Attacker submits:

   * **Front-run**: buys DAI first → pushes price up.
   * Victim’s swap executes next → buys DAI at worse price (high slippage).
   * **Back-run**: sells DAI right after → back to original price.
4. Attacker keeps risk-free profit.

---

### 📦 Real Proof

* Research shows **hundreds of millions** lost yearly to sandwich attacks.
* Example: ETH whale tried swapping on Uniswap, bots sandwiched tx, costing thousands in slippage.

---

### 🛡️ Defense

* **Private relays (Flashbots Protect)**: hide tx from public mempool → bots can’t see it.
* **Set slippage tolerance**: low slippage = attacker profits less.
* **MEV-aware routers**: 1inch, CowSwap → use batch auctions or private orderflow.

---

### ✅ Interview phrasing

> “Sandwich attacks exploit public mempool by bracketing a user’s trade. They cause the victim to suffer slippage, while attacker earns guaranteed profit. The best defenses are using private relays like Flashbots, slippage limits, or MEV-aware DEX routers.”

---

# 📊 Summary

* **Wallet drainers** = phishing approvals → drain tokens.
* **NFT risks** = signature replay, fake marketplaces, reentrancy in receivers.
* **Sandwich attacks (MEV)** = mempool manipulation → extract slippage profit.

👉 All of these highlight the theme: **public mempools and weak UX = huge attack surface.**

---

