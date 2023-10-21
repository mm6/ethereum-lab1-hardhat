## Fall 2023 Blockchain and SQL Fundamentals
### Carnegie Mellon University MSCF
### Due: Wednesday, November 8, 2023 11:59 PM
### 10 Points

<!--
#### Arjun Email: abrar@andrew.cmu.edu
-->


**Learning Objectives:** In this lab the student will set up an Ethereum
development environment (using Truffle and Ganache) and deploy four smart
contracts to an Ethereum test blockchain. The smart contracts will be
written in the Solidity programming language.

Three of these contracts will involve working with fungible tokens and the fourth contract is a non-fungible (NFT) contract.

We will experiment with running "transactions". Transactions cause state
changes, cost ether (for using gas), and may only return responses at a later time.

We will also run "calls". Calls do not change state on the blockchain and
cost nothing to run (no gas). They are processed immediately and provide
an immediate return value.

We will review the various views of the blockchain that Ganache provides.

Solidity event generation and a testing framework are also introduced and the student will modify an existing contract.

The contracts themselves will be studied in detail in class. For
now, we want to set up our system and get some experience deploying
and interacting with the Solidity code and the environment provided
by client side Truffle and server side Ganache.

It is important that you take your time and understand what is going on in these exercises. Start early and leave some time for debugging.

## Part 1. Installations

1) If you are a python programmer, you will want to deactivate your virtual environment during this
project. You may reactivate it later when using python. Use the following command:
```
(base) conda deactivate

```
2) Download and install Google Chrome.
3) Install the Metamask extension for Google Chrome.
   See: https://metamask.io/.
4) Install git.
   See https://git-scm.com/downloads.
5) Mac users need to install Xcode. See https://apps.apple.com/us/app/xcode/id497799835?mt=12.
6) Windows Home should be upgraded to Windows Education. Windows Education has CMU support.
7) Important: remove any existing version of Node.js.
8) Download and install a node version manager (nvm). We need to be particular about the version of Node.js
that we will use. The node version manager will allows us to change versions of Node.js.

   **On Windows** [There is guidance here on installing nvm-windows, node.js, and the node package manager npm.](https://learn.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-windows)

   **On a MAC** [First, install homebrew.](https://brew.sh/) [Second, use homebrew to install nvm.](https://tecadmin.net/install-nvm-macos-with-homebrew/)

9) This command should show the versions of Node.js available:

```
nvm ls-remote
```

10) We want to use Node.js version 16.15.0. Use the following command:

```
nvm install v16.15.0
v16.15.0 is already installed.
Now using node v16.15.0 (npm v8.5.5)

```

11) You should now be able to run Node and look at its version:

```
node --version
v16.15.0
```

12) Use npm to install Truffle.

   We are using version 5.5.23. Use the @ sign in the following command. Ignore the warnings.
   ```
   npm i -g truffle@5.5.23

   ```
13) You should now be able to run Truffle and look at its version. This is the way might
system looks. You may not see all of these components.

```
truffle version
Truffle v5.5.23 (core: 5.5.23)
Ganache v7.3.2
Solidity v0.5.16 (solc-js)
Node v16.15.0
Web3.js v1.7.4

```

14) Download and install a personal Ethereum blockchain - Ganache.
   See: https://trufflesuite.com/ganache/

   <!-- NOTE: The current release of Ganache is only available for x86 architectures.
   On newer M1 and M2 Mac devices, this can still be run by installing Rosetta
   (See: https://support.apple.com/en-us/HT211861). -->

15) If you want to use Visual Studio Code as your editor (recommended) , download and install Visual Studio Code.
    See: https://code.visualstudio.com

16) If you are using Visual Studio Code, you may want to download and install solidity.
   See: https://marketplace.visualstudio.com/items?itemName=JuanBlanco.solidity


## Part 2  Deploying contracts to Ganache (Modified from "Mastering Ethereum" by Antonopoulos and Wood)

1) Run ganache quick start and leave it running for the remainder of this part.
2) Create a new project directory named DBUC_Lab1_Part2 and cd into it.
3) Execute the command:

   ```sh

   truffle init

   ```

4) Examine the directory structure:

   * **contracts** holds solidity source code
      * Migrations.sol is a deployment contract that we do not normally change.
   * **migrations** holds javascript code for efficient redeployments
      * Files here are numbered to specify the order of deployments.
      * For example, 1_initial_migrations.js would run first. 2_deploy_migrartions.js would run second and so on.
      * The first migration would normally deploy the Migrations.sol contract.
   * **test**
      * This directory is for writing tests in Javascript.
It typically uses the mocha framework and Chai Assertions library.
In more complex deployments, this directory structure will mirror
the directory structure required by the application.
   * **truffle-config.js**

      * This Javascript file contains configuration parameters for this truffle project.
      * You may select your Solidity compiler version in this file.

5) Create a new contract in the contracts directory. This file will be named
   Faucet.sol. The content of Faucet.sol is:

   ```js
   pragma solidity ^0.8.0;
   contract Faucet {
      function withdraw(uint withdraw_amount) public {
         require(withdraw_amount <= 100000000000000000000);
         payable(msg.sender).transfer(withdraw_amount);
      }

      fallback() external payable {}
   }
   ```
6) Create a new migration file in the migrations directory. Do
   not remove the initial migration 1_initial_migration.js.
   This new file will be named 2_deploy_migration.js.
   This file will contain a migration script to deploy Faucet.sol.
   The content of 2_deploy_migrations.js is:
   ```js

   // Javascript in 2_deploy_migrations.js to deploy Faucet.sol
   var Faucet = artifacts.require("./Faucet.sol");
   module.exports = function(deployer) {
     deployer.deploy(Faucet);
   };

   ```

7) To create a package.json file, run the following command
   from within the project directory:

 ```sh
   npm init
 ```

   Take the suggested defaults.

8) From the project directory, run the command

 ```sh

  npm install dotenv truffle-wallet-provider ethereumjs-wallet

 ```
   This creates a node_modules directory. Ignore the warnings.

9) To compile and deploy the two contracts (Migrations.sol and Faucet.sol)
   to the blockchain (represented by Ganache), make sure that Ganache is running
   and execute the command:

 ```sh
   truffle migrate --reset
 ```

10) To interact with the deployed contract and accounts, use the truffle console.

      a. From the command line, execute the following commands:

      ```sh
      truffle console
      ```

      b. Access the contract with an asynchronous request. Use a
       callback function and a promise. The callback
       function is defined within the "then" clause.
       Execute the following command within the truffle console.

      ```js
      Faucet.deployed().then(function(x){ myApp = x; });
      ```

      The response should be 'undefined'.

      c. To view the response enter the name myApp.

      ```
      myApp
      ```
      d. To get access to a web3 object, enter three lines of Javascript.
      The first two will return 'undefined'.

      ```js
      var Web3 = require('web3');
      var web3 = new Web3(new Web3.providers.HttpProvider('http://127.0.0.1:7545'));
      web3.isConnected() // should return true if all three lines worked.
      ```
      e. Get the balance on the contract.

      ```js
      contractBalance = web3.eth.getBalance(Faucet.address).toNumber()
      ```

      f. View the account addresses available on Ganache:

      ```js
      web3.eth.accounts
      ```

      g. View the first address. This address is our default address provided
            by Ganache. We are in possession of its private key. Ganache has
            provided this account with 100 eth (but only usable on Ganache).

      ```js
      web3.eth.accounts[0]
      ```

      h. View the contract address.

      ```js
      Faucet.address
      ```

      i. Using web3, transfer eth from the first account to the contract.
      The value returned is the transaction hash. Check the logs in Ganache.

      ```js
      web3.eth.sendTransaction({from:web3.eth.accounts[0],to:Faucet.address,value:web3.toWei(0.5, 'ether')})
      ```

      j. Check that the balance on the contract is higher than before.

      ```js
      web3.eth.getBalance(Faucet.address).toNumber();
      ```

      k. Withdraw some eth from the contract and deposit to account[0]. This returns 'undefined'.

      ```js
      Faucet.deployed().then(instance => {receipt = instance.withdraw(web3.toWei(0.1,'ether'))});
      ```

      l. But we can examine the returned receipt.

      ```js
      receipt
      ```
      m. Check the balance on account[0]. It should be approximately 99590121300000000000. Your answer is in wei and may vary a bit due to different gas fees.

      ```js
      web3.eth.getBalance(web3.eth.accounts[0]).toNumber();
      ```

      n. To exit the truffle console hit control d.

      ```
      truffle(ganache)>ctrl+d
      ```

:checkered_flag:**11) At this point, take three screenshots of Ganache. Take a screenshot of your Ganache Accounts, Blocks, and Transactions. Place these in a clearly labeled single Word or PDF document named Lab1Part2.doc or Lab1Part2.pdf. **


## Part 3 Deploying Contracts to Ganache (Modified from "Mastering Ethereum" by Antonopoulos and Wood)

1) Create a new project directory named DBUC_Lab1_Part3 and cd into it.

2) Execute the command:

```sh
   truffle init
```
3) Run a new instance of ganache. Configure Ganache as follows:

   Select "new workspace".

   Select "add project".

   Choose the file "truffle-config.js" in your "DBUC_Lab1_Part3" project directory.

   Provide the workspace with an appropriate name.

   Save the workspace.

   On the server side (Ganache) you should now be able to view the following tabs:
   Accounts, Blocks, Transactions, Contracts, Events, and Logs. These are all now
   associated with your project.

4) Create a new contract in the contracts directory and name it Faucet.sol.

5) [Copy this code into Faucet.sol.](../../blob/master/Faucet.sol)

6) Next, you will deploy this new contract to Ganache.

      a. Create a file named 2_deploy_migrations.js in the migrations directory:

      ```js
      // Javascript in 2_deploy_migrations.js to deploy Faucet.sol
      var Faucet = artifacts.require("./Faucet.sol");
      module.exports = function(deployer) {
      deployer.deploy(Faucet);
      };
      ```
      b. To create a package.json file, run the following command
         from within the project directory:

      ```sh
         npm init
      ```
      Take the suggested defaults.

      c. From the project directory, run the command

      ```sh

         npm install dotenv truffle-wallet-provider ethereumjs-wallet

      ```
      This creates a node_modules directory. Ignore the warnings.

      d. Compile and deploy the contracts:
      ```sh
      truffle migrate --reset
      ```

      e. Use the console to access the contract and establish a web3 connection:

      From the command line, execute the following commands:

      ```sh
      truffle console
      ```
      Within the console, run the following:

      ```js
      Faucet.deployed().then(function(x){ myFaucet = x; });
      ```

      Within the console, run the following to access web3:


      ```js
      var Web3 = require('web3');
      var web3 = new Web3(new Web3.providers.HttpProvider('http://127.0.0.1:7545'));
      web3.isConnected() // should return true if all three lines worked.
      ```

7) Send a total of 2 ether to the contract with these two commands:

```js
myFaucet.send(web3.toWei(1,"ether")).then(res => { console.log(res.logs[0].event)})
```
```js
myFaucet.send(web3.toWei(1,"ether")).then(res => { console.log(res.logs[0].event, res.logs[0].args)})
```

8) Using Ganache, examine the balance stored in the contract.

9) Send a "withdraw" transaction to the contract. This will be a request to withdraw 2 eth.

In your own words, describe what happens on the client.

In your own words, describe why this failed.

10) Send a single request to withdraw 0.1 eth from the contract.

Show the receipt that is returned from this request.

11)  Make enough of these "0.1 eth withdrawals" from the contract so that it (the contract) runs out of eth. We are interested in the first request that causes the following "require" statement to fail:

```
require((address(this)).balance >= withdraw_amount,"Balance too small for this withdrawal");
```
Force this "require" to fail and show the client side error.

12)  Show a screenshot showing the balance and storage associated with your Faucet contract. The balance and storage associated with a contract are found on the Ganache user interface.

13)  Show a screenshot showing the transactions and events that are associated with your Faucet contract. The transactions and events associated with a contract are found on the Ganache user interface.

Part 3 Submission summary:

     Question 9 Two paragraphs
     Question 10 Transaction receipt
     Question 11 Client side error
     Question 12 screenshot
     Question 13 screenshot

:checkered_flag:**14) Place your submissions in a clearly labeled single Word or PDF document named Lab1Part3.doc or Lab1Part3.pdf.**

## Part 4 Using a Truffle box with MetaCoin.sol, ConvertLib.sol and the testing framework

1) Create a new project directory named DBUC_Lab1_Part4 and cd into it.

2) Execute the command:

```sh
truffle unbox metacoin

```

3) Edit the file truffle-config.js and remove most comment symbols so that the file appears as shown:

```js
module.exports = {

  networks: {
  },

  // Set default mocha options here, use special reporters, etc.
  mocha: {
    // timeout: 100000
  },

  // Configure your compilers
  compilers: {
    solc: {
      version: "0.8.15",      // Fetch exact version from solc-bin (default: truffle's version)
      // docker: true,        // Use "0.5.1" you've installed locally with docker (default: false)
      // settings: {          // See the solidity docs for advice about optimization and evmVersion
      //  optimizer: {
      //    enabled: false,
      //    runs: 200
      //  },
      //  evmVersion: "byzantium"
      // }
    }
  },

};

```

4) Run a new instance of ganache and configure Ganache as before:

   a. Select "new workspace".

   b. Select "add project".

   c. Choose the file "truffle-config.js" in your "DBUC_Lab1_Part4" project directory.

   d. Provide the workspace with an appropriate name.

   e. Save the workspace.

5) Notice that you have two contracts in the contracts subdirectory. One of these is ConvertLib.sol and the other is MetaCoin.sol.

6) Study MetaCoin.sol and ConvertLib.sol and then run the following commands in order.

Note that most of these commands are 'calls' and cost no gas. The transaction sendCoin,
however, costs gas and generates a receipt. Make a copy of this receipt (from sendCoin) for submission.


```sh
truffle migrate --reset
truffle console
```

```js

metaCoinInstance = await MetaCoin.deployed()

```

```

balance = await metaCoinInstance.getBalance.call(accounts[0])
```

```
balance
```
```
balance.toNumber()

```

```js
metaCoinBalance = (await metaCoinInstance.getBalance.call(accounts[0])).toNumber()
```
```js

metaCoinBalance
```


```js
metaCoinEthBalance = (await metaCoinInstance.getBalanceInEth.call(accounts[0])).toNumber()

```
```
metaCoinEthBalance
```

```js

accountOne = accounts[0]
```
```js
accountTwo = accounts[1]
```
```js
accountOneStartingBalance = (await metaCoinInstance.getBalance.call(accountOne)).toNumber()
```
```
accountOneStartingBalance
```
```
accountTwoStartingBalance = (await metaCoinInstance.getBalance.call(accountTwo)).toNumber()
```
```
accountTwoStartingBalance
```
```
amount = 10

```
Keep a copy of the receipt generated by the sendCoin transaction.

```js
metaCoinInstance.sendCoin(accountTwo, amount, { from: accountOne })
```

```js
accountOneEndingBalance = (await metaCoinInstance.getBalance.call(accountOne)).toNumber()
```
```js
accountOneEndingBalance
```
```js
accountTwoEndingBalance = (await metaCoinInstance.getBalance.call(accountTwo)).toNumber()
```
```js
accountTwoEndingBalance
```

7) Exit the console (ctrl-D) and examine the directory named test. It contains a file named
metacoin.js that is provided as part of the truffle box. Look over the javascript code and
compare it with the code we entered above in the console.

8) From the project directory, execute the test code by running:

 ```sh
truffle tests
 ```
9) Take a screenshot of the command line output. That is, show the result of running the 5 tests.

10) Add a new Event to the MetaCoin contract. This event will be fired when
sendCoin is called with an inappropriate transfer of funds. You need to
submit three items:

    a) Submit a copy of your new MetaCoin.sol contract - include your name in the
    program comments.

    b) Show a screenshot of your command line that is testing this event. That
    is, show an example of making a call where the sender has insufficient funds.

    c) Show the transaction receipt that is returned to the caller. This transaction receipt will contain the following line:

    ```js
    event: 'Insufficient_Funds'
    ```

    d) Submit a screenshot of the Ganache Events screen showing the details of the Insufficient_Funds Event.


 11) Note that, from the project directory, you can compile your Solidity code by running:

```sh
  truffle compile
```

Part 4 Submission summary:

              Question 6 Submit a copy of the receipt generated by sendCoin.

              Question 9. Submit a screenshot of the command line output.

              Question 10 a. Submit a modified MetaCoin contract (including your name).

              Question 10 b. Submit a command line screenshot.

              Question 10 c. Submit a copy of the receipt showing insufficient funds.

              Question 10 d. Submit a screenshot of the Ganache Events screen showing the details of the Insufficient_Funds Event.

:checkered_flag:**12) Place your submissions in a clearly labeled single Word or PDF document named Lab1Part4.doc or Lab1Part4.pdf.**


## Part 5 Creating an NFT using Ganache

In this part, we will use the Interplanetary File System (IPFS) and Ganache to store and establish ownership of a non-fungible token (NFT).

The Interplanetary File System (IPFS) is a decentralized file system. The idea is, instead of giving
a name to a file and letting it live on your local machine, we use the hash of the file's content as
the name of the file and store it on a peer on the IPFS network. In this exercise, we will be storing the
file locally (but using IPFS commands).

We will deploy a non-fungible token (NFT) smart contract and store the hash of a metadata file in
the contract's memory. The metadata file and the file itself will be stored on IPFS.

1) Using the directions found here, install the Interplanetary File System (IPFS).

https://docs.ipfs.tech/install/command-line/#system-requirements

   You want the IPFS Kubo for Go. You can select your platform and install the official binary distribution.
   Windows users: it is recommended that you use powershell (instead of command prompt) for these steps.

2) Using these directions, initialize the IPFS repository.

https://docs.ipfs.tech/how-to/command-line-quick-start/

3) These instructions are modified from the video found next. But we are not using the same test network or wallet as shown in the video. Watch the video but follow the directions below.

   https://www.youtube.com/watch?v=IFpU4TNwXec

    a) Create an empty directory and name it nft. Within the nft directory, run

      ```sh
      truffle init
      ```
    b) In the same directory run
    ```sh
    npm install @openzeppelin/contracts
    ```

    c) Within the contracts directory, create UniqueAsset.sol.
    ```js
   // UniqueAsset.sol
   // SPDX-License-Identifier: MIT
   pragma solidity ^0.8.0;
   import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
   import "@openzeppelin/contracts/utils/Counters.sol";
   contract UniqueAsset is ERC721URIStorage {
     using Counters for Counters.Counter;
     Counters.Counter private _tokenIds;
     constructor() ERC721("UniqueAsset","UNA") {}
     function awardItem(address recipient, string memory metadata)
     public returns (uint256) {
       _tokenIds.increment();
       uint256 newItemId = _tokenIds.current();
       _mint(recipient,newItemId);
       _setTokenURI(newItemId,metadata);
      return newItemId;
     }
   }
   ```
   d. Within the migrations directory, create 2_deploy_contracts.js.

   ```js
   const UniqueAsset = artifacts.require("UniqueAsset");
   module.exports = function(deployer) {
      deployer.deploy(UniqueAsset);
   }
   ```
   e) In a shell in the nft directory, run

   ```sh
    npm install fs
   ```
   f) In a shell in the nft directory, run
   ```sh
    npm install @truffle/hdwallet-provider@1.2.3
   ```
   g) Modify truffle-config.js so that it has a compiler version of 0.8.1 and set docker to false. The compiler section should appear as follows:
   ```js
    // Configure your compilers
    compilers: {
      solc: {
        version: "0.8.1",    // Fetch exact version from solc-bin (default: truffle's version)
        docker: false,        // Use "0.5.1" you've installed locally with docker (default: false)
        settings: {          // See the solidity docs for advice about optimization and evmVersion
        optimizer: {
        enabled: false,
        runs: 200
        },
        evmVersion: "byzantium"
        }
      }
    },
   ```
   h) From the nft directory, run
   ```sh
      mkdir credential
   ```
   i) Place a simple file in the credential directory
   ```sh
      echo "This is an important credential" > credential.txt
   ```
   j) Add the credential file to ipfs and make a copy of the content identifier (CID). The CID begins with "Qm".
   ```sh
      ipfs add credential.txt
   ```
   k) Add this metadata file to the credential directory. Name it credentialMetadata.json. Include the CID associated with credential.txt.
   ```json
   {
     "name" : "My cool credential",
     "description" : "This is a credential that I worked very hard to attain.",
     "file" : "https//ipfs.io/ipfs/THE_CREDENTIAL_CID_GOES_HERE"
   }
   ```
   l) Add the metadata file to ipfs:
   ```sh
      ipfs add credentialMetadata.json
   ```
   m) Examine your metadata file using ipfs:
   ```sh
      ipfs cat /ipfs/THE_METADATA_CID_GOES_HERE
   ```
   n) From the nft directory, compile the nft contract:
   ```sh
   truffle compile
   ```
   You will get some warnings, ignore them.

   o) Run Ganache Workspace and point to nft/truffle-config.js.

   p) From the nft directory, deploy the NFT contract to Ganache:
   ```sh
   truffle migrate
   ```
   q) Run the Truffle console:
   ```sh
   truffle console
   ```
   r) Access the contract:
   ```js
   let contract = await UniqueAsset.deployed();
   ```
   s) Visit Ganache and make a copy of the first account address (include the "0x"). This becomes the first argument to the awardItem call. Use the CID of the metadata file as the second argument. Run the following command in the truffle console:
   ```js
   let result = await contract.awardItem("ACCOUNT_ADDR_GOES_HERE","https//ipfs.io/ipfs/THE_META_DATA_CID_GOES_HERE")
   ```
   t) Examine the name of the contract:
   ```js
   let nameOfToken = await contract.name()
   ```
   ```js
   nameOfToken
   ```

   u) Examine the balance of the first account:
   ```js
   let balance = await contract.balanceOf("ACCOUNT_ADDR_GOES_HERE")
   ```
   ```js
   balance.toNumber()
   ```
   v) Examine the balance of the second account. Fill in the blank.

   w) Who is the owner of the Token ID 1?
   ```js
    let owner = await contract.ownerOf("1")
   ```
   ```js
    owner
   ```
   x) Transfer the token to the second account. Save the receipt.
   ```js
   account_from = "0xTHE_ADDRESS_OF_THE_FIRST_ACCOUNT"
   ```
   ```js
   account_to = "0xTHE_ADDRESS_OF_THE_SECOND_ACCOUNT"
   ```
   ```js
   let transfer = await contract.transferFrom(account_from, account_to, 1)
   ```
   ```js
   transfer
   ```
   y) Who is the new owner? Show the command that you use to learn who the new owner is. Fill in the blank.

   :checkered_flag:**11)Place a copy of the transaction receipt from step 3 x) and your answer to question 3 y) in a clearly labeled single Word or PDF document named Lab1Part5.doc or Lab1Part5.pdf.**

   :checkered_flag:**Place your four submission documents and modified Metacoin contract in a single directory and zip that directory. Name the zip file <your-andrew-id>Lab1.zip. Submit this single zip file to Canvas.**

## Grading rubric for the materials in the submission directory

    2 points for completion of Part 2
    0.25 points for correct submission of Part 2
    2 points for completion of Part 3
    0.25 points for correct submission of Part 3
    2 points for completion of Part 4
    0.25 points for correct submission of Part 4
    2 points for completion of Part 5
    0.25 points for correct submission of Part 5

## Penalty for any late  work

    -1 point per day late
