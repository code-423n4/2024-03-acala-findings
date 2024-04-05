## Insufficient Use of Check-Effect-Interaction Pattern in do_withdraw_dex_share() Function

## Impact
Failure to apply the CEI pattern correctly in `do_withdraw_dex_share()` could potentially lead to reentrancy attacks, wherein an external entity could manipulate the contract's state during fund transfer, resulting in unexpected behavior or loss of funds.

## Proof of Concept
In the `Incentives` module, the `do_withdraw_dex_share()` function transfers funds before removing the user's shares:
```
		T::Currency::transfer(lp_currency_id, &Self::account_id(), who, amount)?;
		<orml_rewards::Pallet<T>>::remove_share(who, &PoolId::Dex(lp_currency_id), amount.unique_saturated_into());
```
This approach contradicts the CEI pattern and may potentially lead to reentrancy attacks.

## Mitigation
To address this issue, it is recommended to apply the CEI pattern correctly. First, remove the user's shares, then transfer the assets.