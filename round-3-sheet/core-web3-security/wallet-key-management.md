

# 👛 Hot vs Cold Wallets – Study Card

---

## 1. **What it is**

* **Hot Wallets**: Private keys stored online (connected to internet).

  * Examples: Exchange hot wallets, MetaMask, mobile wallets.
  * ✅ Convenient for fast transfers & liquidity.
  * ❌ Higher attack surface (hacks, malware, phishing).

* **Cold Wallets**: Private keys stored **offline** (not internet-connected).

  * Examples: Hardware wallets (Ledger, Trezor), paper wallets, air-gapped machines.
  * ✅ Strong security against remote hacks.
  * ❌ Less convenient for frequent transactions.

Used together: Exchanges (like Binance) keep **small % in hot wallets** for withdrawals, majority in cold storage.

---

## 2. **How it works (Exploit Flow)**

### Hot Wallet Risk Example

1. Exchange hot wallet private key stored on server.
2. Attacker breaches server → exfiltrates private key.
3. Drains funds instantly (since wallet is online).

### Cold Wallet Protection Flow

1. Funds stored offline (hardware device or paper).
2. Transaction signed offline.
3. Only signed tx (not private key) sent online → key never exposed.

---

## 3. **Real Proof (Case Studies)**

* **Binance Hack (2019, \$40M)**

  * Hot wallet compromise → 7,000 BTC stolen.
  * Attackers used phishing & API key abuse.

* **Mt. Gox (2014, \$450M)**

  * Poor key management, hot wallet drained over time.

* **KuCoin Hack (2020, \$275M)**

  * Hot wallets breached.
  * Luckily much recovered via cooperation & blacklists.

---

## 4. **Defense (Mitigations)**

* **For Hot Wallets**

  * Multi-sig / MPC custody → no single key compromise.
  * Rate limits (per withdrawal / per time window).
  * Behavioral anomaly detection.
  * Minimize hot balance (keep <2–5% of total reserves hot).

* **For Cold Wallets**

  * Hardware wallets with secure enclaves.
  * Air-gapped signing (keys never touch internet).
  * Physical security (vaults, access control).
  * Sharded key storage (e.g., Shamir’s Secret Sharing).

* **Hybrid approach (exchange ops)**

  * Cold: 95%+ of assets.
  * Hot: small buffer for user withdrawals.

---

## 5. **Q\&A Practice**

**Q1.** *Why not keep all funds in cold wallets?*

* Too slow for user withdrawals.
* Exchanges need hot liquidity for real-time trading & withdrawals.
* Hybrid model balances security & usability.

---

**Q2.** *Why are hot wallets the primary target for hackers?*

* Keys online = reachable via phishing, malware, API leaks, server exploits.
* Large balances often rotate through them.

---

**Q3.** *What’s the trade-off between multi-sig and MPC custody?*

* Multi-sig: requires M-of-N signatures. Simple but on-chain logic may leak structure.
* MPC: mathematically splits key shares, one logical signature produced. More private, harder to attack, but complex.

---

**Q4.** *How do hardware wallets improve security?*

* Keys never leave secure chip.
* Signing happens inside device → malware on host PC can’t steal private key.

---

**Q5.** *What happened in Binance’s 2019 hack, and what does it teach us?*

* Attackers compromised API keys, 2FA bypass, gained access to hot wallet.
* Takeaway: minimize hot wallet exposure, monitor abnormal withdrawals, defense-in-depth.

---

## 6. **Traps (Gotchas)**

* **Single-key hot wallets** → one leak = full drain.
* **Cold wallet setup leaks** → if seed phrase photographed/recorded, security fails.
* **“Warm wallets”** (semi-online) misunderstood; still attackable.
* **Social engineering** → SIM-swap phishing leads to hot wallet compromise.
* **User wallets**: many users keep all funds in MetaMask (hot) without hardware backup.

---

# 🔑 MPC & Multisig in Wallet Security – Study Card

---

## 1. **What it is**

* **Multisig (Multi-Signature)**

  * Requires **M-of-N keys** to authorize a transaction.
  * Example: 2-of-3 multisig → any 2 keys out of 3 must sign.
  * Implemented **on-chain** (e.g., Gnosis Safe).
  * ✅ Simple, transparent.
  * ❌ On-chain overhead, signature structure visible to all.

* **MPC (Multi-Party Computation)**

  * Private key **split into shards** across multiple parties.
  * Each party computes partial signature → combined into 1 valid signature.
  * Underlying key is **never reconstructed**.
  * ✅ Off-chain, invisible to blockchain (looks like single normal signature).
  * ❌ More complex infra, requires careful implementation.

Both are used in custody and exchange security (Binance, Fireblocks, Coinbase Custody).

---

## 2. **How it works (Exploit / Flow)**

### 🔒 Multisig Example

1. Smart contract wallet requires 2-of-3 signatures.
2. Transaction proposed → sent to all signers.
3. At least 2 sign with private keys.
4. Contract verifies → executes transaction.
   **Attack surface**: if attacker compromises 2 signers → funds drained.

### 🧩 MPC Example

1. Private key split into 3 shares.
2. Each party signs partial tx → combined mathematically (threshold signature scheme, e.g., ECDSA, EdDSA).
3. Final signature indistinguishable from normal single-key.
   **Attack surface**: attacker must compromise majority of MPC participants.

---

## 3. **Real Proof (Case Studies)**

* **Parity Multisig Bug (2017, \$150M frozen)**

  * Bug in multisig contract → attacker killed library, locked funds permanently.
  * Shows **on-chain multisig contracts** are high-value targets.

* **Fireblocks MPC Custody** (industry example, not hack)

  * Widely adopted by institutions because no single key exists.
  * Even insider compromise of one shard useless.

* **Harmony Horizon Bridge (2022, \$100M lost)**

  * Used 2-of-5 multisig. Attacker compromised 2 keys → full drain.
  * Shows danger of **too-low multisig threshold**.

---

## 4. **Defense (Mitigations)**

* **Multisig**

  * Use higher thresholds (e.g., 4-of-7).
  * Distribute signers across independent entities.
  * Audit contract logic (avoid Parity bug).

* **MPC**

  * Use threshold cryptography with proven libraries (GG18, GG20).
  * Deploy across secure environments (HSMs, enclaves).
  * Rotate key shares periodically.

* **General best practice**

  * Combine MPC + multisig layers (defense-in-depth).
  * Rate-limit withdrawals.
  * Continuous monitoring for abnormal signing activity.

---

## 5. **Q\&A Practice**

**Q1.** *How does MPC differ from multisig?*

* Multisig: on-chain smart contract requires multiple explicit signatures.
* MPC: off-chain threshold scheme produces a single signature that looks normal.
* Trade-off: multisig = simpler, transparent but visible & higher gas; MPC = invisible but more complex infra.

---

**Q2.** *Why did Harmony’s Horizon Bridge fail despite using multisig?*

* Threshold too low (2-of-5).
* Attacker compromised 2 validators’ keys → total control.
* Lesson: decentralize signers and raise threshold.

---

**Q3.** *What risk did the Parity multisig wallet highlight?*

* Smart contract bugs in multisig can lock or drain funds.
* Shows why audits & formal verification are critical for on-chain multisig.

---

**Q4.** *Why do exchanges like Binance prefer MPC custody?*

* Users see a normal wallet (single signature).
* Off-chain infra harder to attack than on-chain smart contract.
* Scales well across many assets (not every chain supports multisig).

---

**Q5.** *What’s the main limitation of MPC?*

* Complexity → if implementation flawed, risk of catastrophic failure.
* Requires strong operational security (each shard holder must be independent).

---

## 6. **Traps (Gotchas)**

* **Low multisig threshold** (2-of-3 or 2-of-5) = vulnerable.
* **Single operator runs multiple signers** → defeats decentralization.
* **MPC false sense of security** → if all shards stored in one cloud provider, attacker still wins.
* **Multisig gas cost** → high overhead for frequent transactions.
* **Parity-style contract bugs** → one small logic error can freeze millions.

---

Perfect — let’s build the **Private Key Leaks, Phishing, and SIM-Swap Risks study card** 🚀

---

# 🔓 Risks of Private Key Leaks, Phishing, & SIM-Swap – Study Card

---

## 1. **What it is**

Private keys = the **ultimate control** over blockchain accounts.
If compromised → attacker has **full ownership** (no recovery, unlike banks).

* **Key Leaks**: Keys accidentally exposed (logs, screenshots, cloud storage).
* **Phishing**: Attacker tricks user into signing malicious tx or revealing seed phrase.
* **SIM-Swap**: Attacker hijacks victim’s phone number → bypasses SMS 2FA → resets accounts/exchange logins.

Root cause: **weak key management + poor user security awareness**.

---

## 2. **How it works (Exploit Flow)**

### 🔑 Key Leak

1. Developer hardcodes private key in GitHub repo.
2. Attacker scans GitHub, finds exposed key.
3. Transfers funds instantly.

### 🎣 Phishing

1. Attacker builds fake MetaMask site or “airdrop” dApp.
2. User connects wallet and unknowingly signs `approve()` or `permit()`.
3. Attacker drains ERC-20 tokens.

### 📱 SIM-Swap

1. Attacker convinces telco to port victim’s phone number.
2. Receives SMS 2FA codes or password reset links.
3. Gains exchange login → withdraws funds.

---

## 3. **Real Proof (Case Studies)**

* **\$24M stolen from dYdX user (2023)**

  * Suspected SIM-swap + phishing combo.

* **Axie Infinity (Sky Mavis, 2022, \$600M Ronin hack)**

  * Attackers spear-phished developers with fake job offers.
  * Gained access to validator keys.

* **Countless GitHub leaks**

  * Bots constantly scan for `0x` private keys in repos → instant theft.

---

## 4. **Defense (Mitigations)**

* **Key Storage**

  * Never hardcode / upload to cloud.
  * Use HSMs, secure enclaves, hardware wallets.

* **Phishing Protection**

  * Verify domains & wallet prompts.
  * Train users: never share seed phrases.
  * Use transaction simulation (e.g., MetaMask’s preview).

* **SIM-Swap Defense**

  * Avoid SMS 2FA → use authenticator apps (TOTP) or hardware keys (YubiKey).
  * Add PIN/port-out lock with carrier.

* **Operational Controls (exchanges)**

  * Whitelist withdrawal addresses.
  * Monitor abnormal approvals.
  * Transaction delay windows.

---

## 5. **Q\&A Practice**

**Q1.** *Why are SIM-swaps especially dangerous for crypto users?*

* SMS 2FA widely used for exchanges.
* Once attacker controls phone number, they can reset logins and bypass OTPs.
* Solution: hardware keys or authenticator apps.

---

**Q2.** *How do phishing attacks bypass wallet security?*

* Users willingly sign malicious transactions (`approve`, `permit`, `setApprovalForAll`).
* Attacker doesn’t need the private key, only permission to spend.

---

**Q3.** *What’s safer: SMS 2FA or Google Authenticator?*

* Authenticator (TOTP) much safer.
* SMS vulnerable to SIM-swap, SS7 attacks, telco insider abuse.

---

**Q4.** *How should developers protect keys in production?*

* Use environment variables, secure key vaults (AWS KMS, HashiCorp Vault).
* Avoid storing in plaintext anywhere.
* Rotate keys regularly.

---

**Q5.** *What’s the single biggest mistake new crypto users make?*

* Storing seed phrases online (Google Drive, email, screenshots).
* Phishers & malware constantly scan cloud accounts for them.

---

## 6. **Traps (Gotchas)**

* **Blind signing**: users click “sign” in MetaMask without reading → malicious approvals.
* **Recovery phrases shared**: fake support staff trick users into giving seeds.
* **Exchanges relying on SMS 2FA** → weak by default.
* **Developers hardcoding test keys** → later forgotten, exploited.
* **AirDrop / fake token scams**: attacker sends worthless token that tricks users into signing malicious tx.

---

✅ That’s the **Private Key Leaks, Phishing & SIM-Swap Risks card** — with **definition, flows, real hacks, mitigations, Q\&A, and traps.**

---
