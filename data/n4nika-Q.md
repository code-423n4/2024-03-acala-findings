## Misdistribution of rewards if they cannot be cleanly distributed

### Impact
In the incentives module, in certain situations where `((REWARDS // AMNT_PARTICIPANTS) * AMNT_PARTICIPANTS) != REWARDS` and the participants claim their rewards, if a new participant deposits shares into the pool after the first accumulation of rewards, upon the next accumulation, the rounding-difference of the first round is distributed between all participants, not only the ones participating during the first accumulation.  
This violates "When a user adds liquidity to the pool, they receive shares, and withdrawn rewards are adjusted accordingly so the new user will start with no reward." written in the scope description on the C4 site for the audit.

### Proof of Concept
Requires adding the following accounts to `modules/incentives/src/mock.rs`:
```Rust
pub const ACC1: AccountId = AccountId::from([6u8; 32]);
pub const ACC2: AccountId = AccountId::from([7u8; 32]);
pub const ACC3: AccountId = AccountId::from([8u8; 32]);
pub const ACC4: AccountId = AccountId::from([9u8; 32]);
pub const ACC5: AccountId = AccountId::from([10u8; 32]);
pub const ACC6: AccountId = AccountId::from([11u8; 32]);
pub const ACC7: AccountId = AccountId::from([12u8; 32]);
pub const ACC8: AccountId = AccountId::from([13u8; 32]);
pub const ACC9: AccountId = AccountId::from([14u8; 32]);
```
```Rust
#[test]
fn bad_distribution() {
	ExtBuilder::default().build().execute_with(|| {
		assert_ok!(TokensModule::deposit(BTC, &VAULT::get(), 10000));

		assert_ok!(IncentivesModule::update_claim_reward_deduction_rates(
			RuntimeOrigin::signed(ROOT::get()),
			vec![(PoolId::Dex(BTC_AUSD_LP), Rate::saturating_from_rational(0, 100))]
		));

		// give RewardsSource tokens
		TokensModule::deposit(BTC, &RewardsSource::get(), 10000);

		// add 11 participants with equal shares
		TokensModule::deposit(BTC_AUSD_LP, &ALICE::get(), 100);
		TokensModule::deposit(BTC_AUSD_LP, &BOB::get(), 100);
		TokensModule::deposit(BTC_AUSD_LP, &CHARLIE::get(), 100);
		TokensModule::deposit(BTC_AUSD_LP, &ACC1::get(), 100);
		TokensModule::deposit(BTC_AUSD_LP, &ACC2::get(), 100);
		TokensModule::deposit(BTC_AUSD_LP, &ACC3::get(), 100);
		TokensModule::deposit(BTC_AUSD_LP, &ACC4::get(), 100);
		TokensModule::deposit(BTC_AUSD_LP, &ACC5::get(), 100);
		TokensModule::deposit(BTC_AUSD_LP, &ACC6::get(), 100);
		TokensModule::deposit(BTC_AUSD_LP, &ACC7::get(), 100);
		TokensModule::deposit(BTC_AUSD_LP, &ACC8::get(), 100);
		TokensModule::deposit(BTC_AUSD_LP, &ACC9::get(), 100);

		// 1000 BTC as reward
		IncentivesModule::update_incentive_rewards(RuntimeOrigin::signed(ROOT::get()), vec![(PoolId::Dex(BTC_AUSD_LP), vec![(BTC, 1000)])]);
		
		// all 11 participants deposit their share
		assert_ok!(IncentivesModule::deposit_dex_share(RuntimeOrigin::signed(ALICE::get()), BTC_AUSD_LP, 100));
		assert_ok!(IncentivesModule::deposit_dex_share(RuntimeOrigin::signed(BOB::get()), BTC_AUSD_LP, 100));
		assert_ok!(IncentivesModule::deposit_dex_share(RuntimeOrigin::signed(CHARLIE::get()), BTC_AUSD_LP, 100));
		assert_ok!(IncentivesModule::deposit_dex_share(RuntimeOrigin::signed(ACC1::get()), BTC_AUSD_LP, 100));
		assert_ok!(IncentivesModule::deposit_dex_share(RuntimeOrigin::signed(ACC2::get()), BTC_AUSD_LP, 100));
		assert_ok!(IncentivesModule::deposit_dex_share(RuntimeOrigin::signed(ACC3::get()), BTC_AUSD_LP, 100));
		assert_ok!(IncentivesModule::deposit_dex_share(RuntimeOrigin::signed(ACC4::get()), BTC_AUSD_LP, 100));
		assert_ok!(IncentivesModule::deposit_dex_share(RuntimeOrigin::signed(ACC5::get()), BTC_AUSD_LP, 100));
		assert_ok!(IncentivesModule::deposit_dex_share(RuntimeOrigin::signed(ACC6::get()), BTC_AUSD_LP, 100));
		assert_ok!(IncentivesModule::deposit_dex_share(RuntimeOrigin::signed(ACC7::get()), BTC_AUSD_LP, 100));
		assert_ok!(IncentivesModule::deposit_dex_share(RuntimeOrigin::signed(ACC8::get()), BTC_AUSD_LP, 100));
		
		// first accumulation of rewards
		IncentivesModule::on_initialize(10);
		
		// everyone claims their rewards
		assert_ok!(IncentivesModule::claim_rewards(RuntimeOrigin::signed(ALICE::get()), PoolId::Dex(BTC_AUSD_LP)));
		assert_ok!(IncentivesModule::claim_rewards(RuntimeOrigin::signed(BOB::get()), PoolId::Dex(BTC_AUSD_LP)));
		assert_ok!(IncentivesModule::claim_rewards(RuntimeOrigin::signed(CHARLIE::get()), PoolId::Dex(BTC_AUSD_LP)));
		assert_ok!(IncentivesModule::claim_rewards(RuntimeOrigin::signed(ACC1::get()), PoolId::Dex(BTC_AUSD_LP)));
		assert_ok!(IncentivesModule::claim_rewards(RuntimeOrigin::signed(ACC2::get()), PoolId::Dex(BTC_AUSD_LP)));
		assert_ok!(IncentivesModule::claim_rewards(RuntimeOrigin::signed(ACC3::get()), PoolId::Dex(BTC_AUSD_LP)));
		assert_ok!(IncentivesModule::claim_rewards(RuntimeOrigin::signed(ACC4::get()), PoolId::Dex(BTC_AUSD_LP)));
		assert_ok!(IncentivesModule::claim_rewards(RuntimeOrigin::signed(ACC5::get()), PoolId::Dex(BTC_AUSD_LP)));
		assert_ok!(IncentivesModule::claim_rewards(RuntimeOrigin::signed(ACC6::get()), PoolId::Dex(BTC_AUSD_LP)));
		assert_ok!(IncentivesModule::claim_rewards(RuntimeOrigin::signed(ACC7::get()), PoolId::Dex(BTC_AUSD_LP)));
		assert_ok!(IncentivesModule::claim_rewards(RuntimeOrigin::signed(ACC8::get()), PoolId::Dex(BTC_AUSD_LP)));

		// new user deposits into the pool
		assert_ok!(IncentivesModule::deposit_dex_share(RuntimeOrigin::signed(ACC9::get()), BTC_AUSD_LP, 100));
		
		// new accumulation, this adds another 1000 BTC as rewards
		IncentivesModule::on_initialize(20);

		assert_ok!(IncentivesModule::claim_rewards(RuntimeOrigin::signed(ACC9::get()), PoolId::Dex(BTC_AUSD_LP)));
		
		// the nerw participant should have (1000 // 12) = 83 BTC 
		// but it has 84, which is a portion of the rounded-away tokens from the first accumulation
		assert_eq!(TokensModule::total_balance(BTC, &ACC9::get()), 83);
	});
}
```