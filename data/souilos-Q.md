
# INFO-1 Staking your entire balance of ACA tokens would lock them in the unbonding queue for a period of 28 days before they become available for withdrawal

## Impact
**Liquidity Constraints:** Locking up tokens for a fixed period reduces liquidity, meaning users cannot readily access their tokens for trading, transfers, or other purposes. This could be problematic if users need access to their funds urgently.

**User Experience:** For users expecting more flexibility in managing their tokens, being unable to unstake for such a long period may lead to dissatisfaction and frustration with the staking platform or protocol.

**Reduced Participation:** Some users may be deterred from staking their tokens if they are concerned about the inability to unstake them for a considerable period. This could impact the overall participation rate in the staking ecosystem.

**Protocol Governance:** If a large portion of tokens are staked and cannot participate in governance activities, it may affect the decentralization and effectiveness of the protocol's governance mechanisms.

From the doc:
https://guide.acalaapps.wiki/staking/aca-staking#unstaking

"Note: i. that unstaking will take 28 days, see Unstaking section below. ii. Keep in mind that you will need to pay transaction fees to unstake and that you won't be able to use your staked ACA to pay those transaction fees. i.e. You may not want to stake MAX ACA and keep some small amount as fees."

## Affected code
https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/earning/src/lib.rs#L138

## Proof of Concept
The ```bond```function allows users to stake their whole amount of ACA tokens.

```rust
	#[pallet::call]
	impl<T: Config> Pallet<T> {
		/// Bond tokens by locking them up to `amount`.
		/// If user available balances is less than amount, then all the remaining balances will be
		/// locked.
		#[pallet::call_index(0)]
		#[pallet::weight(T::WeightInfo::bond())]
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

```rust
impl<T: Config> Pallet<T> {}

impl<T: Config> BondingController for Pallet<T> {
	type MinBond = T::MinBond;
	type MaxUnbondingChunks = T::MaxUnbondingChunks;
	type Moment = BlockNumberFor<T>;
	type AccountId = T::AccountId;

	type Ledger = Ledger<T>;

	fn available_balance(who: &Self::AccountId, ledger: &BondingLedgerOf<T>) -> Balance {
		let free_balance = T::Currency::free_balance(who);
		free_balance.saturating_sub(ledger.total())
	}

	fn apply_ledger(who: &Self::AccountId, ledger: &BondingLedgerOf<T>) -> DispatchResult {
		if ledger.is_empty() {
			T::Currency::remove_lock(T::LockIdentifier::get(), who);
		} else {
			T::Currency::set_lock(T::LockIdentifier::get(), who, ledger.total(), WithdrawReasons::all());
		}
		Ok(())
	}

	fn convert_error(err: bonding::Error) -> DispatchError {
		match err {
			bonding::Error::BelowMinBondThreshold => Error::<T>::BelowMinBondThreshold.into(),
			bonding::Error::MaxUnlockChunksExceeded => Error::<T>::MaxUnlockChunksExceeded.into(),
			bonding::Error::NotBonded => Error::<T>::NotBonded.into(),
		}
	}
}
```
Without any token left in your balance you wouldn't be able to pay the fees to unstake instantly by calling `unbond_instant`.
You would have to follow the unbonding process in `unbond` function.

From the doc:
https://guide.acalaapps.wiki/staking/aca-staking#unstaking
"Unstaking period - Unstaking takes approximately 28 days, or 201,600 blocks from the unstaking request is submitted."

## Tools Used
Manual review.

## Recommended Mitigation Steps
To mitigate the risk of users staking 100% of their entire balance, you can implement a constraint or limit on the maximum amount that users can bond.
You could define a maximum bonding percentage and calculate the maximum bondable amount.