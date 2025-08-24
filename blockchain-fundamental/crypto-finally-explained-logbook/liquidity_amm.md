# 🏦 Liquidity Pools & Automated Market Makers (AMMs)

---

## 1. What is a Liquidity Pool?

* A **liquidity pool** is a smart contract that holds reserves of tokens (e.g., ETH + USDC).
* Instead of trading directly with another person, you trade **against the pool**.
* This makes trades instant — no need to wait for buyers or sellers.

✅ **Key idea**: Liquidity pools replace order books with on-chain token reserves.

---

## 2. What is an AMM (Automated Market Maker)?

* An **AMM** is the algorithm that decides the **price** of tokens in the pool.
* It adjusts prices automatically as traders swap tokens.

### 2.1 The Basic Formula: Constant Product AMM

$$
x \times y = k
$$

* `x` = reserve of Token A
* `y` = reserve of Token B
* `k` = constant (product stays the same)

**Example:**

* Pool starts with 1000 COINX and 1000 COINY → $k = 1{,}000{,}000$.
* You add 200 COINX.
* New COINX reserve = 1200.
* To keep $k$ constant → new COINY reserve = 1,000,000 / 1200 ≈ 833.3.
* Old reserve = 1000 → you get **166.7 COINY**.

⚠️ Notice: You don’t get 200 COINY, because the AMM curve makes the price worse as you buy more → **slippage**.

---

## 3. Types of AMM Models

* **Constant Product (Uniswap v2)** → great for general trading.
* **Constant Sum** → good for pegged assets, but risky if prices diverge.
* **Hybrid (Curve)** → optimized for stablecoins, super low slippage.
* **Multi-asset (Balancer)** → pools with 2+ tokens, like an on-chain ETF.
* **Concentrated Liquidity (Uniswap v3)** → LPs choose price ranges to provide liquidity.

---

## 4. Liquidity Providers (LPs)

* Anyone can deposit tokens into a pool.
* They earn:

  1. **Trading fees** (e.g., 0.3% per swap on Uniswap v2).
  2. Sometimes extra rewards (like governance tokens).

### Risk: Impermanent Loss

* If one token’s price rises faster than the other, LPs can lose compared to just holding.
* Example: If ETH moons while USDC stays flat, you end up with fewer ETH than if you just held ETH.

---

## 5. Real-World Examples

* **Uniswap** → first big AMM, billions in volume daily.
* **Curve** → stablecoin swaps with low slippage.
* **Balancer** → customizable pools (e.g., 50% ETH, 25% USDC, 25% DAI).
* **Pump.fun clones** → memecoin launches with instant liquidity pools.

---

## 6. Security Risks

### 6.1 Smart Contract Bugs

* Bugs in AMM code can drain pools.
* Case: **Balancer hack (2020)** — \~\$500k lost due to handling of deflationary tokens.

### 6.2 Price Manipulation (Oracle Attacks)

* If protocols use AMM price as an “oracle”, attackers can move the price using flash loans.
* Case: **bZx attack (2020)** — flash loans manipulated AMM prices, exploited lending.

### 6.3 Flash Loan Exploits

* Attacker borrows millions instantly, manipulates pool, takes profit, repays loan.

### 6.4 Rug Pulls

* Devs create pool, add liquidity, hype it, then withdraw liquidity → leaving buyers with worthless tokens.

---

## 7. Case Studies

* **Uniswap Success** → showed AMMs can rival centralized exchanges.
* **Curve Wars** → protocols fight for CRV token rewards to attract stablecoin liquidity.
* **SushiSwap “Vampire Attack”** → forked Uniswap, lured LPs with extra rewards.
* **Balancer Hack** → highlights importance of secure token handling in AMMs.

---

## 8. Why It Matters (For Bots & Traders)

* Your **memecoin sniping bot** relies on new liquidity pools being created.
* AMM curves mean **early buyers get much cheaper entry**, so being first matters.
* Risks include:

  * **No buyers after you** → stuck with worthless tokens.
  * **Rug pulls** or honeypots → can’t sell back your tokens.
  * **High competition** → only the fastest bots consistently profit.

---

✅ **Summary**

* Liquidity Pools = token reserves for instant trading.
* AMMs = algorithms (x·y=k, hybrids, etc.) that set prices.
* LPs = earn fees, but face impermanent loss.
* Real-world DEXs (Uniswap, Curve, Balancer) all use these systems.
* Security risks are real: bugs, flash loans, rugs.
* For sniping bots → success = speed + risk management.
