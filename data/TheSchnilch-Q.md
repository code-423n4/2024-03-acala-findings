# Lines of code

https://github.com/code-423n4/2024-03-acala/blame/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L158-L167
https://github.com/code-423n4/2024-03-acala/blame/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L341-L343C63

# Low: In some places in the code of ORML-Rewards, rounding down is mistakenly applied

## Description

The orml-rewards module is designed to calculate how many rewards a user receives. The module tracks how many shares the user has deposited into the protocol and calculates how much the user will receive when they claim their rewards. It then records how much the user has claimed to prevent double claiming. Additionally, a user can transfer their shares to another user using the `transfer_share_and_rewards` function within this module. In some of these functions, rounding to zero is erroneously performed.

```rust
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

Reward inflation is calculated when new shares are added so that no past rewards can be claimed for these shares. The problem is that in this calculation, zero can result if `total_reward * add_amount` is less than `initial_total_shares`. In that case, `reward_inflation` becomes 0, yet the amount is still added. This results in a small number of shares being added without increasing `total_reward`, leading to the possibility of claiming rewards for these added shares. The rewards that were claimed would actually have belonged to another user who can no longer get them. For this attack to be effective, an attacker would need to inject a large number of shares into the system to be able to steal many rewards. This is only feasible if `initial_total_shares` is very large and `total_reward` is very small, or if the attacker makes many small deposits. And these are scenarios that are very unlikely, but if they occur, the attacker can steal rewards.

This bug could also occur in `transfer_share_and_rewards`, as `move_balance` is calculated similarly here. This has roughly the same effects as with `reward_inflation`, since `move_balance` is there so that the new owner of the shares cannot simply claim rewards for them again. Even if `transfer_share_and_rewards` is not used by Acala, the pallet orml_rewards is also used by other projects that may have implemented this function.

## Recommendation

It must be ensured that with `add_share` the `reward_inflation` is greater than 0 if `total_rewards` is not 0. Same with `move_balance`, it has to be reverted if `move_balance` is 0 and withdrawn reward is not.