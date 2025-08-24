## Crypto Bridge
- A crypto bridge (aka blockchain bridge or cross-chain bridge) is a protocol that allows you to move digital assets or data between two different blockchains
- Lock-and-Mint 
    - Step 1: Deposit on source chain
        - You send your BTC to the bridge's custodian wallet (could be controlled by a centralized custodian, a federation of signers, or a decentralized validator set)
        - This BTC locked and cannot move
    - Step 2: Verification
        - The bridge monitors the Bitcoin chain and verifies your deposit happened
        - Depending on design:
            - Centralized bridge: a custodian confirms it
            - Decentralized bridge: multiple validators sign a proof (e.g., using multi-sig, MPC, or consensus)
    - Step 3: Mint on target chain
        - After verification, the bridge mints 1 WBTC on Ethereum and sends it to your Ethereum wallet
- Burn and Release (redeeming back to source chain)
    - Step 1: You burn (destroy) 1 WBTC on Ethereum
    - Step 2: The bridge sees the burn event
    - Step 3: It releases the original 1 BTC from the custodian wallet on Bitcoin to your BTC address
Why Bridges Are Risky
Bridges are huge targets for hackers because they often hold large amounts of locked assets
- If attackers break the verification system or steal validator keys, they can mint tokens without real deposits -> leading to infinite money glitch
- Example hacks:
    - Wormhole (2022) - $325M lost due to bypassed signature verification
    - Ronin Bridge (2022) - $600M stolen after validator keys compromised
    - Poly Network (2021) - $611M stolen via contract exploit
=> So, a crypto bridge = lock assets on chain A + mint representation on Chain B, It works through custodians + validators + smart contracts that verify deposits and issue tokens