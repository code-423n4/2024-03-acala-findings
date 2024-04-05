## [L01] function payout_reward_and_reaccumulate_reward is misleading 
The function name is misleading due to incorrect order. According to the code, it reaccumulates and then payouts 

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L495-L508



## [L02] Crate `parity-wasm` deprecated by the author

https://rustsec.org/advisories/RUSTSEC-2022-0061


## [L03] function set_share is not used anywhere 

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L253-L261



## [L04] function transfer_share_and_rewards isn't used anywhere

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L324-L361 




## [L05] OnUpdateLoan is not used anywhere  

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L576-L588





## [L06] Pointless check on check.min.bond

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/primitives/src/bonding/ledger.rs#L169

There is no reason to check for the min bond on the function rebond because that check is already made in the bond and unbond functions


## [L07] 2 functions with same name can be confusing 

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L607-L618



## [L08] No ensure origin in the IncentivesManager imple

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L544-L574



## [L09] Available balance can be misleading due to not counting min deposit 

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/earning/src/lib.rs#L259-L262




## [L10] OnloanUpdate can break accounting between bonded tokens and shares

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L576-L588



[L10] Incentive Program should allow users to unbond immediately after it ends 



