## Low severity issues

1. Redundant 0 amount check

In function `accumulate_reward` of orml_reward (https://github.com/code-423n4/2024-03-acala/blob/main/src/orml/rewards/src/lib.rs#L120), checks if the `reward_increment` or the input amount is 0 or not. But this check is already done previously. 

For example if the `accumulate_reward` function gets called through `payout_reward_and_reaccumulate_reward` , the check is already present there https://github.com/code-423n4/2024-03-acala/blob/main/src/modules/incentives/src/lib.rs#L502

Or if the `accumulate_reward` function gets called through `transfer_rewards_and_update_records` function of `accumulate_incentives` function, the check is already done there https://github.com/code-423n4/2024-03-acala/blob/main/src/modules/incentives/src/lib.rs#L383

Similar issue, can also be found in `payout` function of incentive module, https://github.com/code-423n4/2024-03-acala/blob/main/src/modules/incentives/src/lib.rs#L595
where the check is done before hand in `claim_one` function https://github.com/code-423n4/2024-03-acala/blob/main/src/orml/rewards/src/lib.rs#L382

2. `ClaimRewardDeductionRates` and `ClaimRewardDeductionCurrency` can be merged to prevent storage space, which in turn helps in preventing slow network performance and resources running out.

https://github.com/code-423n4/2024-03-acala/blob/main/src/modules/incentives/src/lib.rs#L159
https://github.com/code-423n4/2024-03-acala/blob/main/src/modules/incentives/src/lib.rs#L165

They can be merged into a single storage structure, something like 
`pub type ClaimRewardDeductions<T: Config> = StorageMap<_, Twox64Concat, PoolId, (FractionalRate, CurrencyId), ValueQuery>` to prevent unnecessary storage read/write

3. Use `saturating_mul` and `checked_div` instead of `*` and `/` to prevent overflows

https://github.com/code-423n4/2024-03-acala/blob/main/src/orml/rewards/src/lib.rs#L342-L343


