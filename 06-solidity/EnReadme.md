# solidity
#### Chaineye solidity interview questions are collected for those who want to learn.

Twitter: @seek_web3

Chainey community: official website chaineye.info | Chaineye Rust tutorial | WeChat: LGZAXE, add WeChat and then pull the community

All code and tutorials are open source on github: https://github.com/0xchaineye/chaineye-blockchain-interview

### 1. In actual production, what should be done if you want to ensure that the contract addresses of the test network and the main network are consistent?
Option 1: The contract address is determined by nonce and address, so just ensure that nonce and address are the same.

new_address = hash(sender, nonce)
Option 2: Using create2, the whole idea behind this opcode is to make the resulting address independent of future events. No matter what happens on the blockchain, contracts can always be deployed on pre-computed addresses.

The new address is a function of:

0xFF, a constant to prevent collisions with CREATE

Sender's own address

Salt (any value provided by the sender)

The bytecode of the contract to be deployed

new_address = hash(0xFF, sender, salt, bytecode)
CREATE2 guarantees that if deployed using and provided sender, it will be stored in .bytecodeCREATE2saltnew_address

Because the bytecode is included in this calculation, other agents can rely on the fact that if the contract is ever deployed to new_address, it will be one they know. This is the key concept behind counterfactual deployment.

### 2. Pure and view usage principles and scenarios of solidity smart contracts
pure and view principle
pure: Do not read or modify the variables on the block, and use the local CPU resources to calculate our functions. So it is easy to understand without consuming any resources.

view: But since view needs to read the value on the blockchain, why doesn’t it need to consume gas?

It's actually very simple, because as a full node, all information will be saved synchronously and locally.

Then if we want to check the resources on the blockchain, we can also directly query the data on a full node.

I don’t need nodes all over the world to know. They all handle this matter at the same time. I also don’t need to record the call to this function on the blockchain.

So the view still doesn't consume gas.

view: can be called freely as it just "views" the state of the blockchain without changing it

pure: can also be called freely, neither reading nor writing to the blockchain

### 3. How to convert fixed byte array to dynamic byte array
To convert a fixed-length byte array into a dynamic-length byte array, you need to first create a dynamic array and assign values ​​one by one.

pragma solidity ^0.4.23;

contract fixTodynamic{
       bytes6 name = 0x6a6f6e736f6e;

     function Todynamic() view public returns(bytes){
         //return bytes(name);
         bytes memory newName = new bytes(name.length);

         //for loop assigns values ​​one by one
         for(uint i = 0;i<name.length;i++){
            newName[i] = name[i];
         }
         return newName;
     }
}
### 4. Briefly explain the characteristics and differences of the constructor and initialization function of smart contracts.
The constructor is a special function that will be called when deploying the contract. And can only be called at this time. Often used to initialize certain state variables. The initialization function is to initialize the contract and can only be called at one level.

### 5. In Solidity, public, private, internal and external are keywords used to control the access level and visibility of functions and variables in a contract.
public public is the default access level modifier in Solidity, indicating that the function or variable is visible and callable to everyone. It can be accessed and called inside or outside the contract. Functions or variables modified with public will automatically generate a public getter function to obtain the value of the variable.

private private is a modifier used in Solidity to restrict functions or variables to only be accessible within the current contract. Functions or variables modified with private cannot be accessed or called by other contracts or external accounts outside the contract.

internal internal is a modifier used in Solidity to restrict functions or variables to only be accessible within the current contract and its sub-contracts. Functions or variables modified with internal cannot be accessed or called by other contracts or external accounts outside the contract.

external external is a modifier used in Solidity to restrict functions to only be accessed and called by accounts outside the contract. Functions modified with external cannot be accessed and called inside the contract, but can only be called outside the contract.

### 6. Smart contract contract inheritance and visibility
Inheritance is an important feature of object-oriented languages. Inheritance is to simulate real-life phenomena and simplify code writing. For example, cats and dogs are both animals. They all inherit certain traits from animals. Contract inheritance syntax

contract contract name is parent class contract name {

}
Inheritance and visibility
State variables are of public type by default, can be inherited, and can be called externally and internally. Functions are public attributes by default, can be inherited, and can be called externally and internally.

internal: When the inner attribute is added to the state variable, it can still be inherited. The internal attribute can only be called by methods in the contract and cannot be directly called externally. When the inner attribute is added to a function, it can still be inherited. The internal attribute can only be called by methods in the contract and cannot be directly called externally.

external: State variables do not have external attributes, but functions do. When the external attribute is added to a function, it means that the contract can only be called externally and cannot be called internally. Cannot be inherited.

### 7. Briefly explain the differences and connections between storage, memory and calldata in solidity
Storage location: Both storage and memory are locations used to store data, while calldata is just a location used to pass data in function calls.

Life cycle: Variables in storage still exist in the contract memory after the function call ends, while variables in memory are deleted at the end of the function call, and data in calldata are also deleted at the end of the function call.

Access: Variables in storage and memory can be read and written, while data in calldata can only be read and cannot be modified.

Cost: Using storage operations is more expensive than memory because it requires writing to persistent storage and produces permanent state changes. And memory operations are relatively cheap because it just allocates and reads data in temporary memory. For calldata, it can only be read without paying any fees.

Type: In storage and memory, any Solidity type data can be stored, including structures and arrays. However, in calldata, only fixed-size primitive type data can be passed.

Generally speaking, storage and memory are used to store variables, while calldata is used to pass function parameters. Differences in storage location, life cycle, access and cost determine their differences in use.

### 8. What types of contract upgrade methods have you used, how did you operate them, and what matters need to be paid attention to during the upgrade process?
Create a new contract and migrate data: This method requires writing new contract code and migrating data from the old contract to the new contract. This approach needs to ensure the correct migration of data, and the old contract address needs to be passed as a parameter to the new contract so that other contracts and users can find the new contract.

Using library contracts: This approach allows for upgrades by separating the contract's logic from its data. The new logic can be placed in a new library contract, and the new library contract address can be set to a new variable in the old contract. This way, new logic can be called without data needing to be migrated.

Use proxy contracts: This approach is also called the upgrade contract model. The old contract is called the proxy contract, and it only contains some basic logic, such as forwarding transactions and calling the new contract. New contracts contain more complex logic. In this approach, all transactions are sent to the proxy contract, which then forwards the transactions to the new contract for processing.

Before performing a contract upgrade, you need to ensure that there is no data loss or inconsistency during the upgrade process, and security issues need to be considered. When operating, you need to pay attention to ensuring permission control, protecting user assets and other security measures to avoid accidental or malicious operations. It is recommended to conduct contract upgrade experiments in development and testing environments, and use multi-signature and other methods to ensure the security of the upgrade operation.

During the upgrade process, you need to pay attention to the storage slot location of public variables, otherwise data confusion will occur.

### 9. What are the common smart contract audit tools and explain the differences between them?
A smart contract audit tool is a software tool used to check the security and vulnerabilities of smart contract code. Here are several common smart contract audit tools:

Mythril: Mythril is an open source tool based on Python, mainly used to discover possible vulnerabilities in smart contracts. Mythril uses a static analysis method to detect vulnerabilities through static scanning of the code. Mythril supports a variety of smart contracts for the Ethereum Virtual Machine (EVM) and other blockchains.

Oyente: Oyente is a Python-based open source tool for checking possible vulnerabilities in smart contracts. Oyente uses static analysis methods to detect vulnerabilities such as reentrancy, transaction dependencies, and iterator errors.

Securify: Securify is a static analysis-based tool for checking vulnerabilities and flaws in Ethereum smart contracts.

SmartCheck: SmartCheck is a smart contract static analysis and vulnerability detection tool designed to improve the security and reliability of smart contracts. SmartCheck supports multiple virtual machines and smart contract languages, including Solidity, Vyper, Bamboo, etc.

Slither: Slither is an open source tool that helps developers and auditors detect and resolve vulnerabilities in smart contracts. Slither adopts a static analysis method, supports a variety of virtual machines, and can detect reentrancy, state variability, overflow and other vulnerabilities.

Manticore: Manticore is a dynamic binary analysis tool that can be used to analyze vulnerabilities and security in Ethereum smart contracts.

Echidna: Echidna is a symbolic execution-based tool that can be used to check vulnerabilities and security in Ethereum smart contracts.

Teether: Teether is a binary analysis-based tool for checking Ethereum smart contracts for vulnerabilities and flaws.

The above are some common smart contract audit tools that can help developers and auditors find and fix security holes and flaws in smart contracts.

### 10. Contract attack related
Reference tutorial: Click to view

### 11. What contract integration compilation tools do you know, and briefly explain their differences?
hardhat, foundry, remix and truffle

Hardhat: It is a development environment based on Node.js that provides a wealth of plug-ins and functions to support writing, testing and deploying Ethereum smart contracts. It also supports integration with other tools such as Remix and Truffle.

Foundry: is a developer platform for building, deploying and managing Ethereum smart contracts. Foundry provides an easy way to write smart contracts and deploy them onto the Ethereum network.

Remix: is a web-based IDE for writing, testing, and deploying Ethereum smart contracts. It provides an intuitive user interface to easily create, edit and debug smart contracts.

Truffle: is a Node.js-based development framework that provides a set of tools and libraries for writing, testing, and deploying Ethereum smart contracts. Truffle also provides an interactive console for interacting with the Ethereum network, as well as supporting integration with other tools such as Ganache.

### 12. Briefly explain the difference between ERC721 and ERC 1155
ERC721 and ERC1155 are both smart contract standards on Ethereum, used to define different types of digital assets and tokens. The ERC721 standard was originally used to define non-fungible tokens (NFTs), while the ERC1155 standard is used to define fungible and non-fungible tokens.

ERC721 tokens are unique and non-fungible, and each token has its own identifier and properties, making them suitable for representing digital artworks, games, etc.Non-fungible items such as theater props. For example, if you purchase an NFT digital artwork on Ethereum, only you can own this unique token.

ERC1155 tokens are fungible and non-fungible. An ERC1155 token can represent a set of the same digital assets, and these assets can be divided and combined. This makes them suitable for fungible items such as props and virtual currency in games.

The biggest difference between the ERC1155 standard and the ERC721 standard is that one ERC1155 contract can define multiple tokens, while the ERC721 contract can only define one. Additionally, ERC1155 tokens can be transferred in batches, while ERC721 tokens can only be transferred individually.

In addition, ERC-1159 is a new standard proposal designed to solve the problem of token issuance and trading on Ethereum. This standard combines the advantages of ERC20 and ERC721 and can define fungible and non-fungible tokens in the same contract, while also supporting a burning mechanism for token issuance and destruction. In addition, ERC-1159 can also improve the efficiency of token issuance, reduce network congestion, and reduce the cost of token transactions.

### 13. Briefly explain the role of ERC 1559
ERC-1559 is an improvement proposal for Ethereum that aims to improve Ethereum’s transaction fee mechanism. Current Ethereum transaction fees are freely determined by miners based on competition for transactions, which can lead to high fees and transaction delays.

ERC-1559 proposes a new transaction fee mechanism based on block space occupation and market demand, which includes the following points:

The number of transactions contained in each block will be limited to a predefined capacity range, which will result in reduced volatility in transaction fees.

Transaction fees will be automatically calculated by an algorithm to ensure that quickly confirmed transactions receive higher priority.

A portion of transaction fees will be burned rather than paid to miners, which will help reduce Ethereum inflation.

These improvements are expected to increase Ethereum transaction efficiency, lower transaction fees, and reduce Ethereum inflation.

### 14. How to implement guard recovery in smart contract wallet
Set multiple addresses as guardians. Once the private key of the EOA wallet controlling the contract is lost, the guardian can initiate the replacement of the EOA address of the contract to achieve the purpose of guard recovery.

### 15. Briefly explain the function of EIP4337
### 16. Briefly explain the function of EIP4484
### 17.The principle and function of solidity staticcall
Solidity's staticcall method is a method for executing external smart contracts on the Ethereum Virtual Machine (EVM) that does not change state. The following is the underlying code implementation of the staticcall method:

function staticcall(
   address target,
   bytes memory data
) internal view returns (bool success, bytes return memoryData) {
   assembly {
     let result := staticcall(gas(), target, add(data, 0x20), mload(data), 0, 0)
     let size := returndatasize()
     returnData := mload(0x40) // allocate new memory
     mstore(0x40, add(returnData, and(add(add(size, 0x20), 0x1f), not(0x1f)))) // round up to nearest 32 bytes
     mstore(returnData, size) // set length of returned data
     returndatacopy(add(returnData, 0x20), 0, size)
     success := and(result, not(iszero(size)))
   }
}
This implementation uses EVM assembly code. First, the staticcall function takes the target address and the bytecode to be executed as input parameters. Then, it executes the external smart contract on the EVM by using the staticcall instruction. This instruction accepts five parameters: gas limit, target address, starting position of input data, length of input data, and starting position of output data. This function does not change the caller's state.

Within the assembly block, the result of the staticcall instruction is saved in the result variable. Then, the returndatasize instruction is used to get the length of the returned data. Next, this implementation uses EVM assembly code to allocate new memory, set the length of the return data, and copy the return data. Finally, it returns the success status and return data to the caller as output.

Generally speaking, this underlying implementation uses EVM assembly code, which uses some instructions to handle the call of external smart contracts and the processing of returned data in order to implement the static call method.

The staticcall function in Solidity is a special type of function call that allows executing a smart contract function on the Ethereum Virtual Machine without modifying the state. Its purpose is to query data in a smart contract or calculate a certain state without changing the state on the blockchain.

In the staticcall function, the virtual machine creates a new temporary account and passes the call data to the account. Then, the account's code will be executed, but it will not be able to modify the state or call functions with state changes. This process does not consume gas because there are no state changes.

The return value of the staticcall function is a Boolean value used to indicate whether the call was successful. If successful, the function's return value is returned, otherwise an empty byte array is returned.

A common scenario for staticcall is to read data from smart contracts. For example, when you need to get some data from another contract in one contract, you can use the staticcall function to query the data without having to call a function with state changes. This can avoid additional gas costs and state changes.

### 18. The principle and function of assembly in solidity
Solidity is a high-level programming language used for writing smart contracts. It is based on the bytecode instruction set of the EVM (Ethereum Virtual Machine), but sometimes lower-level operations are required, in which case the Assembly language in Solidity can be used.

Assembly is a low-level language that directly operates on the EVM instruction set. Assembly in Solidity can be used to perform some high-level operations, such as inline assembly, and can also be used to access contract storage space and perform low-level memory operations. In addition, Assembly can also be used to optimize code and perform some advanced encryption algorithms.

Assembly code in Solidity can be defined by using the "assembly" keyword in a function declaration. Assembly code uses an assembly language-like syntax and uses a specific set of instructions to operate the EVM. The Assembly language can also directly read and write memory and storage, as well as perform other low-level operations.

In summary, Assembly in Solidity can provide higher-level operations and better performance, but it also requires more writing experience and skills because it requires direct manipulation of the EVM instruction set. For most smart contracts, Solidity's high-level language is sufficient and there is no need to use Assembly.

### 19. How to avoid leaving the contract in an uninitialized state.
An attacker can take over an uninitialized contract. This applies to agents and their implementation contracts, which may impact agents. To prevent the implementation contract from being used, you should call the _disableInitializers function in the constructor to automatically lock it on deployment:

/// @custom:oz-upgrades-unsafe-allow constructor
constructor() {
     _disableInitializers();
}
