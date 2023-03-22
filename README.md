# Ethereum Smart Contract Simulator
An interface to simulate smart contracts. This simulator will use Remix IDE as an API to deploy smart contracts and experiment with different functions and variables. Think of this as a better interface for testing Remix deployed contracts.

## Code Snippets:
```
mkdir ESCS
cd ESCS
npm init -y
```
```
npm install ethers express body-parser cors
```
```
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');

const app = express();
app.use(bodyParser.json());
app.use(cors());

app.post('/compile-and-deploy', (req, res) => {
    // Add your logic for compilation and deployment here
    // using the ethers library and Remix IDE API.

    res.json({ message: 'Smart contract compiled and deployed.' });
});

app.listen(3000, () => {
    console.log('ESCS server listening on port 3000');
});
```
```
npm install -g create-react-app
```
```
create-react-app escs
```
```
cd escs
npm start
```
```
export default async function handler(req, res) {
  if (req.method === 'POST') {
    // Add your logic for compilation and deployment here
    // using the ethers library and Remix IDE API.

    res.status(200).json({ message: 'Smart contract compiled and deployed.' });
  } else {
    res.status(405).json({ message: 'Method not allowed.' });
  }
}
```
```
export default function middleware(req, res, next) {
  // Add middleware logic here

  return next();
}
```
```
npm install @remixproject/plugin
```
```
import { createClient } from '@remixproject/plugin';

async function compileSmartContract() {
  const client = createClient(new window.MessageChannel().port1);

  // Connect to the Remix IDE
  await client.onload();

  // Get the current file from the file explorer
  const currentFile = await client.call('fileManager', 'getCurrentFile');
  const fileContent = await client.call('fileManager', 'getFile', currentFile);

  // Compile the smart contract
  const compilationResult = await client.call('solidity', 'compile', fileContent);

  // Send the compilation result to your Next.js API route
  const response = await fetch('/api/compile-and-deploy', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ compilationResult }),
  });

  // Handle the response from the API route
  const responseData = await response.json();
  console.log(responseData);
}
```
```
export default async function handler(req, res) {
  if (req.method === 'POST') {
    const { compilationResult } = req.body;

    // Add your logic for deployment here using the ethers library.

    res.status(200).json({ message: 'Smart contract compiled and deployed.' });
  } else {
    res.status(405).json({ message: 'Method not allowed.' });
  }
}
```
```
import { createClient } from '@remixproject/plugin';

async function getDeployedContractDetails() {
  const client = createClient(new window.MessageChannel().port1);

  // Connect to the Remix IDE
  await client.onload();

  // Get the last compiled contract
  const lastCompilationResult = await client.call('solidity', 'getCompilationResult');
  const contractName = Object.keys(lastCompilationResult.contracts)[0];
  const contract = lastCompilationResult.contracts[contractName];
  const abi = contract.abiDefinition;

  // Get the deployed contract address
  const accounts = await client.call('udapp', 'getAccounts');
  const deployedContracts = await client.call('udapp', 'getDeployedContracts');
  const contractInstance = deployedContracts.find(c => c.name === contractName);
  const contractAddress = contractInstance ? contractInstance.address : null;

  if (!contractAddress) {
    console.error('Contract not deployed');
    return;
  }

  // Update the ESCS interface with the contract's address and ABI
  updateInterface(contractAddress, abi);
}
```
```
import { ethers } from 'ethers';

function updateInterface(contractAddress, abi) {
  // Initialize the ethers.js contract instance with the contract's address and ABI
  const provider = new ethers.providers.Web3Provider(window.ethereum);
  const signer = provider.getSigner();
  const contract = new ethers.Contract(contractAddress, abi, signer);

  // Render the contract's functions on the ESCS interface and set up event listeners for user interactions
  // ...

  // Example: Interact with the contract's functions using the ethers.js contract instance
  async function callContractFunction(functionName, args) {
    const result = await contract[functionName](...args);
    console.log(`Result of calling ${functionName}:`, result);
  }
}
```
```
-----------------------------------------------
| Ethereum Smart Contract Simulator       O   | <- O = Connect Wallet button
-----------------------------------------------
| Deployed Contracts        | Contract Info   |
|                           |                 |
| [+] Contract 1 (0x123...) | Name: Contract1 |
| [+] Contract 2 (0x456...) | Address: 0x123..|
|                           | Network: Testnet|
|                           |                 |
|                           | Read-only funcs |
| [Load Contract]           | func1() -> Res  |
|                           | func2(arg1)Res  |
|                           |                 |
|                           | Write functions |
|                           | func3(arg1) Tx  |
|                           | func4(arg1) Tx  |
-----------------------------------------------
| Footer: links, copyright, documentation     |
-----------------------------------------------
```
