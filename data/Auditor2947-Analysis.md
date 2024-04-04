
## Analysis Issue Report 


### [File-1]  src/modules/incentives/src/lib.rs
https://github.com/code-423n4/2024-03-acala/blob/main/src/modules/incentives/src/lib.rs


#### Admin Abuse Risks:

The code does not directly address admin abuse risks. However, depending on the implementation of the `Currency` and `orml_rewards` pallets, there may be certain administrative privileges associated with managing rewards and DEX shares. These privileges could potentially be abused if proper access controls and governance mechanisms are not in place.


#### Systemic Risks:

The code does not explicitly handle systemic risks. Systemic risks refer to risks that can affect the entire system, such as vulnerabilities in the underlying blockchain infrastructure or dependencies on external services. It's important to consider these risks outside the scope of the provided code.


#### Technical Risks:

The code primarily focuses on the technical aspects of reward management and DEX share handling. Technical risks could arise from bugs, vulnerabilities, or inefficiencies in the implementation of the `Currency`, `orml_rewards`, and other related pallets. These risks could potentially lead to incorrect reward calculations, unauthorized transfers, or other system malfunctions.


#### Integration Risks:

The code appears to integrate with the `Currency` and `orml_rewards` pallets, which suggests dependencies on these modules. Integration risks may arise if there are compatibility issues or conflicts with other modules or if the external pallets undergo significant changes that impact the functionality of the code. It's important to ensure that the code remains compatible and well-integrated with the required dependencies.


#### Non-Standard Token Risks:

The code does not directly handle non-standard tokens. However, depending on the implementation of the `Currency` pallet and the specific tokens used, there may be risks associated with non-standard tokens. These risks could include potential vulnerabilities, lack of liquidity, or regulatory concerns. It's important to evaluate the risks associated with the specific tokens used within the system.


### [File-2]   src/orml/rewards/src/lib.rs
https://github.com/code-423n4/2024-03-acala/blob/main/src/orml/rewards/src/lib.rs


#### Admin Abuse Risks:

The code does not explicitly address admin abuse risks. It's important to ensure that appropriate access controls and permissions are implemented to prevent unauthorized access or misuse of administrative privileges. Without these controls, there is a risk of unauthorized modifications to the pool's data, such as `adjusting` shares or `rewards`.


#### Systemic Risks:

The code does not seem to address systemic risks explicitly. Systemic risks refer to potential vulnerabilities or weaknesses in the overall system architecture or design. Without a thorough examination of the entire system, it's challenging to identify systemic risks that may be present in this code snippet alone.


#### Technical Risks:

The code may be prone to technical risks such as integer `overflow` or `underflow` if not handled properly. Operations involving `share additions` or `removals`, `reward` calculations, and other numeric calculations need to be carefully implemented to avoid potential vulnerabilities.

There may be security risks associated with the storage and handling of sensitive data, such as account balances and reward information. It's crucial to ensure that appropriate security measures, such as encryption and secure storage, are in place to protect this data from unauthorized access or tampering.


#### Integration Risks:

The code provided does not reveal any specific integration risks. Integration risks typically arise when combining different components or systems, and without knowledge of the broader context or integration points, it's challenging to identify integration risks in this code snippet alone.


#### Non-Standard Token Risks:

The code does not explicitly mention any non-standard token risks. However, if the code interacts with non-standard tokens or implements custom functionalities related to tokens, there may be risks associated with token standards compliance, vulnerabilities in token contracts, or improper handling of tokens. These risks need to be thoroughly assessed and addressed.


### [File-3]    src/modules/earning/src/lib.rs
https://github.com/code-423n4/2024-03-acala/blob/main/src/modules/earning/src/lib.rs



#### Admin Abuse Risks:

The smart contract does not have explicit admin functionality or privileges. It relies on the origin (`sender`) of the transaction to determine the account performing the action. The risk of admin abuse would depend on the context in which this smart contract is used and any additional access control mechanisms implemented outside the scope of the contract itself.

#### Systemic Risks:

The smart contract does not appear to have any systemic risks inherent in its design. However, it relies on the underlying system modules and traits (`frame_support`, `frame_system`, `orml_traits`, etc.) to provide necessary functionality and enforce security guarantees. The security of these dependencies could potentially impact the overall security of this smart contract.

#### Technical Risks:

The smart contract implements a bonding mechanism that allows users to `bond`, `unbond`, and `rebond` tokens. It also supports instant `unbonding` with a fee and withdrawal of unbonded tokens. The technical risks associated with this contract include potential vulnerabilities in the implementation of these bonding and unbonding mechanisms, as well as issues related to the integration with external dependencies such as the `currency` module (`frame_support::traits::Currency`).

#### Integration Risks:

The smart contract integrates with various modules and traits from the `frame_support` and `frame_system` libraries, as well as the `orml_traits` crate. The contract relies on these integrations to access functionality such as `event` handling, `currency` operations, `parameter` storage, and `block number tracking`. The risks associated with integration include compatibility issues, dependency vulnerabilities, and potential inconsistencies or conflicts between different modules or traits.


#### Non-Standard Token Risks:

The smart contract does not explicitly handle non-standard tokens. It relies on the `frame_support::traits::Currency` trait, which suggests compatibility with the standard token implementation in the substrate framework. If non-standard tokens are used, there may be risks associated with their compatibility, security, or behavior within the context of this contract. Additional analysis and testing would be necessary to assess these risks.


### Time spent:
20 hours