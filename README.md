# Hardhat Etherscan Verification

If you open [this](https://etherscan.io/address/0x7be8076f4ea4a4ad08075c2508e481d6c946d12b#writeContract) etherscan link, you can see that you can interact with this smart contract's functions directly through etherscan, similar to how you used to do it on Remix.

![](https://i.imgur.com/IiqNVYe.png)

Are you wondering why that doesnt happen for your contract?

- The reason is that the contract mentioned above is verified on etherscan while your's is not.

So lets learn why and how to verify contracts on etherscan 🚀

## Why?

- Verifying contracts is important because it ensures that the code is exactly what was deployed onto the blockchain
- It also allows the public to read and audit your smart contract code
- Contracts that are verified are considered to be more trust worthy than the ones which are not
- It also gives you an UI interface to interact with the contracts

## Why hardhat etherscan verification?

- Verifying the code manually on etherscan is very hard because you need to make sure that you not only verify your main contract but also the contracts that you are inheriting or using along with your main contract.
- If you deployed your contract for testing and verified it already with the slightest of changes to your contract you will have to again go through the manual process which gets tedious over time.

## Build

Lets now learn how we can leverage hardhat for verifying smart contracts with only some lines of code.

Lets goo 🚀🚀🚀

#### Write some code to verify
- Open up a terminal and execute these commands

    ```bash
    mkdir hardhat-verification
    cd hardhat-verification
    ```

- You will noe need to set up your Hardhat project

    ```bash
    npm init --yes
    npm install --save-dev hardhat
    ```

- In the same directory where you installed Hardhat run:

  ```bash
  npx hardhat
  ```

  - Select `Create a basic sample project`
  - Press enter for the already specified `Hardhat Project root`
  - Press enter for the question on if you want to add a `.gitignore`
  - Press enter for `Do you want to install this sample project's dependencies with npm (@nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers)?`

Now you have a hardhat project ready to go!

If you are not on mac, please do this extra step and install these libraries as well :)

```bash
npm install --save-dev @nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers
```

- We also need to install `hardhat-etherscan` npm package, in your terminal pointing to `hardhar-verification` folder execute this:

  ```bash
  npm install --save-dev @nomiclabs/hardhat-etherscan
  ```
- `hardhat-etherscan` npm package is the package from hardhat which will help us with etherscan verification.

- Now create a new file inside the `contracts` directory called `Verify.sol`.

  ```solidity
  //SPDX-License-Identifier: Unlicense
  pragma solidity ^0.8.4;

  contract Verify {
      string private greeting;

      constructor() {
      }

      function hello(bool sayHello) public pure returns (string memory) {
          if(sayHello) {
              return "hello";
          }
          return "";
      }
  }
  ```

- We will install `dotenv` package to be able to import the env file and use it in our config. Open up a terminal pointing at`hardhat-tutorial` directory and execute this command

  ```bash
  npm install dotenv
  ```

- Mumbai network is one of the testnet's on Polygon. We will learn today how to verify our contracts on mumbai.
- Now create a `.env` file in the `hardhat-tutorial` folder and add the following lines, use the instructions in the comments to get your `ALCHEMY_API_KEY_URL`, `MUMBAI_PRIVATE_KEY` and `POLYGONSCAN_KEY`.If you dont have Mumbai on MetaMask, you can follow [this](https://portal.thirdweb.com/guides/get-matic-on-polygon-mumbai-testnet-faucet) to add it to your MetaMask, make sure that the account from which you get your mumbai private key is funded with mumbai matic, you can get some from [here](https://faucet.polygon.technology/).

  ```bash
    // Go to https://www.alchemyapi.io, sign up, create
    // a new App in its dashboard and select the network as Mumbai, and replace "add-the-alchemy-key-url-here" with its key url
    ALCHEMY_API_KEY_URL="add-the-alchemy-key-url-here"

    // Replace this private key with your Mumbai account private key
    // To export your private key from Metamask, open Metamask and
    // go to Account Details > Export Private Key
    // Be aware of NEVER putting real Ether into testing accounts
    MUMBAI_PRIVATE_KEY="add-the-mumbai-private-key-here"

    // Go to https://polygonscan.com/, sign up, on your account overview page,
    // click on `API Keys`, add a new API key and copy the
    // `API Key Token`
    POLYGONSCAN_KEY="add-the-polygonscan-api-token-here"
  ```

- Lets deploy the contract to `mumbai` network. Create a new file named `deploy.js` under the `scripts` folder. Notice how we are using code to verify the contract.

  ```javascript
  const { ethers } = require("hardhat");
  require("dotenv").config({ path: ".env" });
  require("@nomiclabs/hardhat-etherscan");

  async function main() {
    /*
    A ContractFactory in ethers.js is an abstraction used to deploy new smart contracts,
    so verifyContract here is a factory for instances of our Verify contract.
    */
    const verifyContract = await ethers.getContractFactory("Verify");

    // deploy the contract
    const deployedVerifyContract = await verifyContract.deploy();

    await deployedVerifyContract.deployed();

    // print the address of the deployed contract
    console.log("Verify Contract Address:", deployedVerifyContract.address);

    console.log("Sleeping.....");
    // Wait for etherscan to notice that the contract has been deployed
    await sleep(10000);

    // Verify the contract after deploying
    await hre.run("verify:verify", {
      address: deployedVerifyContract.address,
      constructorArguments: [],
    });
  }

  function sleep(ms) {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }

  // Call the main function and catch if there is any error
  main()
    .then(() => process.exit(0))
    .catch((error) => {
      console.error(error);
      process.exit(1);
    });
  ```

- Now open the `hardhat.config.js` file, we will add the `mumbai` network here so that we can deploy our contract to mumbai and an `etherscan` object so that we can verify our contract on `polygonscan`. Replace all the lines in the `hardhart.config.js` file with the given below lines.

  ```javascript
  require("@nomiclabs/hardhat-waffle");
  require("dotenv").config({ path: ".env" });
  require("@nomiclabs/hardhat-etherscan");

  const ALCHEMY_API_KEY_URL = process.env.ALCHEMY_API_KEY_URL;

  const MUMBAI_PRIVATE_KEY = process.env.MUMBAI_PRIVATE_KEY;

  const POLYGONSCAN_KEY = process.env.POLYGONSCAN_KEY;

  module.exports = {
    solidity: "0.8.4",
    networks: {
      mumbai: {
        url: ALCHEMY_API_KEY_URL,
        accounts: [MUMBAI_PRIVATE_KEY],
      },
    },
    etherscan: {
      apiKey: {
        polygonMumbai: POLYGONSCAN_KEY,
      },
    },
  };
  ```

- Compile the contract, open up a terminal pointing at`hardhat-tutorial` directory and execute this command

  ```bash
    npx hardhat compile
  ```

- To deploy, open up a terminal pointing at `hardhat-tutorial` directory and execute this command

  ```bash
    npx hardhat run scripts/deploy.js --network mumbai
  ```

- It should have printed a link to mumbai polygonscan, your contract is now verified. Click on polygonscan link and interact with your contract there 🚀🚀🚀
