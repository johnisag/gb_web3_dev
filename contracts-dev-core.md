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

### Create Interface

* When we comsume contracts from within other contracts
* Init of the contract instance within the contructor

```solidity
interface IFakeNFTMarketplace {
    function getPrice() external view returns (uint256);
    
    function available(uint256 _tokenId) external view returns (bool);

    function purchase(uint256 _tokenId) external payable;
}

contract CryptoDevsDAO is Ownable {
    // to create instance of the utilised contracts
    IFakeNFTMarketplace nftMarketplace;
    
    constructor(address _nftMarketplace) payable {
        nftMarketplace = IFakeNFTMarketplace(_nftMarketplace);
    }
    // REST OF THE CODE ...
}
```

### Create Modifier

```solidity
// Create a modifier which only allows a function to be
// called if the given proposal's deadline has not been exceeded yet
modifier activeProposalOnly(uint256 proposalIndex) {
    require(
        proposals[proposalIndex].deadline > block.timestamp,
        "DEADLINE_EXCEEDED"
    );
    _;
}

/// @dev voteOnProposal allows a CryptoDevsNFT holder to cast their vote on an active proposal
/// @param proposalIndex - the index of the proposal to vote on in the proposals array
/// @param vote - the type of vote they want to cast
function voteOnProposal(uint256 proposalIndex, Vote vote)
    external
    nftHolderOnly
    activeProposalOnly(proposalIndex)
{
    // TODO: SOME IMPLEMENTATION
}
```

### Withdraw Contract ETH

* Contract Function

```solidity
function withdraw() external onlyOwner {
    uint256 amount = address(this).balance;
    require(amount > 0, "Nothing to withdraw; contract balance empty");
    payable(owner()).transfer(amount);
}
```
