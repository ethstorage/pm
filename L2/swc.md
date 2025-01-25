TL;DR

- Ethereum and its layer 2 solutions are bringing its vision of the "world computer" closer to reality: a permissionless, global operating system designed to replace centralized, trusted intermediaries.
- However, to truly serve global users, the world computer faces significant challenges, including **high costs for validating off-chain computation, EVM performance bottlenecks, storage limitations, barriers to onboard web2 users, and reliance on centralized frontends**.
- **We present the Super World Computer (SWC) Project**: a highly customized L2 built on OP Stack to overcome these challenges.
    - **Advanced fault-proof algorithms** will lower validation costs and accelerate settlement times.
    - **Parallel EVM execution** will boost performance and lower latency.
    - **EthStorage, integrated as Layer 3,** will enable long-term, cost-effective storage for massive datasets.
    - Introduction of a **non-transferable Soul Gas Token** to ease the Web2 to Web3 transition by reducing onboarding costs.
    - A new **web3:// standard** will enable fully decentralized, on-chain applications and dynamic websites.
- The roadmap includes testnet and mainnet launches in **2024–2025,** focusing on performance and scalability improvements.

# What is the super world computer?

Ethereum is often referred to as the "[world computer](https://www.youtube.com/watch?v=j23HnORQXvs).” The "world computer" refers to a decentralized network that functions like a global operating system. It allows users to run applications, store data, and execute smart contracts without centralized control. The concept emphasizes a system where computation and data storage are distributed across many nodes, providing transparency, security, availability, and resistance to censorship.

Unfortunately, due to the limited computational power of Ethereum itself (e.g., in terms of transactions per second), the mission of the world computer to serve people around the world is greatly limited. To address the scalability problem, Ethereum is moving towards its layer 2 (L2), a.k.a., the Rollup-centric roadmap.  The idea is to submit the off-chain transactions to Ethereum L1 and then validate the correctness of off-chain execution using fault proof or validity proof.  In this way, the correctness of the execution result is guaranteed by the Ethereum L1, while the capacity of off-chain execution can be greatly increased. 

While the Rollup approach has the potential to achieve hundreds or even thousands of TPS (transactions per second), Ethereum and its L2 solutions are still missing some critical components needed to become the world computer that serves global users without centralized control:

- **High user onboarding cost**. The gap between traditional web users and the emerging world of Web3 remains considerable, particularly due to the upfront cost of acquiring a gas token. Although airdrops attempt to alleviate this issue, they must carefully identify genuine users to avoid unnecessary token sell-offs. Bridging this gap is a core challenge for realizing the world computer vision.
- **EVM performance limit**. The sequential execution model of the EVM imposes a significant cap on the upper bound of L2 TPS. As a result, running internet-scale applications on L2 remains impractical.
- **High storage cost even with the current L2 approach**. A single node capacity limits the storage capacity of an L2 network, typically a few terabytes.  This constraint means that L2 networks struggle to support internet-scale applications, such as Twitter, which generates approximately 12TB of data daily.
- **Centralized frontend**. Most of the frontends of Ethereum applications heavily rely on centralized servers and DNS mapping, which inherits all the drawbacks of centralized applications, such as single point of failure and censorship attacks. Although ENS and IPFS provide some level of censorship resistance, the web content is static and may disappear at any time.
- **High validation costs on Ethereum**: The validity proof approach demands powerful and expensive proving devices. Moreover, upgrading Layer 2 with new EVM EIPs incurs additional implementation costs in circuit design, which increases the risk of bugs, leading to what's known as "EVM-upgrade scary." While the fault-proof approach is more friendly to EVM upgrades, it involves long settlement times on Layer 1 (up to 7 days) and can become costly during periods of network congestion.

To address all these problems, we are introducing **the super world computer - a highly customized OP Stack rollup** designed to bring the vision of the world computer to life, backed by years of research and development.

|  | Existing Ethereum L2s | Super World Computer |
| --- | --- | --- |
| User Onboarding Cost | Upfront cost to buy ETH as gas token | No cost by receiving Soul Gas Token  |
| Processing | Sequential execution/IO model with ~200M gas/s | Parallel execution/IO model with > 1G gas/s |
| Storage | Limited by single node capacity | 1000x of single node capacity |
| Frontend | DNS + http:// | ENS + web3:// |
| Security | Fault proof or validity proof | Advanced fault proof with fast settlement |

# Our Approach

## Soul Gas Token

To bridge the gap between traditional web users and the growing world of Web3, SWC will adopt a **non-transferable gas token** named Soul Gas Token, where the SWC token (ERC20) is the native gas token of the SWC Rollup, thanks to the recent custom gas token feature of OP Stack.

The concept of Soul Gas Token revolves around facilitating Web2 users' entry into Web3 by airdropping them with a **non-transferable** gas token. This token will enable users to pay for transaction gas fees without the immediate selling pressure. Additionally, this feature does not require support from new wallets—existing EOA wallets will automatically prioritize using the Soul Gas Token for payment before drawing from the user's account balance. This initiative is particularly aimed at those new billions to Web3, providing a seamless transition without the upfront cost of acquiring a gas token.

## Pushing EVM Performance Limits with Parallelization

Parallel EVM emerges as a standout solution for enhancing EVM execution speeds. By leveraging parallel program execution and data read/write operations, it taps into the concurrency capabilities of modern computers, which are typically equipped with 4-8 CPU cores and boast high-bandwidth I/O.

Our exploration in this area focuses on the performance limits of the EVM through parallel optimistic execution, also known as Block STM, and parallel I/O—a strategy designed to reduce latency and boost performance by parallelizing EVM data reads. Some key areas of exploration include:

- **EIP-7650: Programmable Access Lists**, which enables contracts to define multiple data locations for parallel preloading. This method can sustainably reduce the latency of reading multiple data and thus lower gas costs beyond what EIP-2930 offers.
- **Intelligent I/O Preloading**: By designing an EVM capable of learning the common read locations (via SLOAD and CALL) within a contract method, it can preload data in parallel during contract method invocation, thus leveraging I/O parallelization benefits even without explicit access lists.

## Programmable Large Storage with EthStorage as L3

Ethereum’s data availability roadmap allows for the publication of large amounts of data on-chain. However, to limit validator storage requirements, Ethereum discards the DA data (namely binary large objects, BLOBs) older than 18 days. EthStorage, a storage L2, offers a solution for long-term BLOB storage by maintaining a key-to-BLOB-hash mapping on-chain while proving the storage off-chain. This approach reduces storage costs by a factor of 1,000, with 1,000x the capacity, enabling storage of massive datasets far beyond the limitations of sequencers or validators. Additionally, EthStorage’s key-to-BLOB interface enables **programmable large storage**—a critical infrastructure for supporting Web2 applications.

However, programming storage on Ethereum L1 is prohibitively expensive; the gas cost of maintaining on-chain metadata can exceed the storage cost itself. To mitigate these costs, SWC L2 will **integrate EthStorage as L3**, reducing the cost of storing a BLOB to approximately 1/10,000th of the mainnet cost. This dramatically reduced storage cost on the SWC L2 opens up new possibilities for applications on Ethereum, such as fully on-chain AI with all models and training data stored in EthStorage.

## web3:// - Transforming EVM into the Unstoppable Web Server

As the Ethereum scalability initiatives, including our super world computer plan, advance, they set the stage for widespread adoption of fully on-chain applications. However, an essential component remains absent in Ethereum's evolution: a decentralized protocol facilitating direct access to on-chain resources like NFT images and dynamic on-chain websites.

To address this, we proposed a new standard that defines HTTP-style web3:// links in ERC-4804/6860 for navigating dynamic on-chain resources maintained by smart contracts. As a result, it effectively transforms the Ethereum virtual machine (EVM) into an unstoppable decentralized HTTP server. Combined with the programmable large storage enabled by EthStorage, we are bringing to life **the fully decentralized web**, or web3,  a re-architected internet where all centralized entities are replaced by permissionless protocols.

## Advanced Fault Proof Algorithm Supported By OP

OP Stack has recently launched an on-chain fault-proof algorithm, officially promoting Optimism to stage 1.  However, a key concern with fault proofs is that resolving a dispute requires about 73 interactions between the defender and the challenger. To address this, we propose a multi-section fault dispute game that incorporates multiple intermediate claim hashes for each interaction. This approach is expected to reduce the number of interactions and associated gas costs by up to 90%. We are pleased to have received an OP [grant](https://app.charmverse.io/op-grants/page-29596258544520615) to develop this solution further.

|   | Moves | Per move gas cost | Total gas cost | Savings  |
| --- | --- | --- | --- | --- |
| Current (N = 2) | 73 | ~250,000 | ~18,250,000 | 0% |
| N = 16 with calldata | 19 | ~258,192 | ~4,905,648 | 73% |
| N = 256 with calldata | 10 | ~381,072 | ~3,810,720 | 79% |
| N = 4096 with EIP-4844 | 8 | ~250,000 | ~2,000,000 | 89% |

(caption: The estimated on-chain interactions and gas cost of current and proposed methods)

# Roadmap

*(Please note that many of these tasks are in collaboration with partners such as OP Labs, EthStorage, and timelines may be subject to change)*. 

## 2024 Q3 (Finished)

We have launched our first SWC L2 alpha testnet, utilizing the test SWC ERC20 token as a custom gas token. Additionally, we will introduce the inbox contract feature within OP Stack - a capability that allows on-chain smart contracts to validate off-chain batches. This feature will enable functionalities like a decentralized sequencer network and long-term BLOB storage through EthStorage L2.

## 2024 Q4/2025 Q1 (Ongoing)

We are preparing to launch the SWC L2 beta devnet/testnet with new features, including EthStorage as L3 and the soul gas token. To support EthStorage as L3, we will also enable BLOB transactions on OP Stack L2, which is essential for enabling programmable storage capabilities. Meanwhile, we are performing extensive tests to meet high-standard code quality including unit tests, end-to-end tests, integration tests, and tests on devnet.

## 2025 H1

As our beta testnet nears maturity, we plan to launch the SWC L2 mainnet with the default fault-proof implementation. Alongside the mainnet launch, we will collaborate closely with OP Labs on upstream features, including the inbox contract and fault-proof for custom gas tokens, which are targeted for the OP Stack's future upgrades. Additionally, we will introduce our advanced multi-section fault proof on the devnet.

## 2025 H2

Once our multi-section fault proof reaches maturity and passes auditing, we will deploy it on the testnet/mainnet. Simultaneously, we will launch our high-performance EVM testbeds on SWC L2.

*updated on Dec. 18, 2024*
