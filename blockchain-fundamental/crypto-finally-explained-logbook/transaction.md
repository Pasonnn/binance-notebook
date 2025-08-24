
# ⚡ Blockchain Transactions

---

## 1. What is a Transaction?

A **transaction** is any action that changes the state of a blockchain. Examples:

* Trading 1:1 with a friend.
* Swapping money for a physical good.
* Sending or receiving cryptocurrency.
* In blockchain games: winning an item → the item is sent to your wallet → that’s also a transaction.

👉 **Definition:** A **blockchain transaction** is any form of interaction between an individual (wallet) and the blockchain.

* All transactions are **public** (visible on the blockchain).
* Wallet addresses are pseudonymous → people only know it’s yours if you reveal it.

---

## 2. Lifecycle of a Transaction (Ethereum PoS, EVM example)

### Step 0: User Action

* You click **Sign & Send** in your wallet.
* Wallet prepares the transaction object with fields:

  * **nonce** (transaction number for your account).
  * **to/value/data** (receiver, ETH amount, calldata).
  * **gasLimit** (max gas you’ll spend).
  * **maxFeePerGas** & **maxPriorityFeePerGas** (EIP-1559 fee model).
  * **chainId** (for replay protection).
* Wallet fetches the latest nonce & gas estimate from an RPC node.
* ⚡ Tip: On Ethereum today, most transactions are **Type-2** (EIP-1559 format).

---

### Step 1: Transaction Encoding

* Wallet encodes the transaction:

  * Typed envelope + RLP payload.
  * Hash = `keccak256` of the encoded tx.
* Key fields to remember:

  * **nonce** → sequential counter, prevents replay.
  * **to / value / data** → receiver, ETH amount, function call data.
  * **gasLimit** → max gas allowed.
  * **maxFeePerGas** → maximum gas fee you’ll pay.
  * **maxPriorityFeePerGas** → tip to validator/builder.
  * **chainId** → prevents replay on other chains.

---

### Step 2: Cryptographic Signing

* Wallet signs the tx with your **secp256k1 private key**.
* Produces an **ECDSA signature** → (r, s, v).
* Signature is appended → final raw signed tx (bytes).

---

### Step 3: Broadcast to a Node

* Wallet sends the signed tx via `eth_sendRawTransaction` to an RPC (e.g., Infura, Alchemy, or your node).
* The node runs pre-checks:

  * Signature recovers correct sender address.
  * Nonce matches expected.
  * Balance is enough for value + gas.
  * Gas ≥ intrinsic minimum.
* If valid → tx goes into **mempool** and is gossiped across peers.

---

### Step 4: Mempool Propagation

* Transaction spreads across the **peer-to-peer network**.
* Nodes prioritize higher-fee txs.
* If you resubmit with the same nonce but higher fee → “replacement by fee” occurs.

---

### Step 5: Selection & Ordering

* Two paths in Ethereum PoS today:
  **A. Local building:** proposer selects profitable mempool txs.
  **B. MEV-Boost (PBS):** specialized builders create blocks (including MEV bundles). Proposers pick the most profitable block.
* Your tx is included if:

  * Tip ≥ competitive level.
  * Nonce order is valid.
  * Execution doesn’t revert in simulation.

---

### Step 6: Execution in EVM

* Transactions are executed sequentially:

  * Update balances & contract storage.
  * Record gas usage, logs, events.
* After execution, block roots are computed:

  * **stateRoot** (world state).
  * **transactionRoot** (list of txs).
  * **receiptsRoot** (logs & receipts).

---

### Step 7: Block Proposal

* Ethereum PoS has **12s slots**.
* Elected proposer signs block header + gossips the block.

---

### Step 8: Attestations & Fork Choice

* Other validators verify the block:

  * Header validity.
  * State transitions.
  * Proposer’s signature.
* They “attest” to it.
* **Fork-choice rule (LMD-GHOST):** the chain with most attestations becomes the canonical head.
* At this point → wallets show your tx as **confirmed/included**.

---

### Step 9: Finality

* Every **epoch = 32 slots (\~6.4 min)**.
* If enough attestations → epoch is justified.
* Next epoch → previous epoch becomes **finalized** (Casper-FFG).
* Practically → \~13 minutes for strong finality.

---

### Step 10: Receipts & Indexing

* Once block spreads, RPCs can return data:

  * `eth_getTransactionReceipt` → status, gas used, logs, block number.
* Block explorers index your tx, decode logs, and update token transfers.

---

✅ **Summary**

* A transaction = any blockchain state change.
* On Ethereum PoS:

  1. Wallet builds & signs tx.
  2. Tx propagates to mempool.
  3. Block builders/proposers select it.
  4. Execution → block roots → block proposed.
  5. Validators attest → block finalized (\~13 min).
  6. Tx receipt is indexed → visible in explorers.
