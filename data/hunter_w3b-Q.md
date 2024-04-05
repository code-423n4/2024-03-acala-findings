### L-1 :- In the `unbond` function, the `unbond_at` block number is calculated using `frame_system::Pallet::<T>::block_number().saturating_add(T::UnbondingPeriod::get())`.

This calculation assumes that the unbonding period is always in the future. However, if the `UnbondingPeriod` is set to a small value or if the block number overflows, it could lead to an unbonding period in the past, allowing users to bypass the intended unbonding mechanism.

```rs
		pub fn unbond(origin: OriginFor<T>, #[pallet::compact] amount: Balance) -> DispatchResult {
			let who = ensure_signed(origin)?;

			let unbond_at = frame_system::Pallet::<T>::block_number().saturating_add(T::UnbondingPeriod::get());
			let change = <Self as BondingController>::unbond(&who, amount, unbond_at)?;

			if let Some(change) = change {
				T::OnUnbonded::happened(&(who.clone(), change.change));
				Self::deposit_event(Event::Unbonded {
					who,
					amount: change.change,
				});
			}

			Ok(())
		}
```

### L-2 :- The `apply_ledger` function is responsible for setting or removing locks based on the user's bonding ledger.

However, it does not handle potential errors that may arise during the locking or unlocking process. This could lead to situations where locks are not properly set or removed, potentially causing funds to be locked indefinitely or allowing unauthorized withdrawals.

```rs
	fn apply_ledger(who: &Self::AccountId, ledger: &BondingLedgerOf<T>) -> DispatchResult {
		if ledger.is_empty() {
			T::Currency::remove_lock(T::LockIdentifier::get(), who);
		} else {
			T::Currency::set_lock(T::LockIdentifier::get(), who, ledger.total(), WithdrawReasons::all());
		}
		Ok(())
	}

```

### L-3 :- The contract does not handle the case when `T::Currency::withdraw` in `unbond_instant` fails due to insufficient balance.

This can cause the contract to panic and stop functioning. A `try_runtime_exit` or a similar mechanism should be implemented to handle this case.

```rs

		#[pallet::weight(T::WeightInfo::unbond_instant())]
		pub fn unbond_instant(origin: OriginFor<T>, #[pallet::compact] amount: Balance) -> DispatchResult {
			let who = ensure_signed(origin)?;

			let fee_ratio = T::ParameterStore::get(InstantUnstakeFee).ok_or(Error::<T>::NotAllowed)?;

			let change = <Self as BondingController>::unbond_instant(&who, amount)?;

			if let Some(change) = change {
				let amount = change.change;
				let fee = fee_ratio.mul_ceil(amount);
				let final_amount = amount.saturating_sub(fee);

				let unbalance =
					T::Currency::withdraw(&who, fee, WithdrawReasons::TRANSFER, ExistenceRequirement::KeepAlive)?;
				T::OnUnstakeFee::on_unbalanced(unbalance);

				T::OnUnbonded::happened(&(who.clone(), final_amount));
				Self::deposit_event(Event::InstantUnbonded {
					who,
					amount: final_amount,
					fee,
				});
			}

```

### L-4 :- The `unbond_instant` function calculates the fee as `fee_ratio.mul_ceil(amount)`, but it does not check if the resulting fee is greater than the remaining balance.

This can cause the contract to panic or to withdraw more funds than intended.

```rs
			let fee_ratio = T::ParameterStore::get(InstantUnstakeFee).ok_or(Error::<T>::NotAllowed)?;

			let change = <Self as BondingController>::unbond_instant(&who, amount)?;

			if let Some(change) = change {
				let amount = change.change;
				let fee = fee_ratio.mul_ceil(amount);
				let final_amount = amount.saturating_sub(fee);

				let unbalance =
					T::Currency::withdraw(&who, fee, WithdrawReasons::TRANSFER, ExistenceRequirement::KeepAlive)?;
				T::OnUnstakeFee::on_unbalanced(unbalance);
```

### L-5 :- In the `claim_rewards` and `claim_reward` functions, the `PoolInfos::<T>::mutate_exists` method is used instead of `PoolInfos::<T>::mutate`.

This might lead to if the pool info does not exist, as the function will not update the pool info or the withdrawn rewards. If the intention is to only claim rewards for existing pools, it would be better to return an error or log a warning in such cases.

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

### L-6 :- In the `accumulate_reward` function, there's a potential overflow when adding `reward_increment` to `total_reward`.

If the addition operation overflows, it could lead to incorrect reward calculations or even contract failures.

```rs
	pub fn accumulate_reward(
		pool: &T::PoolId,
		reward_currency: T::CurrencyId,
		reward_increment: T::Balance,
	) -> DispatchResult {
		if reward_increment.is_zero() {
			return Ok(());
		}
		PoolInfos::<T>::mutate_exists(pool, |maybe_pool_info| -> DispatchResult {
			let pool_info = maybe_pool_info.as_mut().ok_or(Error::<T>::PoolDoesNotExist)?;

			pool_info
				.rewards
				.entry(reward_currency)
				.and_modify(|(total_reward, _)| {
					*total_reward = total_reward.saturating_add(reward_increment);
				})
				.or_insert((reward_increment, Zero::zero()));

			Ok(())
		})
	}
```

### L-7 :- The `reward_to_withdraw()` function calculates the reward to withdraw using integer division, which can lead to rounding errors and potentially inaccurate reward distributions, especially when dealing with small fractions of the total reward.

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
			.unwrap_or_default()
			.as_u128()
			.unique_saturated_into();
		total_reward_proportion
			.saturating_sub(withdrawn_reward)
			.min(total_reward.saturating_sub(total_withdrawn_reward))
	}
```

### L-8 :- In the `unbond_instant` function, the calculation of the fee might result in an integer overflow if the amount is very large and the fee ratio is significant.

```rs
		pub fn unbond_instant(origin: OriginFor<T>, #[pallet::compact] amount: Balance) -> DispatchResult {
			let who = ensure_signed(origin)?;

			let fee_ratio = T::ParameterStore::get(InstantUnstakeFee).ok_or(Error::<T>::NotAllowed)?;

			let change = <Self as BondingController>::unbond_instant(&who, amount)?;

			if let Some(change) = change {
				let amount = change.change;
				let fee = fee_ratio.mul_ceil(amount);
				let final_amount = amount.saturating_sub(fee);

				let unbalance =
					T::Currency::withdraw(&who, fee, WithdrawReasons::TRANSFER, ExistenceRequirement::KeepAlive)?;
				T::OnUnstakeFee::on_unbalanced(unbalance);

				T::OnUnbonded::happened(&(who.clone(), final_amount));
				Self::deposit_event(Event::InstantUnbonded {
					who,
					amount: final_amount,
					fee,
				});
			}

```

You can mitigate this by using the `checked_mul` function instead of the `mul_ceil` function.

```rs
let fee = fee_ratio.checked_mul(fee_ratio)?;
```

### L-9 :- The `payout_reward_and_reaccumulate_reward` function calls external functions (`<orml_rewards::Pallet<T>>::accumulate_reward` and `T::Currency::transfer`) without following the checks-effects-interactions pattern. This may expose the contract to reentrancy attacks.

```rs
	fn payout_reward_and_reaccumulate_reward(
		pool_id: PoolId,
		who: &T::AccountId,
		reward_currency_id: CurrencyId,
		payout_amount: Balance,
		reaccumulate_amount: Balance,
	) -> DispatchResult {
		if !reaccumulate_amount.is_zero() {
			<orml_rewards::Pallet<T>>::accumulate_reward(&pool_id, reward_currency_id, reaccumulate_amount)?;
		}
		T::Currency::transfer(reward_currency_id, &Self::account_id(), who, payout_amount)?;
		Ok(())
	}
}

```

### L-10 :- The `transfer_share_and_rewards` function could be made more efficient by avoiding the need to iterate over the `rewards` map twice (once in the `SharesAndWithdrawnRewards::<T>::mutate_exists` call, and again in the `SharesAndWithdrawnRewards::<T>::mutate` call).

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

### NC-1 - The event naming in the contract is inconsistent. Some events use past tense (e.g., `Bonded`, `Unbonded`, `Rebonded`), while others use present tense (e.g., `InstantUnbonded`, `Withdrawn`). Standardizing the event naming convention would improve readability and maintainability.
