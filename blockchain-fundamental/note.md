# Blockchain Fundamentals Logbook - Learn DeFi from Crypto Finally Explained

## Liquidity Pool
- Liquidity pool let you trade immediately (dont need to wait people to trade with you).

## AMM
- AMM is stand for Automated Market Maker
- AMM algorithm (basic): Coin_X * Coin_Y = Z
    - For example, if the Liquidity Pool have 1000 coinx, 1000 coiny so the z will be 1.000.000. If you trade 200 coin x for coin y, you will get 167 coin y instead of 200
- Liquidity Providers will invest their coin and get a small amount of fee from traders that interact with the pool
- The AMM algorithm get really complex, and have more than 2 assets inside it at the time.

## Defi 2.0 
- DeFi stand for Decentralize Finance.
- DeFi 2.0 created to solve the core issues of DeFi 1.0
- Every function of DeFi or dApp is powered by smart contract, no human false behind it
- DeFi 2.0 for example OlympusDAO pioneered this. Protocol sells its native token at a discount (bonding) to buy liquidity pool (LP) tokens so liquidity is now protocol-controlled

## Transaction
- What is a transaction:
    - Trade 1:1 with your friend
    - Swap by money - trade money for physical good
- When you send or receive crypto is a transaction
- Everything is a transaction in crypto (for example you play game and win an item, it send to your wallet -> it is a transaction)
- => Blockchain transaction is any form of interaction between an individual and the blockchain.
- Every transactions on blockchain will be public, but they wont know who (the wallet address) if you not tell that the wallet is yours.
- Step by step a transaction from wallet to finalized block (EVM PoS)
    - 0: You click sign, your wallet has already
        - Build a transaction (nonce, to, value, data, gasLimit, maxFeePerGas, maxPriorityFeePerGas, chainId)
        - Pulled nonce and did a gas estimate from some RPC node
        " Tip: On Ethereum, most new transactions are Type-2 (EIP-1559) "
    - 1: Constract the message to sign
        - The wallet encodes the tx (typed exvelope + RLP payload).
        - It forms the exact hash to sign (keccak256) over the typed tx per EIP-2718/1559).
        - Fields you should know cold:
            - nonce: sequential per account (prevents replay/refines order)
            - to/value/data: destination, ETH emount, calldata (function + args).
            - gasLimit: upper bound of gas your are willing to burn
            - maxFeePerGas: cap you'll pay per gas (includes base fee + tip)
            - maxPriorityFeePerGas: the tip to the proposer/builder
            - chainId: EIP-155 replay protection
    - 2: Cryptographic signing
        - Wallet signs with your secp256k1 private key -> ECDSA signature (r, s, v)
        - The signature is appended to the tx object -> raw signed tx (still bytes)
    - 3: Broadcase to a node
        - Wallet calls eth_sendRawTransaction to an RPC provider (Infura,Alchemy/your node)
        - That node pre-validates:
            - Signature recovers your sender address
            - Nonce equals your next expected nonce
            - You have enough balance for value + (gasLimit x maxFeePerGas)
            - Intrinsic gas check (tx cant be under-gassed)
        - If ok, the node drops it into its mempool and start gossiping it across peers.
    - 4: Mempool propagation
        - Peers spread the tx via p2p protocol (devp2p/ETH)
        - Many nodes prioritize higher-typ txs and may evict very low-fee ones.
        - If you send another tx with the same nonce but higher fees, it replaces the older one in mempools ("replacement by fee")
    - 5: Delection and ordering (builder/proposer)
        - Two common paths on Ethereum today:
        - A. Local building (simplified):
            - A slot proposer (chosen validator) builds a block by taking the most profitable mempool txs (maximize tips + MEV), simulating them and ordering them
        - B. PBS/MEV-Boost (what most proposers use):
            - Specialized block builders assemble candidate blocks (including MEV bundles).
            - Relays forward these to the proposer
            - The proposer picks the highest-paying block (maximizes value), then signs it
        - Either way, your tx gets included if: 
            - It pays a competitive effective tip (min(maxFeePerGas - baseFee, maxPriorityFeePerGax))
            It doesnt conflict with earlier nonces from your account
            It doesnt revert during the builder simulation
    - 6: Execution during block building
        - Transactions execute sequentially on the current state via EVM
        - For each tx, the builder.proposer computes:
            - State updates (balances, contract storage).
            - Gas used, logs (events), and a receipt
        - After all txs:
            - stateRoot (Merkle-Patricia trie of world state)
            - transactionRoot (Merkle root of tx list)
            - receiptsRoot (Merkle root of receipts)
            - logsBloom, gasUsed, baseFee, prevRandao, parentHash, etc are sealed into the block header
    - 7: Block proposal (slot time)
        - In Ethereum PoS, time is split into 12s slots
        - The elected proposer for a slot signs the block header and gossips the full block to the network
    - 8: Attestations & Fork-choice (becoming canonical)
        - Other validators int he slot's committee verify the block:
            - Check header validity, execute the block to re-derive roots, verify signature
        - They attest to it. The fork-choice rule (LMD-GHOST) makes the chain with the most recent attenstations the head
        - Once your block is the HEAD, most wallets/explorers show your tx as "confirmed/included"
    - 9: Finality (very hard to revert)
        - Every epoch = 32 slots ~ 6.4 mins
        - With enough attestations, an epoch is justified, and with the next justified epoch, the prior one is finalized (Casper-FFG)
        - Pratically, your block is final about 2 epochs (13 mins) after inclusion under normal confitions
    - 10: Receipts & Indexing
        - As soon as the block spreads, RPCs can answer:
            - eth_getTransactionReceipt -> status, gasUsed, logs, blockNumber, etc.
        - Explores index your tx, decode logs (events), and update token transfers