# QA Report

## Table of Contents

| Issue ID                                                                                                    | Description                                                                                  |
| ----------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| [QA-01](#qa-01-admin-is-a-single-point-of-failure)                                                          | Admin is a single point of failure                                                           |
| [QA-02](#qa-02-consider-adding-better-documentation)                                                        | Consider adding better documentation                                                         |
| [QA-03](#qa-03-setters-should-always-have-equality-checkers)                                                | Setters should always have equality checkers                                                 |
| [QA-04](<#qa-04-onupdateloanhappened()-should-check-for-when-adjustment-==-0>)                              | `OnUpdateLoan::happened()` should check for when `adjustment == 0`                           |
| [QA-05](#qa-05-protocol-does-not-apply-deadlines-when-dealing-with-critical-operations)                     | Protocol does not apply deadlines when dealing with critical operations                      |
| [QA-06](#qa-06-fix-typos)                                                                                   | Fix typos                                                                                    |
| [QA-07](#qa-07-consider-using-bst-instead)                                                                  | Consider using BST instead                                                                   |
| [QA-08](#qa-08-protocol-has-a-minbond-value-but-doesnt-seem-to-check-this-whenever-a-user-attempts-to-bond) | Protocol has a minbond value but doesn't seem to check this whenever a user attempts to bond |
| [QA-09](#qa-09-consider-not-switching-off-important-clippy-protection-methods)                              | Consider not switching off important clippy protection methods                               |
| [QA-10](#qa-10-protocol-does-not-apply-slippages-when-depositing-or-withdrawing-the-dex-shares)             | Protocol does not apply slippages when depositing or withdrawing the dex shares              |
| [QA-11](#qa-11-missing-lockIdentifier-definition-in-scope)                                                  | QA-11 Missing `LockIdentifier` definition in scope                                           |

## QA-01 Admin is a single point of failure

### Proof of Concept

Multiple instances in code where the admin logic could lead to a brick in normal functionality.

For example take a look at https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L83-L85

```rust
		/// The origin which may update incentive related params
		type UpdateOrigin: EnsureOrigin<Self::RuntimeOrigin>;

```

This is an instance where a somewhat `admin` logic has been aplied, note that this is passed in the config's constant paller, now using this search command: [https://github.com/search?q=repo%3Acode-423n4%2F2024-03-acala%20UpdateOrigin&type=code](https://github.com/search?q=repo%3Acode-423n4%2F2024-03-acala%20UpdateOrigin&type=code), we can see that there are three function calls where the expected caller is only accepted to be the `update incentive related params`.

### Impact

Inaccess to update incentive related params if anything was to happen to the `UpdateOrigin`.

### Recommended Mitigation Steps

Consider implementing a backdoor that the admins could use to change the incentive related params if anything were to happen to `UpdateOrigin`.

## QA-02 Consider adding better documentation

### Proof of Concept

This is very rampant in code, for example take a look at https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L143-L190

```rust
	pub fn add_share(who: &T::AccountId, pool: &T::PoolId, add_amount: T::Share) {
		if add_amount.is_zero() {
			return;
		}

		PoolInfos::<T>::mutate(pool, |pool_info| {
			let initial_total_shares = pool_info.total_shares;
			pool_info.total_shares = pool_info.total_shares.saturating_add(add_amount);

			let mut withdrawn_inflation = Vec::<(T::CurrencyId, T::Balance)>::new();

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

			SharesAndWithdrawnRewards::<T>::mutate(pool, who, |(share, withdrawn_rewards)| {
				*share = share.saturating_add(add_amount);
				// update withdrawn inflation for each reward currency
				withdrawn_inflation
					.into_iter()
					.for_each(|(reward_currency, reward_inflation)| {
						withdrawn_rewards
							.entry(reward_currency)
							.and_modify(|withdrawn_reward| {
								*withdrawn_reward = withdrawn_reward.saturating_add(reward_inflation);
							})
							.or_insert(reward_inflation);
					});
			});
		});
	}

```

Evidently, this function is somewhat complex, i.e the type of mathematical operations attached with it, but that isn't the main case here, there are completely no documentations as to why some steps are taken, this heavily stalls the auditing proceess as we and other security researches don't know what the intended behaviour is and can't really sit to break it.

### Impact

Hard time understnading code for users/devs/auditors lead to bad integration

### Recommended Mitigation Steps

Consider adding better documetations to even if not all functions then atleasr the core/complex ones

## QA-03 Setters should always have equality checkers

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L252-L262

```rust

	pub fn set_share(who: &T::AccountId, pool: &T::PoolId, new_share: T::Share) {
		let (share, _) = Self::shares_and_withdrawn_rewards(pool, who);

		if new_share > share {
			Self::add_share(who, pool, new_share.saturating_sub(share));
		} else {
			Self::remove_share(who, pool, share.saturating_sub(new_share));
		}
	}

```

Ths function sets a new share value for a pool, now there are no checks that ` new_share != share` and as such whenever ` new_share == share` the protocol unnecessarily(wrongly attempts to remove via `share.saturating_sub(new_share`)

- Another instance can be seen here, where protocol in all intances of updating the amounts don't check if the provided amount is not already the stored amount and as such unnecessarily updates `v`, i.e https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L280-L312

```rust
		pub fn update_incentive_rewards(
			origin: OriginFor<T>,
			updates: Vec<(PoolId, Vec<(CurrencyId, Balance)>)>,
		) -> DispatchResult {
			T::UpdateOrigin::ensure_origin(origin)?;
			for (pool_id, update_list) in updates {
				if let PoolId::Dex(currency_id) = pool_id {
					ensure!(currency_id.is_dex_share_currency_id(), Error::<T>::InvalidPoolId);
				}

				for (currency_id, amount) in update_list {
					IncentiveRewardAmounts::<T>::mutate_exists(pool_id, currency_id, |maybe_amount| {
						let mut v = maybe_amount.unwrap_or_default();
						if amount != v {
							v = amount;
							Self::deposit_event(Event::IncentiveRewardAmountUpdated {
								pool: pool_id,
								reward_currency_id: currency_id,
								reward_amount_per_period: amount,
							});
						}

						if v.is_zero() {
							*maybe_amount = None;
						} else {
							*maybe_amount = Some(v);
						}
					});
				}
			}
			Ok(())
		}

```

- The same idea can also be applied to the https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L320 `update_claim_reward_deduction_rates()` function.

### Impact

Unnecessary code execution, flawed implementation.

### Recommended Mitigation Steps

As a rule of thumb all setters should always have equality checkers as passing an equal to the already stored value hints a mistake and maybe this attempt to add/remove was meant for a different pool.

## QA-04 `OnUpdateLoan::happened()` should check for when `adjustment == 0`

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L578-L587

```rust
	fn happened(info: &(T::AccountId, CurrencyId, Amount, Balance)) {
		let (who, currency_id, adjustment, _previous_amount) = info;
		let adjustment_abs = TryInto::<Balance>::try_into(adjustment.saturating_abs()).unwrap_or_default();

		if adjustment.is_positive() {
			<orml_rewards::Pallet<T>>::add_share(who, &PoolId::Loans(*currency_id), adjustment_abs);
		} else {
			<orml_rewards::Pallet<T>>::remove_share(who, &PoolId::Loans(*currency_id), adjustment_abs);
		};
	}
```

This function has been used to settle adjustments in protocol, main issue about this report is the fact that adding/removing the share is dependent on if the adjustment is positive or not, this is a somewhat flawed implementation as the `is_positive()` getter would actually return `false` for when `adjustment == 0` and as such lead to an unnecessary attempt of removing `0` shares.

### Impact

Not enough input validation is applied, not the best code structure.

### Recommended Mitigation Steps

Consider checking in the function to ensure that for instances where `adjustment == 0` none of neither `add_shares()` or `remove_shares()` is called.

## QA-05 Protocol does not apply deadlines when dealing with critical operations

### Proof of Concept

Consider https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L511-L543

```rust
	fn do_deposit_dex_share(who: &T::AccountId, lp_currency_id: CurrencyId, amount: Balance) -> DispatchResult {
		ensure!(lp_currency_id.is_dex_share_currency_id(), Error::<T>::InvalidCurrencyId);

		T::Currency::transfer(lp_currency_id, who, &Self::account_id(), amount)?;
		<orml_rewards::Pallet<T>>::add_share(who, &PoolId::Dex(lp_currency_id), amount.unique_saturated_into());

		Self::deposit_event(Event::DepositDexShare {
			who: who.clone(),
			dex_share_type: lp_currency_id,
			deposit: amount,
		});
		Ok(())
	}

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

Both functions are used to either deposit/withdraw dex shares from the lps by passing in the currency which is in short just like a swap, case is that this function does not include any deadline protection whatsoever and the rransaction could be left hanging for a long time and end up being executed in unfavorable situations.

### Impact

This is a pretty popular bug case, where no deadlines have been applied and could lead to users losing funds, sometimes in `$USD` equivalent since their transactions might execute under less favorable conditions than intended.

### Recommended Mitigation Steps

Consider requesting a deadline to which a user would like their transaction to be hanging for, and ensure that if the deadline has passed the function does not get executed.

> Submitting as QA, leaving to judge to upgrade to M if they see fit

## QA-06 Fix typos

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/earning/src/lib.rs#L153-L158

```rust
		/// Start unbonding tokens up to `amount`.
		/// If bonded amount is less than `amount`, then all the remaining bonded tokens will start
		/// unbonding. Token will finish unbonding after `UnbondingPeriod` blocks.
		#[pallet::call_index(1)]
		#[pallet::weight(T::WeightInfo::unbond())]
		pub fn unbond(origin: OriginFor<T>, #[pallet::compact] amount: Balance) -> DispatchResult {
            //..sip
```

This sentence: "Token will finish unbonding after `UnbondingPeriod` blocks." should instead be:
"**Tokens** will finish unbonding after `UnbondingPeriod` blocks."

- Another instancecan be seen here https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L258-L269

```rust
		/// Claim all avalible multi currencies rewards for specific PoolId.
		///
		/// The dispatch origin of this call must be `Signed` by the transactor.
		///
		/// - `pool_id`: pool type
		#[pallet::call_index(2)]
		#[pallet::weight(<T as Config>::WeightInfo::claim_rewards())]
		pub fn claim_rewards(origin: OriginFor<T>, pool_id: PoolId) -> DispatchResult {
			let who = ensure_signed(origin)?;

			Self::do_claim_rewards(who, pool_id)
		}
```

The first line should be "Claim all **avalaible** multi currencies rewards for specific PoolId." instead.

### Impact

### Recommended Mitigation Steps

## QA-07 Consider using BST instead

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L21-L26

```rust
pub struct PoolInfo<Share: HasCompact, Balance: HasCompact, CurrencyId: Ord> {
	/// Total shares amount
	pub total_shares: Share,
	/// Reward infos <reward_currency, (total_reward, total_withdrawn_reward)>
	pub rewards: BTreeMap<CurrencyId, (Balance, Balance)>,
}
```

Evidentlym the reward infos are stores in a B-Tree map which is in short an ordered map based on a `B-Tree`.

Now `B-Trees` represent a fundamental compromise between cache-efficiency and actually minimizing the amount of work performed in a search. In theory, a binary search tree (BST) is the optimal choice for a sorted map, as a perfectly balanced BST performs the theoretical minimum amount of comparisons necessary to find an element (log2n).

### Impact

Non-optimized method of finding elements

### Recommended Mitigation Steps

Consider using binary search trees instead.

## QA-08 Protocol has a minbond value but doesn't seem to check this whenever a user attempts to bond

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/earning/src/lib.rs#L138-L151

```rust
		pub fn bond(origin: OriginFor<T>, #[pallet::compact] amount: Balance) -> DispatchResult {
			let who = ensure_signed(origin)?;

			let change = <Self as BondingController>::bond(&who, amount)?;

			if let Some(change) = change {
				T::OnBonded::happened(&(who.clone(), change.change));
				Self::deposit_event(Event::Bonded {
					who,
					amount: change.change,
				});
			}
			Ok(())
		}
```

We can see that this is the function to use whenver a user wants to bond, navigating here in the contract though https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/earning/src/lib.rs#L252

```rust
	type MinBond = T::MinBond;
```

Protocol is to implement a minbound value, but while attempting the main bonding this check is not applied allowing any body to bond with any minute amount and make the system have a massive backlog of dust bonds

### Impact

Contract's core logic of having a `minbond` in place is flawed as it doesn't apply this on attempts to bond and would lead to the protocol to accept any dust amount as bonds

### Recommended Mitigation Steps

Check that the amount that got bonded is more than the `minbond` value in the config.

## QA-09 Consider not switching off important clippy protection methods

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L1-L3

```rust
#![cfg_attr(not(feature = "std"), no_std)]
#![allow(clippy::unused_unit)]
#![allow(clippy::too_many_arguments)]
```

Protocol disallows clippy to run the auto tool to warn about unused units or too many arguments, now where as too many arguments could be understandable since it's for the `claim_one()` function: https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L363-L385 a justification can't be made for not wanting to use `clippy::unused_unit`

### Impact

Bad code structure

### Recommended Mitigation Steps

In most cases impleemntations that have unused units are mostly flawed and should be sorted out.

## QA-10 Protocol does not apply slippages when depositing or withdrawing the dex shares

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L511-L543

```rust
	fn do_deposit_dex_share(who: &T::AccountId, lp_currency_id: CurrencyId, amount: Balance) -> DispatchResult {
		ensure!(lp_currency_id.is_dex_share_currency_id(), Error::<T>::InvalidCurrencyId);

		T::Currency::transfer(lp_currency_id, who, &Self::account_id(), amount)?;
		<orml_rewards::Pallet<T>>::add_share(who, &PoolId::Dex(lp_currency_id), amount.unique_saturated_into());

		Self::deposit_event(Event::DepositDexShare {
			who: who.clone(),
			dex_share_type: lp_currency_id,
			deposit: amount,
		});
		Ok(())
	}

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

Both functions are implemented as swaps, that's to say they are used to either deposit/withdraw shares from the lps by passing in the currency, case is that these function do not include any slippage protection whatsoever, leaving the end user not bein able to specify the minimum amount of shares they would like to receive and as such they could get gamed, i.e instances where the amounts to be added or removed could get diluted in the rewards module.

### Impact

This is a pretty popular bug case, where no slippage protection could lead to users loss of funds if an attacker manipulates the lps.

> Submitting as QA, leaving to judge to upgrade to M if they see fit

### Recommended Mitigation Steps

Consider requesting a minmimum amount of deposit or shares to receive that a user can specify while calling the functions.

## QA-11 Missing `LockIdentifier` definition in scope

### Proof of Concept

The implementation of `LockableCurrency` in the protocol requires a `LockIdentifier` to be declared, which is a `[u8; 8]` type, this can be seen from https://docs.substrate.io/reference/how-to-guides/pallet-design/implement-lockable-currency/ This identifier must be eight characters long. However, there is no definition of the contracts in-scope, which raises concerns about the correct implementation of the lockable currency feature.

### Impact

Without a properly defined `LockIdentifier`, the protocol might not correctly implement the lockable currency feature. This could lead to issues with locking and unlocking funds, affecting the functionality of staking and voting mechanisms.

### Recommended Mitigation Steps

To address this issue, ensure that a `LockIdentifier` is declared in the scope of the pallet. This identifier should be an eight-character long constant of type `[u8; 8]`.
