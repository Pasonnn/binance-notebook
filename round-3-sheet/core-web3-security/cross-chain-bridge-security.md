# 🌉 Lock-and-Mint vs Burn-and-Release – Study Card

---

## 1. **What it is**

**Cross-chain bridges** allow assets to move between blockchains (e.g., BTC → Ethereum). Two main models:

* **Lock-and-Mint**

  * Asset locked on source chain.
  * Representation minted on destination chain.
* **Burn-and-Release**

  * Representation burned on destination chain.
  * Underlying asset released on source chain.

Together, they form the basic **bridge lifecycle**.

---

## 2. **How it works (Exploit Flow)**

### 🔒 Lock-and-Mint

1. User sends **BTC** to bridge contract/custodian on Bitcoin.
2. Bridge verifies deposit.
3. Bridge mints **WBTC** on Ethereum to user’s wallet.

   * WBTC backed 1:1 by locked BTC.

### 🔥 Burn-and-Release

1. User burns **WBTC** on Ethereum (via bridge contract).
2. Bridge verifies burn event.
3. Bridge custodian releases original BTC on Bitcoin chain.

**Attack Surface:**

* If verification compromised (e.g., fake proof of lock/burn), attacker mints unbacked tokens or withdraws without burning.
* If custodian/validator keys stolen → attacker drains reserves.

---

## 3. **Real Proof (Case Studies)**

* **Wormhole Bridge Hack (2022)**

  * \$325M stolen.
  * Attacker forged a fake signature for minting on Solana side.

* **Ronin Bridge Hack (2022)**

  * \$600M lost (largest ever).
  * Attacker compromised 5/9 validator keys → minted assets without real lock.

* **Poly Network (2021)**

  * \$611M drained.
  * Attacker exploited flawed cross-chain contract verification.

---

## 4. **Defense (Mitigations)**

* **Decentralized validation**

  * Multiple validators (multi-sig, MPC, threshold signatures).
  * ✅ Harder to compromise than 1 custodian.

* **Light client verification**

  * Destination chain verifies source chain state directly (no trusted party).
  * ✅ Strong security.
  * ❌ Expensive, complex (esp. for non-EVM chains).

* **Optimistic bridges**

  * Assume messages valid but allow fraud proofs.
  * ✅ Lighter than full clients.

* **Rate limiting / caps**

  * Limit how much can be minted/released per tx/block.

* **Continuous audits & monitoring**

  * Watch for anomalous mints/burns.

---

## 5. **Q\&A Practice**

**Q1.** *What’s the difference between lock-and-mint vs burn-and-release?*

* Lock-and-Mint: lock real asset on Chain A, mint wrapped asset on Chain B.
* Burn-and-Release: burn wrapped asset on Chain B, release underlying on Chain A.
* Together: ensure circulating supply always backed 1:1.

---

**Q2.** *Why are bridges such high-value attack targets?*

* They hold large locked assets.
* If validation compromised → attacker can mint unbacked assets (infinite money glitch).
* Billions stolen historically.

---

**Q3.** *How does Ronin hack relate to bridge design?*

* Ronin used 9 validators, only 5 needed to sign.
* Attacker stole 5 keys → gained control.
* Compromised **lock-and-mint** model (minted without actual lock).

---

**Q4.** *How can you secure a lock-and-mint bridge?*

* Threshold signatures (MPC).
* Decentralize validators across independent parties.
* Light clients to verify chain state instead of trust.
* Withdraw rate limits.

---

**Q5.** *What’s the trade-off between light clients and MPC bridges?*

* Light clients = highest trust-minimized security, but expensive (gas heavy).
* MPC/multisig = cheaper, but adds trust assumptions.

---

## 6. **Traps (Gotchas)**

* **Validator key compromise** = single point of failure.
* **Replay attacks** = same proof reused across chains.
* **Event spoofing** = fake lock/burn proof accepted.
* **Centralization risk**: If 1 custodian holds keys → not really “trustless.”
* **Liquidity mismatch**: If too much minted relative to locked reserves, peg breaks.

---

# 🗝️ Validator Compromise Risks – Study Card

---

## 1. **What it is**

In cross-chain bridges or PoS systems, **validators (or signers)** secure transactions and verify state.

* They may sign proofs of asset lock/burn or validate consensus.
* If attackers **compromise validator keys** (via phishing, malware, inside collusion, or poor key management), they can:

  * Mint unbacked tokens (bridge context).
  * Release funds without proper burn.
  * Censor or reorder transactions (consensus/MEV).

Root cause: **too much trust placed in too few validators / poor key security**.

---

## 2. **How it works (Exploit Flow)**

1. Bridge uses **M-of-N validators** to confirm lock-and-mint.
2. Attacker compromises majority (M) of validators’ keys.
3. Attacker fabricates a “proof” that assets were locked/burned.
4. Validators (controlled by attacker) sign → contract accepts.
5. Attacker mints wrapped tokens (unbacked) or drains reserves.

**EVM detail:** Validator signatures are verified on-chain. If attacker controls enough keys, forged signatures pass checks.

---

## 3. **Real Proof (Case Studies)**

* **Ronin Bridge Hack (2022, Axie Infinity)**

  * \$600M stolen.
  * 9 validators, only 5 needed to approve.
  * Attacker phished developers, gained control of 5 private keys.
  * Minted unbacked ETH, drained bridge.

* **Harmony Horizon Bridge Hack (2022)**

  * \$100M lost.
  * 2-of-5 multisig validator model.
  * Attacker compromised just 2 keys → total loss.

* **Wormhole (2022)** (different vector)

  * Exploit in signature verification → attacker minted tokens.

---

## 4. **Defense (Mitigations)**

* **Increase validator decentralization**

  * Larger validator set, harder to compromise.

* **Threshold cryptography / MPC**

  * Keys split into shards → no single key compromise possible.

* **HSMs (Hardware Security Modules)**

  * Store private keys in secure hardware.

* **Slashing & monitoring**

  * Misbehaving validators lose stake; anomalies trigger alerts.

* **Rate limits & caps**

  * Even if validators compromised, restrict daily withdrawals/mints.

* **Diverse infrastructure**

  * Validators run by independent orgs, not one company.

---

## 5. **Q\&A Practice**

**Q1.** *Why was Ronin vulnerable despite having 9 validators?*

* Only 5 signatures needed → attacker only had to steal 5 keys.
* Validator set too small and concentrated.

---

**Q2.** *What’s the trade-off between MPC and multisig for validators?*

* Multisig = simple but if M keys stolen, system compromised.
* MPC = one logical signature created from distributed shards → no single key exists, more secure but complex.

---

**Q3.** *How can you reduce validator compromise risk operationally?*

* Use HSMs and key sharding.
* Geographically distribute validators.
* Rotate keys periodically.
* Independent operators with separate infra.

---

**Q4.** *Why are bridges especially sensitive to validator compromise?*

* Validator signatures = proof of locked/burned funds.
* If forged, attacker creates **infinite unbacked tokens**.
* Impact catastrophic (hundreds of millions).

---

**Q5.** *How would you design monitoring for validator compromise?*

* Watch for unusual withdrawal/mint events.
* Alert if large mints without proportional lock.
* Require human/manual approval for unusually large transfers.

---

## 6. **Traps (Gotchas)**

* **Small validator set** = easy compromise (e.g., 2-of-5 in Horizon).
* **Centralized operators** = one org runs most validators.
* **Key reuse** across environments → one breach = multiple validator keys stolen.
* **No rate limiting** → attacker drains entire reserve in 1 tx.
* **Social engineering** (Ronin used spear-phishing job offer to steal keys).

---

# 🌀 Replay & Fake Proof Attacks – Study Card

---

## 1. **What it is**

* **Replay Attack**: A valid transaction/proof on one chain is maliciously **re-used (replayed)** on another chain or context where it is still valid.
* **Fake Proof Attack**: Attacker forges or tricks a bridge/contract into **accepting a proof of an event that never happened** (e.g., fake lock, fake burn).

These often target **cross-chain bridges and interoperability protocols** that rely on signatures or Merkle proofs to verify state.

---

## 2. **How it works (Exploit Flow)**

### 🔄 Replay Attack

1. Bridge uses validator signatures for a lock event on Chain A.
2. Attacker resubmits the same signed proof on Chain B or replays an old lock event.
3. If bridge doesn’t track **nonces / unique IDs**, it releases funds again.
4. Result: double-spend of bridged assets.

### 🏴 Fake Proof Attack

1. Attacker fabricates a Merkle proof or signature.
2. If bridge contract doesn’t fully verify the source chain state (e.g., weak light-client checks), it accepts the fake proof.
3. Bridge mints tokens or releases reserves without actual lock/burn.
4. Attacker drains reserves with “phantom deposits.”

---

## 3. **Real Proof (Case Studies)**

* **Poly Network (2021, \$611M stolen)**

  * Attacker crafted a malicious cross-chain message.
  * Exploited weak verification in contract → “proof” of fake transaction accepted.
  * Drained assets from multiple chains.

* **QBridge Hack (Qubit Finance, 2022, \$80M stolen)**

  * Attacker called bridge contract with crafted data.
  * No real deposit on source chain, but destination accepted fake proof → minted tokens.

* **Nomad Bridge (2022, \$190M stolen)**

  * Initialization bug meant *any submitted message proof was accepted as valid*.
  * Exploit went viral → anyone could copy/paste attack tx.

---

## 4. **Defense (Mitigations)**

* **Nonce / replay protection**

  * Track unique deposit IDs or message hashes; mark as used after processing.

* **Light client verification**

  * Destination chain verifies source chain headers + Merkle proofs.
  * ✅ Hard to fake, secure.
  * ❌ Expensive in gas and complex.

* **Optimistic bridges w/ fraud proofs**

  * Assume valid unless challenged in dispute period.

* **Validator / MPC thresholds**

  * Multiple validators must sign → harder to forge proof.

* **Audit message-passing contracts**

  * Ensure event logs can’t be faked or replayed.

---

## 5. **Q\&A Practice**

**Q1.** *How is a replay attack different from a fake proof attack?*

* Replay = re-using a valid proof again.
* Fake proof = creating a forged proof for an event that never happened.

---

**Q2.** *How can replay attacks be prevented?*

* Track unique nonces or message IDs.
* Mark proofs as consumed once processed.
* Reject duplicates.

---

**Q3.** *Why did Poly Network fail?*

* Contract trusted cross-chain message without robust verification.
* Attacker faked a “lock” message → bridge minted unbacked tokens.

---

**Q4.** *What’s the trade-off of light clients vs multisig validation?*

* Light clients = trust-minimized, secure, but gas-heavy.
* Multisig/MPC = cheap, but adds trust assumption (risk if validators collude).

---

**Q5.** *Why was Nomad hack so devastating?*

* A single bug caused **all proofs to be treated as valid**.
* Anyone could copy the hacker’s transaction and drain funds.
* Showed importance of rigorous auditing of proof verification.

---

## 6. **Traps (Gotchas)**

* **No nonce tracking** → same deposit replayable.
* **Improper chain ID/domain separation** → same signature valid across chains.
* **Weak proof verification logic** → “assume valid” instead of cryptographic check.
* **Trusting logs blindly** → logs can be faked by crafted contract calls.
* **Upgrade bugs** → a new bridge upgrade may accidentally loosen proof checks.

---

✅ That’s the **Replay / Fake Proof Attacks card** — covering **definition, exploit flow, real hacks, defenses, Q\&A, and traps.**

---

