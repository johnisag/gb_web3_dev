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
      connectWallet();
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
