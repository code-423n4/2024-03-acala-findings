
#### ! File by File Analysis Report (one file after another)

## Summary

no | File |
|-|:-|
| [[File-1](#file-1)] | lib.rs |
| [[File-2](#file-2)] | lib.rs | 
| [[File-3](#file-3)] | lib.rs | 


## Analysis Issue Report 


### [File-1]  
#### [lib.rs](https://github.com/code-423n4/2024-03-acala/blob/main/src/modules/incentives/src/lib.rs)


<details>
<summary> See Instances</summary>



#### Admin Abuse Risks:

**Update Origin Check**: The `update_incentive_rewards`, `update_claim_reward_deduction_rates`, and `update_claim_reward_deduction_currency` functions require an origin check, but it's not specified what entities have this privilege. This could potentially lead to admin abuse if not properly controlled.



#### Systemic Risks:

**Reward Accumulation:** The `accumulate_incentives` function accumulates rewards periodically for each pool. If there are any vulnerabilities in this process, it could lead to incorrect reward distributions or loss of funds.

#### Technical Risks:


**Error Handling:** Some functions in the contract handle errors internally, but it's not clear how they propagate and whether they could lead to unexpected behavior.
**Complexity**: The contract has several storage items and complex logic for reward accumulation, deduction rates, and reward claiming. This complexity increases the risk of bugs and vulnerabilities.

#### Integration Risks:

**External Contracts**: The contract interacts with external contracts for currency transfers and reward handling. Any changes or vulnerabilities in these external contracts could impact the functionality of this contract.


#### Non-Standard Token Risks:

**DEX Share Handling:** The contract handles LP tokens for decentralized exchanges (DEX) shares. Any vulnerabilities related to these tokens or interactions with DEXs could pose risks to the contract's functionality and security.

#### Overall:

While the smart contract seems to have comprehensive functionality for handling incentives, staking, and reward distribution, careful attention should be paid to admin privileges, error handling, and integration with external contracts to mitigate potential risks. Additionally, thorough testing and auditing are recommended to ensure the contract's security and reliability.

</details>


### [File-2]   
#### [lib.rs](https://github.com/code-423n4/2024-03-acala/blob/main/src/orml/rewards/src/lib.rs)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

**Accumulate Reward**: The `accumulate_reward` function allows accumulating rewards for a specific pool and currency. There might be a risk if an admin abuses this function to unfairly distribute rewards or manipulate reward balances.
**Set Share**: The `set_share` function allows an admin to set the share of a user in a pool. If not used properly, an admin could potentially manipulate users' shares, leading to unfair distribution of rewards.
**Transfer Share and Rewards**: The `transfer_share_and_rewards` function enables transferring shares and rewards between users. Admin abuse could occur if this function is used to favor specific users or unfairly distribute rewards.




#### Systemic Risks:

**Storage**: The smart contract utilizes storage to maintain pool information, share amounts, and reward balances. Excessive data storage or inefficient data structures could lead to higher costs and slower execution.
**Claim Rewards**: The `claim_rewards` function processes reward claims for users. Inefficient implementation or high gas costs in claiming rewards could affect the overall system performance.


#### Technical Risks:

**Overflow/Underflow**: There might be risks of overflow or underflow in arithmetic operations involving shares, rewards, and balances. These risks should be mitigated by using safe arithmetic operations or appropriate checks.
**Claim One**: The `claim_one` function calculates reward amounts to be withdrawn based on various factors. Incorrect calculations or data manipulation could result in incorrect reward withdrawals.
**Integration with Reward Handler**: The integration with the `RewardHandler` trait might introduce technical risks related to the handling of reward payouts and interactions with external contracts or systems.


#### Integration Risks:

**RewardHandler Integration**: The smart contract relies on an external trait `RewardHandler` for reward payouts. Any changes or vulnerabilities in the implementation of this trait could affect the functionality and security of the contract.
**External Contracts**: Interactions with external contracts or modules, such as currency or reward systems, pose integration risks. Any changes or vulnerabilities in these external components could impact the behavior of the smart contract.


#### Non-Standard Token Risks:

The smart contract does not directly handle non-standard tokens. However, it interacts with reward currencies (`T::CurrencyId`), which could include non-standard tokens. Risks related to the handling of these tokens should be considered, such as compatibility issues or vulnerabilities specific to certain token standards.


#### Overall: 
the smart contract presents risks related to admin abuse, systemic functionality, technical implementation, integration with external components, and potential interactions with non-standard tokens. 

</details>

### [File-3]   
#### [lib.rs](https://github.com/code-423n4/2024-03-acala/blob/main/src/modules/earning/src/lib.rs)



<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

**Bonding**: The `bond` function allows users to bond tokens by locking them up. Admins could abuse this function to lock users' tokens without their consent, leading to loss of control over funds.
**Unbonding**: The `unbond` function starts unbonding tokens for a user. Admins might misuse this to initiate unbonding without user consent, affecting users' liquidity and staking plans.
**Instant Unbonding**: The `unbond_instant` function allows instant unbonding with a fee. Admins could abuse this by imposing arbitrary fees or instant unbonding users' tokens without proper justification.
**Rebonding**: The `rebond` function rebonds tokens from the unbonding period. Admins might misuse this function to rebond tokens without user permission, affecting users' staking strategies.
**Withdraw Unbonded:** The `withdraw_unbonded` function withdraws unbonded tokens. Admins could abuse this function to withdraw users' tokens without their consent, impacting users' financial positions.



#### Systemic Risks:

**Lock Mechanism**: The smart contract uses a lock mechanism (`Currency::set_lock `and `Currency::remove_lock`) to manage bonded tokens. Issues in the lock mechanism implementation could lead to funds being stuck or unlocked incorrectly.
**Bonding Ledger**: The `Ledger` storage maintains bonding ledger information. Inaccurate ledger management could lead to discrepancies in locked token amounts and availability.


#### Technical Risks:

**Overflow/Underflow:** There might be risks of overflow or underflow in arithmetic operations involving token amounts. Proper checks and safeguards should be implemented to prevent these risks.
**Configurability:** The contract's behavior and parameters are configurable (`Parameters` trait). Incorrect parameter values or misconfigurations could lead to unexpected behavior or vulnerabilities.


#### Integration Risks:

**Currency Integration**: The contract integrates with the `Currency` trait for managing token balances and locks. Integration risks such as incorrect token handling or vulnerabilities in the currency implementation should be considered.
**Bonding Controller Integration**: The contract implements the `BondingController` trait for handling bonding operations. Risks related to incorrect implementation or vulnerabilities in the bonding logic could affect the contract's functionality.


#### Non-Standard Token Risks:

The contract does not directly handle non-standard tokens but interacts with tokens through the `Currency` trait. Risks related to token standards, compatibility issues, or vulnerabilities in token contracts should be considered.


#### Overall:

the smart contract presents risks related to admin abuse, systemic functionality, technical implementation, integration with external components, and potential interactions with non-standard tokens. 

</details>


### Time spent:
12 hours