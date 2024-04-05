A. Possible Divide by zero error
[lib.rs#L391-L408](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L391-L408)
The `reward_to_withdraw` function within the contract performs arithmetic operations without adequate bounds checking, potentially leading to inconsistency. Specifically, the division operation (`checked_div`) within the function lacks proper handling for potential division by zero and other edge cases. Here's the relevant code snippet:
```rs
fn reward_to_withdraw(
    share: T::Share,
    total_reward: T::Balance,
    total_shares: U256,
    withdrawn_reward: T::Balance,
    total_withdrawn_reward: T::Balance,
) -> T::Balance {
    let total_reward_proportion: T::Balance = U256::from(share.saturated_into::<u128>())
        .saturating_mul(U256::from(total_reward.saturated_into::<u128>()))
        .checked_div(total_shares)
        .unwrap_or_default() // Potential division by zero is not handled here
        .as_u128()
        .unique_saturated_into();
    total_reward_proportion
        .saturating_sub(withdrawn_reward)
        .min(total_reward.saturating_sub(total_withdrawn_reward))
}
```
This function calculates the proportion of rewards to withdraw based on various inputs such as share amount, total rewards, and total shares. However, it lacks proper checks for potential division by zero or other invalid inputs, which could lead to unexpected behavior or vulnerabilities.

Impact:
The impact of this vulnerability could be significant, potentially leading to incorrect reward calculations or even contract failures. If inputs are not properly validated, the function may produce incorrect results, leading to loss of funds or exploitation by attackers.

Mitigation:
Thorough input validation and bounds checking should be implemented within the `reward_to_withdraw` function. Specifically, the division operation should be wrapped in appropriate error handling to handle potential division by zero and other edge cases gracefully. 