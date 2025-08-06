# Purpose
The purpose of this proposal is to strengthen EthStorage’s mining algorithm by introducing a Proof-of-Stake (PoS) component. By requiring storage providers (miners) to stake EthStorage tokens, we aim to **increase the network’s economic security** and potentially **adjust mining difficulty** in favor of stakers. In essence, miners would not only prove storage of data but also lock up tokens as collateral. This dual requirement ensures miners have financial “skin in the game,” deterring malicious behavior and Sybil attacks. It also opens the door to reducing mining difficulty or granting other benefits for those who stake, as a way to incentivize honest participation.

# Problem Statement and Context
**Current Mining Requirements**: At present (before PoS integration), anyone who wants to join EthStorage as a data provider must meet certain criteria. 
 - **Data Replication**: The miner must download a full replica of the chosen data shard and maintain it on a local SSD, ensuring high availability and redundancy for that shard.
 - **Accounts Setup**: The miner should prepare two Ethereum accounts: one miner account to receive mining rewards, and one signer account to submit mining transactions. The signer account needs a balance of ETH for gas fees (since submitting proofs to the EthStorage smart contract consumes Ethereum gas)
 - **Running the Node**: The miner runs the es-node software, which connects to the EthStorage network, submits proofs to Ethereum, and participates in reward mechanisms.
 
These requirements ensure that only those who actually store the data (full shard) and can interact with Ethereum can become storage providers. However, **there is currently no token staking requirement**. This opens several potential issues:
 - **Lack of Slashing/Penalty Mechanism**: Currently, EthStorage lacks a direct economic penalty for dishonest or unreliable miners. In decentralized storage networks like Filecoin, miners provide collateral tokens that can be slashed (confiscated) if they fail to honor storage commitments. EthStorage does not yet have such a penalty mechanism, meaning dishonest behavior is discouraged only indirectly through potential reputation loss or missed future rewards.
 - **Integration of the Token Utility**: EthStorage plans to launch its native token, which should have clear utility within the network. Incorporating token staking into mining is a natural fit because it directly links the token’s value to network security, aligns miner incentives with network success, and enhances overall economic robustness. Without utilizing the token for staking, EthStorage would miss a critical opportunity to leverage tokenomics for network security.

Once the EthStorage token is launched, we propose making token staking an additional requirement to qualify for mining. Alternatively (or additionally), staking could be used as a mechanism to adjust the mining difficulty – for example, miners who stake the token get an easier difficulty target, effectively boosting their chances of success. The exact approach needs careful consideration, but the end goal is clear: **incorporate PoS into EthStorage’s mining algorithm to enhance security and efficiency**.

# Proposed Solution
To integrate Proof-of-Stake into EthStorage mining, we outline the following design elements:

## Staking Requirement and Stake Amount
To become eligible to mine, we propose that each miner stake a predefined amount of EthStorage tokens by calling a stake() function on the EthStorage smart contract.

 - **Fixed Stake Amount**: Each miner should stake a fixed minimum amount of tokens to qualify. The exact amount (N) will be determined through economic analysis based on token value, target security level, and affordability. This amount could be adjusted later through governance if necessary. It should be sufficiently high to discourage trivial or malicious miners but accessible enough to allow honest, smaller-scale providers to participate.
 - **Eligibility vs. Continuous Scaling**: An open decision is whether additional staking beyond the minimum grants further advantage. Two primary approaches exist:
   - **Binary Eligibility (Recommended)**: Stake the minimum and gain equal treatment with other staked miners in terms of mining difficulty and rewards. This approach prevents large token holders from achieving disproportionately high influence.
   - **Proportional Staking**: Mining power or the probability of successful proofs could scale proportionally with staked token amount, up to a defined limit.
 - **Difficulty Reduction Option**: As noted, another way to integrate stake is by reducing mining difficulty for those who stake. 

## Delayed Stake Activation
One critical aspect to prevent abuse is introducing a **delay between staking and when that stake becomes effective for mining**. Without a delay, a savvy attacker could exploit the system as follows: imagine a miner could attempt to mine without staking, and only if they find a valid proof, they quickly stake and submit the result in the same transaction, then immediately unstake. This would allow them to gain rewards while essentially having had no stake at risk during the mining process – completely defeating the purpose of PoS security.

To prevent malicious “stake-on-the-fly” behavior, we introduce a stake activation delay:
 - When miners stake tokens (calling `stake()`), the contract records a `stake_timestamp`. Staking benefits (eligibility to mine) only activate after a fixed delay (e.g., 24 hours) from this timestamp. For instance, staking at 12:00 PM Monday activates mining eligibility at 12:00 PM Tuesday.
 - The EthStorage contract enforces this activation rule by validating that the current timestamp meets or exceeds `stake_timestamp + activation_delay` when proofs are submitted.
 - This delay ensures miners commit tokens before becoming eligible, preventing attackers from retroactively staking after generating valid proofs.

## Unstaking and Withdrawal Process
To maintain accountability, unstaking should also involve a controlled withdrawal delay, mirroring stake activation:
 - **Unstake Request**: Miners initiate an unstake by calling the `unstake()` function. Tokens remain locked, marked as “pending withdrawal”, and the timestamp (`unstake_timestamp`) is recorded. Miners become ineligible to mine during this withdrawal pending period.
 - **Withdrawal Delay**: Tokens stay locked for a predefined period (e.g., 24 hours), during which the miner’s stake remains vulnerable to slashing if misbehavior is detected.
 - **Final Withdrawal**: After the delay period, miners call `withdraw()` to retrieve their tokens. The smart contract validates that `current_time >= unstake_timestamp + withdrawal_delay` before releasing the stake.

# Risks & Uncertainties
  - **Slashing Mechanism**: One of the most important advantages of PoS is the ability to slash (confiscate) the staked tokens of a misbehaving miner. Slashing provides a strong disincentive for any actions that harm the network. We need to define what constitutes a slashable offense in EthStorage’s context, and how the mechanism will work.
  - **Increased Barrier to Entry**: Requiring a token stake means some interested storage providers might be unable to join if they don’t have enough capital to buy the required tokens. This could reduce the number of miners, especially early on. If the network ends up with too few miners (reducing redundancy), that’s a problem.
