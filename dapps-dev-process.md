---
description: dApp dev process steps
---

# dApps - Dev Process

### **Bootstrap**

* **Boostrap the**  [**React**](https://reactjs.org/) **/** [**Next Js**](https://nextjs.org/) **web app (from project root)**

```shell
npx create-next-app@latest

# start local web server on http://localhost:3000; see that bootstrap worked
cd my-app
npm run dev
```

### **web3 libraries**&#x20;

* **Install required web3 libraries (** [Web3Modal library](https://github.com/Web3Modal/web3modal), [ethers](https://docs.ethers.io/v3/)**)**

```shell
npm install web3modal
npm install ethers
```

### **CSS**&#x20;

* **Create/Update CSS on ./styles/Home.module.css** if required

### **dApp logic**

* **Add dApp logic in ./pages/index.js**

### **Add constants**

* **Add constants (**contract(s) addresses, ABIs,...**)** in **constants/index.js.**&#x20;

```javascript
// ABI comes from contract.json from artefacts folder
export const TOKEN_CONTRACT_ABI = "abi-of-your-token-contract";
export const TOKEN_CONTRACT_ADDRESS = "address-of-your-token-contract";
```
