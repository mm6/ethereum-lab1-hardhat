##  Fall 2024 Blockchain and SQL Fundamentals
### Carnegie Mellon University
### Due to Canvas on Thursday, November 8, 2024 11:59 PM
### 10 Points
### Deliverable: A single .pdf file named Lab1.pdf with clearly labelled answers.

<!--
#### Arjun Email: abrar@andrew.cmu.edu
-->

**Learning Objectives:** In this lab the student will set up an Ethereum
development environment (using Node.js and Hardhat) and deploy smart
contracts that are written in the Solidity programming language.

Our main objective is to get familiar with some of the tooling provided by
Hardhat. We will examine how we can deploy, interact with, and debug smart
contracts.

We will learn that normal accounts as well as contracts may have a token
balance and an eth balance.

We will experiment with running "transactions". Transactions cause state
changes, cost ether (for using gas), and may only return responses at a
later time.

We will also run "calls". Calls do not change state on the blockchain and
cost nothing to run (no gas). They are processed immediately and provide
an immediate return value.

Solidity event generation and a testing framework are also introduced
and the student will modify an existing contract.

The contracts themselves will be studied in detail in class. For
now, we want to set up our system and get some experience deploying
and interacting with the Solidity code and the environment provided
by client side Hardhat and the server side Hardhat network.

This lab exercise is based partly on the Hardhat tutorial [found here.](https://hardhat.org/tutorial)

The deliverable is a single well labelled pdf file with answers for E.1. through E.11.

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

7. We want to use Node.js version 20.8.1 (or, a version close to this). Use the following command:

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
Build a package.json file (holding important information about this project) by running the
following command. Hit return and take the defaults provided.
```
npm init     

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
15. We need access to that toolbox. Edit your hardhat.config.js file and include this line
at the top:

```
require("@nomicfoundation/hardhat-toolbox");
```

16. Create a new subdirectory named contracts. Within contracts, create the following smart
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
17. Using Node Package Execute (npx), compile the code with the following command. We do this in the
directory just above the contracts directory.

Note that this command will download the appropriate compiler.

```
npx hardhat compile

```

18. Create a new directory named test and place the following Javascript code in the test directory in a file named Token.js.
```
// Token.js is stored in the test subdirectory of the Hardhat project.
// Mocha and chai are often used together for unit testing.
// Mocha provides structure for organizing tests. For example, it provides "it"
// and "describe" functions.
// Chai is an assertion library. It provides the "expect" function below.

// import the chai library. We need the function named 'expect'.
const { expect } = require("chai");

describe("Token contract", function () {
  // The it function defines a single test.
  // It takes two arguments - a string description and a callback function containing the code for the test.
  // The name "it" is used for readability.
  // As in, it had better assign the total supply...
  it("Deployment had better assign the total supply of tokens to the owner", async function () {
    // Get the address of the first signer and assign to the constant owner.
    const [owner] = await ethers.getSigners();
    // deploy the contract named Token and set hardhatToken to refer to the deployed code.
    const hardhatToken = await ethers.deployContract("Token");
    // Get the balance of the owner.
    const ownerBalance = await hardhatToken.balanceOf(owner.address);
    // Get the total supply. We expect that to be the same value as the owner's balance.
    expect(await hardhatToken.totalSupply()).to.equal(ownerBalance);
  });
});

```
19. In the project directory, run the test code. Enter the following:

```
npx hardhat test
```

20. Replace the Token.js test file with the test file below and run

```
npx hardhat test
```
again.


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
## Part 2. Debugging Exercises

In this set of exercises, we spend some time making intentional errors in our smart contract to
see how the errors are handled by the testing code. In each case, you will edit the contract,
save it, and compile it. Each of these problems assumes that you start fresh with the contract
as shown above. You need to include clear labels for each answer and place all of the answers
on the single .pdf file.

Note: You do not need to study closely the Javascript in the unit test files. Review the code to see what is going on but focus more on the Solidity code.

E.1. Modify the contract's constructor so that it breaks the "Should show the right owner test".
Do this by setting the owner's address to 0 at the end of the constructor. Use address(0) in the
assignment statement.

E.2. Currently, the contract has 1000000 tokens. Set this value to 10 so that it breaks the
"Should transfer between accounts test".

E.3. Comment out the require statement in the transfer function. This should break the "Should fail if
sender doesn't have enough tokens" test.

E.4. Introduce an error into the contract's balanceOf function. Instead of "return balances[account]", let's
write "return balances[account] - 1". What result do we get when running the tests?

E.5. Introduce a small error into the constructor. Instead of "balances[msg.sender] = totalSupply;", let's write "balances[msg.sender] = totalSupply + 1;". What result do we get when running the tests?

The next two problems ask that you to modify the contract code so that messages are displayed during
the execution of the contract. Again, each of these problems assumes that you start fresh with the contract
as shown above.

E.6. During debugging, Hardhat allows the developer to display log messages from within the contract code.
These messages would be removed prior to deployment. Just below the pragma line, add the following import
statement to your contract:
```
import "hardhat/console.sol";
```
Now, add this line at the end of your constructor:

```
console.log("The constructor ran.");
```
What result do we get when running the tests?

E.7. At the end of your transfer function, add the line:

```
console.log("Transfer executed from %s to %s %s tokens",msg.sender,to,amount);
```
What result do we get when running the tests?

## Part 3. Using the Interactive Console

1) Hardhat provides an interactive console where we can deploy and interact with a contract. In
the HardhatLab1 directory, enter the following two commands:

```
npx hardhat console

```

```
const hardhatToken = await ethers.deployContract("Token");   // deploy the contract returns undefined
```

The library "ethers" provides a set of functions and utilities that allow us to interact with
the Ethereum blockchain. One of those functions is "deployContract". Here we have used it to
deploy our token contract.

2) Next, let's call the contract and ask for the total supply. The variable totalSupply is
public and so we can call hardhatToken.totalSupply().
```
const supply = await hardhatToken.totalSupply();
supply
```
3) Staying in the console (we can always exit with ctrl+d), use "ethers" to find account addresses
that are available on the Hardhat network blockchain. Enter the following:

```
const [Alice, Bob, Carol, Donald] = await ethers.getSigners();
```

Examine the addresses with:
```
Alice.address

```

```
Bob.address
```
And, the contract's address is also available in the target property of the contract reference:

```
hardhatToken.target
```

4) In the console, let's transfer some tokens from Alice to Bob. Note that the first address (Alice) is
the default sender's address. Alice will have to pay for these transactions.

```
await hardhatToken.transfer(Bob.address,20);
```
5) In the console, let's transfer some tokens from Alice to the contract:
```
await hardhatToken.transfer(hardhatToken.target,2);
```

6) Now, let's find the token balances of Alice, Bob, and the contract:

```
aliceBalance = await hardhatToken.balanceOf(Alice.address)
```

```
bobBalance = await hardhatToken.balanceOf(Bob.address)
```
```
contractBalance = await hardhatToken.balanceOf(hardhatToken.target)
```

7) Note that Alice, Bob, and the contract all have some tokens. Alice should have 999,978.
Bob should have 20 and the contract should have 2. But these are token balances. Perhaps
we would like to know how much eth they have.

This is not a call to the contract but is a call to Ethereum more generally.

```
const AliceEthBalance = await ethers.provider.getBalance(Alice.address);

```

```
AliceEthBalance

```

```
const BobEthBalance = await ethers.provider.getBalance(Bob.address);
```
```
BobEthBalance

```
```
const contractEthBalance = await ethers.provider.getBalance(hardhatToken.target);
```
```
contractEthBalance

```
So, we learned the fundamental idea that contracts as well as accounts can have
balances of eth as well as in tokens.

## Part 4. Final Exercise

In this exercise, we deploy and interact with a contract called Faucet.sol.

```

// Solidity code Faucet8.sol from Pg. 150 Mastering Ethereum
// Modified for 5.0 and with comments.

pragma solidity ^0.8.0;      // truffle.config can select the compiler version

contract owned {
    address payable owner;
    // This constructor will be inherited.
    // It takes no arguments and so the
    // migration will be simple.
    constructor() public {
        owner = payable(msg.sender);
    }
    // A modifier is a precondition. If the precondition is not satisfied then
    // the transaction will revert to its prior state.
    modifier onlyOwner {
        require (msg.sender == owner, "Only the creator of this contract may call this function");
    _;
    }
}
// Mortal inherits all features of the contract owned.
// Mortals may die.
contract mortal is owned {
    // only the creator may destroy this contract
    function destroy() public onlyOwner {
        selfdestruct(owner);
    }
}
// Single inheritance
contract Faucet is mortal {
    // Events end up in the receipt of the transaction.
    // The events may be examined by the contracts caller.

    event Withdrawal(address indexed to, uint amount);
    event Deposit(address indexed from, uint amount);

    // Withdraw by anyone as long as it's not greater
    // than 0.1 ether and as long as the contract has ether
    // to spare.
    function withdraw(uint withdraw_amount) public {
        require(withdraw_amount <= 0.1 ether);
        require((address(this)).balance >= withdraw_amount,"Balance too small for this withdrawal");
        payable(msg.sender).transfer(withdraw_amount);
        emit Withdrawal(msg.sender, withdraw_amount);
    }
    // Pay ether to the contract. Generate an event
    // upon deposit.
    fallback() external payable {
        emit Deposit(msg.sender, msg.value);
    }
}
```
E.8. Study this contract. Compile and deploy this contract using Hardhat. Show an interaction demonstrating that eth (not tokens) may be transferred to the contract.

Hint:  After creating a new project and compiling the contract, enter the Hardhat console.
You can deploy the contract from the console with:

```
const faucet = await ethers.deployContract("Faucet");

```

You can get the address of two accounts with this call:

```
const [Alice,Bob] = await ethers.getSigners();

```

You can transfer eth from Alice to the contract with this transaction:

```
const tx = await Alice.sendTransaction({to:faucet.target, value: ethers.parseEther("1.0")})
```

To examine the receipt from the transaction, just enter tx.

Paste a copy of the transaction receipt onto your well labelled single pdf.

Note that, in Ethereum, 1 eth = 10^18 wei. You can see this in the value field of the transaction receipt.

E.9. On your well labelled single pdf file, show an interaction where the withdraw transaction succeeds. Provide a copy of the receipt.

Hint: For Alice (the first account) to execute a withdrawal, we can enter:

```
tx2 = await faucet.withdraw(ethers.parseEther("0.01"))

```

For Bob to make a withdrawal, we need to state that we are working from his account:

```
tx3 = await faucet.connect(Bob).withdraw(4);    // This is 4 wei and not 4 eth

```

E.10. On your well labelled single pdf file, show an interaction where the withdraw transaction fails because of the first require statement. Provide some evidence on your well labelled single pdf. It is OK if this is an error message.

E.11. On your well labelled single pdf file, show an interaction where the withdraw transaction fails because of the second require statement. Provide some evidence on your well labelled single pdf. Again, it is OK if this answer is an error message. Hint: You will need to make more than one withdrawal.
