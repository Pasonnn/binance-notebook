# 🧮 AMM Math Manipulation – Study Card

---

## 1. **What it is**

Automated Market Makers (AMMs) like Uniswap, PancakeSwap, Curve set prices algorithmically based on pool reserves.

* **x \* y = k (constant product)** for Uniswap v2 style.
* Attackers exploit the math to:

  * **Manipulate prices** (especially with flash loans).
  * **Drain liquidity via imbalance**.
  * **Exploit low-liquidity pools** to cause slippage and mispricing.

Root cause: **mathematical assumptions break when adversaries control large trades or exploit protocol integration.**

---

## 2. **How it works (Exploit Flow)**

### Case 1: **Price Manipulation via Flash Loan**

1. Protocol relies on AMM pool price as oracle.
2. Attacker flash-borrows asset → swaps huge amount, skewing reserves.
3. Oracle reads manipulated spot price.
4. Attacker exploits inflated/deflated price in lending/borrowing protocol.
5. Returns flash loan, keeps profit.

### Case 2: **Low Liquidity Exploit**

* In a shallow pool, even a modest trade causes large price moves (slippage).
* Attacker can manipulate price cheaply → useful for oracle attacks or arbitrage.

### Case 3: **Curve StableSwap Math Exploits**

* Curve uses **StableSwap invariant** (for low-slippage stablecoins).
* Attackers manipulate imbalanced pools → force users into unfavorable swaps.

---

## 3. **Real Proof (Case Studies)**

* **Harvest Finance (2020, \~\$24M)**

  * Attackers used Curve’s USDT/USDC pool math.
  * Manipulated stablecoin price via flash loans → drained vaults.

* **bZx Attacks (2020, multiple)**

  * Manipulated Uniswap pool math to trick lending oracles.

* **Inverse Finance (2022, \~\$15M)**

  * Flash loaned tokens → skewed AMM math → borrowed more stablecoins.

---

## 4. **Defense (Mitigations)**

* **Do not use spot AMM price as oracle**

  * Use Chainlink or TWAP instead.
* **Require sufficient liquidity depth**

  * Large pools harder to manipulate.
* **Slippage checks**

  * Enforce `minAmountOut` to protect users from manipulation.
* **TWAP (Time-Weighted Average Price)**

  * Smooths manipulation across blocks.
* **Circuit breakers**

  * Halt trades if price changes > X% in one block.

---

## 5. **Q\&A Practice**

**Q1.** *How does Uniswap’s math set price?*

* In x \* y = k pools, price of Token A relative to Token B = `reserveB / reserveA`.
* Swapping changes reserves, thus price.

---

**Q2.** *Why is low liquidity dangerous in AMMs?*

* Small trades cause huge price swings.
* Attacker can cheaply manipulate prices for oracle exploitation.

---

**Q3.** *What was the Harvest Finance exploit?*

* Used flash loans to pump stablecoin ratio in Curve pools.
* Harvest vault read manipulated pool value → attacker withdrew inflated funds.

---

**Q4.** *What is a TWAP and why is it safer?*

* TWAP = average price over a period.
* Makes it costly to sustain manipulation across many blocks.
* Still vulnerable to long-term attacks if attacker controls liquidity.

---

**Q5.** *If you audit a protocol using Uniswap as oracle, what red flags?*

* Using `getReserves()` spot price directly.
* Small liquidity pools as source of truth.
* No fallback oracle (single point of failure).
* No deviation checks on sudden price swings.

---

## 6. **Traps (Gotchas)**

* **Liquidity imbalance**: Attackers provide/withdraw liquidity strategically to worsen slippage.
* **Flash-loan pump & dump**: Even deep pools can be manipulated if only spot price checked.
* **Stable pools**: Curve math hides manipulation better → harder to detect.
* **Asymmetric pools**: 80/20 pools (Balancer style) change sensitivity differently; devs may miscalculate.
* **Integrations**: Lending protocols blindly trusting AMM math without safeguards.

---

# 📡 Oracle Manipulation (Flash Loans) – Study Card

---

## 1. **What it is**

Oracle manipulation occurs when an attacker **influences the price feed a protocol relies on**, usually by skewing an AMM pool used as an oracle.

* Flash loans amplify this → attacker temporarily controls massive liquidity.
* Typical outcome: **collateral mispricing** → attacker borrows more than they should, drains stablecoins, or forces bad liquidations.

Root cause: **protocols relying on manipulable spot prices from shallow AMMs.**

---

## 2. **How it works (Exploit Flow)**

1. Attacker takes a large flash loan (no collateral needed).
2. Uses loan to **distort pool reserves** (e.g., swap ETH for DAI in huge size).

   * Oracle (Uniswap, Curve) now reports manipulated price.
3. Protocol reads oracle:

   * ETH “appears” very cheap or very expensive.
4. Attacker exploits:

   * Deposits overpriced collateral → borrows too much.
   * Or underprices target → liquidates victims cheaply.
5. Repays flash loan, keeps profit; protocol left insolvent.

---

## 3. **Real Proof (Case Studies)**

* **bZx (2020, multiple exploits)**

  * Uniswap pool used as oracle.
  * Flash-loaned ETH, manipulated price, drained protocol.
  * Losses across incidents > **\$8M**.

* **Harvest Finance (2020)**

  * Manipulated Curve stablecoin pools via flash loans.
  * Vaults mispriced → drained **\$24M**.

* **Cheese Bank (2020)**

  * Oracle flaw + flash loan → **\$3.3M** lost.

---

## 4. **Defense (Mitigations)**

* **Robust oracles**

  * Chainlink, Pyth → aggregate off-chain data, resistant to in-tx manipulation.
* **TWAP (Time-Weighted Average Price)**

  * Average over N blocks, not a single block’s reserves.
* **Multiple sources**

  * Median across different DEXes + Chainlink.
* **Liquidity checks**

  * Ignore pools below X liquidity depth.
* **Circuit breakers**

  * Halt borrowing/liquidations if price changes > X% per block.
* **Flash loan defenses**

  * Limit function scope per tx (e.g., cannot deposit and borrow in same block).

---

## 5. **Q\&A Practice**

**Q1.** *Why are flash loans powerful in oracle manipulation?*

* Let attacker temporarily command massive liquidity → move AMM price drastically.
* No capital risk (must repay in same tx).
* Perfect for single-block manipulation.

---

**Q2.** *Why is a Uniswap spot price unsafe as an oracle?*

* Uniswap price = `reserveB / reserveA`.
* Shallow pools are cheap to skew with flash loans.
* Spot price = 1 block snapshot, not robust against manipulation.

---

**Q3.** *How does a TWAP mitigate oracle manipulation?*

* TWAP averages price over time (say 30 min).
* Attacker must sustain manipulation for many blocks → much more expensive.
* ❌ Still vulnerable to long-term griefing if attacker can hold capital.

---

**Q4.** *How could you audit a lending protocol’s oracle usage?*

* Check if oracle uses **spot vs TWAP vs Chainlink**.
* Verify **liquidity depth** of pools used.
* Look for **single-point-of-failure feeds**.
* Ensure **freshness checks** on price data.

---

**Q5.** *Give an example of a combined exploit (oracle + another bug).*

* bZx used oracle manipulation + over-collateralization flaw.
* Some attacks combine oracle manipulation with **reentrancy** (liquidate during manipulated price).

---

## 6. **Traps (Gotchas)**

* **Assuming TWAP = safe** → griefers can still manipulate over time.
* **Low liquidity pools** used in oracles = cheap to attack.
* **Cross-protocol dependencies**: If one protocol relies on another’s pool, cascade risk.
* **Governance oracles**: Manipulated prices may affect voting power.
* **Single source** = single failure point.

---

# 💵 Stablecoin Depeg Scenarios – Study Card

---

## 1. **What it is**

Stablecoins are tokens pegged to an external value (usually USD).

* **Peg = 1:1 relationship** between stablecoin and underlying asset/collateral.
* A **depeg** happens when stablecoin price drifts significantly from \$1.

Types:

* **Fiat-backed (USDC, USDT, BUSD)** → depend on reserves + trust in issuer.
* **Crypto-collateralized (DAI, sUSD)** → depend on over-collateralization and oracles.
* **Algorithmic (UST, IRON, USDN)** → rely on mint/burn mechanics, most fragile.

---

## 2. **How it works (Exploit / Failure Flow)**

### Case 1: **Collateral Shock**

* Collateral falls quickly (e.g., ETH crash for DAI).
* Not enough collateral to back supply.
* Peg breaks → stablecoin trades below \$1.

### Case 2: **Redemption Risk**

* Centralized issuers (like USDC) → if reserves frozen, holders can’t redeem.
* Example: USDC depegged to \~\$0.87 (March 2023) when \$3.3B stuck in SVB.

### Case 3: **Algorithmic Death Spiral**

* Terra/UST model: mint UST by burning LUNA.
* If UST < \$1 → arbitrage requires minting LUNA → inflates supply → crashes LUNA further → peg collapses.

### Case 4: **Oracle / AMM Manipulation**

* Attacker manipulates oracle or pool ratio, making stablecoin appear “off-peg.”
* Triggers bad liquidations or arbitrage drains.

---

## 3. **Real Proof (Case Studies)**

* **Terra/UST Collapse (2022)**

  * \$40B+ evaporated.
  * Algorithmic model collapsed during sell-off → UST fell below \$0.10.

* **USDC Depeg (March 2023)**

  * Circle had \$3.3B in Silicon Valley Bank.
  * FUD + withdrawal panic drove USDC to \$0.87 before recovering.

* **Iron Finance (2021)**

  * Partially collateralized algorithmic stablecoin.
  * Panic → mass redemptions → TITAN token went to 0 → IRON depegged.

* **sUSD incidents**

  * Oracle lags during ETH volatility → short-term depegs.

---

## 4. **Defense (Mitigations)**

* **Collateral design**

  * Over-collateralization (DAI).
  * Diversified collateral types (avoid single failure point).

* **Oracle robustness**

  * Use aggregated feeds (Chainlink) to price collateral and stablecoin.
  * Deviation checks (ignore >10% sudden moves).

* **Circuit breakers**

  * Pause redemptions/trading if peg deviation too large.

* **Liquidity depth**

  * Deep AMM pools (Curve, Uniswap) make depegs harder to exploit.

* **Transparency & proof-of-reserves**

  * Critical for fiat-backed stables like BUSD, USDC.

---

## 5. **Q\&A Practice**

**Q1.** *Why did UST fail while DAI survived ETH crashes?*

* UST: purely algorithmic, reflexive mint/burn → unstable in panic.
* DAI: over-collateralized with ETH/USDC, liquidation mechanisms protect peg.

---

**Q2.** *What’s the biggest risk to fiat-backed stables?*

* Custodian risk (bank collapse, frozen funds).
* Centralization (issuer can blacklist addresses).
* Proof-of-reserves not always transparent.

---

**Q3.** *How can protocols protect against stablecoin depeg risk?*

* Limit exposure (cap % of collateral in one stablecoin).
* Use multiple stablecoins (DAI, USDC, USDT).
* Monitor deviation with on-chain oracles.

---

**Q4.** *How does Curve help in peg stability?*

* StableSwap invariant gives deep, low-slippage liquidity for stables.
* Arbitrageurs trade to restore 1:1 peg.
* But: can also be used in **depeg attacks** if attacker imbalances pool.

---

**Q5.** *What happens in lending protocol if stablecoin collateral depegs?*

* Example: user deposits 1M USDC at \$1 peg.
* If USDC falls to \$0.80, protocol overvalues collateral → toxic bad debt.
* Liquidators may exploit → insolvency.

---

## 6. **Traps (Gotchas)**

* **Assuming 1 USDC = \$1 forever** → ignores custodian/bank risks.
* **Oracle lag**: depegs happen quickly, oracles may not update fast enough.
* **Stablecoin used as collateral**: if it depegs, lending protocols break.
* **Reflexivity in algorithmic designs**: redemption mechanisms amplify panic.
* **Liquidity flight**: stablecoin peg depends on deep pools; if LPs withdraw → peg fragile.

---

