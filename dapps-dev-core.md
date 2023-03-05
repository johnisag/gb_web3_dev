---
description: Common functionality in dapps
---

# dApps - Dev Core

### **Default Eth Address**

```solidity
address(0) = 0x0000000000000000000000000000000000000000
```

### **Track wallet connected**

```javascript
// walletConnected keeps track of whether the user's wallet is connected or not
const [walletConnected, setWalletConnected] = useState(false);
```

### **Keep Web3 Modal Ref**&#x20;

* **Web3 Modal** (used for connecting to **Metamask**) **reference** **which persists as long as the page is open**

```javascript
import Web3Modal from "web3modal";
const web3ModalRef: any = useRef();
```

### **Get Provider/Signer**

* **Provider** (just view data) &#x20;
* **Signer** (view or modify data)

```javascript
import Web3Modal from "web3modal";
const web3ModalRef: any = useRef();

const getProviderOrSigner = async (needSigner = false) => {
    // Connect to Metamask
    // Since we store `web3Modal` as a reference, we need to access 
    // the `current` value to get access to the underlying object
    const provider = await web3ModalRef.current.connect();
    const web3Provider = new providers.Web3Provider(provider);

    // If user is not connected to the Goerli network, 
    // let them know and throw an error
    const { chainId } = await web3Provider.getNetwork();
    if (chainId !== 5) {
      window.alert("Change the network to Goerli");
      throw new Error("Change network to Goerli");
    }

    if (needSigner) {
      const signer = web3Provider.getSigner();
      return signer;
    }
    return web3Provider;
};
```

### **Get address  **<mark style="color:orange;">****</mark>&#x20;

* **Get address of connected account  **<mark style="color:orange;">**needs signer**</mark>

```javascript
const signer: any = await getProviderOrSigner(true);
const address = await signer.getAddress();
```

### **Create contract instance**

```javascript
import { Contract, providers} from "ethers";

// Get the provider from web3Modal (e.g MetaMask)
// If we need to modify data, we will need a signer
const provider = await getProviderOrSigner(); // OR
const signer: any = await getProviderOrSigner(true);

// Create an instance of MyContract
// ABI can be found in:
// hardhat_folder\artifacts\contracts\MyContract.sol\MyContract.json
const tokenContract = new Contract(CONTRACT_ADDRESS, CONTRACT_ABI, provider);
```

### **Mint tokens**

<pre class="language-javascript"><code class="lang-javascript">import { Contract, providers} from "ethers";

try {
<strong>    const signer = await getProviderOrSigner(true);
</strong>    const tokenContract = new Contract(TOKEN_CREATOR_ADDRESS,TOKEN_CREATOR_ABI,signer);
    // 0.001: the eth cost to create of 1 token
    const value = 0.001 * num_tokens; // or whatever amount per token creation
    const tx = await tokenContract.mint(num_tokens, {
        // Parsing `0.001` string to ether using the utils library from ethers.js
        value: utils.parseEther(value.toString()),
    });
    await tx.wait();
    window.alert("Tokens minted!")
} catch (err) {
    console.error(err);
}
</code></pre>

### **Connect Metamask**&#x20;

* Wrapper function

```javascript
// walletConnected keeps track of whether the user's wallet is connected or not
const [walletConnected, setWalletConnected] = useState(false);

const connectWallet = async () => {
    try {
      // Get the provider from web3Modal, which in our case is MetaMask
      // When used for the first time, it prompts the user to connect their wallet
      await getProviderOrSigner();
      setWalletConnected(true);
    } catch (err) {
      console.error(err);
    }
}

// useEffects are used to react to changes in state of the website
// array at the end represents what state changes will trigger this effect
// whenever the value of `walletConnected` changes - this effect will be called
useEffect(() => {
    // if wallet is not connected, create a new instance of Web3Modal and connect the MetaMask wallet
    if (!walletConnected) {
      // Assign the Web3Modal class to the reference object by setting it's `current` value
      // The `current` value is persisted throughout as long as this page is open
      web3ModalRef.current = new Web3Modal({
        network: "goerli",
        providerOptions: {},
        disableInjectedProvider: false,
      });
      connectWallet().then(() => {
        // OPTIONAL
        //TODO: call function wrappers to get data from contracts
      });
    }
}, [walletConnected]);
```

```html
<button onClick={connectWallet} className={styles.button}>
    Connect your wallet
</button>
```

### Get Contract Owner

```javascript
  const getOwner = async () => {
    try {
        const signer: any   = await getProviderOrSigner(true);
        const contract = getMyContractInstance(signer);

        // call the owner function from the contract
        const _owner  = await contract.owner();
        
        // Get the address associated to signer which is connected to Metamask
        const address = await signer.getAddress();
        
        if (address.toLowerCase() === _owner.toLowerCase()) {
          setIsOwner(true);
        }
    } catch (err: any) {
      console.error(err.message);
    }
  };
```

### Get Contract Instance

* Wrapper function

{% code overflow="wrap" %}
```javascript
// Helper function to return a Contract instance; given a Provider/Signer
const getContractInstance = (
  providerOrSigner: any, 
  address: string, 
  abi: [], 
  is_debug: boolean = false
) => {
  const contractInstance = new Contract(address, abi, providerOrSigner);

  if (is_debug){
    console.log("Getting Contract Instance: " + address);
    console.log("Contract ABI:" + abi);
  }

  return contractInstance;
};
```
{% endcode %}

### Withdraw Contract ETH

```javascript
const withdrawEther = async () => {
  try {
    // get connnected metamask account
    const signer   = await getProviderOrSigner(true);
    const contract = getContractInstance(signer);

    const tx = await contract.withdrawEther();
    setLoading(true);
    await tx.wait();
    setLoading(false);
  } catch (err: any) {
    console.error(err);
    console.error(err.reason);
  }
};
```

### Get ETH Balance

```javascript
export const getEtherBalance = async (provider, address) => {
  try {
    const balance = await provider.getBalance(address);
    return balance;
  } catch (err) {
    console.error(err);
    return 0;
  }
};
```

### Get ERC20 Token Balance

```javascript
export const getTokenBalance = async (provider, address) => {
  try {
    const tokenContract = new Contract(
      TOKEN_CONTRACT_ADDRESS,
      TOKEN_CONTRACT_ABI,
      provider
    );
    const balanceOfER20Token = await tokenContract.balanceOf(address);
    return balanceOfER20Token ;
  } catch (err) {
    console.error(err);
  }
};
```

### Get ENS or Address

* Sets the ENS, if the current connected address has an associated ENS OR ELSE&#x20;
* It sets the address of the connected account&#x20;

```javascript
  // ENS
  const [ens, setENS] = useState("");
  // Save the address of the currently connected account
  const [address, setAddress] = useState("");

  const setENSOrAddress = async (address, web3Provider) => {
    // Lookup the ENS related to the given address
    var _ens = await web3Provider.lookupAddress(address);
    // If the address has an ENS set the ENS or else just set the address
    if (_ens) {
      setENS(_ens);
    } else {
      setAddress(address);
    }
  };
```

### Testing - Get Address

```javascript
// Get three addresses, treat one as the user address
// one as the relayer address, and one as a recipient address
const [_, userAddress, relayerAddress, recipientAddress] = await ethers.getSigners();
```

### Infinite Approve Token Transfer

```javascript
// Have user infinite approve the token sender contract for transferring 'RandomToken'
const approveTxn = await userTokenContractInstance.approve(
  tokenSenderContract.address,
  BigNumber.from(
    // This is uint256's max value (2^256 - 1) in hex
    // Fun Fact: There are 64 f's in here.
    // In hexadecimal, each digit can represent 4 bits
    // f is the largest digit in hexadecimal (1111 in binary)
    // 4 + 4 = 8 i.e. two hex digits = 1 byte
    // 64 digits = 32 bytes
    // 32 bytes = 256 bits = uint256
    "0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff"
  )
);
await approveTxn.wait();
```

