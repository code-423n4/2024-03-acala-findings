# Analysis - Acala Contest

![Acala_Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FDfiqzUd3Mpd.0&w=256&q=75)

## Description overview of `Acala` Contest

![Acala-Image](https://guide.acalaapps.wiki/~gitbook/image?url=https:%2F%2F2484935697-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FHWNwYwj4jC1zxWo5TkwD%252Fuploads%252F7ikUJfGOlbVxxZn7COug%252FAcala%2520banner.jfif%3Falt=media%26token=e0f835a8-406b-4191-b003-bf07a9dcfe1c&width=768&dpr=4&quality=100&sign=5350fc8285f606b2e5272417a9a83bc20e86de0a4aa0391c2438ebc3fb93c505)

**🤔What is Acala Network?**

**Acala** is a DeFi platform built on the Polkadot blockchain. It aims to provide a suite of financial services, including lending, borrowing, staking, and trading.

Acala is a specialized stablecoin and liquidity blockchain that is decentralized, cross-chain by design, and future-proof with forkless upgradability. Acala stablecoin protocol uses multi-collateral-backing mechanism to create a stablecoin soft-pegged to the US Dollar. The Acala stablecoin protocol is essentially a decentralized monetary reserve that creates a stable currency from a basket of reserve assets, where holders of the reserve assets can spend, trade, access other services without price volatility while retaining their exposure to (without selling) the reserve assets.

Acala powered by Polkadot is cross-chain by nature. The reserve assets can be Polkadot native asset (DOT and DOT-derivatives), Acala native asset (ACA), and cross-chain assets like Bitcoin (BTC) and Ether (ETH). Acala stablecoin can be trustlessly integrated by other blockchains connected to Polkadot as well as applications on those chains. The Acala ecosystem continues to expand with a network of decentralized protocols, applications and parachains.

**Key Features of Acala:**

- **Decentralized Lending and Borrowing:** Acala allows users to lend and borrow crypto assets without the need for intermediaries. Lenders earn interest on their deposits, while borrowers can access liquidity at competitive rates.
- **Staking and Rewards:** Users can stake ACA, the native token of Acala, to earn rewards. Staking helps secure the network and provides access to exclusive features.
- **Trading:** Acala offers a decentralized exchange (DEX) where users can trade crypto assets in a non-custodial manner. The DEX is powered by the Substrate framework and provides fast and secure transactions.
- **Cross-Chain Compatibility:** Acala is built on Polkadot and is interoperable with other chains in the Polkadot ecosystem. This allows users to transfer assets and access services across multiple chains.

**Benefits of Using Acala:**

- **High Liquidity:** Acala has a large and growing user base, which provides deep liquidity for lending and trading.
- **Low Fees:** Transaction fees on Acala are significantly lower than on other DeFi platforms.
- **User-Friendly Interface:** Acala's interface is designed to be easy to use, even for beginners.
- **Community Governance:** ACA holders have a say in the development and governance of the Acala platform.

**Use Cases:**

Acala can be used for a variety of financial applications, including:

- **Earning interest on crypto assets:** Lenders can deposit their assets into Acala's lending pools and earn interest.
- **Borrowing crypto assets:** Borrowers can use Acala to access liquidity without selling their assets.
- **Stablecoin payments:** aUSD can be used as a stable medium of exchange for payments and other transactions.
- **Trading crypto assets:** Acala's DEX provides a secure and efficient platform for trading crypto assets.
- **Cross-chain asset transfer:** Acala allows users to transfer assets between Polkadot and other compatible blockchains.

## System Overview

## 1. Scope

### incentives/src/lib.rs:

The `Incentives` module in the Acala blockchain provides a framework for managing rewards and incentives for different protocols and pools. It allows protocols to distribute rewards to users based on their shares and accumulate rewards periodically.

**Key Features:**

- **Incentive Reward Accumulation:** The module accumulates fixed amounts of rewards for specific pool types and reward currencies at regular intervals. Rewards come from a designated "RewardsSource" account.
- **Share Management:** The module tracks the shares of users in different pools, allowing protocols to reward users based on their participation.
- **Reward Distribution:** Users can claim rewards for specific pools, which are distributed to their accounts after deducting any applicable fees or penalties.
- **Incentive Management:** The module allows authorized entities (e.g., governance) to update incentive reward amounts and other parameters related to reward distribution.

**Pool Types:**

- **Loans:** Manages rewards for users participating in the Loans protocol.
- **Dex:** Manages rewards for DEX makers who stake LP tokens.

**Reward Accumulation Mechanism:**

Incentives are accumulated periodically (as defined by the `AccumulatePeriod` configuration) based on the following formula:

```
Reward Amount = Incentive Reward Amount per Period
```

**Reward Distribution:**

When users claim rewards, the module distributes the accumulated rewards to their accounts. However, a portion of the rewards may be deducted as a fee or penalty, as determined by the following factors:

- **Claim Reward Deduction Rate:** A rate applied to all reward currencies for a specific pool.
- **Claim Reward Deduction Currency:** An optional currency that, if specified, applies the deduction rate only to that currency.

**Event Handling:**

The module emits various events to track important actions, such as:

- Depositing DEX shares
- Withdrawing DEX shares
- Claiming rewards
- Updating incentive reward amounts
- Updating claim reward deduction rates
- Updating claim reward deduction currency

**Additional Notes:**

- The module uses the `orml_rewards` pallet to handle the core reward distribution logic.
- The module has an associated trait (`DEXIncentives`) that allows other modules to interact with its DEX-related functionality.
- The module has an associated trait (`IncentivesManager`) that allows other modules to interact with its general reward management functionality.

### rewards/src/lib.rs:

This Substrate pallet provides a framework for managing reward pools and distributing rewards to participants based on their share in the pool. It allows users to add and remove shares, accumulate rewards, and claim their rewards.

**Key Concepts**

- **Reward Pool:** A collection of rewards that are distributed to participants based on their shares.
- **Share:** A measure of a participant's contribution to the pool.
- **Withdrawn Rewards:** The amount of rewards that a participant has already claimed.

**Data Structures**

- **PoolInfo:** Stores information about a reward pool, including the total shares, and the total and withdrawn rewards for each reward currency.
- **SharesAndWithdrawnRewards:** Stores the share and withdrawn rewards for each participant in a pool.

**Functions**

- **accumulate_reward:** Adds a new reward to a pool.
- **add_share:** Increases a participant's share in a pool.
- **remove_share:** Decreases a participant's share in a pool.
- **set_share:** Sets a participant's share in a pool to a specific value.
- **claim_rewards:** Claims all available rewards for a participant in a pool.
- **claim_reward:** Claims rewards for a specific reward currency for a participant in a pool.
- **transfer_share_and_rewards:** Transfers a portion of a participant's share and rewards to another participant.

**Reward Handler**

The pallet relies on a `RewardHandler` trait, which is implemented by a separate module. This trait defines the logic for distributing rewards to participants.

**Analysis**

This pallet provides a flexible and efficient framework for managing reward pools and distributing rewards. It allows users to easily add and remove shares, accumulate rewards, and claim their rewards. The use of BTreeMap data structures for storing pool and participant information ensures efficient access and modification.

The pallet also integrates with a `RewardHandler` trait, which allows developers to customize the logic for distributing rewards. This provides flexibility in implementing different reward distribution mechanisms, such as proportional rewards, tiered rewards, or vesting rewards.

### earning/src/lib.rs:

The Substrate-based pallet, named `Earning`, manages the bonding and unbonding of tokens by users. It allows users to lock up tokens to earn rewards and provides mechanisms for unbonding and withdrawing tokens.

**Key Concepts**

- **Bonding:** The process of locking up tokens to earn rewards.
- **Unbonding:** The process of releasing tokens from the bonded state.
- **Instant Unbonding:** Unbonding tokens immediately by paying a fee.
- **Rebonding:** The process of moving tokens from the unbonding state back to the bonded state.

**Data Structures**

- **Bonding Ledger:** A record of the tokens bonded by a user, including the amount bonded, the unbonding period, and the number of chunks unbonded.
- **Event:** A record of actions related to bonding, unbonding, and withdrawing tokens.

**Functions**

- **Bond:** Bonds the specified amount of tokens, locking them up to earn rewards.
- **Unbond:** Starts the process of unbonding the specified amount of tokens, which will be released after the unbonding period.
- **Unbond Instant:** Unbonds the specified amount of tokens immediately by paying a fee.
- **Rebond:** Rebonds the specified amount of tokens from the unbonding state back to the bonded state.
- **Withdraw Unbonded:** Withdraws all unbonded tokens.

**Event Handling**

The pallet emits events to record actions related to bonding, unbonding, and withdrawing tokens. These events can be used by other modules to track user activity and update their own state accordingly.

**Integration with Other Modules**

The `Earning` pallet integrates with the following modules:

- **Currency:** Used to manage the locking and unlocking of tokens.
- **OnBonded:** Notified when tokens are bonded.
- **OnUnbonded:** Notified when tokens are unbonded.
- **OnUnstakeFee:** Notified when an instant unbonding fee is paid.

**Analysis**

The `Earning` pallet provides a robust and flexible framework for managing token bonding and unbonding in a Substrate-based blockchain. It allows users to easily bond and unbond tokens, and to withdraw unbonded tokens. The integration with other modules ensures that the pallet's actions are properly tracked and accounted for.

## 2. Rewards module

This module exposes capabilities for staking rewards.

### Single asset algorithm

Consider a single pool with a single reward asset, generally, it will behave as next:

```python
from collections import defaultdict

pool = {}
pool["shares"] = 0
pool["rewards"] = 0
pool["withdrawn_rewards"] = 0

users = defaultdict(lambda: dict(shares = 0, withdrawn_rewards = 0))

def inflate(pool, user_share):
    return 0 if pool["shares"] == 0 else pool["rewards"] * (user_share / pool["shares"])

def add_share(pool, users, user, user_share):
    # virtually we add more rewards, but claim they were claimed by user
    # so until `rewards` grows, user will not be able to claim more than zero
    to_withdraw = inflate(pool, user_share)
    pool["rewards"] = pool["rewards"] + to_withdraw
    pool["withdrawn_rewards"] = pool["withdrawn_rewards"] + to_withdraw
    pool["shares"] += user_share
    user = users[user]
    user["shares"] += user_share
    user["withdrawn_rewards"] += to_withdraw

def accumulate_reward(pool, amount):
    pool["rewards"] += amount

def claim_rewards(pool, users, user):
    user = users[user]
    inflation = inflate(pool, user["shares"])
    to_withdraw = min(inflation - user["withdrawn_rewards"], pool["rewards"] - pool["withdrawn_rewards"])
    pool["withdrawn_rewards"]  += to_withdraw
    user["withdrawn_rewards"] += to_withdraw
    return to_withdraw
```

#### Prove

We want to prove that when a new share is added, it does not dilute previous rewards.

The user who adds a share after the reward is accumulated, will not get any part of the previous reward.

Let $R_n$ be the amount of the current reward asset.

Let $s_i$ be the stake of any specific user our of $m$ total users.

User current reward share equals

$$ r*i = R_n \* ({s_i} / {\sum*{i=1}^m s_i}) $$

User $m + 1$ brings his share, so

$$r_i' = R_n * ({s_i} / {\sum_{i=1}^{m+1} s_i}) $$

$r_i > r_i'$, so the original share was diluted and a new user can claim the share of existing users.

What if we increase $R_n$ by $\delta_R$ so that original users get the same share.

We get:

$$ R*n \* ({s_i} / {\sum*{i=1}^m s*i}) = ({R_n + \delta_R}) \* ({s_i} / {\sum*{i=1}^{m+1} s_i})$$

After easy to do algebraic simplification we get

$$ \delta*R = R_n \* ({s_m}/{\sum*{i=1}^{m} s_i}) $$

So for new share we increase reward pool. To compensate for that $\delta_R$ amount is marked as withdrawn from pool by new user.

## 3. Additional Context

- This code is deployed to both Karura and Acala

  - Karura Block Explorer: http://karura.subscan.io
  - Karura RPC: wss://karura-rpc.aca-api.network
  - Acala Block Explorer: http://acala.subscan.io
  - Acala RPC: wss://acala-rpc.aca-api.network

- Continious DoS more than 2 hours is considered valid.

## 4. Main invariants

- No token minting / burning
- Staker always gets more or equal amount of tokens back

## Approach Taken-in Evaluating `Acala`

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the `Acala`.

    I start with the following contracts, which substrate modules are used by `Acala Network` to implement `staking` and `earning` functionality and are in scope for this audit:

                     incentives/src/lib.rs:
                     rewards/src/lib.rs:
                     earning/src/lib.rs

    I started my analysis by examining the intricate structure and functionalities of the `Acala Network` protocol, focusing on the incentives mechanism and reward distribution system. That covers the `Incentives` module in the Acala protocol, responsible for managing rewards and incentives for various protocols and pools, including the Loans and DEX pools.The reward accumulation mechanism, which gathers fixed amounts of rewards for specific pool types and reward currencies at regular intervals. The share management system, which tracks users' shares in different pools, enabling protocols to reward users based on their participation. The reward distribution process, through which users can claim rewards for specific pools, with deductions applied for fees or penalties.The incentive management feature, allowing authorized entities to update incentive reward amounts and other reward distribution parameters.The `orml_rewards` pallet integration, which handles the core reward distribution logic.The associated traits (`DEXIncentives` and `IncentivesManager`) that allow other modules to interact with the DEX-related functionality and the general reward management functionality, respectively.

    Additionally, I explored the reward pool management and distribution in the `rewards` pallet, which provides a framework for managing reward pools and distributing rewards to participants based on their share in the pool.

    Lastly, I examined the `Earning` pallet, a Substrate-based module that manages the bonding and unbonding of tokens by users, allowing them to lock up tokens to earn rewards and providing mechanisms for unbonding and withdrawing tokens.

2.  **Documentation Review**:

    Then went to Review these [documentation](https://guide.acalaapps.wiki/staking/aca-staking), for a more detailed and technical explanation of `Acala Network`.

3.  **Compiling code and running provided tests**:

4.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Codebase Quality

Overall, I consider the quality of the `Acala Network` protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Architecture & Design**                | The protocol features a modular design, segregating functionality into distinct contracts (e.g., `Incentives`, `Rewards`, `Earning`) for clarity and ease of maintenance.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| **Error Handling & Input Validation**    | In the `Acala Network` codebase, includes error handling and input validation through the use of custom error types, such as `Error<T>` and `DispatchError`, which are used to indicate when certain conditions are not met (e.g. `InvalidCurrencyId`, `NotEnough shares`). Additionally, the ensure! macro is used to check for certain conditions and return an error if they are not met. The `claim_rewards` function also includes a check for the emergency shutdown status before distributing rewards. Input validation is also performed in functions such as `update_incentive_rewards` and `update_claim_reward_deduction_rates` by checking the validity of the pool and currency IDs.                                                                                                                                                                                                    |
| **Code Maintainability and Reliability** | The contracts are written with emphasis on sustainability and simplicity. The functions are single-purpose with little branching and low cyclomatic complexity.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Code Comments**                        | There must be different types of comments used in the `Acal Network` protocol: `Single-line comments`, `Multi-line comments`, `Documentation comments` for documentation, explanations, TODOs, and structuring the code for readability. However, the codebase lacks many comments. `NatSpec` tags also allow for automatically generating documentation. The contracts must be accompanied by comprehensive comments to facilitate an understanding of the functional logic and critical operations within the code. Functions must be described purposefully, and complex sections should be elucidated with comments to guide readers through the logic. Despite this, certain areas, particularly those involving intricate mechanics, could benefit from even more detailed commentary to ensure clarity and ease of understanding for developers new to the project or those auditing the code. |
| **Testing**                              | The contracts exhibit a commendable level of test coverage `85%` but with aim to 100% for indicative of a robust testing regime. This coverage ensures that a wide array of `functionalities` and `edge cases` are tested, contributing to the reliability and security of the code. However, to further enhance the testing framework, the incorporation of fuzz testing and invariant testing is recommended. These testing methodologies can uncover deeper, systemic issues by simulating extreme conditions and verifying the invariants of the contract logic, thereby fortifying the codebase against unforeseen vulnerabilities.                                                                                                                                                                                                                                                              |
| **Code Structure and Formatting**        | The codebase benefits from a consistent structure and formatting, adhering to the stylistic conventions and best practices of Rust programming. Logical grouping of functions and adherence to naming conventions contribute significantly to the readability and navigability of the code. While the current structure supports clarity, further modularization and separation of concerns could be achieved by breaking down complex contracts into smaller, more focused components. This approach would not only simplify individual contract logic but also facilitate easier updates and maintenance.                                                                                                                                                                                                                                                                                           |
| **Strengths**                            | The codebase is a well-structured, modular, and extensible implementation of a custom blockchain runtime using the Substrate framework, providing functionality for contract execution, storage management, chain extensions, and more, with a focus on security, performance, and universal compatibility across different blockchain networks.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| **Documentation**                        | While the `Documentation` provides `comprehensive` details for all functions and the system,However currently `no helpful inline comments` available for `auditors` or developers. It is crucial to develop inline comments to offer a comprehensive understanding of the contract's functionality, purpose, and interaction methods inside the codebase.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

## Systemic Risks, Centralization Risks, Technical Risks & Integration Risks

### incentives/src/lib.rs:

1.  **Systemic Risks:**

    - There is no sanctioning mechanism if the rewards source account fails to maintain adequate balances for incentive payouts. The contract could fail to pay out rewards if balances run out.

    - `Emergency shutdown`: The contract includes an emergency shutdown feature that can be triggered by the EmergencyShutdown module.

      ```rs
      use module_support::{DEXIncentives, EmergencyShutdown, FractionalRate, IncentivesManager, PoolId, Rate};
      ```

      If the emergency shutdown is triggered, no further `incentives` will be accumulated for the Loans pool, but other pools will continue to accumulate incentives. This could result in an unfair distribution of rewards.

      ```rs

                     // do not accumulate incentives for PoolId::Loans after shutdown
                  PoolId::Loans(_) if shutdown => {
                     log::debug!(
                        target: "incentives",
                        "on_initialize: skip accumulate incentives for pool {:?} after shutdown",
                        pool_id
                     );
                  }

      ```

2.  **Centralization Risks:**

    - The contract allows the `UpdateOrigin` to update the claim reward deduction rates and the deduction currency for specific pool IDs.

      ```rs

            pub fn update_incentive_rewards(
            origin: OriginFor<T>,
            updates: Vec<(PoolId, Vec<(CurrencyId, Balance)>)>,
         ) -> DispatchResult {
            T::UpdateOrigin::ensure_origin(origin)?;
            for (pool_id, update_list) in updates {
               if let PoolId::Dex(currency_id) = pool_id {
                  ensure!(currency_id.is_dex_share_currency_id(), Error::<T>::InvalidPoolId);
               }


      ```

    This could be exploited by the `UpdateOrigin` to unfairly deduct rewards from users, either by increasing the deduction rates or by targeting specific reward currencies.

3.  **Technical Risks:**

    - The accumulation logic in the `accumulate_incentives` function could fail or get stuck if one of the reward transfers or record updates fails. This would stop incentives from accumulating properly.

    - The `claim_rewards` call could fail during the payout process, preventing users from claiming their rewards.

    -The pending rewards state stored in PendingMultiRewards could get corrupted or stuck if the writes to it fail. Then rewards wouldn't be tracked properly.

4.  **Integration Risks:**

    - The contract's reliance on the `Happened` trait implementation and its interaction with the ORML rewards module adds to the integration complexity and introduces unique risks that may be difficult to anticipate and mitigate.

    - Changes to interfaces with other pallets/protocols like rewards or loans could break incentives syncing

### rewards/src/lib.rs:

1.  **Systemic Risks:**

    - **Reward Currency Management Risk:** The contract manages rewards for multiple currencies using a `BTreeMap`. If the number of reward currencies becomes large, it could impact the performance and storage requirements of the contract.

    - **Overflow/Underflow Risk:** The contract uses fixed-point arithmetic operations (`FixedPointOperand`) for reward calculations. While it attempts to handle overflows and underflows using `saturating` operations, there is still a risk of unexpected behavior or precision loss in certain edge cases. For example, the `add_share` function calls `pool_info.total_shares.saturating_add(add_amount)`, which could wrap around if `add_amount` is a large value.

      ```rs
            PoolInfos::<T>::mutate(pool, |pool_info| {
               let initial_total_shares = pool_info.total_shares;
               pool_info.total_shares = pool_info.total_shares.saturating_add(add_amount);

               let mut withdrawn_inflation = Vec::<(T::CurrencyId, T::Balance)>::new();

               pool_info
      ```

    - **Reward inflation:** The pool's rewards are based on a fixed issuance rate, which could lead to inflation if the demand for the pool's shares does not keep up with the supply.

    - **Reward distribution:** The pool's rewards are distributed based on the number of shares held, which could lead to a concentration of rewards in the hands of a few large shareholders.

    - The `claim_rewards` function allows users to claim rewards based on their share of the total rewards. However, there is no check in place to ensure that the user's share is still valid or that they have not already claimed their rewards. An attacker could potentially claim rewards multiple times or claim rewards on behalf of other users.

      ```rs

         pub fn claim_rewards(who: &T::AccountId, pool: &T::PoolId) {
            SharesAndWithdrawnRewards::<T>::mutate_exists(pool, who, |maybe_share_withdrawn| {
               if let Some((share, withdrawn_rewards)) = maybe_share_withdrawn {
                  if share.is_zero() {
                     return;
                  }

                  PoolInfos::<T>::mutate_exists(pool, |maybe_pool_info| {
                     if let Some(pool_info) = maybe_pool_info {
                        let total_shares = U256::from(pool_info.total_shares.to_owned().saturated_into::<u128>());
                        pool_info.rewards.iter_mut().for_each(
                           |(reward_currency, (total_reward, total_withdrawn_reward))| {
                              Self::claim_one(
                                 withdrawn_rewards,
                                 *reward_currency,
                                 share.to_owned(),
                                 total_reward.to_owned(),
                                 total_shares,
                                 total_withdrawn_reward,
                                 who,
                                 pool,
                              );
                           },
                        );
                     }
                  });
               }
            });
         }

      ```

    - The `transfer_share_and_rewards` function allows users to transfer their share of the rewards to another user. However, there is no check in place to ensure that the `move_share` argument is valid or that the `other` user exists. An attacker could potentially exploit this to transfer shares to non-existent accounts or to manipulate the shares of other users.

      ```rs

            pub fn transfer_share_and_rewards(
            who: &T::AccountId,
            pool: &T::PoolId,
            move_share: T::Share,
            other: &T::AccountId,
         ) -> DispatchResult {
            SharesAndWithdrawnRewards::<T>::mutate(pool, other, |increased_share| {
               let (increased_share, increased_rewards) = increased_share;
               SharesAndWithdrawnRewards::<T>::mutate_exists(pool, who, |share| {
                  let (share, rewards) = share.as_mut().ok_or(Error::<T>::ShareDoesNotExist)?;
                  ensure!(move_share < *share, Error::<T>::CanSplitOnlyLessThanShare);
                  for (reward_currency, balance) in rewards {
                     // u128 * u128 is always less than u256
                     // move_share / share always less then 1 and share > 0
                     // so final results is computable and is always less or equal than u128
                     let move_balance = U256::from(balance.to_owned().saturated_into::<u128>())
                        * U256::from(move_share.to_owned().saturated_into::<u128>())
                        / U256::from(share.to_owned().saturated_into::<u128>());
                     let move_balance: Option<u128> = move_balance.try_into().ok();
                     if let Some(move_balance) = move_balance {
                        let move_balance: T::Balance = move_balance.unique_saturated_into();
                        *balance = balance.saturating_sub(move_balance);
                        increased_rewards
                           .entry(*reward_currency)
                           .and_modify(|increased_reward| {
                              *increased_reward = increased_reward.saturating_add(move_balance);
                           })
                           .or_insert(move_balance);
                     }
                  }
                  *share = share.saturating_sub(move_share);
                  *increased_share = increased_share.saturating_add(move_share);
                  Ok(())
               })
            })
         }
      ```

2.  **Centralization Risks:**

    - The pallet allows for the creation of pools with different reward structures. If a few pools become dominant, it could lead to a concentration of power and potentially unfair distribution of rewards.

3.  **Technical Risks:**

    - There are several instances of unchecked arithmetic operations (e.g., `saturating_add`, `saturating_sub`) that could lead to unexpected behavior if the values overflow or underflow.

    - The `transfer_share_and_rewards` function has a potential for rounding errors due to the fixed-point arithmetic used to calculate the moved rewards.

4.  **Integration Risks:**

    - The contract heavily relies on the correct implementation of various traits and types (e.g., `Config`, `CurrencyId`, `Share`, `Balance`) provided by other modules or libraries. Any changes or issues in these dependencies could break the contract's functionality.

### earning/src/lib.rs:

1. **Systemic Risks:**

   - The possibility of a user's available balance being less than the amount they want to bond, resulting in all of their remaining balances being locked. This could potentially lead to a loss of access to funds for the user.

   - The use of the `InstantUnstakeFee` parameter, which allows users to unbond their tokens instantly by paying a fee. This fee is calculated as a percentage of the amount being unbonded, and could potentially be set to a very high value, leading to a significant loss of funds for the user.

     ```rs

           pub fn unbond_instant(origin: OriginFor<T>, #[pallet::compact] amount: Balance) -> DispatchResult {
              let who = ensure_signed(origin)?;

              let fee_ratio = T::ParameterStore::get(InstantUnstakeFee).ok_or(Error::<T>::NotAllowed)?;

     ```

   - The use of the `LockIdentifier` parameter, which is used to identify the lock that is placed on a user's funds when they are bonded. If this parameter is not set correctly, it could potentially lead to a loss of funds for the user.

   - The use of the `MinBond` and `MaxUnbondingChunks` parameters, which set the minimum amount that can be bonded and the maximum number of unbonding chunks that can be created, respectively. If these parameters are not set correctly, they could potentially lead to a loss of funds for the user.

     ```rs
        type MinBond = T::MinBond;
     type MaxUnbondingChunks = T::MaxUnbondingChunks;
     type Moment = BlockNumberFor<T>;
     ```

   - The module allows for instant unbonding of tokens by paying a fee determined by the `InstantUnstakeFee` parameter. If this parameter is set too low, it could encourage users to frequently unbond their tokens instantly, which may not be desirable for the system's design.

   - **Instant Unstaking Fee Risk:** The contract charges a fee for instant unstaking, which could discourage users from unstaking their tokens and reduce the liquidity of the system.

2. **Centralization Risks:**

   - The contract uses a `ParameterStore` to manage certain parameters, such as the `InstantUnstakeFee`. The centralized control of these parameters could introduce risks if not managed properly.

     ```rs
           pub fn unbond_instant(origin: OriginFor<T>, #[pallet::compact] amount: Balance) -> DispatchResult {

              let who = ensure_signed(origin)?;

              let fee_ratio = T::ParameterStore::get(InstantUnstakeFee).ok_or(Error::<T>::NotAllowed)?;

     ```

   - The locking logic is tied to a single lock identifier (`T::LockIdentifier`), so if that identifier becomes compromised it could allow improper unlocking of funds.

3. **Technical Risks:**

   - The `refund/balancing` mechanisms used for unstaking could have edge cases that lead to loss of funds.

   - The contract assumes that the `T::Currency` implementation correctly handles locking and unlocking of tokens, which could lead to issues if the implementation is faulty.

   - The contract assumes that the `T::Currency` implementation correctly handles locking and unlocking of tokens, which could lead to issues if the implementation is faulty.

4. **Integration Risks:**

   - **Compatibility with Substrate Ecosystem**: The contract heavily relies on Substrate-specific components, such as `frame_support`, `frame_system`, and `orml_traits`. Changes or updates to these Substrate components could introduce integration risks.

   - The contract has an `UnbondingPeriod` parameter, which determines the duration of the unbonding process. This introduces a risk, as users may need to wait for a significant amount of time before they can fully withdraw their unbonded tokens.

   ```rs
         pub fn unbond(origin: OriginFor<T>, #[pallet::compact] amount: Balance) -> DispatchResult {
            let who = ensure_signed(origin)?;

            let unbond_at = frame_system::Pallet::<T>::block_number().saturating_add(T::UnbondingPeriod::get());
            let change = <Self as BondingController>::unbond(&who, amount, unbond_at)?;

   ```

## Suggestions

### What ideas can be incorporated?

**Ideas for Incorporating Unique Features into Acala Network Protocol Rewards and Incentives:**

- **Dynamic Reward Allocation:** Implement a dynamic reward allocation mechanism that adjusts the distribution of rewards based on factors such as pool performance, user activity, and market conditions, ensuring optimal utilization of incentives.
- **Milestone-Based Rewards:** Introduce milestone-based rewards to incentivize users to achieve specific goals or milestones, such as reaching certain liquidity thresholds or participating in governance activities.
- **Cross-Chain Reward Compatibility:** Enable users to earn rewards and incentives for participating in cross-chain activities, such as cross-chain liquidity provision or cross-chain yield farming.
- **Delegated Staking:** Allow users to delegate their staking power to trusted individuals or organizations, enabling them to earn rewards without actively managing their stake.
- **Non-Fungible Token (NFT) Rewards:** Distribute NFTs as rewards for participation in specific activities or milestones, providing users with unique and valuable digital collectibles.

### What’s unique?

1. **Overall Unique Features of Acala Network Protocol:**

   - **Cross-Chain Interoperability:** Acala enables seamless asset transfers and interoperability between Polkadot, Ethereum, and other blockchains via its Cross-Chain Messaging (XCM) protocol.

   - **Native Stablecoin (aUSD):** Acala introduces a decentralized stablecoin, aUSD, backed by a basket of assets, including ACA, DOT, and other cryptocurrencies, providing stability and liquidity in the ecosystem.

   - **Liquid Staking (L-DOT):** Acala offers a liquid staking mechanism for DOT, allowing holders to earn staking rewards while maintaining liquidity and participating in governance.

   - **Substrate-Based:** Acala is built on Substrate, a modular blockchain framework from Parity Technologies, providing flexibility and scalability for its protocol.

   - **EVM Compatibility:** Acala supports the Ethereum Virtual Machine (EVM), enabling developers to easily deploy and execute Ethereum-based smart contracts on the Acala network.

   - **Cross-Chain DEX (Karura):** Acala's parachain, Karura, features a cross-chain decentralized exchange (DEX) that allows users to trade assets seamlessly between different blockchains.

2. **Unique Features of Acala Network Protocol Related to Incentives and Rewards:**

   - **Flexible Incentive Distribution:** Acala's incentive module allows pools to accumulate incentives from multiple sources, providing flexibility in rewarding users for their participation.

   - **Configurable Deduction Rate:** Rewards can be subject to a configurable deduction rate when claimed by users, enabling the protocol to adjust the distribution of incentives as needed.

   - **Adjustable Rewards:** When new users add liquidity to a pool, their rewards are adjusted accordingly, ensuring fair distribution based on their contribution.

   - **Earning through Bonding:** Users can earn rewards by bonding or locking their tokens for a specified period of time.

   - **Unbonding/Unlocking with Fees or Waiting Period:** Unbonding or unlocking tokens may incur a fee or require a waiting period, providing incentives for users to maintain their participation in the protocol.

   - **Extensibility through Hooks:** The earning module implements hooks that allow other modules, such as the incentives module, to implement staking and reward distribution mechanisms.

## Issues surfaced from Attack Ideas

- Find a way to unstake bypass the bonding duration

- Find a way to gain more shares than you should

- Find a way to claim more rewards than you should




### Time spent:
19 hours