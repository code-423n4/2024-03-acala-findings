| Serial No. | Topic                                                                                     |
| ---------- | ----------------------------------------------------------------------------------------- |
| 01         | [Overview](#1-overview)                                                                   |
| 02         | [Architecture Overview](#2-architecture-overview)                                         |
| 03         | [Approach Taken in Evaluating Acala](#3-approach-taken-in-evaluating-acala)               |
| 04         | [Acala Modules Analysis](#4-acala-modules-analysis)                                       |
| 05         | [Call-trace Diagrams](#5-call-trace-diagrams)                                             |
| 06         | [Codebase Quality](#6-codebase-quality)                                                   |
| 07         | [Systematic Risks and Centralization](#7-systematic-risks-and-centralization)                                |

## **1. Overview**

"Building the liquidity layer of Web3 finance". Acala, at it's core is a crosschain DeFi network liquidity hub built for Polkadot. It is an Ethereum compatible, protocol that consists of a decentralized finance network and liquidity hub for Polkadot. Acala’s whole infrastructure also includes a stablecoin network and liquidity staking platform.

The codebase presented for this interaction is the Acala staking and reward system. Its review of staking, reward accumulation and distribution as well as bonding/locking of the staking factor are in-scope for this analysis review and will be delved into deeper.

For this Analysis report, we focused mainly on the staking module.

## **2. Architecture Overview**

During the course of our engagement for conducting this analysis of the staking, bonding and earning modules, we strived to understand the architecture/mechanics of the codebase better first. A summarization of each architecture is as follows:

1. Pools: These facilitate staking in the Acala ecosystem. Pools attract incentives for the stakers in such pool at every given period. These incentives can be claimed at any time - with or without a penalty applied.

2. Rewards: These get paid out to the users and can be referred to as incentives as we mentioned above. But without pools, there are no incentives. What's the point of rewards for stakers when there are zero stakers? None. Zero point. Rewarding is set aside for legitimate stakers. There are no rewards for stakers with 0 factors in pools. Each stake accrues rewards proportional to their staking factor (weight of stake in the pool for the particular user).

3. Bonding: Users typically bond their stakes for a period of time. When that period has elapsed, they can unbond their stake and then withdraw it from pools. This is where the penalty we mentioned in Pools earlier comes in. If a user bonds for 30 days and decides to unbond on day 10. They can bond but only if they forfeit the penalty fee relative to their stake factor.


## **3. Approach Taken in Evaluating Acala**

Before we delved in, it was crucial to understand what each module is doing on a distinct level i.e as a separate package, how each module integrates with the other module. The modules in scope for this audit are three so we took to understanding each one seperately.

We focused mainly on each module's implementation, safeguards, and integration with other modules. With the limited time (we only spent ~1hr each day for 7 days) we had to look into this codebase, we spent the most of our time covering the above mentioned logic Acala.

We added some inline comments on code snippets we cover in this Analysis report to provide context for the reader.

**D1-2:**

- Getting context on Acala from the provided docs: [Docs](https://guide.acalaapps.wiki/staking/aca-staking)
- Grasping code implementations of staking entry and exit points

**D2-4:**

- Brainstorm some attack vectors relevant to doubling rewards, DoS of rewards & staking functions
- Set out building a test suite from scratch
- Further reviews of the staking, earning and bonding modules

**D4-7:**

- Delving deeper into the codebase's test suite
- Draft & submission of report

## **4. Acala Modules Analysis**

### 1. Reward accumulation across Block Initializations
```rs
#[pallet::hooks]
	impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
		fn on_initialize(now: BlockNumberFor<T>) -> Weight {
			// accumulate reward periodically
			if now % T::AccumulatePeriod::get() == Zero::zero() {
				let mut count: u32 = 0;
				let shutdown = T::EmergencyShutdown::is_shutdown();

				for (pool_id, pool_info) in orml_rewards::PoolInfos::<T>::iter() { // @note existing pool iterations for reward accumulation
					if !pool_info.total_shares.is_zero() {
						match pool_id {
							// do not accumulate incentives for PoolId::Loans after shutdown
							PoolId::Loans(_) if shutdown => {
								log::debug!(
									target: "incentives",
									"on_initialize: skip accumulate incentives for pool {:?} after shutdown", // @note debug-printing pretty
									pool_id
								);
							}
							_ => {
								count += 1;
								Self::accumulate_incentives(pool_id);
							}
						}
					}
				}

				T::WeightInfo::on_initialize(count)
			} else {
				Weight::zero()
			}
		}
	}
```
Acala's staking system has this initialization process per blocks. Let's break down this function to tiny bits in order to understand what's happening in the code blocks:

- `on_initialize` is called during block initialization
- Checks are in place for ensuring the current block number (`now`) is greater than the base accumulation period (`T::AccumulatePeriod::get()`)
- If point 2 above is true, then we proceed to accumulate rewards by iterating through all pools stored in `orml_rewards::PoolInfos`.
- For every one of those pools, we check if the total shares for such pool is not zero  -> `if !pool_info.total_shares.is_zero()`
- If the pool shares is indeed not 0, we check to see if the protocol is under the emergency shutdown i.e like the pausing mechanism of OZ `(T::EmergencyShutdown::is_shutdown())`.
- If the contract is paused, we log a message indicating the skip of accumulation of incentives for that pool.
- Otherwise, if the contract is not paused, we increment the counter to move to the next pool and we don't forget to call `accumulate_incentives` for that specific pool we just went through.
- Lastly, we return from the function.

### 2. User Entry point: Staking

```rs
pub fn deposit_dex_share(
    origin: OriginFor<T>, // originator of txn
    lp_currency_id: CurrencyId, // lp currency token
    #[pallet::compact] amount: Balance, // amount to LP
  ) -> DispatchResult { // returns a result
    let who = ensure_signed(origin)?; // beneficiary of stake LP
    Self::do_deposit_dex_share(&who, lp_currency_id, amount)?; // exec the staking
    Ok(()) // end of txn call
  }
```

Staking LP tokens is the same as `deposit_dex_share` within the Acala staking system. Let's break down this function:

- Users execute this function to stake LP
- The origin aka user/address must be `Signed` i.e, only authenticated accounts can execute this function
- `lp_currency_id`: As we have noted in the comment above, this is the type of LP token being staked
- `amount`: the amount of the LP tokens to be staked
- `do_deposit_dex_share`: execute the deposit and allocate the shares to the user

Going back a bit from this function's execution, let's understand that:
- The `ensure_signed` function call is used to make sure that the origin is signed by a valid account. If that is not the case, the transaction to deposit LP tokens reverts.

- The `do_deposit_dex_share` function is called internally to handle the rest of the deposit operation. Any errors encountered during this function's execution will propagate up also cause the entire transaction to revert.

```rs
fn do_deposit_dex_share(who: &T::AccountId, lp_currency_id: CurrencyId, amount: Balance) -> DispatchResult {
	ensure!(lp_currency_id.is_dex_share_currency_id(), Error::<T>::InvalidCurrencyId); // @note checks to make sure the currency is valid

	T::Currency::transfer(lp_currency_id, who, &Self::account_id(), amount)?; // @note transfer the tokens from the caller to us
	<orml_rewards::Pallet<T>>::add_share(who, &PoolId::Dex(lp_currency_id), amount.unique_saturated_into()); // @note mint the shares for the caller

	Self::deposit_event(Event::DepositDexShare {
		who: who.clone(),
		dex_share_type: lp_currency_id,
		deposit: amount,
	}); // @note emit the deposit event
	Ok(()) // @note return
}
```

- The internal `do_deposit_dex_share` function first ensures that the `lp_currency_id` is a valid DEX share currency ID. This validation is in place to prevent stakers from attempting to deposit shares into invalid or non-existent pools
- When the input validation is complete and the currency is valid, we transfer the specified amount of LP tokens from the user's account to the contract's account using the `T::Currency::transfer` function call
- Upon the successful transfer of LP tokens from the caller to us, we then add/mint shares to the user's balance for the specified DEX pool using the `orml_rewards::Pallet<T>::add_share` function call
- Lastly, we fire the `deposit_event` providing details of the deposit and return from the function call

### 3. User Exit point: Withdraw

```rs
pub fn withdraw_dex_share(
		origin: OriginFor<T>,
		lp_currency_id: CurrencyId,
		#[pallet::compact] amount: Balance,
	) -> DispatchResult {
		let who = ensure_signed(origin)?;
		Self::do_withdraw_dex_share(&who, lp_currency_id, amount)?;
		Ok(())
	}
```

Similar to deposits, withdrawing LP is the same as `withdraw_dex_share`. This function allows stakers to withdraw LP tokens, consequently removing/getting rid of the previously allocated shares of the staker from the Pool.

- Close in relation to the `deposit_dex_share` function, the origin must be `Signed` by the transactor for this transaction call ensuring only authenticated accounts can call and execute withdraws
- `lp_currency_id`: This also represnts the type of LP token from which shares are being withdrawn
- `amount`: This last parameter is the amount of LP tokens to be withdrawn by the caller

The internal function `do_withdraw_dex_share` handles most of the execution process for this function as we will see below:

```rs
fn do_withdraw_dex_share(who: &T::AccountId, lp_currency_id: CurrencyId, amount: Balance) -> DispatchResult {
		ensure!(lp_currency_id.is_dex_share_currency_id(), Error::<T>::InvalidCurrencyId);
		ensure!(
			<orml_rewards::Pallet<T>>::shares_and_withdrawn_rewards(&PoolId::Dex(lp_currency_id), &who).0 >= amount,
			Error::<T>::NotEnough,
		); // @note ensure user has enough shares to withdraw from their account & receives accrued rewards

		T::Currency::transfer(lp_currency_id, &Self::account_id(), who, amount)?; // @note LP token transfer back to user
		<orml_rewards::Pallet<T>>::remove_share(who, &PoolId::Dex(lp_currency_id), amount.unique_saturated_into()); // @note share removal that was previously allocated to the caller

		Self::deposit_event(Event::WithdrawDexShare {
			who: who.clone(),
			dex_share_type: lp_currency_id,
			withdraw: amount,
		}); // @note withdraw event emission
		Ok(())
	}
```

As we can see from the function's implementation and our provided comments, the function basically ensures users attempting to withdraw:

1. Have sufficient shares balance to withdraw from
2. Receive their LP token and accrued rewards
3. Proper withdrawal event is emitted

### 4. Reward claims

```rs
#[pallet::call_index(2)]
	#[pallet::weight(<T as Config>::WeightInfo::claim_rewards())]
	pub fn claim_rewards(origin: OriginFor<T>, pool_id: PoolId) -> DispatchResult {
		let who = ensure_signed(origin)?; // @note is the caller signed? this checks that

		Self::do_claim_rewards(who, pool_id) // @note runs the actual rewards claim
	}
```

The `claim_rewards` function allows users to claim all available multi-currency rewards for a specific pool that corresponds to the provided `pool_id` argument. Let's break down the implementation further:

- Similar to both deposit and withdraw functions for LP, the dispatch origin for this claim call must be Signed by the transactor. In order words, only authenticated accounts can call this function
- `origin`: the origin of the transaction, which must be signed by the user
- `pool_id`: Specification of the pool for which rewards are being claimed from

Now, we need to look into the internal `do_claimed_rewards` function to completely grasp the call trace.

```rs
fn do_claim_rewards(who: T::AccountId, pool_id: PoolId) -> DispatchResult { // @note who ==> caller && pool_id ==> pool to claim rewards from
		// orml_rewards will claim rewards for all currencies rewards
		<orml_rewards::Pallet<T>>::claim_rewards(&who, &pool_id);

		PendingMultiRewards::<T>::mutate_exists(pool_id, &who, |maybe_pending_multi_rewards| {
			if let Some(pending_multi_rewards) = maybe_pending_multi_rewards {
				let deduction_rate = Self::claim_reward_deduction_rates(&pool_id);
				let deduction_currency = ClaimRewardDeductionCurrency::<T>::get(pool_id);

				for (currency_id, pending_reward) in pending_multi_rewards.iter_mut() {
					if pending_reward.is_zero() {
						continue;
					}

					let deduction_rate = if let Some(deduction_currency) = deduction_currency {
						// only apply deduction rate to specified currency
						if deduction_currency == *currency_id {
							deduction_rate
						} else {
							Zero::zero()
						}
					} else {
						// apply deduction rate to all currencies
						deduction_rate
					};

					let (payout_amount, deduction_amount) = {
						let should_deduction_amount = deduction_rate.saturating_mul_int(*pending_reward);
						(
							pending_reward.saturating_sub(should_deduction_amount),
							should_deduction_amount,
						)
					};

					// payout reward to claimer and re-accumuated reward.
					match Self::payout_reward_and_reaccumulate_reward(
						pool_id,
						&who,
						*currency_id,
						payout_amount,
						deduction_amount,
					) {
						Ok(_) => {
							// update state
							*pending_reward = Zero::zero();

							Self::deposit_event(Event::ClaimRewards {
								who: who.clone(),
								pool: pool_id,
								reward_currency_id: *currency_id,
								actual_amount: payout_amount,
								deduction_amount,
							});
						}
						Err(e) => {
							log::error!(
								target: "incentives",
								"payout_reward_and_reaccumulate_reward: failed to payout {:?} to {:?} and re-accumulate {:?} {:?} to pool {:?}: {:?}",
								payout_amount, who, deduction_amount, currency_id, pool_id, e
							);
						}
					};
				}

				// clear zero value item of BTreeMap
				pending_multi_rewards.retain(|_, v| *v != 0);

				// if pending_multi_rewards is default, clear the storage
				if pending_multi_rewards.is_empty() {
					*maybe_pending_multi_rewards = None;
				}
			}
		});

		Ok(())
	}
```

- The function facilitates claiming of rewards for a specific pool by a user
- Reward claims: We call `orml_rewards::Pallet<T>::claim_rewards` to claim rewards for all currencies in the specified pool. Keep in mind the `orml_rewards` module handles the rewarding logic

Deduction calculation: For every pending reward in the `PendingMultiRewards` storage, we calculates the deduction amount based on the configured deduction rate (for each user)
- The deduction rate is determined by `claim_reward_deduction_rates` and `ClaimRewardDeductionCurrency::<T>::get(pool_id)`
- If a deduction currency is specified, the deduction rate is then applied to that currency. Otherwise, it is applied to all currencies.
- The deducted amount is then subtracted from the pending reward, resulting in the actual payout amount.

Reward Payout:

- We then `payout_reward_and_reaccumulate_reward` to distribute the rewards to the staker and re-accumulate the deducted amount into the pool (for the loyalty program Acala enforces).
- If the payout and re-accumulation process is deemed successful, the pending reward is updated to zero, and an event (`Event::ClaimRewards`) is emitted to log the details of the claimed rewards.

State mutation and storage reset:

- After processing all pending rewards, the function then keeps only non-zero value items in the `PendingMultiRewards` storage.
- If `pending_multi_rewards` is empty indicating that all rewards have been claimed, the storage entry is then cleared.

### **5. Reward Deduction Rates**

```rs
pub fn update_claim_reward_deduction_rates(
	origin: OriginFor<T>, // @note txn origin
	updates: Vec<(PoolId, Rate)>, // @note data struct for the pool to update and the rate to set
) -> DispatchResult {
	T::UpdateOrigin::ensure_origin(origin)?; // @note ensures caller is authenticated
	for (pool_id, deduction_rate) in updates { // @note loops through pool updates
		if let PoolId::Dex(currency_id) = pool_id {
			ensure!(currency_id.is_dex_share_currency_id(), Error::<T>::InvalidPoolId); // @note ensure pool is a valid pool
		}
		ClaimRewardDeductionRates::<T>::mutate_exists(pool_id, |maybe_rate| -> DispatchResult {
			let mut v = maybe_rate.unwrap_or_default();
			if deduction_rate != *v.inner() {
				v.try_set(deduction_rate).map_err(|_| Error::<T>::InvalidRate)?;
				Self::deposit_event(Event::ClaimRewardDeductionRateUpdated {
					pool: pool_id,
					deduction_rate,
				});
			}

			if v.inner().is_zero() {
				*maybe_rate = None;
			} else {
				*maybe_rate = Some(v);
			}
			Ok(())
		})?;
	}
	Ok(())
}
```

`update_claim_reward_deduction_rates` allows for updating claim reward deduction rates for all reward currencies of a specific Pool

- `updates`: A somewhat data struct of tuples representing the `PoolId` and corresponding deduction rate to be applied
- For each `update` in the `updates` vector/data struct, the function then proceeds to check if the `PoolId` corresponds to a valid DEX pool. If that is the case, we ensure that the associated currency ID is a valid DEX share currency ID
- Moving on, for each `update`, we mutate the `ClaimRewardDeductionRates` storage to update the deduction rate associated with the specified `PoolId`.
- If the new deduction `rate` differs from the current `rate`, it is set, and the `ClaimRewardDeductionRateUpdated` event is emitted to log the update.
- Keep in mind that if the `rate` attempted to be set is the same as the current `rate`, we skip doing the update as it's pointless in that case to set an already set state.
- If the deduction `rate` becomes zero after the update, the corresponding entry in the `ClaimRewardDeductionRates` storage is then removed to conserve storage space

## **5. Call-trace Diagrams**
Entry point for staking:
![Entry](https://rexjoseph.github.io/images/acala_deposit.png)

Reward claims:
![Reward-claims](https://rexjoseph.github.io/images/acala_claim_reward.png)

Exit point for staking:
![Exit](https://rexjoseph.github.io/images/acala_withdraw.png)

## **6. Codebase Quality**

Here are some of our observations of the `Acala` codebase's quality.

### Readability:
The protocol's function implementations are well documented and easy to follow. The code blocks in general follow a consistent naming guide that hints at what the function implementation does and is supposed to return after execution.

### Maintainability:
Each architecture component is seperated into modules. For example, the staking module is separate from the reward modules. They each utilize the other' implementation as we can see in the staking module where claiming of rewards borrows code implementation from the rewards code logic.

This effectively makes the codebase manageable as you can change a function implementation in one module without introducing breaking changes in other modules that utilize it.

### Performance:
Testing suite for the 3 modules in sope are great. They are normal unit-style tests but they live in their separate files away from the production code.

### Robustness:
In most of the function implementation across the codebase, errors are handled from the onset of the function's execution. This is great. It ensures the call can already be reverted without getting up to the point where the invalid state resulting from the bad data is reached to bubble up the error.

## **7. Systematic Risks and Centralization**

This section of the analysis report covers error handling, slight flaws and major systemic risks of the covered components in the Acala Staking System.

### Pool initialization: `/incentives/src/lib.rs`

- Pools are created by Governance in the Acala staking system. So, users are not able to create arbitrary staking pools which is good. But, a malicious governor privileged individual can mass create pools with empty stakes that is zero shares and DoS the entire staking mechanism when we run into reward accumulation and pool iteration during blocks initialization in the code snippet below:

```rs
 for (pool_id, pool_info) in orml_rewards::PoolInfos::<T>::iter() {
  if !pool_info.total_shares.is_zero() {
    match pool_id {
      // do not accumulate incentives for PoolId::Loans after shutdown
      PoolId::Loans(_) if shutdown => {
        log::debug!(
          target: "incentives",
          "on_initialize: skip accumulate incentives for pool {:?} after shutdown",
          pool_id
        );
      }
      _ => {
        count += 1;
        Self::accumulate_incentives(pool_id);
      }
    }
  }
}
```

Assuming the Governace code becomes vulnerable to malicious actors, one way to put an end to staking by DoS is mass creation of pools that are not legitimate and force the loop interation of pools in the next block initialization to fail.


## Insights

### Pool initialization: `/incentives/src/lib.rs`

The current logic of looping through pools is non scalable. Optimization is completely thrown out the window this way. I'm sure the team believes goverance will not be malicious or governance code implementation will always be safe but once any one of these happen to not be the case, the whole staking model rests on the balance. Instead of looping over pools, implement caching or use structs to store necessary pool information and use that during the block initializations. This brings scalability back into the control of the protocol.

In summary, the Acala codebase review was a great learning experience for understanding some niche aspects such as their utilization of block initializations. After having spent a couple days understanding the high level concept, we finally were able to pull up this report for understanding some parts of the protocol.


### Time spent:
7 hours