## L-1: Reliance on block number instead of timestamp

[Incentives are accumulated](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L207) periodically in intervals of [T::AccumulatePeriod](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L190) blocks.  
This is not recommended since the block production time is **not** assured to be constant over time.  
Consequently, inconsistent incentive reward amounts (per time interval) can be accumulated, which is perceived is *unpredictable* and *unfair* by users.


**Recommendation:**  
Use timestamp based incentive accumulation for fixed time periods. See also: [pallet-timestamp](https://marketplace.substrate.io/pallets/pallet-timestamp/)  
Or, if the block number based approach is still desired, add a method which allows to change the *accumulate period* if necessary.

## NC-1: Inconsistent hook naming among *earning* an *incentives* modules
According to the README about the *earning* module:  
> The module implements a set of hooks that can be used by other modules **(i.e. Incentives)** to implement staking.

Hook names in *earning* module: [OnBonded, OnUnbonded](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/earning/src/lib.rs#L62-L63)

Hook names in *incentives* module: [OnEarningBonded, OnEarningUnbonded](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L607-L619)

Since those two modules are meant to be connected via these hooks, the inconsistent naming might be misleading and cause integration issues. 