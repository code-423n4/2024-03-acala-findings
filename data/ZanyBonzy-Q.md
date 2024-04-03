
# 1. Centralization risks

Lines of code* 

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L284
https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L324
https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L360

### Impact
The UpdateOrigin has the power to update reward deduction currency, reward deduction rates and incentive rewards. Actions taking by the party all have serious effects on the protocol. 
### Recommended Mitigation Steps
Consider implementing a multisig account and documenting the potential risks.

***

# 2. Add sanity checks to prevent user griefing

Lines of code* 
https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L320

### Impact
The `update_claim_reward_deduction_rates` allows the UpdateOrigin to set the rates that is removed from a user's rewards as a sort of penalty for lack of loyalty. This can be set to an unusually high value which can grief users.

### Recommended Mitigation Steps
Consider introducing a maximum value that the rate can be set to. 
***


# 3. Users have no incentive to wait for unbonding period

Links to affected code *
https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/earning/src/lib.rs#L44-L48
### Impact
From the protocol documentation, comments and readme, it is discussed that a fee is to be charged for unstaking, however the unbond fee percent is hardcoded and set to 0.
```
define_parameters! {
	pub Parameters = {
		InstantUnstakeFee: Permill = 0,
	}
}
```
Also, looking through the codebase, there's a lack of a `set_fee` function or its equivalent, causing that upon deployment, the instant unbond fee cannot be updated. 

As a consequence, users can simply bypass the unbonding period by instantly unbonding without any consequneces.

### Recommended Mitigation Steps

Introduce the fee parameter

# 4. Unbonding periods can be bypassed if DEX's `lp_currency_id` is set to `NativeCurrencyId` as users can easily call wotdraw dex shares instead

Lines of code* 
https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L248

### Impact

The `withdraw_dex_share` is called by users to remove their shares from the pool.

```rust
	fn do_withdraw_dex_share(who: &T::AccountId, lp_currency_id: CurrencyId, amount: Balance) -> DispatchResult {
		ensure!(lp_currency_id.is_dex_share_currency_id(), Error::<T>::InvalidCurrencyId);
		ensure!(
			<orml_rewards::Pallet<T>>::shares_and_withdrawn_rewards(&PoolId::Dex(lp_currency_id), &who).0 >= amount,
			Error::<T>::NotEnough,
		);

		T::Currency::transfer(lp_currency_id, &Self::account_id(), who, amount)?;
		<orml_rewards::Pallet<T>>::remove_share(who, &PoolId::Dex(lp_currency_id), amount.unique_saturated_into());

		Self::deposit_event(Event::WithdrawDexShare {
			who: who.clone(),
			dex_share_type: lp_currency_id,
			withdraw: amount,
		});
		Ok(())
	}
}
```

Upon bonding, shares are added to the NativeCurrency pool. 
```rust
pub struct OnEarningBonded<T>(sp_std::marker::PhantomData<T>);
impl<T: Config> Happened<(T::AccountId, Balance)> for OnEarningBonded<T> {
	fn happened((who, amount): &(T::AccountId, Balance)) {
		<orml_rewards::Pallet<T>>::add_share(who, &PoolId::Earning(T::NativeCurrencyId::get()), *amount);
	}
}
```

What this causes is that if then liquidity pool currency, is set to the native currency, users, rather than unbond and wait for the unbonding period will just withdraw the shares directly by calling the `withdraw_dex_shares` function.
***
