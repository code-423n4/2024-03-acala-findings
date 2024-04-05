# Table of Contents

- [Overview](#overview)
- [Introduction Acala Network](#introduction)
  - [Stablecoin Creation](#stablecoin-creation)
  - [Cross-Chain Compatibility](#cross-chain-compatibility)
  - [Liquidity Provision](#liquidity-provision)
  - [Key Use Cases](#key-use-cases)
  - [Stablecoin Mechanisms](#stablecoin-mechanisms)
  - [Governance](#governance)
  - [Conclusion](#conclusion)
- [Codebase Quality and Systemic Risks](#codebase-quality-and-systemic-risks)
  - [Approach We Followed When Reviewing the Code](#approach-we-followed-when-reviewing-the-code)
  - [Earning Module](#earning-module)
    - [System Risk](#system-risk)
    - [Economic Risk](#economic-risk)
    - [Security Risk](#security-risk)
    - [Governance Risk](#governance-risk)
  - [Incentives](#incentives)
    - [System Risk](#system-risk-1)
  - [Rewards](#rewards)
    - [System Risk](#system-risk-2)
    - [Technical Risks](#technical-risks)
- [Analysis of the Code Base and Diagram Architecture](#analysis-of-the-code-base-and-diagram-architecture)
  - [Earning](#earning)
  - [Incentives](#incentives-1)
  - [Rewards](#rewards-1)
- [Conclusion](#conclusion-1)


# Overview

Structuring the code is crucial for conducting a thorough analysis of the code base. This enables auditors to predict the level of contract difficulty, identify critical contracts, and determine security-critical features such as payment functions or assembly usage. Additionally, this approach accurately measures the time required to perform a detailed audit and helps determine the cost of a project audit. To achieve efficiency, clarity, and improvements, it is essential to have a comprehensive view of the entire architecture. This enables the identification of the basic structure, relationships between modules, and key design patterns. By understanding the big picture, we can analyze the complexity of the code and reveal its strengths and potential for improvement with confidence.


# Introduction
Acala is a specialized stablecoin and liquidity blockchain that is decentralized, cross-chain by design, and future-proof with forkless upgradability. Acala stablecoin protocol uses multi-collateral-backing mechanism to create a stablecoin soft-pegged to the US Dollar. It offers a unique approach to stablecoin issuance, cross-chain compatibility, and upgradability without the need for forks. Let's dive into its key features and benefits.

![usecaseacala](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/5e20a1e7-e87c-4648-b82b-8a1b81ecd61c)
![acalaintroduction](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/0405071b-49af-46f4-acfa-404f73bd3dc0)

## Stablecoin Creation
Acala's stablecoin protocol utilizes a multi-collateral backing mechanism to create a stablecoin, Acala Dollar (aUSD), pegged to the US Dollar. This stablecoin is created from a basket of reserve assets, including DOT, LDOT, and ACA, ensuring stability while allowing users to retain ownership of their reserve assets.

## Cross-Chain Compatibility
Powered by Polkadot, Acala is inherently cross-chain. It supports Polkadot native assets like DOT and ACA, as well as cross-chain assets such as Bitcoin and Ether. This cross-chain compatibility enables trustless integration with other blockchains and applications connected to Polkadot.

## Liquidity Provision
Users can obtain Acala stablecoin through AcalaSwap or by depositing collateral into a stablecoin vault. This mechanism allows for increased liquidity of underlying crypto assets without the need to sell them, providing flexibility for traders and businesses alike.

## Key Use Cases

- **Trustless Minting:** Minting Acala stablecoin is a sovereign and liberating act, allowing users to hedge volatility and pay for goods and services without selling their underlying assets.
- **Leveraged Trading:** Traders can leverage their underlying assets by collateralizing them to mint stablecoin, essentially multiplying their exposure to the asset.
- **Yield Farming:** Assets like LDOT can be used as collateral to multiply exposure to underlying assets while earning annual percentage rates (APR).
- **Increased Capital Efficiency:** Liquidity providers can use LP collateralized super loans to increase capital efficiency when providing liquidity to pools.

## Stablecoin Mechanisms
Acala's stablecoin protocol employs various mechanisms, including risk parameter management, autonomous liquidation, oracles, and emergency shutdown, to ensure stability and robustness in tracking the dollar in different market conditions.

## Governance
Acala's governance framework includes a Referenda chamber, a General Council, and specialized councils like the Financial Council and the Homa Staking Council. These councils oversee the network's operations, upgrades, and protocol parameters, ensuring the network's stability and security.

## Conclusion
Acala's innovative approach to stablecoin creation, cross-chain compatibility, and governance makes it a promising platform for decentralized finance (DeFi) applications. Its ability to provide stability, liquidity, and flexibility to users and developers alike positions it as a key player in the evolving DeFi ecosystem.


# Codebase quality and Systemic risks 

Overall, I consider the quality of the Acala Network codebase to be excellent. The code appears to be very mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Module              | Codebase Quality                                                                                                                 | Systemic Risks                                                                                                                                      |
|---------------------|----------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| Earning Module      | - Well-structured with clear documentation.                                                                                    | - **Bonding and Unbonding:** Potential vulnerabilities if input parameters are not properly validated.                                             |
|                     | - Consistent naming conventions and parameter descriptions.                                                                     | - **Instant Unbonding:** Vulnerability if fee parameters are not properly validated, leading to excessive fees or revenue loss.                  |
|                     | - Error handling for various scenarios.                                                                                        | - **Rebonding:** Vulnerability if rebonding amount is not properly validated, leading to issues.                                                   |
|                     | - Events emitted for tracking important actions.                                                                                | - **Withdrawal:** Vulnerability if unbonding period is not properly validated, leading to early or delayed withdrawals.                             |
|                     |                                                                                                                                  | - **Ledger Management:** Vulnerability if ledger updates are not handled correctly, affecting token bonding and unbonding.                         |
|                     |                                                                                                                                  | - **Configuration Parameters:** improper configuration settings affecting token bonding and unbonding process.               |
| Incentives Module   | - Comprehensive documentation outlining module functionality.                                                                  | - **Systemic risks:** Dependency on orml_rewards pallet may introduce bugs or issues impacting functionality.                                      |
|                     | - Configuration traits defined for flexibility and customization.                                                               | - **Technical risks:** Dependencies on Substrate-specific types and functions may introduce bugs or issues.                                        |
|                     | - Error handling implemented for various scenarios.                                                                             | - **Currency and pool ID validation:** improper validation of currency and pool IDs.                                        |
|                     | - Internal functions for managing state are well-defined.                                                                       | - **Rate and amount validation:** improper validation of rates and amounts.                                                |
|                     | - Events emitted for tracking key actions.                                                                                      | - **Weight and gas costs:** improper calculation or management of weight and gas costs.                                    |
|                     |                                                                                                                                  | - **Emergency shutdown:** improper implementation or management of emergency shutdown mechanism.                           |
| Rewards Module      | - Detailed configuration and error handling.                                                                                   | - **Input Validation:** improper validation of input values leading to overflow, underflow, or incorrect behavior.           |
|                     | - Functions for all key operations defined.                                                                                     | - **Error Handling:** improper handling of error cases and invalid operations.                                            |
|                     | - System risks identified and documented.                                                                                       | - **Edge Cases:** behavior in edge cases, ensuring correct functionality.                                                   |
|                     | - Technical risks and mitigation strategies outlined.                                                                           | - **Efficiency:** inefficient code, especially with large numbers of shares or rewards.                                     |
|                     | - Comprehensive integration considerations.                                                                                    | - **Integration Risks:** integration with other systems or modules, ensuring security and vulnerability management.      |
|                     | - Risks related to overflow, underflow, and manipulation of pools and shares discussed.                                          | - **Overflow and Underflow:** arithmetic operations, ensuring protection against overflow and underflow issues.             |
|                     | - Technical risks identified with solutions proposed.                                                                           | - **Pool and Share Manipulation:** adding, removing, and claiming rewards, ensuring system integrity.                        |

# Approach we followed when reviewing the code

# Earning Module
This module used for bonding and unbonding tokens. It is implemented as a Substrate pallet and includes the following features:

- **Bonding**: Locking up tokens to earn rewards.
- **Unbonding**: Unlocking bonded tokens after a certain period of time.
- **Instant unbonding**: Unlocking bonded tokens immediately by paying a fee.
- **Rebonding**: Rebonding tokens that are in the unbonding period.
- **Withdrawal**: Withdrawing unbonded tokens.

parameters:

- `InstantUnstakeFee`: The fee for instant unbonding.
- `MinBond`: The minimum amount of tokens required for bonding.
- `UnbondingPeriod`: The period of time required for unbonding.
- `MaxUnbondingChunks`: The maximum number of unbonding chunks allowed.
- `LockIdentifier`: The identifier for the lock on the tokens.

storage items:

- `Ledger`: A mapping of account IDs to bonding ledgers.

errors:

- `BelowMinBondThreshold`: The bonding amount is below the minimum threshold.
- `MaxUnlockChunksExceeded`: The number of unbonding chunks exceeds the maximum allowed.
- `NotAllowed`: The operation is not allowed.
- `NotBonded`: The account is not bonded.

events:

- `Bonded`: A user has bonded tokens.
- `Unbonded`: A user has unbonded tokens.
- `InstantUnbonded`: A user has unbonded tokens instantly by paying a fee.
- `Rebonded`: A user has rebonded tokens.
- `Withdrawn`: A user has withdrawn unbonded tokens.

extrinsics:

- `bond`: Bond tokens by locking them up to a certain amount.
- `unbond`: Start unbonding tokens up to a certain amount.
- `unbond_instant`: Unbond tokens instantly by paying a fee.
- `rebond`: Rebond tokens from the unbonding period.
- `withdraw_unbonded`: Withdraw all unbonded tokens.

## System Risk

**Bonding and Unbonding:** 
- The `bond` and `unbond` functions could be vulnerable if the `amount` parameter is not properly validated. For example, if the amount is less than the minimum bond threshold or greater than the available balance, it could cause issues. 
- The `unbond` function could also be vulnerable if the `unbond_at` parameter is not set correctly, causing tokens to be unbonded at the wrong time.

**Instant Unbonding:** 
- The `unbond_instant` function could be vulnerable if the `InstantUnstakeFee` parameter is not properly validated. If the fee ratio is set too high, it could cause users to pay excessive fees, while if it's set too low, it could cause the system to lose revenue.

**Rebonding:** 
- The `rebond` function could be vulnerable if the `amount` parameter is not properly validated. If the amount is greater than the available unbonded balance, it could cause issues.

**Withdrawal:** 
- The `withdraw_unbonded` function could be vulnerable if the `unbonding_period` parameter is not properly validated. If the period is set too short, it could cause tokens to be withdrawn before they are fully unbonded, while if it's set too long, it could cause unnecessary delays in withdrawing tokens.

**Ledger Management:** 
- The `apply_ledger` function could be vulnerable if the ledger is not properly updated. If the ledger is not correctly locked or unlocked, it could cause issues with token bonding and unbonding.

**Configuration Parameters:** 
- relies on several configuration parameters, such as `MinBond`, `UnbondingPeriod`, and `MaxUnbondingChunks`. If these parameters are not properly set, it could cause issues with token bonding and unbonding.

### Economic Risk
The `bond`, `unbond`, `unbond_instant`, `rebond`, and `withdraw_unbonded` functions deal with locking and unlocking tokens, posing economic risks such as potential loss of funds due to bugs or vulnerabilities in these functions.

### Security Risk
Any vulnerabilities in the smart contract logic or dependencies could be exploited by malicious actors, including potential attacks on the `Currency` trait implementations or other dependencies used in the module.

### Governance Risk
Changes to the parameters or logic of the module could impact the governance of the system, for example, changes to `MinBond`, `UnbondingPeriod`, or `MaxUnbondingChunks` could affect the behavior of the bonding and unbonding process.

# Incentives

Substrate pallet for managing incentives. It provides a mechanism for recording and managing shares, accumulating rewards, and distributing rewards to users based on their shares in various pools. The pools can be of different types, such as Loans and DEX.

**Storage items:**
- IncentiveRewardAmounts: a mapping from pool to its fixed incentive amounts of multi-currencies per period.
- ClaimRewardDeductionRates: a mapping from pool to its claim reward deduction rate.
- ClaimRewardDeductionCurrency: a mapping from pool to its claim reward deduction currency.
- PendingMultiRewards: a mapping from pool and account to its pending rewards amount.

also includes a number of events that are emitted when certain actions are performed, such as depositing or withdrawing DEX shares, claiming rewards, and updating incentive reward amounts.

has several configuration traits that must be implemented, including Config, RuntimeEvent, AccumulatePeriod, NativeCurrencyId, RewardsSource, UpdateOrigin, Currency, EmergencyShutdown, and PalletId.

**Functions:**
- deposit_dex_share
- withdraw_dex_share
- claim_rewards
- update_incentive_rewards
- update_claim_reward_deduction_rates
- update_claim_reward_deduction_currency

These functions can be called by external actors to manage the state of the pallet.

**Internal functions:**
- do_deposit_dex_share
- do_withdraw_dex_share
- do_claim_rewards
- payout_reward_and_reaccumulate_reward
- get_incentive_reward_amount

These functions are used by the public-facing functions to perform the actual work of managing shares and rewards.

also implements the DEXIncentives and IncentivesManager traits, which provide additional functionality for managing DEX shares and rewards.

Finally, includes several types for handling events, including OnUpdateLoan, OnEarningBonded, and OnEarningUnbonded. These types are used to handle events that occur in other parts of the system and trigger updates to the state of the pallet.

## System Risk

- **Systemic risks**: relies on the orml_rewards pallet to manage shares and rewards. If there is a bug or issue in the orml_rewards pallet, it may affect the functionality and validity of this pallet.
- **Technical risks**: uses various Substrate-specific types and functions, such as `frame_support`, `frame_system`, and `sp_runtime`. If there are any bugs or issues in these dependencies, it may cause to become invalid or lose validation.
- **Currency and pool ID validation**: uses various currency and pool IDs throughout the code. If these IDs are not properly validated, it may cause to become invalid or lose validation.
- **Rate and amount validation**: uses various rates and amounts throughout the code. If these values are not properly validated, it may cause to become invalid or lose validation.
- **Weight and gas costs**: has various weight and gas costs associated with its functions. If these costs are not properly calculated or managed, it may cause to become invalid or lose validation.
- **Emergency shutdown**: has an emergency shutdown mechanism, which may be triggered by certain conditions. If the shutdown mechanism is not properly implemented or managed, it may cause to become invalid or lose validation.


# Rewards

For runtime module for managing a reward pool. A reward pool is a system where users can deposit a certain type of asset (called "shares") and earn rewards in various currencies over time. The rewards are distributed proportionally to the amount of shares each user has deposited.

## Configuration
configured with several types and a handler. The types include Share, Balance, PoolId, and CurrencyId. The handler is an instance of RewardHandler, which is responsible for handling the actual reward distribution.

## Errors
defines two errors: `PoolDoesNotExist` and `ShareDoesNotExist`. These errors are thrown when a user tries to interact with a pool or share that doesn't exist.

## Storage
uses two storage items: `PoolInfos` and `SharesAndWithdrawnRewards`. `PoolInfos` stores information about each reward pool, including the total shares and rewards. `SharesAndWithdrawnRewards` stores information about each user's shares and withdrawn rewards in each pool.

## Functions
provides several functions for interacting with the reward pool:

- `accumulate_reward`: adds a reward to a pool in a specific currency.
- `add_share`: adds shares to a pool for a specific user.
- `remove_share`: removes shares from a pool for a specific user.
- `set_share`: sets the number of shares for a specific user in a pool.
- `claim_rewards`: claims all available rewards for a specific user in a pool.
- `claim_reward`: claims all available rewards for a specific user in a pool for a specific currency.
- `transfer_share_and_rewards`: transfers a portion of a user's shares and rewards to another user.

## System Risk

### Input Validation
Ensure that input values like `add_amount`, `remove_amount`, `new_share`, and `move_share` are valid and within expected ranges to prevent overflow, underflow, or incorrect behavior.

### Error Handling
Ensure that error cases are handled properly, and invalid operations are not allowed. For example, when trying to remove more shares than the account owns, or when attempting to claim rewards without any shares.

### Edge Cases
Test the system with edge cases, such as when the total shares or total rewards are zero, to ensure that the system behaves correctly in such scenarios.

### Efficiency
Consider optimizing the code for efficiency, especially in operations that involve large numbers of shares or rewards.

### Integration Risks
When integrating this code with other systems or modules, ensure that the integration is secure and does not introduce vulnerabilities.

### Overflow and Underflow
Use `saturating_add` and `saturating_sub` to prevent overflow and underflow issues. Ensure that all arithmetic operations are protected against these issues.

### Pool and Share Manipulation
Allow adding and removing shares, as well as claiming rewards. Make sure that these operations cannot be exploited to manipulate the reward pool or shares in an unintended way.

## Technical Risks

### Incorrect Calculations
Use `U256` for some calculations, which might lead to precision loss when converting back to `Balance` type. Ensure that all calculations are correct and do not lead to unintended behavior.

### Concurrency Issues
Handle concurrent access to the storage. If multiple transactions are executed simultaneously, it might lead to race conditions or inconsistent state. Use Substrate's storage APIs correctly to handle concurrent access.

### Error Handling
Avoid using `unwrap()` as it might panic if the option is `None`. Replace these with proper error handling to prevent panics.

### Performance
Consider performance optimizations, such as zero-cost abstractions and efficient memory management. Ensure that the code is optimized for performance and that there are no unnecessary performance bottlenecks.

### Scalability
Handle a large number of users and transactions. Ensure that the code can scale horizontally and vertically as the system grows.

### Monitoring and Logging
Implement logging and monitoring capabilities to help in debugging and troubleshooting issues. Ensure that the logging and monitoring are comprehensive and provide enough information to diagnose and resolve issues quickly.


# Analysis of the code base and diagram architecture

## Earning
![earning](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/e519017c-8076-4755-93c7-ed71d3f64ed8)

## Incentives
![incentives](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/f75042d0-c71b-4a9e-8e52-c02f556f3223)

## Rewards
![rewards](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/e3391e08-f423-448f-8fa9-e20df8333926)

# Conclusion

In general, the Acala Network project exhibits an interesting and well-developed architecture we believe the team has done a good job regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from potential malicious use cases. Additionally, it is recommended to improve the documentation and comments in the code to enhance understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

### Time spent:
32 hours