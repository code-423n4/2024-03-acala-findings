# Analysis Report for Acala

![](https://pbs.twimg.com/profile_images/1379064847439175692/upOKBONH_400x400.jpg)

## Table of Contents

- [Approach](#approach)
- [Brief Overview](#brief-overview)
- [Scope and Architecture Overview](#scope-and-architecture-overview)
- [Earning Module](#earning-module)
  - [Earning Module Framework](#earning-module-framework)
  - [Line by Line Deep Dive into Earning Module's Code Snippets For Non-Rust Accustommed Readers](#line-by-line-deep-dive-into-earning-modules-code-snippets-for-non-rust-accustommed-readers)
  - [Earning Module Testing Suite Setup](#earning-module-testing-suite-setup)
- [Rewards Module](#rewards-module)
  - [Rewards Module Framework](#rewards-module-framework)
  - [Line by Line Deep Dive into Rewards Module's Code Snippets For Non-Rust Accustomed Readers](#line-by-line-deep-dive-into-rewards-modules-code-snippets-for-non-rust-accustomed-readers)
  - [Rewards Module Testing Suite Setup](#rewards-module-testing-suite-setup)
- [Incentives Module](#incentives-module)
  - [Incentives Module Framework](#incentives-module-framework)
  - [Line by Line Deep Dive into Incentives Module's Code Snippets For Non-Rust Accustomed Readers](#line-by-line-deep-dive-into-incentives-modules-code-snippets-for-non-rust-accustomed-readers)
  - [Incentives Module Testing Suite Setup](#incentives-module-testing-suite-setup)
- [Business Logic ](#business-logic)
- [Centralization Risks](#centralization-risks)
- [Systemic Risks](#systemic-risks)
- [Recommendations](#recommendations)
- [Security Researcher Logistics](#security-researcher-logistics)
- [Conclusion](#conclusion)
- [Resources](#resources)

## Approach

Started with the `readMe`, then a comprehensive analysis of the provided docs to understand protocol functionality and key points, ambiguities were discussed with the sponsors(deva), and possible risk areas were outlined.

Followed by a manual review of each contract in scope, testing function behavior, protocol logic against expectations, and working out potential attack vectors. Vulnerabilities related to dependencies and inheritances, were also assessed. Comparisons with similar protocols was also performed to identify recurring issues and evaluate fix effectiveness.

Finally, identified issues from the security review were compiled into a comprehensive audit report.

## Brief Overview

Acala is a decentralized stablecoin and liquidity blockchain designed for cross-chain integration and equipped with forkless upgradability. It employs a multi-collateral-backing mechanism to establish a stablecoin closely pegged to the US Dollar, functioning as a decentralized monetary reserve that stabilizes currency through a mix of assets. Users can leverage the stablecoin for transactions and services, mitigating price volatility while maintaining exposure to their reserve assets, including Polkadot's DOT, Acala's ACA, and cross-chain assets like BTC and ETH.

Now, within the Acala ecosystem, which is bolstered by Polkadot’s cross-chain capabilities, there are multiple different Substrate modules, but only 3 are in scope for this contest, i.e the ones that focus on the staking and earning functionalities: the `Incentives` module for staking LP tokens and managing staking pool rewards, the `Rewards` module for reward calculation and distribution, and the `Earning` module that supports a bonded or locked token system for time-bound token locking.

## Scope and Architecture Overview

## Earning Module

### [Earning Module Framework](https://github.com/code-423n4/2024-03-acala/tree/main/src/modules/earning/src)

The Earning Module within the Acala framework is designed to facilitate the bonding and unbonding of tokens, serving as a cornerstone for staking and earning processes. It provides the infrastructure for users to engage actively with the platform's financial activities, allowing them to bond tokens, initiate unbonding, and manage their investments effectively over time.

At the core of this module is the functionality that enables users to bond their tokens as a form of investment or collateral, represented through bonding operations. This is encapsulated within the `bond` function, which locks the specified amount of tokens from the user's balance, adhering to the minimum bond threshold and other regulatory parameters set within the system:

```rust
pub fn bond(origin: OriginFor<T>, #[pallet::compact] amount: Balance) -> DispatchResult { ... }
```

Furthermore, the module offers an `unbond` function, allowing users to initiate the withdrawal of their bonded tokens, subject to an unbonding period that ensures the stability and security of the financial ecosystem. This period acts as a cooldown, during which the tokens are prepared for release back to the user's control:

```rust
pub fn unbond(origin: OriginFor<T>, #[pallet::compact] amount: Balance) -> DispatchResult { ... }
```

Users can request to unbond or unlock their tokens, after which they must wait for the unbonding period to complete before the tokens are available for withdrawal. This mechanism ensures a steady and predictable change in the platform's liquidity and staking composition.

For those seeking immediate access to their funds, the module provides an `unbond_instant` feature, enabling users to bypass the standard unbonding timeline at the cost of an instant unstake fee. This mechanism ensures that while operational flexibility is afforded, it is balanced with a fee structure to discourage frivolous or excessive instant unbonding actions:

```rust
pub fn unbond_instant(origin: OriginFor<T>, #[pallet::compact] amount: Balance) -> DispatchResult { ... }
```

Additionally, the module supports rebonding of funds that are in the unbonding phase, allowing users to reverse their unbonding decision and reallocate their funds back to the bonded state, thus maintaining their investment in the platform:

```rust
pub fn rebond(origin: OriginFor<T>, #[pallet::compact] amount: Balance) -> DispatchResult { ... }
```

The withdrawal process is streamlined through a `withdraw_unbonded` function, which finalizes the unbonding process and transfers the unbonded tokens back to the user's free balance, completing the investment cycle:

```rust
pub fn withdraw_unbonded(origin: OriginFor<T>) -> DispatchResult { ... }
```

The Earning Module also incorporates a set of hooks and interfaces that allow for seamless interaction with other modules within the Acala ecosystem, such as the Incentives Module. These hooks enable the Earning Module to react to events like token bonding and unbonding, ensuring that the broader logic of staking, rewards, and penalties is consistently applied across the platform's various functionalities.

In summary, the Earning Module in the Acala ecosystem is a dynamic and user-centric framework that provides robust functionalities for token bonding and unbonding. It is designed to support the financial engagement of users within the platform, it efacilitates staking rewards through a simplified algorithm focused on single asset pools. Here's how it operates:

- The pool tracks total shares, rewards, and withdrawn rewards.
- User-specific data includes their share count and the rewards they've withdrawn.

Adding shares involves inflating the pool's rewards proportionally to the new shares, simulating the user's immediate claim on these added rewards. This adjustment ensures that new shares don't dilute the rewards for existing stakeholders.

```python
def add_share(pool, users, user, user_share):
    ...
```

To manage reward distribution, the module allows for reward accumulation within the pool and enables users to claim their proportional share of these rewards based on their stake:

```python
def accumulate_reward(pool, amount):
    ...

def claim_rewards(pool, users, user):
    ...
```

The core principle here is to maintain fair reward distribution, preventing new shares from accessing previously accumulated rewards. The mathematical proof provided establishes that to avoid diluting existing shares when new stakes are added, the reward pool must increase accordingly. This ensures that the reward per share for existing users remains constant, and the additional rewards for new shares are accounted for separately, marked as withdrawn by the new user.

### Line by Line Deep Dive into Earning Module's Code Snippets For Non-Rust Accustommed Readers

<details>
  <summary>Click To See Line by Line Deep Dive</summary>

- `#![allow(clippy::unused_unit)]`: This line is a compiler directive that suppresses a specific Clippy warning about unused unit expressions. It's not directly related to the pallet code but is a general Rust directive.

- `#[frame_support::pallet]`: This attribute macro from the `frame_support` crate is used to mark the module as a pallet. It's a mandatory part of defining a pallet in Substrate [1].

- `pub mod module {`: This line starts the definition of a module named `module`. The `pub` keyword makes the module public, meaning it can be accessed from outside the module.

- `use super::*;`: This line imports all items from the parent module into the current module. It's a common pattern in Rust to simplify access to items defined in the parent module.

- `#[pallet::config]`: This attribute macro marks the following trait as the configuration trait for the pallet. It's used to define the types and traits that the pallet depends on [1].

- `pub trait Config: frame_system::Config {`: This line starts the definition of a trait named `Config` that inherits from `frame_system::Config`. This trait is used to configure the pallet, specifying the types it depends on.

- `type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;`: This line defines a type alias for the runtime event type. It specifies that the runtime event type must implement the `From<Event<Self>>` trait and the `IsType` trait with the runtime event type of the frame system configuration.

- `type Currency: LockableCurrency<Self::AccountId, Balance = Balance>;`: This line defines a type alias for the currency type. It specifies that the currency type must implement the `LockableCurrency` trait with the account ID type and balance type defined by the pallet.

- `type ParameterStore: ParameterStore<Parameters>;`: This line defines a type alias for the parameter store. It specifies that the parameter store type must implement the `ParameterStore` trait with the parameters defined by the pallet.

- `type OnBonded: Happened<(Self::AccountId, Balance)>;`: This line defines a type alias for the event that is triggered when tokens are bonded. It specifies that the event type must implement the `Happened` trait with a tuple containing the account ID and balance.

- `type OnUnbonded: Happened<(Self::AccountId, Balance)>;`: Similar to the previous line, this defines a type alias for the event that is triggered when tokens are unbonded.

- `type OnUnstakeFee: OnUnbalanced<NegativeImbalanceOf<Self>>;`: This line defines a type alias for the event that is triggered when there is an unbalance due to unstaking fees. It specifies that the event type must implement the `OnUnbalanced` trait with the negative imbalance type defined by the pallet.

- `#[pallet::constant]`: This attribute macro is used to define constants for the pallet.

- `type MinBond: Get<Balance>;`: This line defines a constant for the minimum bond amount.

- `type UnbondingPeriod: Get<BlockNumberFor<Self>>;`: This line defines a constant for the unbonding period.

- `type MaxUnbondingChunks: Get<u32>;`: This line defines a constant for the maximum number of unbonding chunks.

- `type LockIdentifier: Get<LockIdentifier>;`: This line defines a constant for the lock identifier.

- `type WeightInfo: WeightInfo;`: This line defines a type alias for the weight information of the pallet's extrinsics.

- `pub type BondingLedgerOf<T> = bonding::BondingLedgerOf<Pallet<T>>;`: This line defines a type alias for the bonding ledger.

- `type NegativeImbalanceOf<T> = <<T as Config>::Currency as Currency<<T as frame_system::Config>::AccountId>>::NegativeImbalance;`: This line defines a type alias for the negative imbalance of the currency.

- `#[pallet::error]`: This attribute macro is used to define the error types that can be returned from the pallet's functions [9].

- `pub enum Error<T> {`: This line starts the definition of an enumeration named `Error` that can be used to represent various error conditions.

- `#[pallet::event]`: This attribute macro is used to define the event types for the pallet [9].

- `#[pallet::generate_deposit(pub(crate) fn deposit_event)]`: This line generates a function named `deposit_event` that can be used to deposit events into the runtime's event storage.

- `pub enum Event<T: Config> {`: This line starts the definition of an enumeration named `Event` that represents the events that can be emitted by the pallet.

- `#[pallet::storage]`: This attribute macro is used to define the storage items for the pallet [1].

- `pub type Ledger<T: Config> = StorageMap<_, Twox64Concat, T::AccountId, BondingLedgerOf<T>, OptionQuery>;`: This line defines a storage map named `Ledger` that maps account IDs to bonding ledgers.

- `#[pallet::pallet]`: This attribute macro marks the following struct as the pallet struct. It's a mandatory part of defining a pallet in Substrate [1].

- `#[pallet::without_storage_info]`: This attribute macro indicates that the pallet does not provide storage information.

- `pub struct Pallet<T>(_);`: This line defines a struct named `Pallet` that is parameterized by a generic type `T`.

- `#[pallet::hooks]`: This attribute macro is used to define optional hooks for the pallet. Hooks allow the pallet to execute logic at specific points in the block making process [9].

- `#[pallet::call]`: This attribute macro is used to define the callable functions for the pallet. These functions can be invoked by transactions [1].

- `impl<T: Config> Pallet<T> {`: This line starts the implementation block for the `Pallet` struct. It specifies that the implementation is for the `Pallet` struct parameterized by a type `T` that implements the `Config` trait.

- The following lines define various callable functions for the pallet, such as `bond`, `unbond`, `unbond_instant`, `rebond`, and `withdraw_unbonded`. Each function is annotated with `#[pallet::call_index]` to specify its index in the callable function table, and `#[pallet::weight]` to specify its computational weight.

This code snippet is a comprehensive example of defining a pallet in Substrate, including its configuration, storage, events, errors, and callable functions.

#### BondingController's Implementation for the pallet

The `BondingController` trait for the `Pallet` struct is parameterized by a type `T` that implements the `Config` trait. This means that the `Pallet` struct is providing an implementation of the `BondingController` trait.

- `type MinBond = T::MinBond;`: This specifies that the minimum bond amount is determined by the `MinBond` type defined in the `Config` trait.

- `type MaxUnbondingChunks = T::MaxUnbondingChunks;`: Similarly, this is specifying that the maximum number of unbonding chunks is determined by the `MaxUnbondingChunks` type defined in the `Config` trait.

- `type Moment = BlockNumberFor<T>;`: This line is specifying that the moment type is the block number for the runtime configuration `T`.

- `type AccountId = T::AccountId;`: This line defines an associated type `AccountId` for the `BondingController` implementation, specifying that the account ID type is determined by the `AccountId` type defined in the `Config` trait.

- `type Ledger = Ledger<T>;`: This line is specifying that the ledger type is the `Ledger` defined in the pallet with the runtime configuration `T`.

- `fn available_balance(who: &Self::AccountId, ledger: &BondingLedgerOf<T>) -> Balance {`: This function calculates the available balance for a given account, taking into account the account's free balance and the total amount locked in the ledger.

- `fn apply_ledger(who: &Self::AccountId, ledger: &BondingLedgerOf<T>) -> DispatchResult {`: This function applies the ledger to the account, either removing the lock if the ledger is empty or setting a new lock with the total amount in the ledger.

- `fn convert_error(err: bonding::Error) -> DispatchError {`: This function converts errors from the bonding module into dispatch errors that can be returned by the pallet's functions.

This implementation block is crucial for defining how the pallet behaves when it is used with a specific configuration. It specifies the types and behaviors related to bonding tokens, such as the minimum bond amount, the maximum number of unbonding chunks, and how to calculate and apply the ledger to an account. This is a common pattern in Substrate pallets, allowing for modular and configurable blockchain logic [11][11].

```markdown
#### Citations

[1] https://docs.rs/frame-support/latest/frame_support/attr.pallet.html
[2] https://stackoverflow.com/questions/68885677/substrate-rust-unresolved-imports-node-template-runtime
[3] https://storage.googleapis.com/mangata-docs-node/frame_support/pallet_macros/index.html
[4] https://subdaily.io/dev/learn-to-build-substrate-pallets-from-scratch-dissecting-the-frame-pallet/
[5] https://github.com/paritytech/substrate/discussions/7788
[6] https://substrate.stackexchange.com/questions/6949/the-trait-bound-t-as-frame-systemconfigaccountid-palletconfig-is-not
[7] https://docs.substrate.io/reference/frame-pallets/
[8] https://docs.substrate.io/learn/runtime-development/
[9] https://docs.substrate.io/reference/frame-macros/
[10] https://substrate.stackexchange.com/questions/7804/how-to-conditionally-compile-functions-in-palletcall
[11] https://crates.parity.io/frame_support/attr.pallet.html
[12] https://docs.rs/frame-support/latest/frame_support/pallet_macros/index.html
[13] https://docs.substrate.io/tutorials/build-application-logic/use-macros-in-a-custom-pallet/
[14] https://www.americansurplus.com/used-pallet-rack-for-sale-in-illinois/
[15] https://www.chicagopalletracking.com/
[11] https://docs.substrate.io/build/custom-pallets/
```

</details>

### Earning Module Testing Suite Setup

- Git clone `https://github.com/code-423n4/2024-03-acala`
- cd into the `Acala/src/modules/earning/src` directory
  - Build: `cargo build`
  - Test: `cargo test`

## Rewards Module

### [Rewards Module Framework](https://github.com/code-423n4/2024-03-acala/tree/main/src/orml/rewards/src)

The Rewards Module serves as a fundamental component within the protocol, it contains the base methods for calculating and distributing rewards. It is used by the incentives module to calculate and distribute rewards to users.

The `PoolInfo` struct, encapsulates essential details such as total shares and a `BTreeMap` that records the total and withdrawn rewards for each currency. This structure is crucial for maintaining a detailed account of the rewards landscape within the pool, enabling transparent and equitable reward distribution.

Key functionalities of the contract include the ability to accumulate rewards, add or remove shares, and claim rewards, pivotal for the seamless operation and interaction with the reward pools. For instance, the `accumulate_reward` function allows for the periodic addition of rewards to a pool, thereby adjusting the total rewards available for distribution:

```rust
pub fn accumulate_reward(
    pool: &T::PoolId,
    reward_currency: T::CurrencyId,
    reward_increment: T::Balance,
) -> DispatchResult { ... }
```

Participants interact through functions like `add_share` and `remove_share`, which enable them to adjust their stake in the pool and consequently, their entitlement to the pool's rewards. These functions reflect the dynamic nature of participation and reward distribution within the DeFi platform:

```rust
pub fn add_share(who: &T::AccountId, pool: &T::PoolId, add_amount: T::Share) { ... }

pub fn remove_share(who: &T::AccountId, pool: &T::PoolId, remove_amount: T::Share) { ... }
```

The process of claiming rewards is streamlined through functions such as `claim_rewards`, which calculates the amount due to each participant and also facilitates the distribution of these rewards, ensuring participants receive their fair share relative to their contribution to the pool's total shares:

```rust
pub fn claim_rewards(who: &T::AccountId, pool: &T::PoolId) { ... }
```

To accommodate different scenarios, the contract also includes functionality for transferring shares between accounts, updating reward amounts, and adjusting share allocations. These features allow for a high degree of flexibility and adaptability in managing reward distributions, catering to the evolving needs of the platform's participants.

### Line by Line Deep Dive into Rewards Module's Code Snippets For Non-Rust Accustommed Readers

<details>
  <summary>Click To See Line by Line Deep Dive</summary>

1. `#![cfg_attr(not(feature = "std"), no_std)]`: This line is a conditional compilation attribute that disables the standard library (`std`) if the `std` feature is not enabled. This is common in Substrate pallets to ensure compatibility with both Wasm and native targets.

2. `#![allow(clippy::unused_unit)]`: This line allows the code to compile without warnings for unused unit values, which are often used in Rust for functions that do not return a meaningful value.

3. `#![allow(clippy::too_many_arguments)]`: This line suppresses warnings from the Clippy linter about functions having too many arguments, which is a common issue in complex systems like blockchain pallets.

4. `mod mock; mod tests;`: These lines declare two modules, `mock` and `tests`, which are likely used for testing the pallet's functionality.

5. `use frame_support::pallet_prelude::*;`: This line imports various types and traits from the `frame_support` crate, which is a core part of the Substrate framework providing essential utilities for developing pallets.

6. `use orml_traits::RewardHandler;`: This line imports the `RewardHandler` trait from the `orml_traits` crate, which is likely used to handle reward distribution logic.

7. `use parity_scale_codec::{FullCodec, HasCompact};`: This line imports the `FullCodec` and `HasCompact` traits from the `parity_scale_codec` crate, which are used for serialization and deserialization of data.

8. `use scale_info::TypeInfo;`: This line imports the `TypeInfo` trait from the `scale_info` crate, which is used for runtime type information.

9. `use sp_core::U256;`: This line imports the `U256` type from the `sp_core` crate, which is a 256-bit unsigned integer type used for arithmetic operations.

10. `use sp_runtime::{...};`: This line imports various traits and types from the `sp_runtime` crate, which provides runtime functionality for Substrate blockchains.

11. `use sp_std::{borrow::ToOwned, collections::btree_map::BTreeMap, fmt::Debug, prelude::*};`: This line imports various types and traits from the `sp_std` crate, which is a standard library for Substrate pallets.

12. `/// The Reward Pool Info.`: This line is a documentation comment that describes the following struct.

13. `pub struct PoolInfo<Share: HasCompact, Balance: HasCompact, CurrencyId: Ord> {...}`: This line defines a struct `PoolInfo` that represents information about a reward pool. It includes fields for total shares, and a map of rewards by currency ID.

14. `impl<Share, Balance, CurrencyId> Default for PoolInfo<Share, Balance, CurrencyId> {...}`: This block implements the `Default` trait for `PoolInfo`, providing a default value for instances of the struct.

15. `pub use module::*;`: This line re-exports everything from the `module` module, making it accessible from outside the module.

16. `#[frame_support::pallet]`: This attribute marks the following module as a pallet, which is a component of the Substrate runtime.

17. `pub mod module {...}`: This line declares a module named `module`, which contains the pallet's logic.

18. `#[pallet::config]`: This attribute marks the following trait as the configuration trait for the pallet, defining the types and traits it requires.

19. `pub trait Config: frame_system::Config {...}`: This line defines a trait `Config` that extends the `frame_system::Config` trait, specifying the types and traits required by the pallet.

20. `type WithdrawnRewards<T> = BTreeMap<<T as Config>::CurrencyId, <T as Config>::Balance>;`: This line defines a type alias for a map of withdrawn rewards by currency ID.

21. `#[pallet::error]`: This attribute marks the following enum as the pallet's error type, defining possible errors that can occur.

22. `pub enum Error<T> {...}`: This line defines an enum `Error` that lists possible errors for the pallet.

23. `#[pallet::storage]`: This attribute marks the following storage items as part of the pallet's storage, defining how data is stored on-chain.

24. `#[pallet::getter(fn pool_infos)]`: This attribute defines a getter function for accessing the `PoolInfos` storage item.

25. `#[pallet::storage]`: This attribute marks the following storage item as part of the pallet's storage.

26. `#[pallet::getter(fn shares_and_withdrawn_rewards)]`: This attribute defines a getter function for accessing the `SharesAndWithdrawnRewards` storage item.

27. `#[pallet::pallet]`: This attribute marks the following struct as the pallet's entry point, defining the pallet's metadata and configuration.

28. `#[pallet::without_storage_info]`: This attribute indicates that the pallet does not provide detailed storage information, which is common for pallets that manage complex storage structures.

29. `pub struct Pallet<T>(_);`: This line defines a struct `Pallet` that represents the pallet itself, with a generic parameter `T` that must implement the `Config` trait.

30. `impl<T: Config> Pallet<T> {...}`: This block implements methods for the `Pallet` struct, providing functionality for managing reward pools, such as accumulating rewards, adding and removing shares, and claiming rewards.

Citations:
[1] https://docs.substrate.io/reference/frame-pallets/
[2] https://substrate.stackexchange.com/questions/2784/how-to-get-a-percent-portion-of-a-balance
[3] https://docs.substrate.io/tutorials/build-application-logic/add-a-pallet/
[4] https://substrate-developer-hub.github.io/substrate-how-to-guides/docs/basics/basic-pallet-integration/
[5] https://subdaily.io/dev/learn-to-build-substrate-pallets-from-scratch-dissecting-the-frame-pallet/
[6] https://docs.substrate.io/tutorials/collectibles-workshop/03-create-pallet/
[7] https://substrate-developer-hub.github.io/substrate-how-to-guides/docs/pallet-design/add-contracts-pallet/
[8] https://blog.knoldus.com/pallets-in-substrate-and-using-them-in-runtime/
[9] https://substrate-starterkit.dev/
[10] https://blog.logrocket.com/custom-blockchain-implementation-rust-substrate/
[11] https://substrate-developer-hub.github.io/substrate-how-to-guides/docs/tutorials/Kitties/Part%201/basic-setup/
[12] https://github.com/topics/substrate-pallet
[13] https://docs.substrate.io/build/custom-pallets/
[14] https://marketplace.substrate.io/pallets/pallet-balances/
[15] https://marketplace.substrate.io/pallets/pallet-assets/

</details>

### Rewards Module's Testing Suite Setup

- Git clone `https://github.com/code-423n4/2024-03-acala`
- cd into the `Acala/src/orml/rewards/src` directory
  - Build: `cargo build`
  - Test: `cargo test`

## Incentives Module

### [Incentives Module Framework](https://github.com/code-423n4/2024-03-acala/tree/main/src/modules/incentives/src/lib.rs)

The Incentives module in Acala's architecture plays a pivotal role in managing the distribution of rewards across different liquidity pools within the platform. Scheduled on a periodic basis, this module ensures that incentives are systematically accumulated for each pool, with the funding sourced directly from the designated RewardsSource. This setup allows for a structured reward distribution where each pool is capable of receiving multiple types of incentives, enhancing the reward opportunities for participants.

Users engaged with these pools can claim their accrued rewards at any given time, offering flexibility and immediate access to their earnings. The module is designed with a configurable deduction rate mechanism, which applies to the rewards at the time of claiming, ensuring a fair and balanced distribution system. Additionally, when users contribute liquidity to a pool, they are granted shares that represent their stake in the pool. This share mechanism is intricately linked with the reward system, where withdrawn rewards are meticulously adjusted. This adjustment ensures that new users begin their journey in the pool without any pre-accumulated rewards, maintaining equity among all participants.

In the realm of reward accumulation, the module stands out for its systematic approach. It periodically (as determined by `AccumulatePeriod`) aggregates fixed amounts of rewards for each pool based on predefined incentives. The rewards are sourced from designated accounts, necessitating sufficient funding in these accounts before initiating any incentive plan. This meticulous process ensures a consistent and fair distribution of rewards to participants.

One of the core functionalities of the Incentives Module is its support for staking and unstaking liquidity provider (LP) tokens. Participants in the DEX can stake their LP tokens to gain shares in the liquidity pool, which in turn entitles them to a portion of the pool's rewards. The process of staking and unstaking is facilitated through user-friendly functions that ensure seamless interaction with the platform:

```rust
pub fn deposit_dex_share(origin: OriginFor<T>, lp_currency_id: CurrencyId, amount: Balance) -> DispatchResult {
    ...
    Self::do_deposit_dex_share(&who, lp_currency_id, amount);
}

pub fn withdraw_dex_share(origin: OriginFor<T>, lp_currency_id: CurrencyId, amount: Balance) -> DispatchResult {
    ...
    Self::do_withdraw_dex_share(&who, lp_currency_id, amount);
}
```

Claiming rewards is another pivotal feature. Participants can claim their accumulated rewards from various pools, thereby realizing their gains from staking or other activities within the platform. The module's architecture also facilitates administrative control over the incentive mechanisms, allowing for the dynamic adjustment of incentive reward amounts and deduction rates:

```rust
pub fn claim_rewards(origin: OriginFor<T>, pool_id: PoolId) -> DispatchResult {
    ...
    Self::do_claim_rewards(who, pool_id);
}

pub fn update_incentive_rewards(origin: OriginFor<T>, updates: Vec<(PoolId, Vec<(CurrencyId, Balance)>)>) -> DispatchResult {
    ...
    for (pool_id, update_list) in updates {
        ...
        IncentiveRewardAmounts::<T>::mutate_exists(pool_id, currency_id, |maybe_amount| { ... });
    }
}
```

Administratively, the module empowers platform operators to fine-tune the rewards ecosystem to reflect changing dynamics and strategic priorities. Through functions like `update_incentive_rewards` and `update_claim_reward_deduction_rates`, administrators can modify the reward structures and parameters to optimize the incentive distribution process and align it with the platform's goals.

### Line by Line Deep Dive into Incentives Module's Code Snippets For Non-Rust Accustommed Readers

<details>
  <summary>Click To See Line by Line Deep Dive</summary>

```rust
#![cfg_attr(not(feature = "std"), no_std)]
```

- This line conditionally compiles the module for a `no_std` environment, typical in blockchain runtime development, to ensure compatibility with the limited execution environment.

```rust
#![allow(clippy::unused_unit)]
#![allow(clippy::upper_case_acronyms)]
```

- These lines disable specific Clippy lints for the whole module. Clippy is a Rust linting tool to help ensure code quality.

```rust
use frame_support::{pallet_prelude::*, transactional, PalletId};
use frame_system::pallet_prelude::*;
```

- Imports types and macros from `frame_support` and `frame_system`, which are part of Substrate's framework, providing functionalities needed to build a pallet.

```rust
use module_support::{DEXIncentives, EmergencyShutdown, FractionalRate, IncentivesManager, PoolId, Rate};
```

- Imports specific traits and types that are likely part of the local module or a supporting module within the project.

```rust
use orml_traits::{Happened, MultiCurrency, RewardHandler};
```

- Imports from `orml_traits`, which provides a set of common traits used in Open Runtime Module Library (ORML) for DeFi applications.

```rust
use primitives::{Amount, Balance, CurrencyId};
```

- Imports basic types like `Amount`, `Balance`, and `CurrencyId` used to represent monetary values and currency identifiers.

```rust
use sp_runtime::{
    traits::{AccountIdConversion, UniqueSaturatedInto, Zero},
    DispatchResult, FixedPointNumber,
};
```

- Imports from `sp_runtime`, which provides core runtime functionalities including mathematical traits and result types.

```rust
use sp_std::{collections::btree_map::BTreeMap, prelude::*};
```

- Imports standard library features and a B-tree map for efficient storage and retrieval.

```rust
#[frame_support::pallet]
pub mod module {
    use super::*;
```

- Defines a new pallet module, using the Substrate pallet macro to simplify pallet construction. `use super::*;` imports all the items from the outer scope.

```rust
    #[pallet::config]
    pub trait Config: frame_system::Config + orml_rewards::Config {
        ...
    }
```

- The `Config` trait defines the configuration interface for the pallet, inheriting from `frame_system::Config` and other configuration traits necessary for the pallet's operation.

```rust
    #[pallet::error]
    pub enum Error<T> {
        NotEnough,
        InvalidCurrencyId,
        ...
    }
```

- Defines errors that the pallet might encounter during its execution.

```rust
    #[pallet::event]
    pub enum Event<T: Config> {
        DepositDexShare { ... },
        WithdrawDexShare { ... },
        ...
    }
```

- Declares events that the pallet can emit to inform external entities of the changes within the pallet.

```rust
    #[pallet::storage]
    pub type IncentiveRewardAmounts<T: Config> = ...
```

- Declares storage items for the pallet. In this case, `IncentiveRewardAmounts` stores the reward amounts for each pool and currency.

```rust
    #[pallet::pallet]
    #[pallet::without_storage_info]
    pub struct Pallet<T>(_);
```

- Defines the main pallet structure. The `without_storage_info` attribute opts out of generating storage metadata for this pallet, which can be a necessity for runtime optimization.

```rust
    #[pallet::hooks]
    impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
        ...
    }
```

- Implements the `Hooks` trait for the pallet to allow it to perform actions at specific stages in the block lifecycle, such as initialization.

```rust
    #[pallet::call]
    impl<T: Config> Pallet<T> {
        ...
    }
```

- Within the `call` implementation, functions are defined that can be called by external entities or users to interact with the pallet's functionality.

```rust
impl<T: Config> Pallet<T> {
    pub fn account_id() -> T::AccountId {
        ...
    }
    ...
}
```

- Additional implementation blocks for the pallet can define helper functions, internal logic, and interfaces that are not directly exposed as callable functions.

```rust
impl<T: Config> DEXIncentives<T::AccountId, CurrencyId, Balance> for Pallet<T> {
    ...
}
```

- Specific implementations of traits for the pallet, providing

functionalities like DEX incentives management in this context.

```rust
impl<T: Config> RewardHandler<T::AccountId, CurrencyId> for Pallet<T> {
    ...
}
```

- Another trait implementation that allows the pallet to handle reward-related functionalities.

```rust
pub struct OnEarningBonded<T>(sp_std::marker::PhantomData<T>);
```

- A struct used for type safety and logic encapsulation, often in the context of event handling or specific callbacks.

Each of these sections introduces specific functionalities and integrations within the Substrate pallet, leveraging Rust's type system and Substrate's framework to build a comprehensive DeFi incentives module.

</details>

### Incentives Module Testing Suite Setup

- Git clone `https://github.com/code-423n4/2024-03-acala/`
- cd into the `Acala/src/modules/incentives/src` directory
  - Build: `cargo build`
  - Test: `cargo test`

## Business Logic

> This section provides the protocol's business logic in summary

### Incentives Module

The business logic of the Incentive Module is pretty simple. It just facilitates reward accumulation based on pre-defined incentive amounts and distributes them to users based on their pool shares.

### Earning Module

The Earning Module's business logic centers around managing user bonds and enables the following actions:

- **Bonding:** Users can lock up tokens to become bonded.
- **Unbonding:** Users can initiate unbonding of their tokens, starting a waiting period before full withdrawal.
- **Instant Unbonding:** Users can unbond tokens immediately by paying a fee.
- **Rebonden:** Users can rebond tokens previously set for unbonding.
- **Withdrawing Unbonded:** Users can withdraw tokens that have completed the unbonding period.

The logic appears sound, but consideration should be given to the following aspects:

- **Minimum Bond Threshold:** The contract enforces a minimum bond amount. This should be carefully chosen to discourage users from exploiting the system with negligible contributions while still allowing small-scale participation.
- **Unbonding Period:** The waiting period for unbonding helps maintain liquidity within the system. The chosen duration should balance user flexibility with potential risks of sudden outflows.
- **Instant Unbonding Fee:** The fee incentivizes users to participate in the regular unbonding process and helps mitigate potential liquidity risks from mass instant unbonding. The fee structure should be designed to achieve this balance.

### Rewards Module

The business logic of the reward pool contract appears sound. It facilitates accumulation of rewards based on pre-defined contribution amounts (represented by shares) and distributes them proportionally to users based on their share holdings. However, careful consideration should be given to the following aspects:

- **Transparency:** The process for determining reward allocation should be transparent and well-defined.
- **Fairness:** The reward distribution mechanism should be fair and unbiased, ensuring all users with shares participate proportionally.

## Centralization Risks

- The Incentives module manager can withdraw anybody's dex shares without their permission via the incentive manager trait for the pallet, seen in these snippets https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#L553, causing a user not to be in control of when they want withdraw their shares and when they do not.
- The Incentives module manager can claim anybody's dex rewards without their permission via the incentive manager trait for the pallet, seen in these snippets https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/modules/incentives/src/lib.rs#557 causing a user not to be in control of when they want to claim rewards or not.
- In the Earning module, the admin decides the amount of the `minbond`, if this is very low, it could cause an account that binds not to have any tokens on chain say, it's lower than the chain's existential deposit.
- Same idea for the fee, in the Earning module, the admin could pass in a very low percentage, or say the minbond is very low both above the Existential Deposit, however whenever the fee is calculated it's below the Existential Deposit, then when this fee gets transfered via `unbond_instant()` it could get missing, since the value is less than ED that's been sent, and if the receiver does not have any tokens on chain these tokesn might just be lost, due to the rules arounf Acala/Karura: https://wiki.acala.network/get-started/acala-network/acala-account#existential-deposit
- Admins could unnecessarily set the unbonding period to be very long forcing the users to have to pay the instant unbond fee so as to use `unbond_instant`

## Systemic Risks

- In the Incentives module, protocol utilizes the `UpdateOrigin` as the origin that might update incentive related params, but this value, i.e `UpdateOrigin` is being set as a constant in the config of the allet, emaning it can't be changed and thing happening to the account leads to a complete DOS of the ability to update the incentive params.
- Bugs in the smart contract code could lead to unintended consequences, such as incorrect reward calculations or unintended token transfers, for example a user is allowed to remove their shares partially, but this logic is flawed cause while removing their shares the `claim_reward()` function assumes that the user is rmoving all their shares.
- Protocol supports an Emergency shutdown mode, whereas this is very important in some cases it's very risky as tehre is a need to always ensure the parties having access to this are not misusing it.
- The incentive module relies on a specific origin (`T::UpdateOrigin`) to update incentive rewards and claim reward deduction rates. Compromise of just this account could lead to unauthorized modifications.
- External dependencies from inherited contracts/pallets, etc.
- The Incentive Module, contains an `OnUpdateLoan` struct where there is a `happened()` function, this function in shorts checks the adjustments and then adjusts as needed, but this function potentially leads to unnecessary computational cost, taking into accout that it uses the `is_positive()` to know if to add or remove shares, however this function returns false for when the adjustment is `0` and as such wouldlead to an unnecessary attempt at removing the shares.

## Recommendations

- Consider setting a maximum accepted `unbonding` duration, this way users are sure that an admin can never indefinitely grief them from their tokens after unbonding.
- Enhance documentation quality, currently the code's natspec and protocol's provided docs, do not always align.
- Thorough more tests there should be an implemented to verify the behavior of individual contract functions and even integral functionalities.
- Regular security audits by reputable firms are essential to identify and address potential vulnerabilities.
- Onboard more developers, cause multiple eyes checking out a protocol can be invaluable. More contributors can significantly reduce potential risks and oversights. Evidence of where this could come in handy can be gleaned from codebase typos. Drawing from the [broken windows theory](https://en.wikipedia.org/wiki/Broken_windows_theory), such lapses may signify underlying code imperfections.
- Improve protocol's testability, one thing to note about tests is that _there's always room for further refinement and improvement._, a few out of the box ideas need to also be attached to test cases.
- To support the above, even fuzz and invariant tests could be further incorporated to help secure protocol.
- There seems to be a need to improve the naming conventions in some areas of protocol to ease user/developer experience while trying to use/understand protocol.

## Security Researcher Logistics

Our attempt on reviewing the Codebase spanned around 50 hours distributed over 8 days:

- 4 hours dedicated to writing this analysis.
- 5 hours (first day) spent exploring the whole system to grasp the foundations before diving into it line by line
- 2 hours were allocated for discussions with sponsors on the private discord group regarding potential vulnerabilities.
- 8 hours over the course of the 8 days _(~60 minutes per day)_ was spent hovering around the Acala and Karura docs, matching this up with substrate docs to get a better grounded knowledge of implemented functionalities.
- The remaining time was spent on finding issues and writing the report for each of them.

## Conclusion

The codebase was a very great learning experience, albeit very hard to review since this my second attempted rust contest, though it was a pretty hard nut to crack, nonetheless during my review, I uncovered a few issues within the protocol and they need to be fixed. Recommended steps should be taken to protect the protocol from potential attacks. Timely audits and codebase cleanup for mitigations should also be conducted to keep the codebase fresh and up to date with evolving security times.

## Resources

- [Audit Page](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/)

- [Documentation](https://guide.acalaapps.wiki/staking/aca-staking)

- [Website](https://acala.network)

- [Acala's Wiki](https://wiki.acala.network/get-started/acala-network/)

- [Previous Audits](https://github.com/AcalaNetwork/Acala/tree/master/audit)

- [Karura's Wiki](https://wiki.acala.network/get-started/get-started/karura-launch-phases)

- [Karura Block Explorer](http://karura.subscan.io)

- [Main Repo](http://github.com/AcalaNetwork/Acala)

- [Acala Block Explorer](http://acala.subscan.io)

- [Guide For Detailed Instructions To Setup Dev env.](https://docs.substrate.io/install/)


### Time spent:
50 hours