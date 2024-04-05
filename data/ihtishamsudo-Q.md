## L-01 Incorrect calculation of Rewards during share transfer process

In the [transfer_share_and_rewards](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L326) function, the [code segment](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L348-L354) responsible for updating the rewards map does not handle the scenario where there could be existing rewards for other currencies. 

If there are already accumulated rewards for other currencies in the [increased_rewards](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L348) map, inserting a new entry for [reward_currency](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L349) with [move_balance](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L353) would overwrite those existing rewards. This can lead to incorrect reward balances and distort the accuracy of reward distribution.

Snippet : 
```rust
increased_rewards
    .entry(*reward_currency)
    .and_modify(|increased_reward| {
        *increased_reward = increased_reward.saturating_add(move_balance);
    })
    .or_insert(move_balance);
```

### Recommendation

ensure that when inserting a new entry for reward_currency, you consider the possibility of existing entries for other currencies and properly aggregate the rewards. A potential fix involves using the `or_insert_with` closure to insert a new entry without overwriting existing entries for other currencies:

```diff
increased_rewards
    .entry(*reward_currency)
    .and_modify(|increased_reward| {
        *increased_reward = increased_reward.saturating_add(move_balance);
    })
-    .or_insert(move_balance);
+    .or_insert_with(|| move_balance);
```

## L-02 `claim_one` function doesn't check if the total_shares is zero or not

The [claim_one](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L364) function is responsible for handling the claiming of rewards for a single currency. It calculates the amount of rewards to withdraw for a given share based on the total reward, total shares, and previously withdrawn rewards.

Problem arise because it explicitly miss the check for if the [total_share](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L369) is zero or not. If the total_share is zero, error will occur and the function will panic.

Snippet : 

```rust
fn claim_one(
		withdrawn_rewards: &mut BTreeMap<T::CurrencyId, T::Balance>,
		reward_currency: T::CurrencyId,
		share: T::Share,
		total_reward: T::Balance,
		total_shares: U256,
		total_withdrawn_reward: &mut T::Balance,
		who: &T::AccountId,
		pool: &T::PoolId,
	) {	//@audit if !total_shares.is_zero() missing 
		let withdrawn_reward = withdrawn_rewards.get(&reward_currency).copied().unwrap_or_default();
		let reward_to_withdraw = Self::reward_to_withdraw(
			share,
			total_reward,
			total_shares,
			withdrawn_reward,
			total_withdrawn_reward.to_owned(),
		);
		if !reward_to_withdraw.is_zero() {
			*total_withdrawn_reward = total_withdrawn_reward.saturating_add(reward_to_withdraw);
			withdrawn_rewards.insert(reward_currency, withdrawn_reward.saturating_add(reward_to_withdraw));

			// pay reward to `who`
			T::Handler::payout(who, pool, reward_currency, reward_to_withdraw);
		}
	}
```

### Recommendation

Ensure that the `claim_one` function checks if the `total_shares` is zero or not before proceeding with the reward calculation. If the `total_shares` is zero, the function should return an error or handle the scenario.


```diff
fn claim_one(
    ...
) {
    let withdrawn_reward = withdrawn_rewards.get(&reward_currency).copied().unwrap_or_default();
    
+    // Ensure total_shares is not zero to avoid panic error
+    if !total_shares.is_zero() {
        let reward_to_withdraw = Self::reward_to_withdraw(
            share,
            total_reward,
            total_shares,
            withdrawn_reward,
            total_withdrawn_reward.to_owned(),
        );
        if !reward_to_withdraw.is_zero() {
            *total_withdrawn_reward = total_withdrawn_reward.saturating_add(reward_to_withdraw);
            withdrawn_rewards.insert(reward_currency, withdrawn_reward.saturating_add(reward_to_withdraw));

            // pay reward to `who`
            T::Handler::payout(who, pool, reward_currency, reward_to_withdraw);
        }
    }
}
```

## L-03 `claim_reward` function doesn't check if specific reward currency exist in pool 

[claim_reward](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L293) function appears to be designed to allow an account to claim rewards from a specific pool. 

However, there is no check to ensure that the specified reward currency exists in the pool before attempting to claim rewards for it. This can lead to a scenario where an account tries to claim rewards for a currency that does not exist in the pool, which can result in an error or unexpected behavior.

## Recommendation 

it's recommended to add a check to verify that the specified reward currency exists in the pool before proceeding with the reward claim process. If the currency does not exist, the function should return early or handle the situation appropriately.

```diff
pub fn claim_reward(who: &T::AccountId, pool: &T::PoolId, reward_currency: T::CurrencyId) {
    SharesAndWithdrawnRewards::<T>::mutate_exists(pool, who, |maybe_share_withdrawn| {
        if let Some((share, withdrawn_rewards)) = maybe_share_withdrawn {
            if share.is_zero() {
                return;
            }

            PoolInfos::<T>::mutate(pool, |pool_info| {
                if let Some((total_reward, total_withdrawn_reward)) = pool_info.rewards.get_mut(&reward_currency) {
                    // Check if the reward currency exists in the pool
                    if let Some(total_reward) = total_reward {
                        let total_shares = U256::from(pool_info.total_shares.to_owned().saturated_into::<u128>());
                        Self::claim_one(
                            withdrawn_rewards,
                            reward_currency,
                            share.to_owned(),
                            total_reward.to_owned(),
                            total_shares,
                            total_withdrawn_reward,
                            who,
                            pool,
                        );
+                    } else {
+                        // For demonstration, i'll log an error message
+                        log::error!("Reward currency {:?} does not exist in pool {:?}", reward_currency, pool);
                    }
                }
            });
        }
    });
}
```

## L-04 Incomplete Share Update in `set_share` function

[set_share]() function is responsible for setting the share of a specified account in a pool to a new value. 

When the function compares the new share with the current share, If the new share is greater than the current share, it calls [Self::add_share](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L257) to add the difference between the new share and the current share.

If the new share is less than or equal to the current share, it calls [Self::remove_share](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L259) to remove the difference between the current share and the new share.

However, if the new share is `equal` to the current share, neither [add_share]((https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L257)) nor [remove_share](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L259)) is called. As a result, the function doesn't update the share at all in this scenario. This could be problematic if the intention is to set the share to a specific value regardless of its current value.

## Recommendation 

It is necessary to ensure that the share is always updated to the new value, even if it's equal to the current value.

```diff
pub fn set_share(who: &T::AccountId, pool: &T::PoolId, new_share: T::Share) {
		let (share, _) = Self::shares_and_withdrawn_rewards(pool, who);

		if new_share > share {
			Self::add_share(who, pool, new_share.saturating_sub(share));
		} else {
			Self::remove_share(who, pool, share.saturating_sub(new_share));
		}
+       // Update the share to the new value even if it's equal to the current value        
+       // This ensures that the share is always set to the specified value
+       <SharesAndWithdrawnRewards<T>>::insert(pool, who, (new_share, BTreeMap::new()));        
	}
```

## L-05 Missing Check for Empty Rewards Map in `add_share` Function
 
 The [add_share](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L143) function is responsible for adding shares to a specified account in a pool. However, it lacks a check for the case where the rewards map associated with the pool is empty. 

 Without the check for an empty rewards map, the function may attempt to iterate over non-existent rewards, leading to runtime errors.


## Recommendation

it's crucial to implement a check for an empty rewards map before iterating over the rewards. If the rewards map is empty, the function should handle this case appropriately

```diff
pub fn add_share(who: &T::AccountId, pool: &T::PoolId, add_amount: T::Share) {
		if add_amount.is_zero() {
			return;
		}

		PoolInfos::<T>::mutate(pool, |pool_info| {
			let initial_total_shares = pool_info.total_shares;
			pool_info.total_shares = pool_info.total_shares.saturating_add(add_amount);

			let mut withdrawn_inflation = Vec::<(T::CurrencyId, T::Balance)>::new();
+			 //@audit pool info rewards should be checked if it exists or not
+            // Check if rewards map is empty
+            if pool_info.rewards.is_empty() {
+            // Handle the case when rewards map is empty
+            return;
+            }
			pool_info
				.rewards
				.iter_mut()
				.for_each(|(reward_currency, (total_reward, total_withdrawn_reward))| {
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
					*total_reward = total_reward.saturating_add(reward_inflation);
					*total_withdrawn_reward = total_withdrawn_reward.saturating_add(reward_inflation);

					withdrawn_inflation.push((*reward_currency, reward_inflation));
				});

			CODE_SNIPPET....
	}
```

## NC-01 Edge Cases should retunr proper error messages instead of early success

The [accumulate_reward](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L120) function is accumulating rewards for a specific pool and currency. 
t incorrectly handles the case where the reward increment is zero. Instead of returning an error to indicate that no reward increment was provided, it [returns Ok(())](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L126), indicating success.

## Recommendation

[accumulate_reward]((https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L120)) function should returns a proper error when the reward increment is zero.

```diff
pub fn accumulate_reward(
    pool: &T::PoolId,
    reward_currency: T::CurrencyId,
    reward_increment: T::Balance,
) -> DispatchResult {
    if reward_increment.is_zero() {
-       return Ok(());        
+       return Err(Error::<T>::ZeroRewardIncrement.into());  //demonstration for specifying the error
    }
    CODE_SNIPPET...
}
```

Implement the same for all other instance where edge cases are returning early success instead of proper error messages.