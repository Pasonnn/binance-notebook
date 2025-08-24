
# 🔐 Reentrancy – Study Card

---

## 1. **What it is**

Reentrancy happens when a smart contract makes an external call (e.g., `call`, `transfer`, or `send`) **before updating its own state**, allowing the external contract to call back into the original function and exploit the unchanged state.

* Root cause: **improper ordering of state changes + external calls**.
* Common in **withdrawals, token transfers, vaults**.

---

## 2. **How it works (Exploit Flow)**

Step-by-step:

1. Victim contract has a `withdraw()` function:

   * Sends ETH → *then* updates user balance.
2. Attacker deposits ETH into Victim.
3. Attacker calls `withdraw()`.
4. Victim executes `msg.sender.call{value: amount}("")`.
5. Attacker’s fallback function triggers, calling `withdraw()` again **before balance updates**.
6. Victim still thinks attacker has full balance → repeats until drained.

EVM detail: The external call hands control to attacker with full gas unless limited → attacker re-enters.

---

## 3. **Real Proof (Case Study)**

* **DAO Hack (2016)**

  * Loss: ~~3.6M ETH (~~\$60M at the time, >\$40B at peak prices).
  * Vulnerability: `splitDAO()` sent ETH before updating balances.
  * Attacker recursively called split function, draining funds.
  * Led to Ethereum hard fork (ETH vs ETC).

---

## 4. **Defense (Mitigations)**

* **Checks-Effects-Interactions pattern (CEI)**

  * Update balances/state *before* external calls.
  * ✅ Simple, minimal overhead.
  * ❌ Easy to forget, not enforced by compiler.

* **ReentrancyGuard (mutex)**

  * Boolean lock (e.g., OpenZeppelin `nonReentrant` modifier).
  * ✅ Easy to apply, protects multiple functions.
  * ❌ Doesn’t fix bad state logic, adds minor gas cost.

* **Pull-payment pattern**

  * Users withdraw themselves via separate function, no inline transfers.
  * ✅ Minimizes external calls inside critical functions.
  * ❌ Less user-friendly; requires extra step.

* **Limit gas stipend**

  * Use `transfer()` (2300 gas) instead of raw `call()`.
  * ✅ Limits fallback attack surface.
  * ❌ Not future-proof (EIPs changed gas rules → `transfer()` may fail).

---

## 5. **Q\&A Practice**

**Q1.** *Why does `call{value: ...}` increase reentrancy risk compared to `transfer()`?*

* `call()` forwards **all gas** by default, letting attacker run complex logic.
* `transfer()` only forwards **2300 gas**, limiting fallback execution.
* But since EIP-1884 and later gas repricing, `transfer()` can fail → best practice today: use `call()` but pair with CEI or mutex.

---

**Q2.** *Can reentrancy happen across different functions?*

* Yes → **cross-function reentrancy**.
* Example: Attacker calls `deposit()` inside fallback while `withdraw()` is running → state inconsistencies.
* Defense: use mutex that protects **all state-mutating functions**.

---

**Q3.** *How would you detect reentrancy in an audit?*

* Look for:

  * External calls (`call`, `delegatecall`, ERC20/721 transfers) **before** state updates.
  * Missing CEI.
  * Lack of `ReentrancyGuard`.
* Simulation: create malicious fallback contract to test recursive calls.

---

**Q4.** *Is reentrancy only about sending ETH?*

* No. Token standards like **ERC-777** and **ERC-1155** allow hooks (`tokensReceived`) that enable attacker code execution.
* Even “read-only reentrancy” exists (e.g., price oracles) → attacker manipulates state mid-calculation.

---

**Q5.** *How would you secure a lending protocol against reentrancy?*

* Use CEI for deposit/borrow/withdraw.
* Separate accounting (shares instead of raw balances).
* Add `nonReentrant` to state-mutating functions.
* Carefully review **cross-function interactions** (borrow inside withdraw).

---

## 6. **Traps (Gotchas)**

* **ERC-777 hooks** → attacker runs code on receipt of tokens.
* **Cross-function reentrancy** (deposit↔withdraw).
* **External calls in constructors** (vulnerable before full state init).
* **“Read-only” reentrancy** can manipulate on-chain oracles (Curve pools).
* **Assuming `transfer()` is safe forever** → gas repricing can break it.

---

✅ That’s the **full study card** for **Reentrancy**.
Next time they ask you, you’ll be able to explain:
👉 what it is → how it works → real hack → how to fix → tricky interview Q\&A → traps.

---

# 🔢 Integer Overflow / Underflow – Study Card

---

## 1. **What it is**

In Solidity (pre-0.8.0), integers are **fixed-size unsigned/signed values**.

* **Overflow** → when adding exceeds max value (wraps around to 0).
* **Underflow** → when subtracting below 0 (wraps around to max).

Example with `uint8` (0–255):

* `uint8 x = 255; x + 1 → 0` (overflow).
* `uint8 y = 0; y - 1 → 255` (underflow).

Root cause: **EVM arithmetic does not throw errors** (before Solidity 0.8).

---

## 2. **How it works (Exploit Flow)**

1. Victim contract tracks user balances in `uint256`.
2. A withdrawal function subtracts amount:

   ```solidity
   balances[msg.sender] -= _amount;
   msg.sender.call{value: _amount}("");
   ```
3. If balance was `0` and user withdraws `1`:

   * Pre-0.8: balance underflows → wraps to `2^256 - 1`.
   * Attacker now has massive balance → drains contract.

---

## 3. **Real Proof (Case Studies)**

* **BeautyChain Token (BEC), 2018**

  * Bug: missing SafeMath, arithmetic overflow in `batchTransfer`.
  * Exploit: attacker transferred tokens causing overflow → huge token balances.
  * Impact: supply inflated to \~8e58 tokens; exchange trading collapsed.

* **Uniswap V1 / Lendf.me, 2020**

  * Combined **reentrancy + underflow** in ERC-777 and ERC-20 logic.
  * Hackers exploited arithmetic errors in balance accounting.
  * Impact: \~\$25M stolen.

---

## 4. **Defense (Mitigations)**

* **Use Solidity 0.8+**

  * Built-in overflow/underflow checks → auto-revert on error.
  * ✅ Zero overhead in most modern EVMs.
* **Use SafeMath (OpenZeppelin)** (for <0.8 contracts).

  * Functions like `SafeMath.add`, `SafeMath.sub`.
  * ✅ Explicit checks, clear revert messages.
* **Validate inputs**

  * Require conditions (`require(balances[msg.sender] >= _amount)`).
* **Invariant testing**

  * Unit/invariant tests to detect unexpected wraparounds.

---

## 5. **Q\&A Practice**

**Q1.** *Why are overflow/underflow vulnerabilities less common today?*

* Solidity 0.8+ has **checked arithmetic by default** → reverts on overflow/underflow.
* Legacy contracts still vulnerable if deployed pre-0.8 or with inline assembly math.

---

**Q2.** *How did the BEC overflow work?*

* Attacker called `batchTransfer()` with values causing sum > `2^256`.
* Overflow wrapped balance → attacker received **near-infinite tokens**.
* Destroyed supply integrity → exchanges halted trading.

---

**Q3.** *How could underflow interact with reentrancy?*

* Example: attacker withdraws from contract with 0 balance.
* Underflow sets balance to max `uint256`.
* Reentrant calls drain contract since balance check passes.

---

**Q4.** *What’s the gas impact of SafeMath vs Solidity 0.8 checks?*

* SafeMath adds **extra function calls + checks**, minor gas cost.
* Solidity 0.8 built-in checks → optimized at compiler level, cheaper.

---

**Q5.** *Can overflow still happen in Solidity 0.8?*

* Yes, if developer uses **`unchecked { ... }`** block to save gas.
* If used incorrectly, same vulnerabilities can reappear.

---

## 6. **Traps (Gotchas)**

* Assuming Solidity auto-protects **all cases** — but inline assembly or `unchecked` can still be unsafe.
* ERC-20 implementations may hide subtle arithmetic bugs (esp. in custom tokens).
* Loop counters using `i++` can overflow if unbounded (theoretically).
* Developers migrating old code but forgetting SafeMath.


---


# 🔑 Access Control Flaws – Study Card

---

## 1. **What it is**

Access control = who is authorized to call sensitive functions in a smart contract (e.g., mint, upgrade, pause, withdraw).

* **Flaws** happen when contracts:

  * Don’t restrict sensitive functions.
  * Use weak primitives (`tx.origin` instead of `msg.sender`).
  * Misconfigure roles (anyone can upgrade/mint).
* Root cause: **improper enforcement of permissions**.

---

## 2. **How it works (Exploit Flow)**

1. Victim contract has `function mint(uint amount)` without `onlyOwner`.
2. Attacker calls `mint(1_000_000)` → creates unlimited tokens.
3. If contract checks `tx.origin == owner`:

   * Attacker writes malicious contract that calls Victim.
   * Victim sees `tx.origin == owner` (if owner calls attacker contract).
   * Attacker hijacks privileged actions.

---

## 3. **Real Proof (Case Studies)**

* **Parity Multisig Wallet (2017, \$150M frozen)**

  * Bug: `initWallet()` function could be re-initialized by anyone.
  * Attacker made themselves owner → triggered `selfdestruct`.
  * All funds frozen permanently.

* **Lendf.me / imBTC (2020, \~\$25M stolen)**

  * Partly due to **reentrancy** + weak permission checks in ERC-777 integration.

* **dForce “YAM Finance” clones**

  * Weak role configs allowed unlimited mint/burn.

---

## 4. **Defense (Mitigations)**

* **Use standardized libraries**

  * OpenZeppelin’s `Ownable` or `AccessControl` for roles.
  * ✅ battle-tested, audited.
* **Principle of least privilege**

  * Granular roles (`MINTER_ROLE`, `PAUSER_ROLE`, `UPGRADER_ROLE`).
  * Avoid single “god-mode” owner.
* **Multi-sig / MPC for admin keys**

  * ✅ reduces single point of failure.
  * ❌ operational overhead.
* **Timelocks on upgrades/critical ops**

  * ✅ community time to react.
* **Avoid `tx.origin`**

  * Always use `msg.sender` for authorization.

---

## 5. **Q\&A Practice**

**Q1.** *Why is `tx.origin` dangerous for access control?*

* `tx.origin` traces back to **EOA that started the transaction**.
* If owner calls a malicious contract, that contract can forward calls to victim, with `tx.origin == owner`.
* Attack bypasses checks.
* Safer: always use `msg.sender` (direct caller).

---

**Q2.** *What’s the risk with upgradeable contracts and access control?*

* If upgrade function isn’t protected (e.g., `upgradeTo()`), attacker can replace logic contract with malicious one.
* Solution: restrict with `onlyOwner`, use multi-sig, timelocks.

---

**Q3.** *How would you audit for access control flaws?*

* List all privileged functions.
* Verify each has correct modifier (`onlyOwner`, role-based).
* Trace how owner/roles are initialized (constructor or `initialize`).
* Check that roles can’t be reassigned by non-privileged accounts.
* Look for `tx.origin`, hardcoded addresses, or public init functions.

---

**Q4.** *What’s the difference between `Ownable` and `AccessControl` in OpenZeppelin?*

* `Ownable`: single privileged address (`owner`).
* `AccessControl`: role-based, supports multiple roles & granular privileges.
* In larger systems, `AccessControl` is safer and more flexible.

---

**Q5.** *How can you reduce risk of admin key compromise?*

* Use Gnosis Safe (multi-sig).
* Split control (e.g., 2-of-3 signers).
* Add timelock so community can monitor changes.

---

## 6. **Traps (Gotchas)**

* **Uninitialized contracts**: dev forgets to call `initialize()`, attacker can call and become owner.
* **Shadowed modifiers**: function meant to be `onlyOwner` but missing.
* **Hardcoded test keys** left in code.
* **Overpowered owner**: one key controls upgrade, mint, pause.
* **Forgotten revoke**: role not revoked after migration.
* **Centralization risk**: Binance/large protocols often criticized if too much control in one admin wallet.

---

# ⚡ Front-running / MEV (Maximal Extractable Value) – Study Card

---

## 1. **What it is**

* **Front-running**: When an attacker (or miner/validator) sees a pending transaction in the mempool and inserts their own transaction before it to gain profit.
* **MEV**: Broader category — **Maximal Extractable Value** — profit extracted by ordering, including, or censoring transactions in a block.

  * Includes front-running, back-running, sandwich attacks, liquidation sniping, arbitrage.
* Root cause: **transparent mempool + deterministic block ordering controlled by validators/bots**.

---

## 2. **How it works (Exploit Flow)**

Example: **Sandwich Attack on a DEX trade**

1. Alice submits swap: `1,000 ETH → USDC` on Uniswap.
2. Attacker sees it in mempool.
3. Attacker sends:

   * **Tx1:** Buy ETH before Alice (pushes price up).
   * **Tx2:** Alice’s big swap executes at worse price (slippage).
   * **Tx3:** Attacker sells ETH after Alice at profit.
4. Result: Alice suffers higher slippage, attacker profits risk-free.

EVM detail: Bots monitor mempool (via RPC) → craft tx with higher gas fee or priority tip → miners/validators include attacker’s tx first.

---

## 3. **Real Proof (Case Studies)**

* **DEX Sandwiching (Uniswap, SushiSwap, PancakeSwap)**

  * Billions siphoned over years by MEV bots.
* **Ethereum MEV (Flashbots data)**

  * > \$1.5B extracted since 2020.
* **Curve Finance arbitrage MEV**

  * Large whales exploited slippage + imbalanced pools.

---

## 4. **Defense (Mitigations)**

* **For users**

  * Set tight slippage tolerance.
  * Use private RPC / Flashbots Protect (skip public mempool).
  * Use aggregators that route via protected relays.

* **For protocols**

  * **Batch auctions** (match orders at single clearing price).
  * **Commit-reveal schemes** (hide trade details until commit phase).
  * **Fair ordering** (e.g., chain-based randomized ordering, SUAVE, Eden Network).
  * **MEV redistribution** → shared among validators/users (e.g., MEV-Boost).

---

## 5. **Q\&A Practice**

**Q1.** *How does a sandwich attack work?*

* Attacker sees pending trade, buys before (raising price), lets victim swap at worse price, then sells after.
* Profit = victim’s slippage loss.

---

**Q2.** *Why can validators extract MEV more easily than normal users?*

* Validators control transaction ordering in a block → guaranteed front-running/back-running without competing in gas wars.
* Normal users rely on bidding wars with gas fees.

---

**Q3.** *What’s the difference between front-running and back-running?*

* **Front-running**: insert tx *before* victim to profit from victim’s impact.
* **Back-running**: insert tx *after* victim to capture arbitrage opportunities (e.g., after liquidation or large trade).

---

**Q4.** *How can protocols reduce MEV risk?*

* Use batch auctions (no advantage to ordering).
* Oracle updates on TWAP, not single trades.
* Commit-reveal for orders (hide until reveal).
* Private transaction relays (skip mempool exposure).

---

**Q5.** *Is all MEV bad?*

* No. Some MEV is **toxic** (hurts users: sandwiches).
* Some MEV is **positive** (keeps markets efficient: arbitrage, liquidations).
* The challenge is to minimize toxic MEV and share positive MEV fairly.

---

## 6. **Traps (Gotchas)**

* **Slippage setting**: Users set 1–5% slippage; MEV bots exploit full tolerance.
* **Gas fee illusions**: Attackers can bribe validators directly (Flashbots bundles) → don’t show in mempool.
* **Cross-chain MEV**: Bots exploit price differences across chains via bridges.
* **Protocol design flaw**: Exposing sensitive operations (e.g., liquidation, governance voting) to mempool sniping.
* **L2 is not immune**: Rollups still have sequencer ordering power.

---

