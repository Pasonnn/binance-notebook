

# 🌉 Crypto Bridges

---

## 1. Definition

A **crypto bridge** (also called a **blockchain bridge** or **cross-chain bridge**) is a protocol that allows users to transfer digital assets or data from one blockchain to another.

Since most blockchains are isolated (BTC, Ethereum, Solana, etc.), bridges are essential for cross-chain interoperability.

---

## 2. How Bridges Work

### 🔒 Lock-and-Mint Model (most common)

* **Step 1: Deposit on Source Chain**

  * You send your BTC to the bridge’s custodian wallet.
  * Custodian can be:

    * Centralized entity
    * Federation of signers (multi-sig)
    * Decentralized validator set (MPC or consensus).
  * Your BTC is **locked** and cannot move.

* **Step 2: Verification**

  * The bridge monitors the Bitcoin blockchain.
  * Depending on design:

    * **Centralized bridge:** one custodian confirms the deposit.
    * **Decentralized bridge:** multiple validators sign a proof of deposit.

* **Step 3: Mint on Target Chain**

  * After verification, the bridge **mints a wrapped token** on the destination chain (e.g., 1 WBTC on Ethereum).
  * The WBTC is sent to your Ethereum wallet.

---

### 🔥 Burn-and-Release Model (redeeming back)

* **Step 1: Burn on Target Chain**

  * You burn (destroy) your wrapped token (e.g., 1 WBTC) on Ethereum.
* **Step 2: Bridge Verification**

  * The bridge monitors Ethereum and sees the burn event.
* **Step 3: Release on Source Chain**

  * The original BTC locked in custody is released back to your Bitcoin wallet.

---

## 3. Why Bridges Are Risky 🚨

Bridges are among the **biggest honeypots in crypto** because they often hold **hundreds of millions in locked assets**.

### Main Risks

* **Custodian compromise:** If attackers get the custodian/validator keys, they can unlock or mint tokens without deposits.
* **Verification flaws:** Bugs in the proof/verification logic can let attackers mint unlimited wrapped tokens (“infinite money glitch”).
* **Smart contract exploits:** Vulnerabilities in bridge contracts can allow attackers to bypass checks.

---

## 4. Famous Bridge Hacks (Case Studies)

* **Wormhole (2022):**

  * Exploit: Bypassed signature verification.
  * Loss: \~\$325M.

* **Ronin Bridge (Axie Infinity, 2022):**

  * Exploit: Private validator keys stolen (social engineering).
  * Loss: \~\$600M.

* **Poly Network (2021):**

  * Exploit: Smart contract bug allowed attacker to take control of funds.
  * Loss: \~\$611M (later returned).

💡 Lesson: Bridges fail not because the concept is bad, but because they are **extremely complex** systems combining **cryptography, smart contracts, and off-chain validation** — any weak link = total collapse.

---

## 5. Summary

* A **bridge = lock on Chain A + mint representation on Chain B**.
* Works via **custodians + validators + smart contracts**.
* **Huge systemic risk:** If compromised, attackers can mint fake tokens or drain funds.
* **Alternatives:** Some projects are exploring safer solutions like native cross-chain messaging (e.g., Cosmos IBC, LayerZero).

Great question 🙌 — digging into **real-world bridge hacks** is one of the best ways to understand *why* these systems are risky. I’ll break down **Wormhole, Ronin, and Poly Network** in detail:

---

# 🔓 Case Studies: Famous Bridge Hacks

---

## 1. 🪱 Wormhole Hack (February 2022)

* **Bridge type:** Solana ↔ Ethereum
* **What happened:**

  * Wormhole used a **guardian network** (validators) to verify cross-chain deposits.
  * The attacker found a bug in the **Solana contract’s signature verification**.
  * They managed to **forge a message** that looked valid, without needing the required guardian signatures.
* **Impact:**

  * Attacker minted **120,000 wrapped ETH (wETH)** on Solana **without depositing real ETH**.
  * Equivalent to \~\$325M at the time.
* **Aftermath:**

  * Jump Crypto (backer of Wormhole) **bailed out the protocol** by depositing their own ETH to cover the hole.
* **Lesson:**

  * Even if the system looks decentralized, a *single logic flaw* in verification = total compromise.

---

## 2. 🐉 Ronin Bridge Hack (March 2022)

* **Bridge type:** Ethereum ↔ Ronin (sidechain used by Axie Infinity)
* **What happened:**

  * Ronin bridge used a **9-validator model**.
  * Only **5 of 9 signatures** were required to approve transactions.
  * The attacker (later attributed to the **North Korea Lazarus Group**) managed to compromise **4 validators owned by Sky Mavis** + 1 validator from a third party.
  * With 5 signatures, they had **full control of the bridge**.
* **Impact:**

  * Attacker drained **173,600 ETH + 25.5M USDC** (\~\$600M).
  * Biggest crypto hack in history (as of 2022).
* **Aftermath:**

  * Axie paused the bridge.
  * Partially reimbursed users via a fundraising round and Binance loan.
* **Lesson:**

  * Over-centralization (few validators, many controlled by one entity) is dangerous.
  * Bridges are only as secure as their validator set.

---

## 3. 🌐 Poly Network Hack (August 2021)

* **Bridge type:** Multi-chain (Ethereum, Binance Smart Chain, Polygon, etc.)
* **What happened:**

  * Poly Network’s contracts had a **flaw in access control**.
  * The attacker found a way to **call the contract’s function that changes “owners”** of the bridge contracts.
  * Once ownership was hijacked, the attacker could instruct the bridge to release funds anywhere.
* **Impact:**

  * Attacker stole \~\$611M across multiple chains.
* **Aftermath:**

  * The hacker claimed they did it “for fun” and returned most of the funds.
  * Poly called the attacker a “white-hat.”
* **Lesson:**

  * Smart contract bugs in *administration logic* are catastrophic.
  * Bridges are very complex multi-chain systems, so attack surfaces are larger.

---

# ⚠️ Key Takeaways from All Hacks

* **Wormhole:** Cryptography/signature verification must be bulletproof.
* **Ronin:** Decentralization of validators is not optional — too few = single point of failure.
* **Poly Network:** Poor access control in contracts can hand over ownership to attackers.

💡 In all cases, bridges held **massive amounts of user funds**, making them irresistible targets.
