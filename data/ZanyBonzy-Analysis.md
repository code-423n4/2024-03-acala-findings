#  **Advanced Analysis Report for <img width= auto src="https://gist.github.com/assets/112232336/ea241e1b-42c4-43f0-a8cc-123eda1b97d9" alt="logo">**

[Audit approach](#1-audit-approach)

[Brief Overview](#2-brief-overview)

[Scope and Architecture Overview](#3-scope-and-architecture-overview)

[Codebase Overview](#4-codebase-overview)

[Centralization Risks](#5-centralization-risks)

[Systemic Risks](#6-systemic-risks)

[Recommendations](#7-recommendations)

[Conclusions](#8-conclusions)

[Resources](#9-resources)


## **1. Audit approach**

**Phase 1**: Protocol Familiarization: The review began by analyzing the provided documentation, including the readme, checking out the protocol's website, followed by a preliminary examination of the relevant contracts and their tests.

**Phase 2**: Deep Contract Review: A meticulous, line-by-line review of the in-scope contracts was conducted, meticulously following the expected function interaction flow.

**Phase 3**: Issue Discussion and Resolution: Potential pitfalls identified during the review were discussed with the developers to clarify design choices and false positives from genuine issues.

**Phase 4**: Reporting: Audit reports were finalized based on the findings.

***

## **2. Brief Overview**

- Acala protocol offers, among others, a liqudity, staking and earning platform for users deposit or stake their `ACA` tokens to earn more rewards. This is facilitated by a simple process of staking and claiming whenever users see fit.
- Users also have the options of removing their stakes, either instantly for a fee, or by waiting for the 28 days cooldown period. There is also an option to restake previously unstaked tokens.  
***
## **3. Scope and Architecture Overview**

For the audit scope, the contracts are sectioned into three different sections.

***

### **3.2. Incentives pallet**

- This pallet is responsible depositing the tokens into the liquidity pool. It functions as the accounting book for keeping track of users' deposits and accumulated incentives.

*Liquidity deposit and withdrawal* - By calling the `deposit_dex_share` function, a user can deposit their tokens into the pool, to become eligible for various incentives. They receive a number of shares in return. This liquidity can later be withdrawn by the user if they so wish, by calling the `withdraw_dex_share` function. The function claims the user's remaining rewards, removes the user's shares from the pool and transfers them their liquidity.

*Rewards claim* - By calling the `claim_rewards` function, users can claim all the rewards available to them for a specific pool. If all rewards of a currency have been claimed, the currency is removed from the rewards list.

*Admin functiions* - The UpdateOrigin admin has the power to update the incentive rewards for diffent pools by calling the `update_incentive_rewards` function, He can also update the various deduction rates via the `update_claim_reward_deduction_rates` and the rewards currency via the `update_claim_reward_deduction_rates` function.

```

deposit_dex_share
    |
    |--> do_deposit_dex_share
    |       |
    |       |--> T::Currency::transfer
    |       |
    |       |--> <orml_rewards::Pallet<T>>::add_share
    |       |
    |       |--> Self::deposit_event
    |
withdraw_dex_share
    |
    |--> do_withdraw_dex_share
    |       |
    |       |--> T::Currency::transfer
    |       |
    |       |--> <orml_rewards::Pallet<T>>::remove_share
    |       |
    |       |--> Self::deposit_event
    |       
claim_rewards
    |
    |--> do_claim_rewards
    |       |
    |       |--> <orml_rewards::Pallet<T>>::claim_rewards
    |       |
    |       |--> PendingMultiRewards::<T>::mutate_exists
    |               |
    |               |--> Self::payout_reward_and_reaccumulate_reward
    |                       |
    |                       |--> <orml_rewards::Pallet<T>>::accumulate_reward
    |                       |
    |                       |--> T::Currency::transfer
    |                       |
    |                       |--> Self::deposit_event
    |
update_incentive_rewards
    |
    |--> IncentiveRewardAmounts::<T>::mutate_exists
    |       |
    |       |--> Self::deposit_event
    |
update_claim_reward_deduction_rates
    |
    |--> ClaimRewardDeductionRates::<T>::mutate_exists
    |       |
    |       |--> Self::deposit_event
    |
update_claim_reward_deduction_currency
    |
    |--> ClaimRewardDeductionCurrency::<T>::mutate_exists
    |       |
    |       |--> Self::deposit_event
    |
accumulate_incentives
    |
    |--> transfer_rewards_and_update_records
    |       |
    |       |--> T::Currency::transfer
    |       |
    |       |--> <orml_rewards::Pallet<T>>::accumulate_reward
    |
payout_reward_and_reaccumulate_reward
    |
    |--> <orml_rewards::Pallet<T>>::accumulate_reward
    |
    |--> T::Currency::transfer
    |
    |--> Self::deposit_event

```
***
### **3.2. Earning pallet**

- This contract provides a way for users to stake the protocol's native tokens for a period of time. Users can also unlock these tokens, while waiting for a particular cooldown period. If they aren't interested in waiting, they can unlock instantly for a fee. While the cooldown period is ongoing, users can decide to restake their tokens.

*Tokens bonding/unbonding process* - A user calls the `bond` function to lock the tokens in the protocol. To unlock the tokens, he either calls the `unbond_instant` to unlock the tokens for a fee, or `unbond` to wait for the stipulated time period. While waiting, the user can call `rebond` function to relock the unlocked tokens. Finally, upon the passage of the unlocking period, the user can call `withdraw_unbonded` to completely withdraw their tokens.

```
bond()
 |
 |--> OnBonded event
 |
 |--> Bonded event

unbond()
 |
 |--> OnUnbonded event
 |
 |--> Unbonded event

unbond_instant()
 |
 |--> OnUnstakeFee event
 |
 |--> OnUnbonded event
 |
 |--> InstantUnbonded event

rebond()
 |
 |--> OnBonded event
 |
 |--> Rebonded event

withdraw_unbonded()
 |
 |--> Withdrawn event

```
***
### **3.3. Rewards pallet**

- This pallet functions as the internal library upon which the other pallets are built. Due to its important status in calculating and handling protocol shares/rewards accounting, it is isolated from direct user interactions. 

Important functions here include: 

`accumulate_reward` which increases the rewards available in a pool.

`add_share` which increase the amount of shares in the pool when liquidity is added.

`remove_share` removes shares from pool when liquidity is removed.

`transfer_share_and_rewards` which allows users to transfer shares and corresponding rewards to other users. and so on.

```
accumulate_reward
    |
    |--> PoolInfos::mutate_exists
            |
            |--> rewards.entry
                    |
                    |--> and_modify
                    |--> or_insert

add_share
    |
    |--> PoolInfos::mutate
            |
            |--> rewards.iter_mut
                    |
                    |--> for_each
                            |
                            |--> SharesAndWithdrawnRewards::mutate
                                    |
                                    |--> withdrawn_inflation.into_iter
                                            |
                                            |--> for_each
                                                    |
                                                    |--> withdrawn_rewards.entry
                                                            |
                                                            |--> and_modify
                                                            |--> or_insert

remove_share
    |
    |--> Self::claim_rewards
    |--> SharesAndWithdrawnRewards::mutate_exists
            |
            |--> PoolInfos::mutate_exists
                    |
                    |--> rewards.iter_mut
                            |
                            |--> for_each
                                    |
                                    |--> if let Some
                                            |
                                            |--> rewards.remove

set_share
    |
    |--> add_share
    |--> remove_share

claim_rewards
    |
    |--> SharesAndWithdrawnRewards::mutate_exists
            |
            |--> PoolInfos::mutate_exists
                    |
                    |--> rewards.iter_mut
                            |
                            |--> for_each
                                    |
                                    |--> claim_one

claim_reward
    |
    |--> SharesAndWithdrawnRewards::mutate_exists
            |
            |--> PoolInfos::mutate
                    |
                    |--> if let Some
                            |
                            |--> claim_one

transfer_share_and_rewards
    |
    |--> SharesAndWithdrawnRewards::mutate
            |
            |--> SharesAndWithdrawnRewards::mutate_exists
                    |
                    |--> for_each
                            |
                            |--> increased_rewards.entry
                                    |
                                    |--> and_modify
                                    |--> or_insert

claim_one
    |
    |--> reward_to_withdraw

reward_to_withdraw
    |
    |--> U256::from
            |
            |--> saturating_mul
            |--> checked_div
            |--> unique_saturated_into

```
***
## **3. Codebase Overview**

- **Audit Information** - For the purpose of the security review, Acala comprises three smart contracts totaling over 1135 RLoC, holding an external import contract. There are five major structs and interfaces defined in scope and the codebase holds a test coverage of about 85% with mostly unit tests. Its core design principle is composition, enabling efficient and flexible integration.

- **Documentation** - The provided documentation provides serves as more of a marketing piece than a technical one, lacking various intricate details about the contracts and how they're expected to function. It's not really of much use to someone interested in the technial aspects. The contracts are moderately commented.

- **Protocol Ecosystem** - The protocol's ecosystem relies on the rust language, are scheduled to be deployed on Kurara and Acala substrate based networks. Testing and compilations are handled with cargo. 
 
- **Token Support** - The protocol mainly works with ERC20 tokens, which functions as the protocol's reward currencies and liquidity tokens.

- **Attack Vectors** - Paramount attack vectors for the protocol includes gaming rewards, bypassing fees for instant unbonding, gaining more shares than user should. Other points to consider are breaking of main invariants, griefing vectors from malicious users, and wrong accounting.

***


## **4. Centralization Risks** 
The protocol employs the actions of the `UpdateOrigin`. While this admin is essentially trusted and assumed to be correctly configured, having such a centralized power puts the protocol at risks, some of which are: 
- Maliciously removing reward currencies before all rewards have been claimed, causing the users rewards to be locked.
- Arbitrarily changing the rewards incentives and reward deduction rate to grief users.

***
## **5. Systemic Risks** 

- Low engagement, can cause low liquidity in pools and consequently, affect protocol.
- External dependencies from compilers, network issues and so on.
- Smart contract bugs which at best might be contained to erring contract and at worst could affect entire protocol.
- Non standard erc20 tokens as assets which can be a source of attack. 
- Slippage and deadline issues due to interactions with pools.

***
## **6. Recommendations**

- The `transfer_share_and_rewards` function in the rewards library doesn't prevent users from transferring to themselves. An analysis of the function shows that users can potentially increase their shares and rewards by doing just that. This is something to look into, even though the function cannot be accessed from outside, it can still have an effect in cases of integrations.
- The functions to add and remove liquidity to the pools lack a protection from slippage and which can lead to loss of funds for users as they'd receive less tokens than they expect.
- A minimum deposit amount can be enforced to protect from the possibilty of a first depositor bug, or from the reward calculation being forcefully scaled up by malicious users or from bloating the storage runtime with spam.
- A proper pause function can be implemented to protect users in cases of turbulent market situations and black swan events. It also helps prevent malicious activities or bugs from causing significant damage while the team investigates the potential issues without affecting users' funds.
- A deadline parameter can also be included to prevent transactions from being executed at an unfavorable time due to network congestion or other factors that could affect the outcome of the trade. By setting a deadline, users specify until what time they are willing to have their transaction included in the blockchain.
- Consider upgrading the various cargo packages in use to stay up to date with latest security fixes.
***
## **7. Conclusions**

-  As an endnote, the codebase was pretty well designed, albeit hard to crack, due its fairly unconventional nautre. All the same, a number of risks were identified and they need to be fixed. Recommended measures should be implemented to protect the protocol from potential attacks. Timely audits and codebase cleanup should be conducted to keep the codebase fresh and up to date with evolving security times.
***
## **8. Resources**

- [C4 ReadMe](https://code4rena.com/audits/2024-03-acala#top)
- [Protocol documentation](https://guide.acalaapps.wiki/staking/aca-staking)
- [Previous audits](https://github.com/AcalaNetwork/Acala/tree/master/audit)
- [Karura scan](https://karura.subscan.io/)
- [Acala scan](https://acala.subscan.io/)

### Time spent:
36 hours