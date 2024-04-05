# Code and comment mismatch

In `earning::lib.rs`, lines 134 and 135 it says `If user available balances is less than amount, then all the remaining balances will be locked.` in the comments, but the functions `bond` does not check like that. Consider either removing those comments or putting the code to lock balances if user available balances is less than amount. 

[earning::lib.rs#L134-L135](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/earning/src/lib.rs#L134C1-L135C14)


```rust

	#[pallet::call]
	impl<T: Config> Pallet<T> {
		/// Bond tokens by locking them up to `amount`.
@audit  /// If user available balances is less than amount, then all the remaining balances will be
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