
## [L1] In `deposit_dex_share`, the incorrect `lp_currency_id` is passed, resulting in a silent failure

The `do_deposit_dex_share` function only checks `lp_currency_id` `is_dex_share_currency_id`

If the `lp_currency_id` does not exist in the pool,ths `<orml_rewards::Pallet<T>>::add_share` function does nothing,
however, the user's token has been sent to the contract, which will cause the loss of the user's funds.

```rust
	fn do_deposit_dex_share(who: &T::AccountId, lp_currency_id: CurrencyId, amount: Balance) -> DispatchResult {
		ensure!(lp_currency_id.is_dex_share_currency_id(), Error::<T>::InvalidCurrencyId);

@>		T::Currency::transfer(lp_currency_id, who, &Self::account_id(), amount)?;
@>		<orml_rewards::Pallet<T>>::add_share(who, &PoolId::Dex(lp_currency_id), amount.unique_saturated_into());

		Self::deposit_event(Event::DepositDexShare {
			who: who.clone(),
			dex_share_type: lp_currency_id,
			deposit: amount,
		});
		Ok(())
	}

    pub fn add_share(who: &T::AccountId, pool: &T::PoolId, add_amount: T::Share) {
		if add_amount.is_zero() {
			return;
		}

		PoolInfos::<T>::mutate(pool, |pool_info| {
			.....
            //If the PoolId does not exist in the pool, nothing to do.
		});
	}
```

## [l2] Duplicate reward_currencies will cause `get_pending_rewards` to return incorrect `reward_balances`.

```rust
	fn get_pending_rewards(pool_id: PoolId, who: T::AccountId, reward_currencies: Vec<CurrencyId>) -> Vec<Balance> {
		let rewards_map = PendingMultiRewards::<T>::get(pool_id, who);
		let mut reward_balances = Vec::new();
		for reward_currency in reward_currencies {
			let reward_amount = rewards_map.get(&reward_currency).copied().unwrap_or_default();
			reward_balances.push(reward_amount);
		}
		reward_balances
	}
```

`get_pending_rewards` does not remove duplicates for the passed array `reward_currencies`,
If, this parameter can be entered by the user , a malicious user enters a duplicate `CurrencyId`, the `get_pending_rewards` function returns duplicate `reward_balances`.