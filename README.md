# Aave Flash Loan Arbitrage using Hardhat 
## Sample Hardhat Project

This project demonstrates a basic Hardhat use case. It comes with a sample contract, a test for that contract, and a script that deploys that contract.

## Step 1: Initialize a New Hardhat Project
Open your terminal and navigate to your desired project directory. Run the following commands:

npm install --save-dev hardhat
npx hardhat
Follow the prompts to create a new Hardhat project. Choose the default settings for simplicity.

## Step 2: Install Dependencies
We’ll need to install some additional dependencies for our project. Open your terminal and run the following commands:
```
npm install --dev @nomiclabs/hardhat-ethers@npm:hardhat-deploy-ethers ethers @nomiclabs/hardhat-etherscan @nomiclabs/hardhat-waffle chai ethereum-waffle hardhat hardhat-contract-sizer hardhat-deploy hardhat-gas-reporter prettier prettier-plugin-solidity solhint solidity-coverage dotenv
npm install @aave/core-v3
```
## Step 3: Project Structure
Your project directory should now have the following structure:
```
- contracts/
  - FlashLoanArbitrage.sol
  - Dex.sol
- deploy/
  - 00-deployDex.js
  - 01-deployFlashLoanArbitrage.js  
- scripts/
- test/
- hardhat.config.js
- package.json
- README.md
```
create .env file, add both SEPOLIA_RPC_URL and PRIVATE_KEY by your proper credentials as follows:
Open hardhat.config.js, and update it.
Additionally, create a new file called helper-hardhat-config.js in the root directory and add the following needed details keeping in ming that we’re using the Sepolia test network for all save addresses:
 1. PoolAddressesProvider,
 2. daiAddress,
 3. usdcAddress
After all the adjustments made above, here’s how our project structure should look:
```
- contracts/
  - FlashLoanArbitrage.sol
  - Dex.sol
- deploy/
  - 00-deployDex.js
  - 01-deployFlashLoanArbitrage.js  
- scripts/
- test/
-.env
- hardhat.config.js
-helper-hardhat-config
- package.json
- README.md
```
## Step 4: Contracts
In this tutorial, we’ll be working with two smart contracts:

Dex.sol: This contract simulates a decentralized exchange where arbitrage opportunities occur.
FlashLoanArbitrage.sol: This contract handles flash loans and arbitrage operations.
Let’s dive into these contracts and understand each line of code. First, we’ll explore FlashLoanArbitrage.sol.
Read the Dex Contract Explanation:

The Dex.sol contract simulates a decentralized exchange. Let's dive into its key features:
```
Dex Contract: The main contract defines storage variables for the owner, DAI and USDC addresses, and instances of IERC20 for DAI and USDC.
Token Deposits: depositUSDC and depositDAI functions allow users to deposit USDC and DAI tokens, updating their balances.
Token Exchange: buyDAI and sellDAI functions simulate token exchanges, buying DAI with USDC and selling DAI for USDC based on exchange rates.
Balance Tracking: The contract tracks individual user balances for DAI and USDC with mappings.
Token Withdrawal: The withdraw function enables the contract owner to withdraw tokens from the contract.
```

Read the FlashLoanArbitrage Contract Explanation:

The FlashLoanArbitrage.sol contract is the core of our arbitrage strategy. It utilizes Aave flash loans to execute profitable trades. Let's break down the key aspects of the contract:
### Imports and Interfaces
- Import required contracts and interfaces from Aave and OpenZeppelin.
  - FlashLoanSimpleReceiverBase
  - IPoolAddressesProvider
  - IERC20

### IDex Interface
- Define the interface for the decentralized exchange (DEX).
- Include methods like depositUSDC, depositDAI, buyDAI, and sellDAI.

### FlashLoanArbitrage Contract
- Inherits from...
- Initializes addresses and contracts for DAI, USDC, and the DEX.
- Implements the executeOperation function for arbitrage.

### Flash Loan Execution
- Execute arbitrage operation within the executeOperation function.
- Steps: deposit funds, buy DAI, deposit DAI, and then sell DAI.

### Loan Repayment
- Repay flash loan amount plus premium to Aave.
- Approve the Pool contract to pull the owed amount.

### Flash Loan Request
- Initiate flash loan using the requestFlashLoan function.
- Call flashLoanSimple from the POOL contract.
# Step 5: Deploying Smart Contracts
Deploying your smart contracts is the next crucial step. Let’s take a look at the deployment scripts.

### 00-deployDex.js
### 01-deployFlashLoanArbitrage.js
### Token Approval and Allowance
- Include functions like approveUSDC, approveDAI, allowanceUSDC, and allowanceDAI.
- Approve tokens and check allowances for the DEX.

### Balance Inquiry and Withdrawal
- The getBalance function checks the balance of a token.
- The withdraw function allows the contract owner to withdraw tokens.
Let’s start by deploying the Dex.sol contract:
```
npx hardhat deploy --tags dex --network sepolia
```
```
The Dex contract address is “0x82d6c12e57a3eb6E18fCc791fFbD84615BDb2a31” which is added to the FlashLoanArbitrage contract
```

Then we deploy FlashLoanArbitrage.sol by running the command below:
```
npx hardhat deploy --tags FlashLoanArbitrage --network sepolia
```
```
which outputs the FlashLoanArbitrage contract address below:
“0xc30b671E6d94C62Ee37669b229c7Cd9Eab2f7098”
```
## Step 5: Testing Smart Contracts
Let’s now text out the contracts using Remix IDE, but to be more specific here’s where the Flash Loan Arbitrage:
```
// exchange rate indexes
uint256 dexARate = 90;
uint256 dexBRate = 100;
```
Here, we buy 1DAI at 0.90 and sell it at 100.00 .
When coping the code to Remix IDE, consider change the imports below in both contracts accordingly:
```
import {IERC20} from "@aave/core-v3/contracts/dependencies/openzeppelin/contracts/IERC20.sol";
import {FlashLoanSimpleReceiverBase} from "https://github.com/aave/aave-v3-core/blob/master/contracts/flashloan/base/FlashLoanSimpleReceiver.sol";
import {IPoolAddressesProvider} from "https://github.com/aave/aave-v3-core/blob/master/contracts/interfaces/IPoolAddressesProvider.sol";
```
### Add liquidity to Dex.sol:
- USDC 1500
- DAI 1500
### Approve:
- USDC 1000000000
- DAI 1200000000000000000000
### Request Loan — USDC (6 decimal):
- 0xda9d4f9b69ac6C22e444eD9aF0CfC043b7a7f53f,1000000000 // 1,000 USDC
Here’s the transtions explanation:

- Transfer 1000 USDC from the aave LendingPool contract toFlashLoanArbitrage contract,
- Deposit 1000 USDC from the FlashLoanArbitrage contract to Dex contract,
- Purchase of DAI From Dex contract to FlashLoanArbitrage contract,
- Deposit of the DAI amount in the Dex contract,
- Transfer 1,111.1111 USDC from Dex contract to FlashLoanArbitrage
- Rapay 1000 USDC + 0.05% of flashloan fee (1000.5 USDC).


After a successfull tranaction, we’ll chek back our balalnce wich increases up to 110.611100 USDC
