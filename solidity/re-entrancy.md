# Re-Entrancy

## 1. Introduction:

Re-Entrancy allows an attacker to recursively call a function from a vulnerable contract, to whatever malicious intent they have. (usually, we're talking about completely draining the vulnerable contract's balance)To learn about DAO's hack(the "historical" origin of this vulnerability) [check out this post.](https://coinmarketcap.com/alexandria/article/a-history-of-the-dao-hack)&#x20;

As usual, we will not showcase the actual exploit of the vulnerable contract. [For that, you may find this video useful.](https://www.youtube.com/watch?v=4Mm3BCyHtDY)

## 2. Typical vulnerable code:

{% code title="VulnerableContract.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract VulnerableContract {
  
  mapping(address => uint) public balances;

  function sendTo(address _to) public payable {
    balances[_to] += msg.value ;
  }

  function balanceOf(address _who) public view returns (uint balance) {
    return balances[_who];
  }

  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      // the amount is sent to the caller of the contract. 
      (bool result,) = msg.sender.call.value(_amount)("");
      if(result) {
        _amount;
      }
      // balance gets updated after VulnerableContract sends the _amount.
      balances[msg.sender] -= _amount;
    }
  }

  receive() external payable {}
}
```
{% endcode %}

The issue is found on line `26`. Since the `balances[]` of the sender is not updated properly(before actually sending the money to the `msg.sender`) an attacker can recursively call the `withdraw` function. For more details on how this exploit works, from the attacker's perspective, [you can watch this cool video](https://www.youtube.com/watch?v=4Mm3BCyHtDY).

## 3. Mitigations:

### 3.1. Checks-Effects-Interactions.

The Checks-Effects-Interactions pattern is essentially a way of organizing statements such that the **state** of the contract is updated consistently before actually interacting with other contracts/ addresses. By placing effects(eg. decreasing the user balance) before interactions(eg. sending `eth` to the user), we ensure that all state changes are done before any potential reentrancy point, leaving the state of the contract consistent on each call/ interaction.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract VulnerableContract {
  
  mapping(address => uint) public balances;

  function sendTo(address _to) public payable {
    balances[_to] += msg.value ;
  }

  function balanceOf(address _who) public view returns (uint balance) {
    return balances[_who];
  }

  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      // updating the balance before sending the _amount
      balances[msg.sender] -= _amount;
      (bool result,) = msg.sender.call.value(_amount)("");
      if(result) {
        _amount;
      }
      
    }
  }

  receive() external payable {}
}
```

### 3.2. Implementing a guard.

Making use of a reentrant guard, [such as this one](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard).

## 4. Takeaways:

It’s essential to program our contracts to be safe against reentrancy by organizing code according to the **checks-effects-interactions** pattern. Moreover, it is important to consider using already available solutions, such as the ones provided by OpenZeppelin, in order to properly mitigate the risk against such attacks.

{% hint style="info" %}
You can find more details about this topic here:

* [Re-entrancy after Istanbul.](https://blog.openzeppelin.com/reentrancy-after-istanbul/)
* [Reentrancy | Hack Solidity](https://www.youtube.com/watch?v=4Mm3BCyHtDY)
* [Solidity By Example | Re-entrancy](https://solidity-by-example.org/hacks/re-entrancy/)
{% endhint %}
