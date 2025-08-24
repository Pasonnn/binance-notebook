## Oracle
### Definitions
- Oracles are entities that provide real-world data, or information, to smart contracts on blockchain networks
- Software Oracle: Web based database to check the data, help smart contract find nessesary data to execute.
- Hardware Orable: They are hardware in physically environment, more security.
- Crypto Oracle use cases:
    - Chainlink: Most popular oracle software on DeFi
    - For example: Aave use oracle to validate your token A price to tracks your deposit value, and calculate the LTV (Loan to Value). If price of your token A drop to 82.5% of your borrow, your deposited coin A get liquidation.
- Challenge and Risk:
    - Single point of failure: If someone manipulate all data sources of oracle, it may make oracles failing and take advantages of that failing data 
