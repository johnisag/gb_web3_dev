---
description: Common functionality in contracts development
---

# Contracts - Dev Core

### **Create ERC20**&#x20;

* **Create an ERC20 Contract**

<pre class="language-solidity"><code class="lang-solidity">// SPDX-License-Identifier: MIT
<strong>pragma solidity ^0.8.0;
</strong>
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract CryptoDevToken is ERC20, Ownable {
    // whatever price per token
    uint256 public constant tokenPrice = 0.001 ether;
    // the max total supply is e.g. 10000 for My Tokens
    uint256 public constant maxTotalSupply = 10000 * 10**18;
    
    constructor() ERC20("My Own Token", "MOT") {}
    
    function mint(uint256 amount) public payable {
        // value of ether: should be equal or greater than tokenPrice * amount;
        uint256 _requiredAmount = tokenPrice * amount;
        require(msg.value >= _requiredAmount, "Ether sent is incorrect");
        // total tokens + amount &#x3C;= 10000, otherwise revert the transaction
        uint256 amountWithDecimals = amount * 10**18;
        require(
           (totalSupply() + amountWithDecimals) &#x3C;= maxTotalSupply,
           "Exceeds the max total supply available."
        );
        // call the internal function from Openzeppelin's ERC20 contract
        _mint(msg.sender, amountWithDecimals);
      }
      
      function withdraw() public onlyOwner {
        uint256 amount = address(this).balance;
        require(amount > 0, "Nothing to withdraw; contract balance empty");
        
        address _owner = owner();
        (bool sent, ) = _owner.call{value: amount}("");
        require(sent, "Failed to send Ether");
      }
      
      // Function to receive Ether. msg.data must be empty
      receive() external payable {}

      // Fallback function is called when msg.data is not empty
      fallback() external payable {}
}
</code></pre>

### Create Modifier

```
// Some code
```
