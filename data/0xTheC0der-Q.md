## L-1: Reliance on block number instead of timestamp

[Incentives are accumulated](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L207) periodically in intervals of [T::AccumulatePeriod](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L190) blocks.  
This is not recommended since the block production time is **not** assured to be constant over time.  
Consequently, inconsistent incentive reward amounts (per time interval) can be accumulated, which is perceived is *unpredictable* and *unfair* by users.


**Recommendation:**  
Use timestamp based incentive accumulation for fixed time periods. See also: [pallet-timestamp](https://marketplace.substrate.io/pallets/pallet-timestamp/)  
Or, if the block number based approach is still desired, add a method which allows to change the *accumulate period* if necessary.