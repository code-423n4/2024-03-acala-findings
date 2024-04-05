# 🛠️ Acala
**Building the liquidity layer of Web3 finance.**

## Conceptual overview of the project:

The Acala project, encompassing both the Acala and Karura networks, offers a decentralized finance (DeFi) hub on the Polkadot and Kusama ecosystems, respectively. At its core, Acala is designed to provide a suite of financial services and products, leveraging blockchain technology to offer a secure, efficient, and interoperable platform for users.

Users interact with Acala through a variety of DeFi services. One of the primary functionalities is the ability to stake and earn rewards, enabling users to lock up certain assets in return for rewards, thus contributing to the network's security and efficiency. The Earning pallet, for instance, allows users to bond their tokens and earn yields over time, which can be unbonded with or without a penalty, depending on the conditions set by the protocol.

Another significant feature is the LiquidCrowdloan, which permits users to participate in Polkadot's parachain slot auctions by locking up DOT tokens. In return, participants receive a liquid token, representing their staked value, which they can trade, use in DeFi protocols, or stake further to earn additional rewards. This mechanism ensures that users can support their preferred projects in the parachain auctions without losing liquidity.

The project also introduces various precompiled contracts, such as the Xtokens and Homa precompiles, which facilitate cross-chain transfers and operations, enabling users to seamlessly move assets across different blockchains within the Polkadot ecosystem. This interoperability is a key aspect of Acala, as it allows for a wide range of financial activities without the usual barriers seen in traditional finance.

StableAssets and the StableAsset precompile provide the functionality for creating and managing stablecoins, which are crucial for DeFi applications by offering a stable medium of exchange, lending, borrowing, and earning interest. The stablecoin mechanism within Acala ensures that users can transact with minimal price volatility compared to other cryptocurrencies.

From a user perspective, Acala presents a comprehensive DeFi platform that not only offers a range of financial services, such as staking, liquidity provision, and stablecoin management, but also integrates these services within a cross-chain ecosystem. Its focus on interoperability, liquidity, and user participation makes it a pioneering project in the DeFi space, aiming to bridge the gap between various blockchains and provide a unified platform for financial activities.

<br/>

[![Screenshot-from-2024-04-05-19-23-46.png](https://i.postimg.cc/SR35mXVT/Screenshot-from-2024-04-05-19-23-46.png)](https://postimg.cc/2LdTwSsh)

## System Overview

### Pallets:
- **Incentives**: Manages rewards and incentives for liquidity providers and stakers.

### Breakdown of the Functions:
- **deposit_dex_share**: Allows users to deposit DEX shares to earn rewards.
- **withdraw_dex_share**: Enables users to withdraw their DEX shares.
- **claim_rewards**: Users can claim their pending rewards.
- **update_incentive_rewards**: Admin function to update incentive rewards for various pools.
- **update_claim_reward_deduction_rates**: Adjusts the rate at which rewards are deducted upon claiming.
- **update_claim_reward_deduction_currency**: Sets the currency in which reward deductions are taken.
- **on_initialize**: Routine run at the beginning of each block to distribute rewards.

### Key Functions:
- **claim_rewards**: Central function for users to claim their accumulated rewards.
- **update_incentive_rewards**: Critical admin function to set or adjust the rewards users can earn from different pools.
- **on_initialize**: Automated function that triggers at the start of each block to process and distribute incentives based on defined parameters and user actions.







### Pallets:
- **Earning**: Manages bonded assets, allowing users to earn rewards by locking up tokens.

### Breakdown of the Functions:
- **bond**: Allows users to bond tokens to participate in earning rewards.
- **unbond**: Enables users to start the unbonding process for previously bonded tokens.
- **unbond_instant**: Users can instantly unbond tokens by paying a fee.
- **rebond**: Reverts the unbonding process, rebonding previously unbonded tokens.
- **withdraw_unbonded**: Completes the unbonding process, making funds available for withdrawal after the unbonding period.

### Key Functions:
- **bond**: Core function for users to participate and lock their tokens to start earning rewards.
- **unbond** and **unbond_instant**: Key functions for users to retrieve their bonded tokens, with an instant option available for a fee.
- **withdraw_unbonded**: Essential for users to claim their unbonded tokens after the specified unbonding period, finalizing the withdrawal process.







### Pallets:
- **Rewards**: Manages distribution and calculation of rewards for different activities within the protocol.

### Breakdown of the Functions:
##### Reward Accumulation:
- **accumulate_reward**: Adds rewards to a specific pool, increasing the total rewards available for distribution.

##### Share Management:
- **add_share**: Allocates shares to a participant, allowing them to earn rewards based on their share.
- **remove_share**: Removes a participant's shares, adjusting their potential rewards.

##### Reward Claiming:
- **claim_rewards**: Allows participants to claim their earned rewards from a specific pool.
- **claim_reward**: Enables claiming of rewards for a specific currency from a pool.

##### Share Adjustment:
- **set_share**: Sets the exact share amount for a participant, adjusting their reward potential directly.
- **transfer_share_and_rewards**: Transfers share (and potentially rewards) from one account to another, maintaining reward continuity.

### Key Functions:
- **accumulate_reward**: Fundamental for adding new rewards to pools, affecting the total rewards participants can earn.
- **add_share** and **remove_share**: Central to managing participant's involvement in reward pools, directly impacting their reward potential.
- **claim_rewards**: Critical for participants to retrieve their earned rewards, realizing their gains from participation in the protocol activities.




  


## Codebase Quality

Overall, I consider the quality of the Acala codebase to be of high caliber. The codebase exhibits mature software engineering practices with a strong emphasis on security, modularity, and clear documentation. 

| Codebase Quality Categories                     | Comments |
| ----------------------------------------------- | -------- |
| **Code Organization and Modularity**            | The codebase is well-organized, with distinct modules or pallets responsible for specific functionalities. This modular design aids in maintainability and scalability. |
| **Documentation and Comments**                  | Documentation within the codebase is thorough, including comments within the code that explain the functionality and purpose of functions and modules, enhancing understandability for new contributors. |
| **Testing and Coverage**                        | The project includes comprehensive tests covering various scenarios and edge cases. The presence of mock implementations and tests indicates a commitment to ensuring code reliability. |
| **Security Practices**                          | The codebase demonstrates a strong focus on security, with checks and validations to prevent overflow, underflow, and unauthorized access. Previous audits and prompt fixes to reported issues show a proactive approach to security. |
| **Error Handling and Logging**                  | Error handling is comprehensive, with clear distinctions between different error types and appropriate logging for critical events, aiding in debugging and monitoring. |
| **Consistency and Coding Standards**            | The project adheres to Rust's idiomatic practices and the Substrate framework's guidelines, contributing to a consistent and high-quality codebase. |
| **Dependencies Management**                     | Dependency management appears to be handled cautiously, with dependencies kept up-to-date and efforts to minimize unnecessary external dependencies, reducing potential security risks. |





## Core Functionality
1. **Decentralized Finance (DeFi) Ecosystem**: At its core, Acala serves as a DeFi hub within the Polkadot ecosystem, providing a suite of financial applications including a decentralized exchange (DEX), a stablecoin platform, and staking derivatives, catering to a wide range of DeFi activities.

2. **Stablecoin Platform (aUSD)**: One of Acala's flagship features is its decentralized stablecoin, aUSD, pegged to the USD. This stablecoin is central to the ecosystem, enabling stable value exchange, lending, borrowing, and yield generation.

3. **Decentralized Exchange (DEX)**: The DEX allows for seamless trading between various assets on the Acala network, supporting liquidity provisioning, yield farming, and other DeFi activities, enhancing the ecosystem's liquidity and accessibility.

4. **Liquid Staking (LDOT)**: Acala's liquid staking solution allows users to stake their DOT tokens while maintaining liquidity through LDOT, a derivative that represents staked DOT. This mechanism enables users to participate in network security while engaging in other DeFi activities.

5. **Earning Module**: The earning module provides users with opportunities to earn rewards through various DeFi strategies, optimizing yield across different assets and protocols within the Acala ecosystem.

6. **Cross-chain Capabilities (XCM)**: Leveraging Polkadot's cross-chain message passing (XCM), Acala facilitates interoperability and asset transfers across different blockchains within the Polkadot network, broadening its DeFi services reach.

7. **Governance and Upgradability**: Acala features a governance model that allows token holders to participate in decision-making processes, contributing to the network's evolution and upgradability, ensuring the protocol remains adaptable and community-driven.




### Liquid Staking (LDOT) Overview:

Liquid Staking on Acala allows users to stake their DOT tokens (Polkadot's native cryptocurrency) and receive LDOT in return. LDOT represents the staked DOT but remains liquid, meaning it can be traded, used in DeFi protocols for lending, borrowing, or providing liquidity, without losing the ability to earn staking rewards. This process enhances capital efficiency and ensures users don't have to choose between participating in network security and engaging in other DeFi activities.

### Workflow:

1. **Stake DOT**: Users stake their DOT tokens through the Acala platform. This action is facilitated by the protocol, which manages the staking process on the Polkadot network.

2. **Receive LDOT**: In return for staking DOT, users receive an equivalent amount of LDOT tokens. LDOT represents the staked DOT plus any staking rewards earned, adjusted for any slashing events.

3. **Use LDOT in DeFi**: Holders can use LDOT across various DeFi protocols on Acala for lending, borrowing, and liquidity provision, among other uses, without losing their staking position.

4. **Unstake and Withdraw**: Users can choose to unstake their DOT by returning the LDOT tokens. The protocol then initiates the unstaking process on Polkadot, which includes an unbonding period as per Polkadot's staking rules.

5. **Claim Staking Rewards**: While LDOT is staked, it accrues staking rewards. Users can claim these rewards through the protocol, which converts them into additional LDOT or allows for withdrawal in DOT after the unstaking process.

### Code Snippet:

- **Deposit DEX shares**:
    ```rust
	fn deposit_dex_share(who: &T::AccountId, lp_currency_id: CurrencyId, amount: Balance) -> DispatchResult {
    ```
    
- **Withdraw DEX shares**:
    ```rust
    	fn withdraw_dex_share(who: &T::AccountId, lp_currency_id: CurrencyId, amount: Balance) -> DispatchResult {
    ```
    
- **Claiming Rewards**:
    ```rust
    	fn claim_rewards(who: T::AccountId, pool_id: PoolId) -> DispatchResult {
    ```



<br/>

[![download.png](https://i.postimg.cc/HskMpBXZ/download.png)](https://postimg.cc/jCB2c68P)


### Earning Module:
The Earning Module in the Acala project is designed to manage the staking, earnings, and distribution of rewards within the ecosystem. This module facilitates users to bond or lock their tokens to earn rewards, while also providing the flexibility to unbond or unlock these tokens with or without a penalty, depending on the chosen method. Here’s a technical overview and workflow of how the Earning Module operates within the protocol:

### Workflow:

1. **Bonding Tokens**: Users bond their tokens to the protocol to participate in earning rewards. The bonding process locks the tokens, thereby securing them in the protocol's custody and making them eligible for earning rewards based on the protocol's staking and reward distribution mechanisms.

   ```rust
   // Bond tokens
   pub fn bond(origin: OriginFor<T>, amount: Balance) -> DispatchResult {...}
   ```

2. **Earning Rewards**: Based on the protocol's reward distribution logic, bonded tokens earn rewards over time. The module calculates these rewards according to the amount of bonded tokens and the duration for which they are bonded.

3. **Unbonding Tokens**: Users can choose to unbond their tokens. Depending on the protocol's configuration, this can either be instant, with a fee, or it can require waiting for a specified unbonding period without any fee.

   ```rust
   // Start unbonding tokens
   pub fn unbond(origin: OriginFor<T>, amount: Balance) -> DispatchResult {...}
   ```

4. **Withdrawing Unbonded Tokens**: After the unbonding period, users can withdraw their tokens back to their custody. The module ensures that the tokens are no longer earning rewards and are released from the lock.

   ```rust
   // Withdraw all unbonded tokens
   pub fn withdraw_unbonded(origin: OriginFor<T>) -> DispatchResult {...}
   ```

5. **Reward Distribution**: The module periodically calculates and distributes rewards to all participants based on the configured logic. This includes the total reward pool available, individual users' contributions (bonded tokens), and the duration of bonding.


The Earning Module's architecture is critical for incentivizing participation in the protocol and securing the network. By balancing the incentive mechanisms with user flexibility, it plays a pivotal role in the ecosystem's overall health and growth.



<br/>

[![Screenshot-from-2024-04-05-23-02-33.png](https://i.postimg.cc/bwZ9Pb5t/Screenshot-from-2024-04-05-23-02-33.png)](https://postimg.cc/SnpM7XkS)







## Archietecture and WorkFlow
| File Name  | Core Functionality                                                                                                                                                                           | Technical Characteristics                                                                                                                                                                                                                                                                                                                                 | Importance and Management                                                                                                                                                                                                                                        |
|------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `earning.rs`               | This pallet allows users to bond or lock their tokens for a period to earn rewards. Unbonding or unlocking before the term requires a fee/penalty, or waiting for the unbonding period to finish.                                                                                                                         | - Utilizes the `LockableCurrency` trait for token locking.<br>- Implements `OnBonded` and `OnUnbonded` traits for hooking into bonding events.<br>- Manages bonding periods and penalties through configurable parameters.                                                                                                               | High importance for managing user investments and staking within the protocol. Managed through a combination of bonding parameters and the integration of on-chain events to adjust user balances and permissions based on their staking and unstaking actions.                                                         |
| `incentives.rs`            | Facilitates various incentive mechanisms for liquidity provision, staking, and other participatory actions within the ecosystem. It defines reward pools, accumulation of rewards, and distribution mechanisms to users based on their contributions.                                                                         | - Defines reward pools and mechanisms for accumulating and distributing rewards.<br>- Utilizes `MultiCurrency` for handling different types of rewards.<br>- Implements logic for updating incentives and managing claims through user interactions.                                                                                     | Crucial for encouraging user participation and liquidity provision within the ecosystem. It is managed by regularly updating incentive mechanisms and parameters to align with ecosystem goals and user engagement levels.                          
| rewards.rs | Handles the calculation and distribution of rewards across different pools, facilitating a fair and efficient rewards mechanism for various on-chain activities. | Implements a robust set of features for reward management, including multi-currency support, share-based reward distribution, and automatic claim mechanisms. It emphasizes security with checks against overflow and abuse scenarios, ensuring the reliability of reward calculations and distributions. | Vital for the Acala network's ecosystem, ensuring users are compensated for their contributions. It underpins the economic incentives that drive behavior on the platform and requires careful oversight to maintain balance and fairness in reward distribution. |





## Approach Taken while auditing the codebase
I began the audit by immersing myself in the comprehensive documentation provided for the Acala protocol. This initial deep dive was critical to grasp the overarching architecture, functionalities, and the specific intricacies of the system. The documentation served as a foundational layer, enabling me to align my auditing efforts with the protocol's intended security and functional benchmarks.

Following this preparatory phase, I organized my audit strategy around a sequence that mirrored the logical and dependency flow of the protocol as outlined in the documentation. The sequence in which I approached the files for audit was as follows:

1. **Earning Module (`lib.rs`)**: As a core part of the protocol that directly interacts with user stakes and rewards, this was my starting point. Understanding the earning mechanics was crucial since it laid the groundwork for how rewards and penalties are computed and distributed.

2. **Incentives Module (`lib.rs`)**: Building upon the understanding of the earning mechanisms, I moved on to audit the incentives module. This module's significance stems from its role in managing and distributing incentives, which are vital for ensuring network participation and security.

3. **Rewards Module (`lib.rs`)**: The final piece of the puzzle, the rewards module, was audited last. This module is integral to the protocol as it oversees the actual distribution of rewards, an area ripe for vulnerabilities if not handled correctly.

With each file, my approach was methodical and twofold: Catch common pitfalls and vulnerabilities, followed by a detailed manual review focusing on business logic, security controls, and potential edge cases. This blend of automated and manual examination ensured no stone was left unturned.

During the manual review phase, I paid particular attention to function calls, data handling, and state management operations. By tracing how data flowed across these modules, I aimed to uncover any potential security risks such as unauthorized access, improper state changes, or issues that could lead to funds being locked or lost.




## Test analysis

**Setup**

Installed Substrate:

```bash
sudo apt install build-essential
```

Navigate to `/src`

```bash
cargo build
```
After installing and building contracts using this command to execute all the test scripts:

```bash
cargo test
```



### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content. In particular, tests have been written successfully.

-   2) Overall line coverage percentage provided by your tests : 85%

### What could they have done better?

-  1): It is recommended to increase the test coverage to 100% so make sure that each and every line is battle tested

## Security Approach of the Project

### Successful current security understanding of the project;

1- The project has already underwent several [Audits](https://github.com/AcalaNetwork/Acala/tree/master/audit)

### What the project should add in the understanding of Security;

1- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/

2- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

3- I also recommend that you have an "Economic Audit" for projects based on such complex mathematics and economic models. An example Economic Audit is provided in the link below;
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)

4 - As the project team, you can consider applying the multi-stage audit model.

[![sla.png](https://i.postimg.cc/nhR0kN3w/sla.png)](https://postimg.cc/Sn96Q1FW)

Read more about the MPA model;
https://mpa.solodit.xyz/

5 - I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19












### Systemic Risks

In auditing the Acala project, several systemic risks became apparent, reflecting both the complex interplay of its components and the broader context in which it operates within the DeFi ecosystem. These systemic risks are critical not only due to their potential impact on the Acala network but also due to the cascading effects they might have on connected systems and users.

1. **Cross-Chain Interactions (XCM)**: Acala's integration with the Polkadot ecosystem through Cross-Chain Message Passing (XCM) introduces a layer of risk associated with the cross-chain transfer of assets. Vulnerabilities or design flaws in the handling of cross-chain messages could lead to loss or theft of assets as they are transferred across chains.

2. **Governance Attacks**: Given the governance mechanisms in place for managing aspects of the Acala network, there is a risk of governance attacks where malicious actors could potentially gain control or unduly influence governance decisions. This could lead to unfavorable changes in protocol parameters, diversion of funds, or other actions that could harm the network.


### Centralization Risks

Centralization risks stem from the roles and permissions granted to certain addresses within the protocol.

**Governance Centralization:** Acala's governance mechanism allows token holders to vote on proposals that can alter the protocol's behavior, parameters, and upgrades. If a small number of holders or a single entity accumulates a significant portion of the governance tokens, they could exert undue influence over the protocol's direction and decisions.

**Parameter Management Centralization:** The ability to adjust critical protocol parameters, such as reward distribution rates, staking requirements, or fee structures, might be restricted to a small group of administrators or governed through a process where participation is limited.

### Technical Risks

**Smart Contract Vulnerabilities:** Bugs or logical errors in the smart contracts can lead to loss of funds, unauthorized access, or unintended behavior. Given the complexity of contracts like the rewards, earnings and incentive the attack surface is significant.

**Scalability Concerns:** As transaction volumes grow, the platform must scale without compromising performance or security.

### Integration Risks

Integration risks are primarily associated with the protocol's interactions with external DeFi platforms.

**Cross-Chain Communication Failures:** Acala's functionality relies heavily on cross-chain messaging and interoperability, particularly with Polkadot and other parachains through XCM (Cross-Consensus Message format). Misalignments, incompatibilities, or failures in these cross-chain communication protocols can disrupt Acala's services, affecting liquidity, staking, and other decentralized finance (DeFi) functionalities.

**Smart Contract Interactions:** Acala's ecosystem includes various precompiled contracts. Flaws, vulnerabilities, or unexpected behaviors in these external contracts could lead to losses or adverse effects on Acala's operations and its users' assets.




NOTE: I don't track time while auditing or writing report, so what the time I specified is just a number







### Time spent:
2 hours