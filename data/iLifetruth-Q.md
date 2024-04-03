# Lack of Sanity Check

Description: If the user calls the function IncentivesModule::claim_rewards(), there is a lack of check when both deduction_rate and deduction_currency exist for the specified pool_id in the sub-function do_claim_rewards(). If the deduction_currency has been set for the pool corresponding to the pool_id in advance, then there should be a corresponding check for the existence of the deduction_rate (because if the deduction_rate is not set in advance, the deduction_rate obtained through the function claim_reward_deduction_rates() returns 0 at line 422).   

src/modules/incentives/src/lib.rs  #line 416
```rust
    fn do_claim_rewards(who: T::AccountId, pool_id: PoolId) -> DispatchResult {
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

===============================
The following unit test code shows that the transaction execution in this case does not return an error, and the system administrator cannot detect the existence of this problem, resulting in the following impacts:

This test code is based on the unit test claim_rewards_works():
```rust
#[test]
fn uninitialized_reward_deduction_rates() {
    ExtBuilder::default().build().execute_with(|| {
        assert_ok!(TokensModule::deposit(ACA, &VAULT::get(), 10000));
        assert_ok!(TokensModule::deposit(AUSD, &VAULT::get(), 10000));
        assert_ok!(TokensModule::deposit(LDOT, &VAULT::get(), 10000));
        assert_ok!(IncentivesModule::update_claim_reward_deduction_rates(
            RuntimeOrigin::signed(ROOT::get()),
            vec![
                (PoolId::Dex(BTC_AUSD_LP), Rate::saturating_from_rational(20, 100)),
                (PoolId::Loans(ACA), Rate::saturating_from_rational(20, 100)),
            ]
        ));
        assert_ok!(IncentivesModule::update_claim_reward_deduction_rates(
            RuntimeOrigin::signed(ROOT::get()),
            vec![
                (PoolId::Dex(BTC_AUSD_LP), Rate::saturating_from_rational(40, 100)),
                (PoolId::Loans(ACA), Rate::saturating_from_rational(40, 100)),
            ]
        ));
        assert_ok!(IncentivesModule::update_claim_reward_deduction_rates(
            RuntimeOrigin::signed(ROOT::get()),
            vec![
                (PoolId::Dex(BTC_AUSD_LP), Rate::saturating_from_rational(50, 100)),
                (PoolId::Loans(ACA), Rate::saturating_from_rational(60, 100)),
            ]
        ));
        assert_ok!(IncentivesModule::update_claim_reward_deduction_rates(
            RuntimeOrigin::signed(ROOT::get()),
            vec![(PoolId::Loans(ACA), Rate::saturating_from_rational(80, 100)),]
        ));
        assert_ok!(IncentivesModule::update_claim_reward_deduction_rates(
            RuntimeOrigin::signed(ROOT::get()),
            vec![(PoolId::Loans(ACA), Rate::saturating_from_rational(90, 100)),]
        ));

        // alice add shares before accumulate rewards
        RewardsModule::add_share(&ALICE::get(), &PoolId::Loans(BTC), 100);
        RewardsModule::add_share(&ALICE::get(), &PoolId::Dex(BTC_AUSD_LP), 100);

        // bob add shares before accumulate rewards
        RewardsModule::add_share(&BOB::get(), &PoolId::Dex(BTC_AUSD_LP), 100);

        // accumulate rewards for different pools
        assert_ok!(RewardsModule::accumulate_reward(&PoolId::Loans(BTC), ACA, 2000));
        assert_ok!(RewardsModule::accumulate_reward(&PoolId::Dex(BTC_AUSD_LP), ACA, 1000));
        assert_ok!(RewardsModule::accumulate_reward(&PoolId::Dex(BTC_AUSD_LP), AUSD, 2000));

        // bob add share after accumulate rewards
        RewardsModule::add_share(&BOB::get(), &PoolId::Loans(BTC), 100);

        // accumulate LDOT rewards for PoolId::Loans(BTC)
        assert_ok!(RewardsModule::accumulate_reward(&PoolId::Loans(BTC), LDOT, 500));

        // alice claim rewards for PoolId::Loans(BTC)
        assert_eq!(
            RewardsModule::pool_infos(PoolId::Loans(BTC)),
            PoolInfo {
                total_shares: 200,
                rewards: vec![(ACA, (4000, 2000)), (LDOT, (500, 0))].into_iter().collect(),
            }
        );
        assert_eq!(
            RewardsModule::shares_and_withdrawn_rewards(PoolId::Loans(BTC), ALICE::get()),
            (100, Default::default())
        );
        assert_eq!(TokensModule::free_balance(ACA, &VAULT::get()), 10000);
        assert_eq!(TokensModule::free_balance(LDOT, &VAULT::get()), 10000);
        assert_eq!(TokensModule::free_balance(ACA, &ALICE::get()), 0);
        assert_eq!(TokensModule::free_balance(LDOT, &ALICE::get()), 0);
        assert_ok!(IncentivesModule::claim_rewards(
            RuntimeOrigin::signed(ALICE::get()),
            PoolId::Loans(BTC)
        ));

    });
}
```

===============================
Impact: Unexcepted Business Logic
In the given code, if the deduction_rate is not configured, zero will be used as the deduction rate when calculating the deduction amount. This may lead to the following consequences:

1. Excessive reward payment: If the deduction_rate is not configured, then all rewards will be paid in full without any deductions. This may lead to excessive reward payments beyond expectations. （And this cannot be checked immediately and intuitively）
2. Unfair reward allocation: If the deduction_rate is not configured, different reward recipients may be treated differently because there is no unified deduction standard. This may lead to unfair reward allocation.
3. Financial issues: If the deduction_rate is not configured, it may lead to financial issues because there is no accurate reward deduction budget. This may result in excessive or insufficient reward payments, affecting financial status.

Therefore, to ensure the smooth process of claiming rewards, it is recommended to correctly configure the deduction_rate in the code to avoid the occurrence of the above problems.
