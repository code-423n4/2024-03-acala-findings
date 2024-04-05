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



## [L09] Available balance can be misleading due to not counting min existential deposit 

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/earning/src/lib.rs#L259-L262




## [L10] OnloanUpdate can break accounting between bonded tokens and shares

If the earning module is hooked into OnloanUpdate, then it would break the invariant of having the total bonded token match the total amount of shares

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L576-L588



## [L11] function Claim_reward is not used anywhere in the codebase

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L293-L317


## [L12] DepositEvent naming convention is confusing 

https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L535

Deposit Event is used for events that have nothing to do with depositing funds which can be confusing 


## [L13] do_deposit_dex_share does not check if a user has the required amount of capital to deposit 

https://github.com/code-423n4/2024-03-acala/blob/28c60b339bc011d59fa05fda8276c8e70a7f5779/src/modules/incentives/src/lib.rs#L510-L523

The function allows a user to to deposit funds to receive shares. However, there is no check whether a user has the required capital to make the deposit. This is shown here 

https://github.com/code-423n4/2024-03-acala/blob/28c60b339bc011d59fa05fda8276c8e70a7f5779/src/modules/incentives/src/lib.rs#L514

 Without this check a user could potentially gain shares that they did not deserve. For example, a user can make a deposit of 1000 token to receive shares, when they only have 10 tokens, but since the function does not check whether a user has the required funds, the function will grant the user extra shares that was not deserved.