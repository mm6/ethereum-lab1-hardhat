## Fall 2023 Blockchain and SQL Fundamentals
### Carnegie Mellon University MSCF
### Due: Wednesday, November 8, 2023 11:59 PM
### 10 Points

<!--
#### Arjun Email: abrar@andrew.cmu.edu
-->


**Learning Objectives:** In this lab the student will set up an Ethereum
development environment (using Node.js and Hardhat) and deploy smart
contracts that are written in the Solidity programming language.

We will experiment with running "transactions". Transactions cause state
changes, cost ether (for using gas), and may only return responses at a later time.

We will also run "calls". Calls do not change state on the blockchain and
cost nothing to run (no gas). They are processed immediately and provide
an immediate return value.

Solidity event generation and a testing framework are also introduced
and the student will modify an existing contract.

The contracts themselves will be studied in detail in class. For
now, we want to set up our system and get some experience deploying
and interacting with the Solidity code and the environment provided
by client side Hardhat and the server side Hardhat network.

In class, we will delve into the specifics of the contracts. However,
for the time being, our goal is to establish our system and gain
some practical experience in deploying and interacting with the
Solidity code. We’ll be using the Hardhat environment on both the
client and server sides.

This lab exercise is based partly on the Hardhat tutorial [found here.](https://hardhat.org/tutorial)

## Part 1. Installations

1. If you are a python programmer, you may want to deactivate your virtual
environment during this project. You may reactivate it later when using
python. Use the following command:
```
(base) conda deactivate

```
2. Download and install Google Chrome.
3. Install the Metamask extension for Google Chrome.
   See: https://metamask.io/.
4. Install git.
   See https://git-scm.com/downloads.
5. Download and install a node version manager (nvm).

   **On Windows** [There is guidance here on installing nvm-windows, node.js, and the node package manager npm.](https://learn.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-windows)

   **On a MAC** [First, install homebrew.](https://brew.sh/) [Second, use homebrew to install nvm.](https://tecadmin.net/install-nvm-macos-with-homebrew/)

6. This command should show the versions of Node.js available:

```
nvm ls-remote
```

7. We want to use Node.js version 20.8.1. Use the following command:

```
nvm install 20
nvm use 20
Now using node v20.8.1 (npm v10.2.1)

```

8. You should now be able to run Node.js and look at its version:

```
node --version
v20.8.1
```

9. Create a new project directory named HardhatLab1.

```
mkdir HardhatLab1

```
10. Change directory into the new directory.

```
cd HardhatLab1
```
Build a package.json file (holding important information about this project) by running the command:

```
npm init     hit return and take the defaults provided

```

11. Now, within the HardhatLab1 directory, install hardhat:

```
npm install --save-dev hardhat

```
12. Next, within the HardhatLab1 directory, initialize Hardhat with the Node Package Execute (npx) command:

```
npx hardhat init
```

You will need to select "Create an empty hardhat.config.js".

13. The npx command creates a hardhat.config.js. In addition, if you are asked to do so,
run the following npm command.

```
npm install --save-dev "hardhat@^2.18.2"

```

14. Within the project directory, install the Hardhat toolbox:

```
npm install --save-dev @nomicfoundation/hardhat-toolbox

```
We need access to that toolbox. Edit your hardhat.config.js file and include this line
at the top:

```
require("@nomicfoundation/hardhat-toolbox");
```

Create a new subdirectory named contracts. Within contracts, create the following smart
contract named Token.sol:

```
//SPDX-License-Identifier: UNLICENSED

// Solidity files have to start with this pragma.
// It will be used by the Solidity compiler to validate its version.
pragma solidity ^0.8.0;


// This is the main building block for smart contracts.
contract Token {
    // Some string type variables to identify the token.
    string public name = "My Hardhat Token";
    string public symbol = "MHT";

    // The fixed amount of tokens, stored in an unsigned integer type variable.
    uint256 public totalSupply = 1000000;

    // An address type variable is used to store ethereum accounts.
    address public owner;

    // A mapping is a key/value map. Here we store each account's balance.
    mapping(address => uint256) balances;

    // The Transfer event helps off-chain applications understand
    // what happens within your contract.
    event Transfer(address indexed _from, address indexed _to, uint256 _value);

    /**
     * Contract initialization.
     */
    constructor() {
        // The totalSupply is assigned to the transaction sender, which is the
        // account that is deploying the contract.
        balances[msg.sender] = totalSupply;
        owner = msg.sender;
    }

    /**
     * A function to transfer tokens.
     *
     * The `external` modifier makes a function *only* callable from *outside*
     * the contract.
     */
    function transfer(address to, uint256 amount) external {
        // Check if the transaction sender has enough tokens.
        // If `require`'s first argument evaluates to `false` then the
        // transaction will revert.
        require(balances[msg.sender] >= amount, "Not enough tokens");

        // Transfer the amount.
        balances[msg.sender] -= amount;
        balances[to] += amount;

        // Notify off-chain applications of the transfer.
        emit Transfer(msg.sender, to, amount);
    }

    /**
     * Read only function to retrieve the token balance of a given account.
     *
     * The `view` modifier indicates that it doesn't modify the contract's
     * state, which allows us to call it without executing a transaction.
     */
    function balanceOf(address account) external view returns (uint256) {
        return balances[account];
    }
}
```
15. Using Node Package Execute (npx), compile the code with the following command. We do this in the
directory just above the contracts directory.

Note that this command will download the appropriate compiler.

```
npx hardhat compile

```

16. Create a new directory named test and place the following Javascript code in the test directory in a file named Token.js.
```
const { expect } = require("chai");

describe("Token contract", function () {
  it("Deployment should assign the total supply of tokens to the owner", async function () {
    const [owner] = await ethers.getSigners();

    const hardhatToken = await ethers.deployContract("Token");

    const ownerBalance = await hardhatToken.balanceOf(owner.address);
    expect(await hardhatToken.totalSupply()).to.equal(ownerBalance);
  });
});
```
In the project directory, run the test code. Enter the following:

```
npx hardhat test
```




```
// This is an example test file. Hardhat will run every *.js file in `test/`,
// so feel free to add new ones.

// Hardhat tests are normally written with Mocha and Chai.

// We import Chai to use its asserting functions here.
const { expect } = require("chai");

// We use `loadFixture` to share common setups (or fixtures) between tests.
// Using this simplifies your tests and makes them run faster, by taking
// advantage of Hardhat Network's snapshot functionality.
const {
  loadFixture,
} = require("@nomicfoundation/hardhat-toolbox/network-helpers");

// `describe` is a Mocha function that allows you to organize your tests.
// Having your tests organized makes debugging them easier. All Mocha
// functions are available in the global scope.
//
// `describe` receives the name of a section of your test suite, and a
// callback. The callback must define the tests of that section. This callback
// can't be an async function.
describe("Token contract", function () {
  // We define a fixture to reuse the same setup in every test. We use
  // loadFixture to run this setup once, snapshot that state, and reset Hardhat
  // Network to that snapshot in every test.
  async function deployTokenFixture() {
    // Get the Signers here.
    const [owner, addr1, addr2] = await ethers.getSigners();

    // To deploy our contract, we just have to call ethers.deployContract and await
    // its waitForDeployment() method, which happens once its transaction has been
    // mined.
    const hardhatToken = await ethers.deployContract("Token");

    await hardhatToken.waitForDeployment();

    // Fixtures can return anything you consider useful for your tests
    return { hardhatToken, owner, addr1, addr2 };
  }

  // You can nest describe calls to create subsections.
  describe("Deployment", function () {
    // `it` is another Mocha function. This is the one you use to define each
    // of your tests. It receives the test name, and a callback function.
    //
    // If the callback function is async, Mocha will `await` it.
    it("Should set the right owner", async function () {
      // We use loadFixture to setup our environment, and then assert that
      // things went well
      const { hardhatToken, owner } = await loadFixture(deployTokenFixture);

      // `expect` receives a value and wraps it in an assertion object. These
      // objects have a lot of utility methods to assert values.

      // This test expects the owner variable stored in the contract to be
      // equal to our Signer's owner.
      expect(await hardhatToken.owner()).to.equal(owner.address);
    });

    it("Should assign the total supply of tokens to the owner", async function () {
      const { hardhatToken, owner } = await loadFixture(deployTokenFixture);
      const ownerBalance = await hardhatToken.balanceOf(owner.address);
      expect(await hardhatToken.totalSupply()).to.equal(ownerBalance);
    });
  });

  describe("Transactions", function () {
    it("Should transfer tokens between accounts", async function () {
      const { hardhatToken, owner, addr1, addr2 } = await loadFixture(
        deployTokenFixture
      );
      // Transfer 50 tokens from owner to addr1
      await expect(
        hardhatToken.transfer(addr1.address, 50)
      ).to.changeTokenBalances(hardhatToken, [owner, addr1], [-50, 50]);

      // Transfer 50 tokens from addr1 to addr2
      // We use .connect(signer) to send a transaction from another account
      await expect(
        hardhatToken.connect(addr1).transfer(addr2.address, 50)
      ).to.changeTokenBalances(hardhatToken, [addr1, addr2], [-50, 50]);
    });

    it("Should emit Transfer events", async function () {
      const { hardhatToken, owner, addr1, addr2 } = await loadFixture(
        deployTokenFixture
      );

      // Transfer 50 tokens from owner to addr1
      await expect(hardhatToken.transfer(addr1.address, 50))
        .to.emit(hardhatToken, "Transfer")
        .withArgs(owner.address, addr1.address, 50);

      // Transfer 50 tokens from addr1 to addr2
      // We use .connect(signer) to send a transaction from another account
      await expect(hardhatToken.connect(addr1).transfer(addr2.address, 50))
        .to.emit(hardhatToken, "Transfer")
        .withArgs(addr1.address, addr2.address, 50);
    });

    it("Should fail if sender doesn't have enough tokens", async function () {
      const { hardhatToken, owner, addr1 } = await loadFixture(
        deployTokenFixture
      );
      const initialOwnerBalance = await hardhatToken.balanceOf(owner.address);

      // Try to send 1 token from addr1 (0 tokens) to owner.
      // `require` will evaluate false and revert the transaction.
      await expect(
        hardhatToken.connect(addr1).transfer(owner.address, 1)
      ).to.be.revertedWith("Not enough tokens");

      // Owner balance shouldn't have changed.
      expect(await hardhatToken.balanceOf(owner.address)).to.equal(
        initialOwnerBalance
      );
    });
  });
});

```
