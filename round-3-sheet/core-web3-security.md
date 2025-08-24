
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

