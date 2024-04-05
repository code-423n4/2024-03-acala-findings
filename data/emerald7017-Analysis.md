## Project Overview:
Acala is building the liquidity layer of Web3 finance by providing a decentralized platform for staking, earning, and incentivizing liquidity. The core modules under review are:
1. [Earning](https://github.com/code-423n4/2024-03-acala/tree/main/src/modules/earning/): Allows users to bond/lock tokens for a certain period to earn rewards. 
2. [Rewards](https://github.com/code-423n4/2024-03-acala/tree/main/src/orml/rewards/): Calculates and distributes rewards based on user shares in reward pools.
3. [Incentives](https://github.com/code-423n4/2024-03-acala/tree/main/src/modules/incentives/): Enables reward pools to accumulate incentives and users to claim rewards.

## Evaluation Approach:
- Staking and unbonding logic 
- Share calculation and allocation
- Reward computation and distribution 
- Interactions between modules
- Potential economic attacks and DoS vectors
- Access control and admin privileges

## Architecture Analysis
The Earning, Rewards and Incentives modules follow a modular design pattern, with clear separation of concerns. **However, some potential architectural risks were identified:**
- High degree of coupling between modules, with cross-module interactions that could lead to unintended consequences if not properly managed.
- Heavy reliance on accurate share calculation in Rewards module. Errors here could have ripple effects.
- Use of fixed-point arithmetic may introduce precision loss.

**Recommendation: Consider more rigorous testing around module interactions. Evaluate safer alternatives to fixed-point math.**

## Code Quality:
The codebase is well-structured and follows Rust best practices. Key observations:
- Proper use of visibility modifiers and encapsulation.
- Consistent error handling via `DispatchError`.
- Effective use of traits for abstraction.
- Comprehensive test suite covering key functionalities.

However, some areas for improvement were noted:
- Lack of detailed documentation for critical functions.
- Some large functions could be refactored for better readability.
- More descriptive variable names would enhance code clarity.

**Recommendation: Prioritize documentation improvements and establish coding guidelines to maintain consistency.**

## Centralization Risks
The `UpdateOrigin` trait is used for admin-controlled parameters, including:
- `UnbondingPeriod` and `InstantUnstakeFee` in Earning module
- Reward rates and deduction rates in Incentives module

If `UpdateOrigin` is not properly managed, it could lead to centralized control and admin abuse.

**Recommendation: Ensure `UpdateOrigin` is set to a decentralized governance mechanism, such as a DAO or multisig. Implement timelocks and limits on parameter changes.**

## Mechanism Review:
1. [Earning](https://github.com/code-423n4/2024-03-acala/tree/main/src/modules/earning/): The bonding, unbonding, and withdrawing mechanisms are clearly defined and appear secure. However, the instant unbond fee (`InstantUnstakeFee`) could potentially be set to an unreasonably high value by admin.

2. [Rewards](https://github.com/code-423n4/2024-03-acala/tree/main/src/orml/rewards/): The share-based reward distribution model is vulnerable to rounding errors due to fixed-point math. This could lead to precision loss and unfair allocation over time.

3. [Incentives](https://github.com/code-423n4/2024-03-acala/tree/main/src/modules/incentives/): The reward claiming process is well-designed, with proper checks and balances. However, the deduction rate applied during claims is admin-controlled and could be abused.

**Recommendation: Implement reasonable limits on admin-controlled parameters. Consider moving to integer-based arithmetic for share calculation.**

## Systemic Risks:
- Flash loan attacks: An attacker could potentially use flash loans to temporarily inflate their shares and claim more rewards than deserved. While costly, this could drain reward pools over time.

- DoS attacks: If `UnbondingPeriod` is set to an unreasonably high value, it could effectively lock user funds and disrupt staking operations.

- Cascading failures: With the high interdependence between modules, a serious bug in one module could propagate and cause system-wide issues.

**Recommendation: Incorporate flash loan resistance mechanisms, such as time-weighted average shares. Implement circuit breakers and emergency pause functionality to halt the system in case of critical failures.**

## Conclusion:
The Acala codebase is well-designed and follows best practices, but there are some notable risks around centralization, precision loss, and economic attacks. By implementing the recommended safeguards and continuing to prioritize security and decentralization, Acala can provide a robust and trustworthy liquidity layer for Web3 finance.


Part II || Weaknesses/exploits that could allow users to bypass bonding durations, gain excessive shares, or claim unearned rewards.
------------------------------------------------------------------------------

`1. Bypassing Bonding Durations:`
- The `Earning` module enforces bonding durations through the `UnbondingPeriod` parameter, which determines the time users must wait before withdrawing their bonded funds.
- The `unbond` function (line 145) correctly calculates the `unbond_at` timestamp by adding `UnbondingPeriod` to the current block number.
- The `withdraw_unbonded` function (line 222) checks if the current block number is greater than or equal to the `unbond_at` timestamp before allowing withdrawals.

A potential weakness lies in the `InstantUnstakeFee` parameter, which allows users to instantly unbond funds by paying a fee. If this fee is set too low, it could effectively nullify the bonding duration.

`Recommendation: Implement a minimum fee threshold and a maximum unbonding limit to prevent abuse of the instant unbond feature.`

`2. Gaining Excessive Shares:`
- The `Rewards` module calculates user shares based on their bonded amount and the total shares in the pool.
- The `add_share` function (line 132) correctly updates the user's shares and the pool's total shares based on the added stake.
- The `remove_share` function (line 195) properly updates the shares when a user unbonds.

The share calculation relies on fixed-point arithmetic, which could lead to rounding errors and potential share inflation over time.

// Example: [add_share function](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L158-L167)
```rust
// Example: add_share function
let reward_inflation = if initial_total_shares.is_zero() {
    Zero::zero()
} else {
    U256::from(add_amount.to_owned().saturated_into::<u128>())
        .saturating_mul(total_reward.to_owned().saturated_into::<u128>().into())
        .checked_div(initial_total_shares.to_owned().saturated_into::<u128>().into())
        .unwrap_or_default()
        .as_u128()
        .saturated_into()
};
```

Recommendation: Consider using integer-based arithmetic with appropriate scaling factors to minimize rounding errors. Regularly audit share balances to detect and correct any discrepancies.

3. Claiming Unearned Rewards:
- The `Incentives` module allows users to claim their earned rewards based on their shares in the reward pool.
- The `claim_rewards` function (line 164) calculates the claimable rewards by comparing the user's current share of the total rewards with their previously withdrawn rewards.
- The claimed rewards are correctly deducted from the pool's balance and added to the user's withdrawn rewards.

However, there is a risk of users manipulating their share balance to claim more rewards than earned, especially if the share calculation is vulnerable to rounding errors.

// Example: [claim_rewards function](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L375-L381)
```rust
// Example: claim_rewards function
let reward_to_withdraw = Self::reward_to_withdraw(
    share,
    total_reward,
    total_shares,
    withdrawn_reward,
    total_withdrawn_reward.to_owned(),
);
```

Recommendation: Implement additional checks to verify the consistency between a user's claimed rewards and their historical share balance. Use integer-based arithmetic to minimize rounding errors.

4. Economic Attacks:
- The Acala ecosystem may be vulnerable to economic attacks, such as flash loan exploits, where attackers temporarily inflate their share balance to claim unearned rewards.
- The current codebase does not have explicit protections against flash loan attacks.

Recommendation: Incorporate flash loan resistance mechanisms, such as time-weighted average shares or a cooldown period for share updates. Monitor the system for suspicious activity and implement emergency stop mechanisms.

5. Centralization Risks:
- The `UpdateOrigin` trait is used to control critical parameters, such as the `UnbondingPeriod`, `InstantUnstakeFee`, and reward rates.
- If the `UpdateOrigin` is not properly decentralized, it could lead to admin abuse and centralized control over the staking and earning system.

Recommendation: Ensure that the `UpdateOrigin` is set to a decentralized governance mechanism, such as a DAO or a multisig contract. Implement strict access controls and monitoring for admin actions.

In conclusion, while the Acala codebase follows best practices and has a robust design, there are potential weaknesses in the share calculation, instant unbonding, and economic attack resistance. By addressing these issues through the recommended solutions, Acala can significantly strengthen the security and integrity of its staking and earning modules.

### The core components of Acala include:
1. Earning: Allows users to bond (lock) their tokens for a specified period to earn rewards.
2. Rewards: Calculates and distributes rewards to users based on their shares in reward pools.
3. Incentives: Enables reward pools to accumulate incentives and users to claim their earned rewards.

By offering attractive earning opportunities and incentivizing liquidity, Acala aims to create a vibrant ecosystem that supports the growth and adoption of Web3 finance.

The Acala system consists of three main modules: Earning, Rewards, and Incentives. These modules work together to provide a seamless staking and earning experience for users.

1. Earning Module:
- Allows users to bond (lock) their tokens for a specified period.
- Enforces bonding durations and manages the unbonding process.
- Supports instant unbonding with a configurable fee.
- Emits events for bonding and unbonding actions.

2. Rewards Module:
- Calculates and distributes rewards to users based on their shares in reward pools.
- Maintains a record of user shares and withdrawn rewards.
- Provides functions for adding and removing shares, and claiming rewards.
- Supports reward distribution for multiple reward currencies.

3. Incentives Module:
- Enables reward pools to accumulate incentives from external sources.
- Allows users to claim their earned rewards based on their shares in the pool.
- Applies a configurable deduction rate during the reward claiming process.
- Manages the incentive reward amounts and emission rates.

### Functions
1. [Earning Module](https://github.com/code-423n4/2024-03-acala/tree/main/src/modules/earning/):
- `bond`: Allows users to bond (lock) their tokens for a specified period.
- `unbond`: Initiates the unbonding process for a user's bonded tokens.
- `unbond_instant`: Allows instant unbonding of tokens by paying a fee.
- `rebond`: Allows users to rebond their unbonding tokens.
- `withdraw_unbonded`: Allows users to withdraw their unbonded tokens after the unbonding period.

2. [Rewards Module](https://github.com/code-423n4/2024-03-acala/tree/main/src/orml/rewards/):
- `add_share`: Adds a user's share to a reward pool based on their bonded amount.
- `remove_share`: Removes a user's share from a reward pool when they unbond.
- `claim_rewards`: Allows users to claim their earned rewards from a reward pool.
- `claim_reward`: Allows users to claim a specific reward currency from a reward pool.
- `transfer_share_and_rewards`: Transfers a portion of a user's share and rewards to another user.

3. [Incentives Module](https://github.com/code-423n4/2024-03-acala/tree/main/src/modules/incentives/):
- `deposit_dex_share`: Allows users to deposit their DEX shares to earn incentives.
- `withdraw_dex_share`: Allows users to withdraw their DEX shares.
- `claim_rewards`: Allows users to claim their earned incentive rewards.
- `update_incentive_rewards`: Updates the incentive reward amounts for specific pools.
- `update_claim_reward_deduction_rates`: Updates the deduction rates applied during reward claiming.

### Gram
1. Users:
- Participate in staking by bonding their tokens.
- Earn rewards based on their shares in reward pools.
- Claim their earned rewards.
- Withdraw their unbonded tokens after the unbonding period.

2. Administrators:
- Set and update system parameters, such as bonding durations, instant unbond fees, and reward rates.
- Monitor the system's operation and performance.
- Respond to any security incidents or network issues.

3. Liquidity Providers:
- Provide liquidity to DEX pools and earn incentives.
- Deposit and withdraw their DEX shares.
- Claim their earned incentive rewards.

4. External Reward Sources:
- Provide incentive rewards to specific reward pools.
- Determine the reward amounts and emission rates.

Architecture and Workflow:
1. User Bonding:
- Users bond their tokens using the `bond` function in the Earning module.
- The bonded tokens are locked for a specified period.
- The Earning module emits a `Bonded` event, which is captured by the Incentives module.

2. Share Calculation:
- The Incentives module listens for `Bonded` events and updates the user's share in the corresponding reward pool using the `add_share` function in the Rewards module.
- The user's share is calculated based on their bonded amount and the pool's total shares.

3. Reward Accumulation:
- External reward sources provide incentive rewards to specific reward pools.
- The Incentives module accumulates these rewards and updates the pool's reward balances.

4. Reward Claiming:
- Users can claim their earned rewards using the `claim_rewards` function in the Incentives module.
- The Rewards module calculates the claimable rewards based on the user's share and the pool's total rewards.
- The claimed rewards are deducted from the pool's balance and added to the user's withdrawn rewards.
- A configurable deduction rate is applied during the reward claiming process.

5. Unbonding and Withdrawing:
- Users can initiate the unbonding process using the `unbond` function in the Earning module.
- The unbonding period starts, during which the tokens remain locked.
- After the unbonding period, users can withdraw their unbonded tokens using the `withdraw_unbonded` function.

6. Instant Unbonding:
- Users can opt for instant unbonding using the `unbond_instant` function in the Earning module.
- An instant unbond fee is deducted from the unbonded amount.
- The remaining tokens are immediately available for withdrawal.

7. Rebonding:
- Users can rebond their unbonding tokens using the `rebond` function in the Earning module.
- The rebonded tokens are added back to the user's bonded balance.

_This architecture and workflow ensure a seamless interaction between the Earning, Rewards, and Incentives modules, enabling users to participate in staking, earn rewards, and claim their earned incentives._

### 1. Potential Weakness: Instant Unbonding Fee Manipulation
Reference: [`pallets/earning/src/lib.rs#unbond_instant`](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/earning/src/lib.rs#L180-L205)

The `unbond_instant` function allows users to instantly unbond their tokens by paying a fee specified by the `InstantUnstakeFee` parameter. However, if this fee is set too low, it could be abused to circumvent the normal unbonding process and potentially destabilize the system.

```rust
#[pallet::weight(T::WeightInfo::unbond_instant())]
pub fn unbond_instant(origin: OriginFor<T>, #[pallet::compact] amount: Balance) -> DispatchResult {
    let who = ensure_signed(origin)?;
    let fee_ratio = T::ParameterStore::get(InstantUnstakeFee).ok_or(Error::<T>::NotAllowed)?;
    let change = <Self as BondingController>::unbond_instant(&who, amount)?;
    // ... (apply fee and emit event)
    Ok(())
}
```

Insight:
- Implement a minimum fee threshold to ensure the instant unbonding fee cannot be set too low.
- Introduce a maximum instant unbonding limit to prevent large-scale unbonding in a short period.
- Consider adding a cooldown period between instant unbonding requests to mitigate potential abuse.

### 2. Potential Weakness: Share Calculation Precision Loss
Reference: [`pallets/rewards/src/lib.rs#add_share`](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L143-L167)

The share calculation in the Rewards module uses fixed-point arithmetic, which may lead to precision loss and potential share inflation over time. This could result in users receiving slightly more or fewer rewards than expected.

```rust
fn add_share(pool: &T::PoolId, who: &T::AccountId, add_amount: T::Share) {
    // ... (calculate reward_inflation)
    let reward_inflation = if initial_total_shares.is_zero() {
        Zero::zero()
    } else {
        U256::from(add_amount.to_owned().saturated_into::<u128>())
            .saturating_mul(total_reward.to_owned().saturated_into::<u128>().into())
            .checked_div(initial_total_shares.to_owned().saturated_into::<u128>().into())
            .unwrap_or_default()
            .as_u128()
            .saturated_into()
    };
    // ... (update shares and withdrawn_rewards)
}
```

Insight:
- Consider using integer-based arithmetic with appropriate scaling factors to minimize precision loss.
- Implement a mechanism to periodically reconcile and adjust share balances based on the actual reward distribution.

### 3. Potential Weakness: Centralization Risk in Parameter Updates
Reference: [`pallets/earning/src/lib.rs`, `pallets/incentives/src/lib.rs#UpdateOrigin`](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L280-L311)

The `UpdateOrigin` trait is used to control critical parameters such as the `UnbondingPeriod`, `InstantUnstakeFee`, and reward rates. If the `UpdateOrigin` is not properly decentralized, it could lead to centralized control and potential abuse by administrators.

```rust
#[pallet::weight(T::WeightInfo::update_incentive_rewards(updates.len() as u32))]
pub fn update_incentive_rewards(origin: OriginFor<T>, updates: Vec<(PoolId, Vec<(CurrencyId, Balance)>)>) -> DispatchResult {
    T::UpdateOrigin::ensure_origin(origin)?;
    // ... (update incentive reward amounts)
    Ok(())
}
```

Insight:
- Implement a decentralized governance mechanism, such as a DAO or multisig contract, to manage the `UpdateOrigin`.
- Establish strict guidelines and approval processes for parameter updates to prevent unilateral changes.
- Introduce time-locks and voting periods for critical parameter changes to allow community input and oversight.

### 4. Potential Weakness: Economic Attacks and Flash Loan Vulnerability
Code Reference: `pallets/rewards/src/lib.rs`, `pallets/incentives/src/lib.rs`

The current implementation of the Rewards and Incentives modules does not have explicit protections against economic attacks, such as flash loan exploits. An attacker could potentially manipulate their share balance using flash loans to claim more rewards than deserved.

Insight:
- Implement flash loan resistance mechanisms, such as time-weighted average share calculation or a cooldown period for share updates.
- Introduce a minimum staking period or lock-up mechanism to prevent instant share inflation.
- Monitor the system for suspicious activities and implement emergency stop functionality to halt reward distribution in case of detected exploits.

## 5. Potential Weakness: Lack of Comprehensive Documentation
Reference: Throughout the codebase

While the Acala codebase is well-structured and follows best practices, there is a lack of comprehensive documentation for critical functions and modules. This can make it challenging for developers and auditors to understand the system's intricacies and potential vulnerabilities.

Insight:
- Prioritize the development of detailed documentation for all critical functions, modules, and system components.
- Include clear explanations of the purpose, input/output parameters, and potential edge cases for each function.
- Provide documentation on the overall system architecture, interactions between modules, and recommended secure usage patterns.

### 6. Potential Weakness: Absence of Formal Verification
Reference: Throughout the codebase

The Acala codebase currently relies on traditional testing and auditing techniques to ensure its security and correctness. However, the absence of formal verification methods may leave room for undetected vulnerabilities and edge cases.

Insight:
- Consider incorporating formal verification techniques, such as model checking or theorem proving, to mathematically prove the correctness of critical system properties.
- Identify key invariants and security properties that should hold true throughout the system's execution and formally verify their adherence.
- Collaborate with academic institutions or specialized firms to conduct formal verification audits and validate the system's security guarantees.

### Time spent:
9 hours