### ACA Staking Overview

ACA staking rewards participants with ecosystem tokens, allows for voting on emission distributions, and includes a loyalty bonus mechanism for unstaking. By staking ACA, users not only contribute to the network's liquidity but also retain their governance voting rights. The process is straightforward: create a wallet, transfer ACA to it, connect to the Acala Apps, and stake your desired amount of ACA. Rewards can be claimed, and ACA can be unstaked with a 28-day waiting period, although unstaked ACA does not yield earnings during this time. A unique loyalty bonus incentivizes timely reward claims, distributing a portion of claimed amounts among participants.

| File Name    | File Description    |
|--------------|---------------------|
| src/modules/incentives/src/lib.rs   | This code base is a part of the Acala platform, designed to manage and distribute multi-currency incentives and rewards across various pools like loans and decentralized exchanges, leveraging a modular approach to integrate with other protocols for share management and reward distribution. |
| src/orml/rewards/src/lib.rs   | This code base is a Rust module for a blockchain platform, focused on managing reward pools, including share allocation and reward distribution across various currencies. It supports operations like adding and removing shares, claiming rewards, and handling reward accumulation efficiently.  |
| src/modules/earning/src/lib.rs      | This code is a module for the Acala platform that manages bonding and unbonding of tokens, allowing users to lock tokens as a bond, initiate unbonding with or without a fee for instant access, and rebond or withdraw unbonded tokens, leveraging parameters for flexibility and integrating events for tracking these actions.                    |

### Architecture Diagram
[![art-drawio.png](https://i.postimg.cc/L5ytb9jC/art-drawio.png)](https://postimg.cc/5QCQHM7L) 
**Architecture Overall System Overview:**

- **User Interactions:** Users interact with the system by bonding or unbonding tokens, managing rewards, and participating in various incentive programs. User actions trigger the corresponding modules within the platform.

- **Earning Module:** Handles the bonding and unbonding processes for tokens. Users can bond tokens to participate in the platform's economy, choose to unbond tokens with or without a fee for immediate access, and manage their bondings.

- **Rewards Management Module:** Manages rewards across different pools and mechanisms. This module is responsible for accumulating rewards over time, based on user shares in different pools, and distributing these rewards.

- **Incentives Module:** Specifically designed to handle incentives separate from regular rewards. It supports different pool types (e.g., loans, decentralized exchange participation) and accumulates incentive rewards periodically from a predefined source.

- **Currency Module:** Interacts closely with the Earning Module to lock or unlock tokens based on bonding actions. It ensures that the token balances are updated accordingly.

- **Ledger:** A central component that records the state changes related to token bonding, rewards, and incentive participation. It's updated based on actions from the Earning, Rewards Management, and Incentives Modules.

- **Reward Pool and Incentive Rewards Pool:** These are pools from which rewards and incentives are drawn. They're periodically updated to reflect the accumulation of rewards and distribution to users.

- **Events:** Events are emitted by the Earning, Rewards Management, and Incentives Modules to inform users about significant actions or changes, such as successful bonding, unbonding, reward claims, and incentive updates.

This architecture facilitates a flexible and dynamic platform for token bonding, rewards management, and incentive distribution, catering to the diverse needs of its users.

### Sequence Diagram

[![seq-drawio-5.png](https://i.postimg.cc/Qxh5pTZX/seq-drawio-5.png)](https://postimg.cc/F1C7v1Kq) 

**Overall Sequence Overview:**

1. **Bonding Tokens:** The user initiates the bonding process, which triggers the Earning Module to lock the specified number of tokens through the Currency Module. Upon successful locking, the ledger is updated, and an event is emitted to notify the user.

2. **Claiming Rewards:** The user requests to claim rewards, activating the Rewards Management Module to retrieve reward details from the Reward Pool. After updating the rewards ledger, an event is emitted for the user notification.

3. **Participating in Incentives:** The user decides to participate in incentives, prompting the Incentives Module to gather incentive details from the Incentive Rewards Pool. It updates the incentives ledger accordingly and notifies the user through an emitted event.

4. **Unbonding Tokens:** When the user chooses to unbond tokens, the Earning Module works with the Currency Module to unlock the tokens. The ledger is updated to reflect this change, followed by an event notification to the user.

This sequence diagram illustrates the flow of actions and interactions within the system from the user's perspective, highlighting how various modules cooperate to manage bonding, rewards, and incentives.

### src/modules/incentives/src/lib.rs
[![f1-uml-drawio-6.png](https://i.postimg.cc/3JBkT8f9/f1-uml-drawio-6.png)](https://postimg.cc/N5Ks8ccr) 
## Overview of the Incentives Module Functionality

### Configuration (Config)
This section defines the configuration traits for the module, specifying the types for runtime events, currency handling, rewards accumulation period, and other operational parameters.

- **RuntimeEvent**: Defines the type used for runtime events.
- **Currency**: Specifies the currency type for transactions within the module.
- **ParameterStore**: A storage for module parameters like the instant unstake fee.
- **AccumulatePeriod, NativeCurrencyId, RewardsSource**: Configure the period for reward accumulation, the native currency ID, and the source account for rewards.
- **UpdateOrigin**: Sets the origin allowed to update incentive-related parameters.
- **EmergencyShutdown**: Allows for an emergency shutdown of the incentives, affecting reward accumulation.

### Storage
The module uses several storage items to track the state of incentives, pools, and rewards.

- **IncentiveRewardAmounts**: Stores the reward amounts for each pool and currency.
- **ClaimRewardDeductionRates**: Records the deduction rates applicable to claim rewards.
- **ClaimRewardDeductionCurrency**: Indicates the currency that deduction rates apply to.
- **PendingMultiRewards**: Tracks the pending rewards for each user, pool, and currency.

### Events
Events are emitted to notify users of significant actions within the module.

- **DepositDexShare, WithdrawDexShare, ClaimRewards**: Inform about share deposits, withdrawals, and reward claims.
- **IncentiveRewardAmountUpdated, ClaimRewardDeductionRateUpdated, ClaimRewardDeductionCurrencyUpdated**: Notify about updates to incentive rewards, deduction rates, and deduction currencies.

### Calls
User-callable functions allow interacting with the module's functionality, such as staking LP tokens, claiming rewards, and updating incentive parameters.

- **deposit_dex_share**: Stakes LP tokens to participate in DEX incentives.
- **withdraw_dex_share**: Unstakes LP tokens, removing participation in DEX incentives.
- **claim_rewards**: Allows users to claim their pending rewards.
- **update_incentive_rewards**: Updates the incentive reward amounts for specific pools.
- **update_claim_reward_deduction_rates**: Adjusts the deduction rates for reward claims.
- **update_claim_reward_deduction_currency**: Changes the currency to which the deduction rate applies.

### Internal Functions
These functions are not directly called by users but are crucial for the module's operation.

- **accumulate_incentives**: Periodically updates the reward amounts for each pool based on configured incentives.
- **transfer_rewards_and_update_records**: Handles the transfer of rewards to the module account and records the distribution.
- **do_claim_rewards**: Processes the reward claim by a user, adjusting pending rewards and applying any deductions.
- **payout_reward_and_reaccumulate_reward**: Manages the payout of rewards to users and re-accumulation of any deducted amounts.

### Implementation of Traits
The module implements several traits to integrate with other parts of the system.

- **DEXIncentives**: Manages the deposit and withdrawal of DEX shares.
- **IncentivesManager**: Provides functions for managing incentive rewards and claiming rewards.
- **Happened**: Hooks for external events, such as loan updates, bonding, and unbonding events.

### Utility Functions
Utility functions support the main functionality, such as calculating deduction rates and managing account IDs for reward distribution.

Overall, the Incentives Module is designed to manage and distribute multi-currency rewards within a blockchain platform, supporting various types of incentive pools and flexible reward mechanisms.
[![f1-seq-drawio-5.png](https://i.postimg.cc/Xq6kTj14/f1-seq-drawio-5.png)](https://postimg.cc/xJgmm2S4) 
### src/orml/rewards/src/lib.rs
[![f2-uml-drawio-6.png](https://i.postimg.cc/hGz4fTc4/f2-uml-drawio-6.png)](https://postimg.cc/gwpb7wx1) 
## Functionality Overview of the Smart Contract

### **Configuration (Config)**
Defines essential types used across the smart contract, including parameters for shares, balances, pool IDs, currency IDs, and handling of rewards.

### **Structs and Types**
- **PoolInfo**: Represents the details of a reward pool, including total shares and reward information mapped by currency.
- **WithdrawnRewards**: A map that tracks the amount of rewards already withdrawn by currency ID.

### **Storage Items**
- **PoolInfos**: Tracks information for each reward pool, including shares and rewards.
- **SharesAndWithdrawnRewards**: Records the share amount and withdrawn rewards for each account in every pool.

### **Error Types**
Enumerates potential errors that might occur in the contract, such as nonexistent pools or shares, or attempting operations that exceed certain limits.

### **Public Functions**

#### **accumulate_reward**
Updates a reward pool with an increment of rewards for a specific currency. This function ensures rewards are added to the pool's total and are available for distribution.

#### **add_share**
Increases the share count for an account in a specified pool. It updates both the total shares of the pool and the individual account's share, adjusting the reward distribution proportionally.

#### **remove_share**
Decreases an account's share in a pool, potentially triggering the rewards claim process first. This adjustment might affect the reward distribution and updates the pool's total shares accordingly.

#### **set_share**
Sets the share amount for an account in a pool to a specific value, adding or removing shares as necessary to reach the desired amount. This function is a flexible tool for adjusting an account's participation in a pool.

#### **claim_rewards**
Initiates the reward claim process for an account, allowing them to collect earned rewards from all currencies in a pool based on their share.

#### **claim_reward**
Similar to `claim_rewards`, but targets a specific currency within a pool. This allows for more granular control over the reward claim process.

#### **transfer_share_and_rewards**
Facilitates the transfer of shares and associated rewards from one account to another within the same pool. This function maintains the integrity of reward distribution while enabling share mobility.

### **Utility Functions**

#### **claim_one**
A helper function that calculates and processes the payout for a single currency's rewards to an account, based on their share in the pool.

#### **reward_to_withdraw**
Determines the amount of a specific currency's rewards that an account is entitled to withdraw, taking into account their share and previously withdrawn rewards.

These functionalities collectively enable a comprehensive rewards management system within the smart contract, allowing for detailed tracking and distribution of rewards based on participant shares in various pools.
[![f2-seq-drawio-4.png](https://i.postimg.cc/sfbwGZS6/f2-seq-drawio-4.png)](https://postimg.cc/xqyKW86M)
### src/modules/earning/src/lib.rs 
[![f3-uml-drawio-5.png](https://i.postimg.cc/59n7MQVr/f3-uml-drawio-5.png)](https://postimg.cc/14VcGt40) 
## Overview of the Earning Module Functionality

### **Bonding Operations**
This functionality allows users to lock their tokens into the system, signifying a commitment and enabling participation in various platform activities or rewards.

#### **bond**
- **Purpose**: Locks a specified amount of tokens as a bond.
- **Mechanism**: Checks the user's balance and locks the specified amount. If the balance is insufficient, it locks the available balance.
- **Outcome**: Emits a `Bonded` event with details of the transaction.

#### **unbond**
- **Purpose**: Initiates the unbonding process for a specified amount of tokens.
- **Mechanism**: Sets a future block number (based on the `UnbondingPeriod`) when the tokens will be available for withdrawal.
- **Outcome**: Emits an `Unbonded` event with transaction details.

#### **unbond_instant**
- **Purpose**: Allows instant unbonding of tokens for a fee.
- **Mechanism**: Calculates the fee based on the `InstantUnstakeFee` parameter and the amount being unbonded. Deducts the fee and makes the remaining balance available immediately.
- **Outcome**: Emits an `InstantUnbonded` event, detailing the transaction and the fee charged.

#### **rebond**
- **Purpose**: Rebonds tokens that are in the unbonding process.
- **Mechanism**: Cancels the unbonding process for the specified amount, effectively rebonding the tokens.
- **Outcome**: Emits a `Rebonded` event with transaction details.

#### **withdraw_unbonded**
- **Purpose**: Withdraws tokens that have completed the unbonding process.
- **Mechanism**: Checks if the unbonding period has passed and makes the tokens available for withdrawal.
- **Outcome**: Emits a `Withdrawn` event, indicating the amount withdrawn.

### **Parameter Management**
Handles the parameters that affect the bonding and unbonding processes.

#### **Parameters Structure**
Defines parameters such as the `InstantUnstakeFee`, which is the fee charged for the instant unbonding of tokens.

### **Error Handling**
Enumerates potential errors that can occur during bonding operations, such as attempting to bond below the minimum threshold or exceeding the maximum number of unbonding chunks.

### **Event Notifications**
Defines events that notify users and external systems of important state changes within the module, including bonding, unbonding, and parameter updates.

### **Storage Items**
Maintains records of bonded tokens, unbonding schedules, and other relevant information to manage user balances and states effectively.

### **Utility Functions**
Includes utility functions and interfaces for interacting with other parts of the system, such as the currency module for locking and unlocking tokens, and the parameter store for accessing module parameters.

This module plays a crucial role in managing the financial commitments of users within the platform, facilitating participation in staking, governance, or rewards programs by handling the complexities of token bonding and unbonding.
[![f3-seq-drawio-5.png](https://i.postimg.cc/CxDsdz3S/f3-seq-drawio-5.png)](https://postimg.cc/PCXwRrfR) 

### Centralization Risk

- **Access Control and Permissions**: If the update functions (e.g., updating incentive rewards, unbonding parameters) are restricted to a limited number of addresses or roles, there's a risk of centralization where a few entities control significant aspects of the system. This could lead to manipulation or unfair advantages.
- **Parameter Store Control**: The management and update mechanisms of parameters like `InstantUnstakeFee` can centralize power if not governed democratically. The parameters affecting the system's economics and operations could be misused if controlled by a centralized entity.

### Systematic Risk

- **Single Point of Failure**: The reliance on specific modules or external dependencies (e.g., `orml_traits`, `Currency`, `LockableCurrency`) introduces systematic risks where failures or vulnerabilities in these dependencies could compromise the entire system.
- **Liquidity and Market Impact**: The functionalities around bonding, unbonding, and rewards management could significantly impact the liquidity and market behavior of the underlying tokens. Mismanagement or exploitation of these features could lead to market manipulation or unintended economic consequences.

### Architecture Risks

- **Upgradeability and Compatibility**: The system's ability to adapt to future requirements, blockchain updates, or changes in external modules without causing disruptions or vulnerabilities is crucial. Rigid architectural designs might pose risks in terms of scalability and maintainability.
- **Inter-module Communication**: The interactions between different modules (e.g., rewards distribution, bonding ledger updates) need to be seamless and secure. Miscommunications or errors in data exchange could lead to inconsistencies or vulnerabilities in token management and rewards distribution.

### Complexity Risks

- **Smart Contract Vulnerabilities**: The complexity of functions managing bonds, unbonds, rebonds, and reward calculations increases the surface for potential exploits or bugs within the smart contract code. Complex logic and state management need thorough testing and auditing to mitigate risks.
- **User Experience and Understanding**: The complexity of the system, especially around the mechanics of bonding, unbonding, instant unbond fees, and reward claims, could lead to a poor user experience. Misunderstandings or misinterpretations of the functionalities could result in user errors or loss of funds.
- **Operational Complexity**: The need for continuous parameter adjustments (like unbonding periods, fees) and the management of various types of incentives and rewards introduce operational complexities. These require diligent oversight and governance to ensure the system's objectives are met without unintended consequences.

Mitigating these risks involves comprehensive governance structures, rigorous testing and auditing processes, clear documentation and communication strategies, and the implementation of fail-safes and recovery mechanisms to ensure the system's integrity and user trust.
### Conclusion

The system presents a sophisticated framework for managing tokens through bonding, unbonding, and rewards distribution, catering to various stakeholder incentives. While it harnesses the potential for dynamic participation and engagement, it necessitates rigorous oversight, meticulous design consideration, and continuous community involvement to mitigate inherent risks associated with centralization, architecture, and complexity. Ensuring transparency, security, and scalability will be paramount for sustaining user trust and system longevity.

### Time spent:
25 hours