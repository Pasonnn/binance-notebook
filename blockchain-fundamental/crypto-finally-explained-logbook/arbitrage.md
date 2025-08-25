
# 🔄 Arbitrage – Study Card

---

## 1. **What it is (Definition)**

* **Arbitrage** = taking advantage of price differences for the same asset across markets.
* Buy where it’s **cheaper**, sell where it’s **more expensive**.
* In crypto:

  * **CEX–CEX arbitrage** (Binance vs Coinbase).
  * **DEX–DEX arbitrage** (Uniswap vs Curve).
  * **Cross-chain arbitrage** (ETH price differs across chains).
* Constraints: gas fees, slippage, transaction speed.
* In DeFi, **bots compete** — success often comes down to being fastest.

---

## 2. **How it works (Exploit Flow)**

### Simple DEX Arbitrage Example

1. ETH/USDC price = 5000 on Uniswap.
2. ETH/USDC price = 5050 on Curve.
3. Bot buys ETH on Uniswap at 5000.
4. Instantly sells ETH on Curve at 5050.
5. Profit = (5050 – 5000) – gas & fees.

### Flash Loan Arbitrage

1. Take flash loan of 10,000 USDC.
2. Buy ETH on Uniswap at low price.
3. Sell ETH on Curve at higher price.
4. Repay loan + fee in same tx.
5. Keep profit → no upfront capital risk.

---

## 3. **Real Proof (Case Studies)**

* **Flash Boys 2.0 paper (2019)** → highlighted how bots extract arbitrage & MEV on Ethereum.
* **MEV Bots on Ethereum**

  * Billions earned via arbitrage across AMMs (Uniswap, Sushiswap, Curve, Balancer).
* **Curve pool imbalances**

  * During USDT/USDC imbalances, arbitrageurs profit by restoring peg.

---

## 4. **Defense (Mitigations / Considerations)**

* Not really “defense” since arbitrage = market efficiency. But:

  * **Users**: protect against sandwich/front-running bots (set low slippage).
  * **Protocols**:

    * Use **TWAP oracles** so one big trade doesn’t instantly trigger arbitrage exploits elsewhere.
    * Batch auctions (fair sequencing) reduce MEV arbitrage.
* **Traders** must account for:

  * Gas fees (esp. on Ethereum).
  * Slippage → reduces expected profit.
  * Latency → if slower, arbitrage gone.

---

## 5. **Q\&A Practice**

**Q1.** *Why is arbitrage important in DeFi?*

* Keeps markets efficient (prices aligned across AMMs).
* Arbitrageurs are like “free market makers” correcting prices.

---

**Q2.** *What’s the role of flash loans in arbitrage?*

* Allow traders to arbitrage without initial capital.
* Must repay in same block.
* High risk if trade fails → tx reverts.

---

**Q3.** *What’s the risk of arbitrage in shallow liquidity pools?*

* Big trades → high slippage → potential losses.
* Bots often skip pools with low depth since fees kill profit.

---

**Q4.** *What is MEV arbitrage?*

* Validators (or bots paying validators) reorder transactions to capture arbitrage first.
* Example: user submits large swap → validator front-runs with arbitrage.

---

**Q5.** *How can arbitrage create opportunities for attackers?*

* Combined with flash loans + oracle manipulation:

  * Attacker manipulates price in small pool.
  * Arbitrage bot “corrects” → attacker profits.
  * Or attacker times exploit during arbitrage loop.

---

## 6. **Traps (Gotchas)**

* **Gas fees**: arbitrage may appear profitable but tx cost wipes gains.
* **Competition**: Many bots → only fastest wins.
* **Atomicity risk**: Flash loan arbitrage must succeed; if fails, tx reverts (you lose gas).
* **Cross-chain latency**: Arbitrage harder across chains since block times differ.
* **MEV**: Validators/bots can steal your arbitrage by copying tx with higher gas bid.

---
