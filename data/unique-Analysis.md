# Introduction

Acala is the decentralized financial hub of Polkadot that makes it fast and easy to use or build financial applications, improving trading efficiency and saving valuable time. The platform offers a suite of DeFi primitives including a stablecoin, DEX, and staking deritatives.

# he approach I followed when reviewing the code

Accordingly, I analyzed and audited the subject in the following steps;

**1. Initial Scope and Documentation Review**:

&nbsp;          Thoroughly went through the Contest Readme File and project documentation to understand the protocol's objectives and functionalities.

**2. High-level Architecture Understanding**:

&nbsp;       Performed an initial architecture review of the codebase by going through all files without going into function details.

3. **Code Review**:

&nbsp;          Conducted a line-by-line code review focusing on understanding code functionalities.

- Understand Codebase Functionalities: Began by going through the functionalities of the codebase to gain a clear understanding of its operations.
- Identify Access Control Issues: Thoroughly examined the codebase for any access control problems that might allow unauthorized users to execute critical functions.
- Evaluate Function Execution Order: Checked random sequence of function executions to ensure the protocol's logic cannot be disrupted by changing the order of calls.

**4. Report Writing**:

&nbsp;           Write Report by compiling all the insights I gained throughout the line by line code review.

# System Overview

## Scope

### Overview of Each Contract:

Scope

- <ins>Incentives</ins>: Each period, pools will accumulate incentives and rewards are distributed to them from RewardsSource. Each pool can receive multiple incentives. Users can claim rewards from the pool at any time. Deduction rate is configurable and is applied to the rewards when they are claimed. When a user adds liquidity to the pool, they receive shares, and withdrawn rewards are adjusted accordingly so the new user will start with no reward.
    
- <ins>Rewards</ins>: This module contains the base methods for calculating and distributing rewards. It is used by the incentives module to calculate and distribute rewards to users.
    
- <ins>Earning</ins>: Users will bond/lock their tokens for a period of time. Unbonding/unlocking is possible by paying a fee/penalty, or requesting to unbond/unlock and waiting for the unbonding period to finish before you can withdraw. The module implements a set of hooks that can be used by other modules (i.e. Incentives) to implement staking.
    

# Architecture Overview

## Incentives Module:

- Manages incentives and rewards for different pools like Dex and Loans.
- Provides flexibility in configuring reward parameters.
- Integrates with other Acala modules to facilitate incentives distribution.

## Rewards Distribution System:

- Handles reward accumulation and distribution across various pools.
- Supports multiple reward currencies and efficient share management.
- Ensures fair distribution of rewards based on users' shares in each pool.

## Earning Module:

- Facilitates token bonding, unbonding, and earning functionalities.
- Implements mechanisms for stake management and reward distribution.
- Integrates with Acala's broader ecosystem to support staking and earning activities.

&nbsp;

## Codebase Quality

Overall, I consider the quality of the Acala codebase to be Good. The code appears to be mature and well-developed. I have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories | Comments |
| --- | --- |
| **Code Maintainability and Reliability** | The Acala Protocol contracts demonstrates good maintainability through modular structure, consistent naming . It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space. |
| ****Testing**** | The audit scope of the contracts to be audited is 85% and it should be aimed to be 100%. |
| **Code Comments** | Some functions   in the code are not well-documented, making it harder for developers to understand their purpose and behavior. Adding appropriate comments and documentation can improve code maintainability and understandability. |
| ****Code Structure and Formatting**** | The codebase contracts are well-structured and formatted when analyzing the code structure and formatting of the Acala protocol overall, I evaluate the organization of modules, the clarity of function and variable names, the consistency in coding style, the presence of documentation, adherence to coding standards, and the overall readability and maintainability of the codebase. |
| **Documentation** | The documentation of Acala was very clear and helpful. It addressed my questions thoroughly and provided valuable insights, and it was very easy to find the details about each contract. |
| **Error** | The codebase includes assertions for validating conditions, ensuring that certain prerequisites are met before executing critical operations. Proper error handling mechanisms contribute to the reliability of the contract by preventing unexpected behaviors and vulnerabilities. |

# Systemic Risks

## Incentives Module:

- Complexity in configuring and managing incentives parameters could lead to misconfigurations.
- Potential security vulnerabilities in reward distribution logic may result in unfair rewards allocation.

## Rewards Distribution System:

- Risk of bugs or vulnerabilities in the reward accumulation and distribution algorithms.
- Possibility of economic exploits or attacks affecting the fairness of reward distribution.

## Earning Module:

- Vulnerabilities in bonding, unbonding, or reward withdrawal mechanisms could lead to token loss or manipulation.
- Risks associated with the economic incentives model, such as potential for gaming or manipulation by users.

## Conclusion

Overall Acala came up with a very strong solution with some weak points in commenting and less docs . But the approach used for the core working of protocol is really solid and up to the industry standard.

### Time spent:

35 hours

&nbsp;

### Time spent:
35 hours