# Project 1: Simple Token Contract (ERC-20) 🪙

**Difficulty:** Beginner | **Estimated Time:** 8-12 hours | **Technologies:** Solidity, Hardhat, OpenZeppelin

## 🎯 Project Overview

Build your first smart contract by implementing a complete ERC-20 token from scratch. This project introduces you to Solidity fundamentals, token standards, and smart contract deployment while creating a production-ready cryptocurrency token.

## 🚀 What You'll Build

A fully functional ERC-20 token contract with the following features:
- ✅ Standard ERC-20 functionality (transfer, approve, allowance)
- ✅ Minting and burning capabilities
- ✅ Access control for administrative functions
- ✅ Pausable functionality for emergency stops
- ✅ Detailed events and error handling
- ✅ Gas-optimized implementation
- ✅ Comprehensive test suite
- ✅ Frontend integration with React

## 🛠️ Technical Requirements

### Core ERC-20 Functions
```solidity
interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address to, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
    
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}
```

### Extended Features
```solidity
// Additional functionality beyond basic ERC-20
interface IExtendedToken {
    function mint(address to, uint256 amount) external;
    function burn(uint256 amount) external;
    function burnFrom(address account, uint256 amount) external;
    function pause() external;
    function unpause() external;
    
    event Mint(address indexed to, uint256 amount);
    event Burn(address indexed from, uint256 amount);
    event Paused(address account);
    event Unpaused(address account);
}
```

## 📋 Step-by-Step Implementation Guide

### Phase 1: Project Setup and Basic Structure (2-3 hours)

#### 1. Initialize Hardhat Project
```bash
mkdir simple-token-project
cd simple-token-project
npm init -y

# Install dependencies
npm install --save-dev hardhat
npm install --save-dev @nomicfoundation/hardhat-toolbox
npm install @openzeppelin/contracts
npm install dotenv

# Initialize Hardhat
npx hardhat
# Choose "Create a JavaScript project"
```

#### 2. Project Structure
```
simple-token-project/
├── contracts/
│   ├── SimpleToken.sol
│   └── interfaces/
│       └── ISimpleToken.sol
├── scripts/
│   ├── deploy.js
│   └── interact.js
├── test/
│   ├── SimpleToken.test.js
│   └── integration.test.js
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   └── hooks/
│   └── public/
├── hardhat.config.js
├── .env
└── README.md
```

#### 3. Hardhat Configuration
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
      accounts: [process.env.PRIVATE_KEY],
      gasPrice: 20000000000, // 20 gwei
    },
    sepolia: {
      url: process.env.SEPOLIA_RPC_URL,
      accounts: [process.env.PRIVATE_KEY]
    }
  },
  etherscan: {
    apiKey: process.env.ETHERSCAN_API_KEY
  },
  gasReporter: {
    enabled: true,
    currency: 'USD',
    gasPrice: 20
  }
};
```

### Phase 2: Smart Contract Implementation (3-4 hours)

#### 1. Basic Token Contract
```solidity
// contracts/SimpleToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Pausable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

/**
 * @title SimpleToken
 * @dev Implementation of a basic ERC20 token with additional features
 * Features:
 * - Standard ERC20 functionality
 * - Minting and burning
 * - Pausable transfers
 * - Access control
 * - Reentrancy protection
 */
contract SimpleToken is ERC20, ERC20Burnable, ERC20Pausable, Ownable, ReentrancyGuard {
    // Maximum supply cap
    uint256 public constant MAX_SUPPLY = 1_000_000_000 * 10**18; // 1 billion tokens
    
    // Minting rate limit (tokens per day)
    uint256 public constant DAILY_MINT_LIMIT = 1_000_000 * 10**18; // 1 million tokens
    uint256 public lastMintTimestamp;
    uint256 public dailyMintedAmount;
    
    // Events
    event Mint(address indexed to, uint256 amount);
    event BurnFrom(address indexed from, uint256 amount);
    event MintLimitUpdated(uint256 newLimit);
    
    // Custom errors
    error ExceedsMaxSupply(uint256 requested, uint256 available);
    error ExceedsDailyMintLimit(uint256 requested, uint256 available);
    error InvalidAmount();
    error InvalidAddress();
    
    /**
     * @dev Constructor that gives msg.sender all of existing tokens
     * @param name Token name
     * @param symbol Token symbol
     * @param initialSupply Initial token supply
     */
    constructor(
        string memory name,
        string memory symbol,
        uint256 initialSupply
    ) ERC20(name, symbol) {
        if (initialSupply > MAX_SUPPLY) {
            revert ExceedsMaxSupply(initialSupply, MAX_SUPPLY);
        }
        
        _mint(msg.sender, initialSupply);
        lastMintTimestamp = block.timestamp;
    }
    
    /**
     * @dev Mint new tokens to specified address
     * @param to Address to mint tokens to
     * @param amount Amount of tokens to mint
     */
    function mint(address to, uint256 amount) external onlyOwner nonReentrant {
        if (to == address(0)) revert InvalidAddress();
        if (amount == 0) revert InvalidAmount();
        
        // Check max supply
        if (totalSupply() + amount > MAX_SUPPLY) {
            revert ExceedsMaxSupply(amount, MAX_SUPPLY - totalSupply());
        }
        
        // Check daily mint limit
        _checkDailyMintLimit(amount);
        
        _mint(to, amount);
        emit Mint(to, amount);
    }
    
    /**
     * @dev Burn tokens from specified account (with allowance)
     * @param account Account to burn tokens from
     * @param amount Amount of tokens to burn
     */
    function burnFrom(address account, uint256 amount) public override {
        if (account == address(0)) revert InvalidAddress();
        if (amount == 0) revert InvalidAmount();
        
        super.burnFrom(account, amount);
        emit BurnFrom(account, amount);
    }
    
    /**
     * @dev Pause all token transfers
     * Can only be called by the owner
     */
    function pause() external onlyOwner {
        _pause();
    }
    
    /**
     * @dev Unpause all token transfers
     * Can only be called by the owner
     */
    function unpause() external onlyOwner {
        _unpause();
    }
    
    /**
     * @dev Check and update daily mint limit
     * @param amount Amount to be minted
     */
    function _checkDailyMintLimit(uint256 amount) internal {
        // Reset daily counter if a day has passed
        if (block.timestamp >= lastMintTimestamp + 1 days) {
            dailyMintedAmount = 0;
            lastMintTimestamp = block.timestamp;
        }
        
        // Check if minting would exceed daily limit
        if (dailyMintedAmount + amount > DAILY_MINT_LIMIT) {
            revert ExceedsDailyMintLimit(
                amount,
                DAILY_MINT_LIMIT - dailyMintedAmount
            );
        }
        
        dailyMintedAmount += amount;
    }
    
    /**
     * @dev Get remaining mintable amount for today
     * @return Remaining amount that can be minted today
     */
    function getRemainingDailyMintAmount() external view returns (uint256) {
        if (block.timestamp >= lastMintTimestamp + 1 days) {
            return DAILY_MINT_LIMIT;
        }
        return DAILY_MINT_LIMIT - dailyMintedAmount;
    }
    
    /**
     * @dev Override required by Solidity for multiple inheritance
     */
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal override(ERC20, ERC20Pausable) {
        super._beforeTokenTransfer(from, to, amount);
    }
    
    /**
     * @dev Get token information
     * @return Token details as a struct
     */
    function getTokenInfo() external view returns (
        string memory tokenName,
        string memory tokenSymbol,
        uint8 tokenDecimals,
        uint256 tokenTotalSupply,
        uint256 maxSupply,
        bool isPaused
    ) {
        return (
            name(),
            symbol(),
            decimals(),
            totalSupply(),
            MAX_SUPPLY,
            paused()
        );
    }
}
```

### Phase 3: Comprehensive Testing (2-3 hours)

#### 1. Unit Tests
```javascript
// test/SimpleToken.test.js
const { expect } = require("chai");
const { ethers } = require("hardhat");
const { loadFixture } = require("@nomicfoundation/hardhat-network-helpers");

describe("SimpleToken", function () {
  async function deployTokenFixture() {
    const [owner, addr1, addr2] = await ethers.getSigners();
    
    const SimpleToken = await ethers.getContractFactory("SimpleToken");
    const token = await SimpleToken.deploy(
      "SimpleToken",
      "STK",
      ethers.utils.parseEther("1000000") // 1 million initial supply
    );
    
    return { token, owner, addr1, addr2 };
  }
  
  describe("Deployment", function () {
    it("Should set the right owner", async function () {
      const { token, owner } = await loadFixture(deployTokenFixture);
      expect(await token.owner()).to.equal(owner.address);
    });
    
    it("Should assign the total supply to the owner", async function () {
      const { token, owner } = await loadFixture(deployTokenFixture);
      const ownerBalance = await token.balanceOf(owner.address);
      expect(await token.totalSupply()).to.equal(ownerBalance);
    });
    
    it("Should set correct token details", async function () {
      const { token } = await loadFixture(deployTokenFixture);
      expect(await token.name()).to.equal("SimpleToken");
      expect(await token.symbol()).to.equal("STK");
      expect(await token.decimals()).to.equal(18);
    });
  });
  
  describe("Transfers", function () {
    it("Should transfer tokens between accounts", async function () {
      const { token, owner, addr1 } = await loadFixture(deployTokenFixture);
      const transferAmount = ethers.utils.parseEther("100");
      
      await expect(token.transfer(addr1.address, transferAmount))
        .to.emit(token, "Transfer")
        .withArgs(owner.address, addr1.address, transferAmount);
      
      expect(await token.balanceOf(addr1.address)).to.equal(transferAmount);
    });
    
    it("Should fail if sender doesn't have enough tokens", async function () {
      const { token, addr1 } = await loadFixture(deployTokenFixture);
      const initialBalance = await token.balanceOf(addr1.address);
      
      await expect(
        token.connect(addr1).transfer(owner.address, ethers.utils.parseEther("1"))
      ).to.be.revertedWith("ERC20: transfer amount exceeds balance");
    });
  });
  
  describe("Minting", function () {
    it("Should allow owner to mint tokens", async function () {
      const { token, owner, addr1 } = await loadFixture(deployTokenFixture);
      const mintAmount = ethers.utils.parseEther("1000");
      
      await expect(token.mint(addr1.address, mintAmount))
        .to.emit(token, "Mint")
        .withArgs(addr1.address, mintAmount);
      
      expect(await token.balanceOf(addr1.address)).to.equal(mintAmount);
    });
    
    it("Should not allow non-owner to mint", async function () {
      const { token, addr1 } = await loadFixture(deployTokenFixture);
      
      await expect(
        token.connect(addr1).mint(addr1.address, ethers.utils.parseEther("1000"))
      ).to.be.revertedWith("Ownable: caller is not the owner");
    });
    
    it("Should not exceed max supply", async function () {
      const { token, addr1 } = await loadFixture(deployTokenFixture);
      const maxSupply = await token.MAX_SUPPLY();
      const currentSupply = await token.totalSupply();
      const excessAmount = maxSupply.sub(currentSupply).add(1);
      
      await expect(
        token.mint(addr1.address, excessAmount)
      ).to.be.revertedWithCustomError(token, "ExceedsMaxSupply");
    });
  });
  
  describe("Burning", function () {
    it("Should allow token burning", async function () {
      const { token, owner } = await loadFixture(deployTokenFixture);
      const burnAmount = ethers.utils.parseEther("100");
      const initialSupply = await token.totalSupply();
      
      await token.burn(burnAmount);
      
      expect(await token.totalSupply()).to.equal(initialSupply.sub(burnAmount));
    });
    
    it("Should allow burning from approved account", async function () {
      const { token, owner, addr1 } = await loadFixture(deployTokenFixture);
      const burnAmount = ethers.utils.parseEther("100");
      
      // Transfer tokens to addr1
      await token.transfer(addr1.address, ethers.utils.parseEther("200"));
      
      // Approve owner to burn from addr1
      await token.connect(addr1).approve(owner.address, burnAmount);
      
      await expect(token.burnFrom(addr1.address, burnAmount))
        .to.emit(token, "BurnFrom")
        .withArgs(addr1.address, burnAmount);
    });
  });
  
  describe("Pausable", function () {
    it("Should pause and unpause transfers", async function () {
      const { token, owner, addr1 } = await loadFixture(deployTokenFixture);
      
      // Pause the contract
      await token.pause();
      expect(await token.paused()).to.be.true;
      
      // Should not allow transfers when paused
      await expect(
        token.transfer(addr1.address, ethers.utils.parseEther("100"))
      ).to.be.revertedWith("Pausable: paused");
      
      // Unpause the contract
      await token.unpause();
      expect(await token.paused()).to.be.false;
      
      // Should allow transfers when unpaused
      await expect(
        token.transfer(addr1.address, ethers.utils.parseEther("100"))
      ).to.not.be.reverted;
    });
  });
});
```

### Phase 4: Deployment and Verification (1-2 hours)

#### 1. Deployment Script
```javascript
// scripts/deploy.js
const { ethers } = require("hardhat");

async function main() {
  const [deployer] = await ethers.getSigners();
  
  console.log("Deploying contracts with the account:", deployer.address);
  console.log("Account balance:", (await deployer.getBalance()).toString());
  
  // Deploy the token
  const SimpleToken = await ethers.getContractFactory("SimpleToken");
  const token = await SimpleToken.deploy(
    "SimpleToken",
    "STK",
    ethers.utils.parseEther("1000000") // 1 million initial supply
  );
  
  await token.deployed();
  
  console.log("SimpleToken deployed to:", token.address);
  console.log("Transaction hash:", token.deployTransaction.hash);
  
  // Wait for a few confirmations
  await token.deployTransaction.wait(5);
  
  // Verify the contract on Etherscan
  if (network.name !== "hardhat") {
    console.log("Verifying contract on Etherscan...");
    try {
      await hre.run("verify:verify", {
        address: token.address,
        constructorArguments: [
          "SimpleToken",
          "STK",
          ethers.utils.parseEther("1000000")
        ],
      });
      console.log("Contract verified successfully");
    } catch (error) {
      console.log("Verification failed:", error.message);
    }
  }
  
  // Save deployment info
  const deploymentInfo = {
    network: network.name,
    address: token.address,
    deployer: deployer.address,
    timestamp: new Date().toISOString(),
    constructorArgs: [
      "SimpleToken",
      "STK",
      ethers.utils.parseEther("1000000").toString()
    ]
  };
  
  console.log("Deployment completed:", deploymentInfo);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

### Phase 5: Frontend Integration (2-3 hours)

#### 1. React Token Dashboard
```javascript
// frontend/src/components/TokenDashboard.js
import React, { useState, useEffect } from 'react';
import { ethers } from 'ethers';
import TokenABI from '../contracts/SimpleToken.json';

const TOKEN_ADDRESS = process.env.REACT_APP_TOKEN_ADDRESS;

function TokenDashboard() {
  const [provider, setProvider] = useState(null);
  const [signer, setSigner] = useState(null);
  const [contract, setContract] = useState(null);
  const [account, setAccount] = useState('');
  const [balance, setBalance] = useState('0');
  const [tokenInfo, setTokenInfo] = useState({});
  const [transferTo, setTransferTo] = useState('');
  const [transferAmount, setTransferAmount] = useState('');
  
  useEffect(() => {
    initializeProvider();
  }, []);
  
  const initializeProvider = async () => {
    if (window.ethereum) {
      const provider = new ethers.providers.Web3Provider(window.ethereum);
      const signer = provider.getSigner();
      const contract = new ethers.Contract(TOKEN_ADDRESS, TokenABI.abi, signer);
      
      setProvider(provider);
      setSigner(signer);
      setContract(contract);
      
      // Get account
      const accounts = await provider.send("eth_requestAccounts", []);
      setAccount(accounts[0]);
      
      // Load token info
      await loadTokenInfo(contract, accounts[0]);
    }
  };
  
  const loadTokenInfo = async (contract, account) => {
    try {
      const [name, symbol, decimals, totalSupply, maxSupply, isPaused] = 
        await contract.getTokenInfo();
      const balance = await contract.balanceOf(account);
      
      setTokenInfo({
        name,
        symbol,
        decimals,
        totalSupply: ethers.utils.formatEther(totalSupply),
        maxSupply: ethers.utils.formatEther(maxSupply),
        isPaused
      });
      
      setBalance(ethers.utils.formatEther(balance));
    } catch (error) {
      console.error('Error loading token info:', error);
    }
  };
  
  const handleTransfer = async () => {
    if (!contract || !transferTo || !transferAmount) return;
    
    try {
      const tx = await contract.transfer(
        transferTo,
        ethers.utils.parseEther(transferAmount)
      );
      
      console.log('Transaction sent:', tx.hash);
      await tx.wait();
      console.log('Transaction confirmed');
      
      // Reload balance
      await loadTokenInfo(contract, account);
      setTransferTo('');
      setTransferAmount('');
    } catch (error) {
      console.error('Transfer failed:', error);
    }
  };
  
  return (
    <div className="token-dashboard">
      <h1>Simple Token Dashboard</h1>
      
      <div className="token-info">
        <h2>Token Information</h2>
        <p>Name: {tokenInfo.name}</p>
        <p>Symbol: {tokenInfo.symbol}</p>
        <p>Total Supply: {tokenInfo.totalSupply}</p>
        <p>Max Supply: {tokenInfo.maxSupply}</p>
        <p>Status: {tokenInfo.isPaused ? 'Paused' : 'Active'}</p>
      </div>
      
      <div className="account-info">
        <h2>Your Account</h2>
        <p>Address: {account}</p>
        <p>Balance: {balance} {tokenInfo.symbol}</p>
      </div>
      
      <div className="transfer-section">
        <h2>Transfer Tokens</h2>
        <input
          type="text"
          placeholder="Recipient address"
          value={transferTo}
          onChange={(e) => setTransferTo(e.target.value)}
        />
        <input
          type="number"
          placeholder="Amount"
          value={transferAmount}
          onChange={(e) => setTransferAmount(e.target.value)}
        />
        <button onClick={handleTransfer}>Transfer</button>
      </div>
    </div>
  );
}

export default TokenDashboard;
```

## 🎯 Advanced Features

### Gas Optimization
```solidity
// Gas-optimized batch operations
function batchTransfer(
    address[] calldata recipients,
    uint256[] calldata amounts
) external {
    require(recipients.length == amounts.length, "Arrays length mismatch");
    
    uint256 totalAmount = 0;
    for (uint256 i = 0; i < amounts.length; i++) {
        totalAmount += amounts[i];
    }
    
    require(balanceOf(msg.sender) >= totalAmount, "Insufficient balance");
    
    for (uint256 i = 0; i < recipients.length; i++) {
        _transfer(msg.sender, recipients[i], amounts[i]);
    }
}
```

### Token Vesting
```solidity
// Add vesting functionality
mapping(address => VestingSchedule) public vestingSchedules;

struct VestingSchedule {
    uint256 totalAmount;
    uint256 releasedAmount;
    uint256 startTime;
    uint256 duration;
}

function createVestingSchedule(
    address beneficiary,
    uint256 amount,
    uint256 duration
) external onlyOwner {
    vestingSchedules[beneficiary] = VestingSchedule({
        totalAmount: amount,
        releasedAmount: 0,
        startTime: block.timestamp,
        duration: duration
    });
    
    _transfer(msg.sender, address(this), amount);
}
```

## 📚 Learning Outcomes

After completing this project, you will have:
- Implemented a complete ERC-20 token contract
- Mastered Solidity basics and smart contract structure
- Applied security best practices and access control
- Built comprehensive test suites for smart contracts
- Deployed and verified contracts on testnets
- Integrated smart contracts with React frontends
- Understood gas optimization techniques

---

**Ready to build your token?** This project provides the foundation for all future smart contract development and introduces you to the core concepts you'll use throughout the course.

**Next Project:** [Voting System](../02-voting-system/README.md) - Learn governance mechanisms and democratic decision-making on the blockchain.

---

*This project demonstrates essential Solidity skills that every smart contract developer needs. Take your time to understand each concept thoroughly!*
