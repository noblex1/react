# Module 01: Blockchain Fundamentals and Ethereum

**Duration:** 1 Week | **Difficulty:** Beginner

## 🎯 Learning Objectives

By the end of this module, you will:
- ✅ Understand blockchain technology and distributed systems
- ✅ Master Ethereum architecture and the Ethereum Virtual Machine (EVM)
- ✅ Learn about accounts, transactions, and gas mechanics
- ✅ Understand consensus mechanisms and network security
- ✅ Set up development environment and connect to networks
- ✅ Interact with Ethereum using Web3 tools
- ✅ Understand the role of smart contracts in the ecosystem

## 📚 What is Blockchain?

### Core Concepts

**Blockchain** is a distributed ledger technology that maintains a continuously growing list of records (blocks) that are linked and secured using cryptography.

#### Key Properties
- **Decentralization** - No single point of control
- **Immutability** - Data cannot be altered once confirmed
- **Transparency** - All transactions are publicly visible
- **Consensus** - Network agrees on the state of the ledger
- **Cryptographic Security** - Protected by advanced cryptography

### Blockchain Structure
```
Block N-1          Block N            Block N+1
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│ Block Header│   │ Block Header│   │ Block Header│
│ - Prev Hash │◄──│ - Prev Hash │◄──│ - Prev Hash │
│ - Merkle Root│   │ - Merkle Root│   │ - Merkle Root│
│ - Timestamp │   │ - Timestamp │   │ - Timestamp │
│ - Nonce     │   │ - Nonce     │   │ - Nonce     │
├─────────────┤   ├─────────────┤   ├─────────────┤
│ Transactions│   │ Transactions│   │ Transactions│
│ - Tx 1      │   │ - Tx 1      │   │ - Tx 1      │
│ - Tx 2      │   │ - Tx 2      │   │ - Tx 2      │
│ - Tx 3      │   │ - Tx 3      │   │ - Tx 3      │
└─────────────┘   └─────────────┘   └─────────────┘
```

### Cryptographic Foundations

#### Hash Functions
```javascript
// Example: SHA-256 hashing
const crypto = require('crypto');

function hash(data) {
  return crypto.createHash('sha256').update(data).digest('hex');
}

console.log(hash('Hello Blockchain')); 
// Output: a1b2c3d4e5f6... (64 character hex string)
```

#### Digital Signatures
```javascript
// Ethereum uses ECDSA (Elliptic Curve Digital Signature Algorithm)
// Private Key → Public Key → Ethereum Address

// Example flow:
// 1. Generate private key (256-bit random number)
// 2. Derive public key using elliptic curve multiplication
// 3. Hash public key and take last 20 bytes for address
```

#### Merkle Trees
```
        Root Hash
       /         \
   Hash AB     Hash CD
   /    \      /    \
Hash A Hash B Hash C Hash D
  |     |      |     |
 Tx A  Tx B   Tx C  Tx D
```

## 🔷 Ethereum Architecture

### Ethereum Virtual Machine (EVM)

The **EVM** is a runtime environment for smart contracts in Ethereum. It's a stack-based virtual machine that executes bytecode.

#### EVM Characteristics
- **Deterministic** - Same input always produces same output
- **Isolated** - Contracts run in sandboxed environment
- **Turing Complete** - Can perform any computation (with gas limits)
- **State Machine** - Maintains global state across all accounts

#### EVM Stack
```
┌─────────────────────────────────────┐
│            Smart Contracts          │ ← High-level languages (Solidity)
├─────────────────────────────────────┤
│              EVM Bytecode           │ ← Compiled contract code
├─────────────────────────────────────┤
│         Ethereum Virtual Machine    │ ← Execution environment
├─────────────────────────────────────┤
│            Ethereum Node            │ ← Network participant
├─────────────────────────────────────┤
│           Operating System          │ ← Host system
└─────────────────────────────────────┘
```

### Ethereum Accounts

#### Externally Owned Accounts (EOAs)
```javascript
// EOA Structure
{
  address: "0x742d35Cc6634C0532925a3b8D4C9db96590c6C87",
  balance: "1000000000000000000", // 1 ETH in wei
  nonce: 42, // Number of transactions sent
  // No code storage
}
```

#### Contract Accounts
```javascript
// Contract Account Structure
{
  address: "0x1f9840a85d5aF5bf1D1762F925BDADdC4201F984",
  balance: "500000000000000000", // 0.5 ETH in wei
  nonce: 1, // Number of contracts created
  codeHash: "0xa1b2c3...", // Hash of contract code
  storageRoot: "0xd4e5f6..." // Root of storage trie
}
```

### Transactions and Gas

#### Transaction Structure
```javascript
// Ethereum Transaction
{
  from: "0x742d35Cc6634C0532925a3b8D4C9db96590c6C87",
  to: "0x1f9840a85d5aF5bf1D1762F925BDADdC4201F984",
  value: "1000000000000000000", // 1 ETH in wei
  gas: 21000, // Gas limit
  gasPrice: "20000000000", // 20 Gwei
  nonce: 42,
  data: "0x", // Empty for simple transfer
  v: 28, // Recovery ID
  r: "0x...", // Signature component
  s: "0x..." // Signature component
}
```

#### Gas Mechanics
```solidity
// Gas costs for common operations
SSTORE (storage write): 20,000 gas
SLOAD (storage read): 800 gas
ADD/SUB/MUL: 3 gas
DIV/MOD: 5 gas
SHA3: 30 gas + 6 gas per word
CREATE (deploy contract): 32,000 gas
```

## 🔗 Consensus Mechanisms

### Proof of Work (Legacy)
```javascript
// Simplified PoW mining algorithm
function mineBlock(blockData, difficulty) {
  let nonce = 0;
  const target = "0".repeat(difficulty);
  
  while (true) {
    const hash = calculateHash(blockData + nonce);
    if (hash.startsWith(target)) {
      return { nonce, hash };
    }
    nonce++;
  }
}
```

### Proof of Stake (Current)
```javascript
// Validator selection in PoS
function selectValidator(validators, randomSeed) {
  const totalStake = validators.reduce((sum, v) => sum + v.stake, 0);
  const randomValue = randomSeed % totalStake;
  
  let cumulativeStake = 0;
  for (const validator of validators) {
    cumulativeStake += validator.stake;
    if (randomValue < cumulativeStake) {
      return validator;
    }
  }
}
```

## 🛠️ Development Environment Setup

### Essential Tools Installation

#### 1. Node.js and npm
```bash
# Install Node.js (v18+ recommended)
# Download from https://nodejs.org/

# Verify installation
node --version
npm --version
```

#### 2. Hardhat Development Framework
```bash
# Create new project
mkdir my-smart-contract
cd my-smart-contract
npm init -y

# Install Hardhat
npm install --save-dev hardhat

# Initialize Hardhat project
npx hardhat
# Choose "Create a JavaScript project"
```

#### 3. Essential Dependencies
```bash
# OpenZeppelin contracts (security-audited libraries)
npm install @openzeppelin/contracts

# Hardhat plugins
npm install --save-dev @nomicfoundation/hardhat-toolbox
npm install --save-dev @nomiclabs/hardhat-etherscan
npm install --save-dev dotenv
```

#### 4. Hardhat Configuration
```javascript
// hardhat.config.js
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

module.exports = {
  solidity: {
    version: "0.8.19",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200
      }
    }
  },
  networks: {
    hardhat: {
      chainId: 31337
    },
    goerli: {
      url: process.env.GOERLI_RPC_URL,
      accounts: [process.env.PRIVATE_KEY]
    },
    sepolia: {
      url: process.env.SEPOLIA_RPC_URL,
      accounts: [process.env.PRIVATE_KEY]
    }
  },
  etherscan: {
    apiKey: process.env.ETHERSCAN_API_KEY
  }
};
```

### MetaMask Wallet Setup

#### 1. Install MetaMask
- Download from [metamask.io](https://metamask.io/)
- Create new wallet or import existing
- **NEVER share your seed phrase!**

#### 2. Add Test Networks
```javascript
// Goerli Testnet Configuration
Network Name: Goerli Test Network
RPC URL: https://goerli.infura.io/v3/YOUR_PROJECT_ID
Chain ID: 5
Currency Symbol: ETH
Block Explorer: https://goerli.etherscan.io

// Sepolia Testnet Configuration
Network Name: Sepolia Test Network
RPC URL: https://sepolia.infura.io/v3/YOUR_PROJECT_ID
Chain ID: 11155111
Currency Symbol: ETH
Block Explorer: https://sepolia.etherscan.io
```

#### 3. Get Test ETH
```bash
# Goerli Faucets
https://goerlifaucet.com/
https://faucets.chain.link/goerli

# Sepolia Faucets
https://sepoliafaucet.com/
https://faucet.sepolia.dev/
```

## 🌐 Interacting with Ethereum

### Using Ethers.js Library

#### 1. Basic Connection
```javascript
const { ethers } = require("ethers");

// Connect to network
const provider = new ethers.providers.JsonRpcProvider("https://goerli.infura.io/v3/YOUR_PROJECT_ID");

// Get network information
async function getNetworkInfo() {
  const network = await provider.getNetwork();
  console.log("Network:", network.name);
  console.log("Chain ID:", network.chainId);
  
  const blockNumber = await provider.getBlockNumber();
  console.log("Latest block:", blockNumber);
}
```

#### 2. Account Operations
```javascript
// Create wallet from private key
const wallet = new ethers.Wallet(process.env.PRIVATE_KEY, provider);

// Get account balance
async function getBalance(address) {
  const balance = await provider.getBalance(address);
  return ethers.utils.formatEther(balance);
}

// Send transaction
async function sendTransaction(to, amount) {
  const tx = await wallet.sendTransaction({
    to: to,
    value: ethers.utils.parseEther(amount)
  });
  
  console.log("Transaction hash:", tx.hash);
  
  // Wait for confirmation
  const receipt = await tx.wait();
  console.log("Transaction confirmed in block:", receipt.blockNumber);
}
```

#### 3. Contract Interaction
```javascript
// Contract ABI (Application Binary Interface)
const abi = [
  "function balanceOf(address owner) view returns (uint256)",
  "function transfer(address to, uint256 amount) returns (bool)"
];

// Connect to contract
const contract = new ethers.Contract(contractAddress, abi, wallet);

// Read from contract (view function)
async function getTokenBalance(address) {
  const balance = await contract.balanceOf(address);
  return ethers.utils.formatEther(balance);
}

// Write to contract (state-changing function)
async function transferTokens(to, amount) {
  const tx = await contract.transfer(to, ethers.utils.parseEther(amount));
  const receipt = await tx.wait();
  console.log("Transfer completed:", receipt.transactionHash);
}
```

## 🏋️ Hands-on Exercises

### Exercise 1: Network Exploration
Create a script that explores Ethereum network data:

```javascript
// network-explorer.js
const { ethers } = require("ethers");

async function exploreNetwork() {
  const provider = new ethers.providers.JsonRpcProvider("https://goerli.infura.io/v3/YOUR_PROJECT_ID");
  
  // Get latest block
  const latestBlock = await provider.getBlock("latest");
  console.log("Latest Block:", {
    number: latestBlock.number,
    hash: latestBlock.hash,
    timestamp: new Date(latestBlock.timestamp * 1000),
    transactions: latestBlock.transactions.length
  });
  
  // Get gas price
  const gasPrice = await provider.getGasPrice();
  console.log("Current Gas Price:", ethers.utils.formatUnits(gasPrice, "gwei"), "Gwei");
  
  // Analyze a transaction
  if (latestBlock.transactions.length > 0) {
    const txHash = latestBlock.transactions[0];
    const tx = await provider.getTransaction(txHash);
    console.log("Sample Transaction:", {
      hash: tx.hash,
      from: tx.from,
      to: tx.to,
      value: ethers.utils.formatEther(tx.value),
      gasLimit: tx.gasLimit.toString(),
      gasPrice: ethers.utils.formatUnits(tx.gasPrice, "gwei")
    });
  }
}

exploreNetwork().catch(console.error);
```

### Exercise 2: Wallet Operations
Build a simple wallet manager:

```javascript
// wallet-manager.js
const { ethers } = require("ethers");

class SimpleWallet {
  constructor(privateKey, providerUrl) {
    this.provider = new ethers.providers.JsonRpcProvider(providerUrl);
    this.wallet = new ethers.Wallet(privateKey, this.provider);
  }
  
  async getBalance() {
    const balance = await this.provider.getBalance(this.wallet.address);
    return ethers.utils.formatEther(balance);
  }
  
  async sendEther(to, amount) {
    const tx = await this.wallet.sendTransaction({
      to: to,
      value: ethers.utils.parseEther(amount)
    });
    
    console.log(`Sent ${amount} ETH to ${to}`);
    console.log(`Transaction hash: ${tx.hash}`);
    
    const receipt = await tx.wait();
    console.log(`Confirmed in block: ${receipt.blockNumber}`);
    
    return receipt;
  }
  
  async getTransactionHistory(count = 10) {
    // This would require additional API calls or indexing service
    // For now, we'll just show the concept
    console.log(`Getting last ${count} transactions for ${this.wallet.address}`);
  }
}

// Usage
async function main() {
  const wallet = new SimpleWallet(
    process.env.PRIVATE_KEY,
    "https://goerli.infura.io/v3/YOUR_PROJECT_ID"
  );
  
  console.log("Wallet Address:", wallet.wallet.address);
  console.log("Balance:", await wallet.getBalance(), "ETH");
}

main().catch(console.error);
```

### Exercise 3: Gas Estimation
Create a gas estimation tool:

```javascript
// gas-estimator.js
const { ethers } = require("ethers");

async function estimateGasCosts() {
  const provider = new ethers.providers.JsonRpcProvider("https://goerli.infura.io/v3/YOUR_PROJECT_ID");
  
  // Get current gas price
  const gasPrice = await provider.getGasPrice();
  const gasPriceGwei = ethers.utils.formatUnits(gasPrice, "gwei");
  
  console.log(`Current Gas Price: ${gasPriceGwei} Gwei`);
  
  // Estimate costs for common operations
  const operations = [
    { name: "Simple ETH Transfer", gas: 21000 },
    { name: "ERC-20 Transfer", gas: 65000 },
    { name: "Uniswap Swap", gas: 150000 },
    { name: "Contract Deployment", gas: 500000 }
  ];
  
  console.log("\nEstimated Transaction Costs:");
  operations.forEach(op => {
    const costWei = gasPrice.mul(op.gas);
    const costEth = ethers.utils.formatEther(costWei);
    console.log(`${op.name}: ${costEth} ETH (${op.gas} gas)`);
  });
}

estimateGasCosts().catch(console.error);
```

## 🎯 Challenge: Block Explorer

Build a simple block explorer that can:

**Requirements:**
1. **Block Information** - Display block details by number or hash
2. **Transaction Details** - Show transaction information and status
3. **Address Lookup** - Get balance and transaction history for any address
4. **Gas Tracker** - Monitor gas prices and estimate costs
5. **Network Stats** - Show network statistics and health

**Implementation Structure:**
```javascript
class BlockExplorer {
  constructor(providerUrl) {
    this.provider = new ethers.providers.JsonRpcProvider(providerUrl);
  }
  
  async getBlock(blockNumber) {
    // Implement block retrieval
  }
  
  async getTransaction(txHash) {
    // Implement transaction lookup
  }
  
  async getAddressInfo(address) {
    // Implement address information
  }
  
  async getGasTracker() {
    // Implement gas price tracking
  }
  
  async getNetworkStats() {
    // Implement network statistics
  }
}
```

## 📚 Additional Resources

### Essential Reading:
- [Ethereum Whitepaper](https://ethereum.org/en/whitepaper/)
- [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
- [Mastering Ethereum](https://github.com/ethereumbook/ethereumbook)

### Development Tools:
- [Hardhat Documentation](https://hardhat.org/docs)
- [Ethers.js Documentation](https://docs.ethers.io/)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/)

### Network Resources:
- [Etherscan](https://etherscan.io/) - Ethereum block explorer
- [Ethereum Gas Tracker](https://etherscan.io/gastracker)
- [EIP Repository](https://eips.ethereum.org/) - Ethereum Improvement Proposals

## ✅ Module Completion Checklist

Before moving to the next module, make sure you can:
- [ ] Explain blockchain fundamentals and consensus mechanisms
- [ ] Understand Ethereum architecture and the EVM
- [ ] Set up development environment with Hardhat
- [ ] Connect to Ethereum networks using providers
- [ ] Interact with accounts and send transactions
- [ ] Estimate gas costs for different operations
- [ ] Use MetaMask for wallet operations
- [ ] Read blockchain data using Ethers.js
- [ ] Understand the role of smart contracts

## 🚀 What's Next?

In **Module 02**, we'll dive into:
- Solidity programming language fundamentals
- Setting up smart contract development workflow
- Writing your first smart contract
- Compiling and deploying contracts

---

**Ready to continue?** 👉 **[Module 02: Solidity Basics and Development Environment](../02-solidity-basics/README.md)**

---

*Estimated completion time: 8-12 hours*
