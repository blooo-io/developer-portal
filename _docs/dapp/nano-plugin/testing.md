---
title: Testing
subtitle:
tags: [dapp, DApp plugin]
category: "Live App: Dapp integration"
author:
toc: true
layout: doc
---

To test the code, we will use the [Zondax Zemu Testing Framework](https://github.com/Zondax/zemu). Tests can quickly be added and run on the emulator [Speculos](https://github.com/LedgerHQ/speculos) and take and compare snapshots to confirm they are correct.

<!--  -->
{% include alert.html style="important" text="The tests provided in the boilerplate plugin are only available for Nano S and X for the moment." %}
<!--  -->

We won't be using a docker image for this one. Instead, install `yarn` and the dependencies on your machine. So let's open a new terminal window.

<!--  -->
{% include alert.html style="important" text="
Make sure you don't use the previously launched docker container to run/install this.
" %}
<!--  -->

You need:
1. NodeJS and npm: [instructions here](https://nodejs.org/en/download/package-manager/)
2. Yarn: `npm install -g yarn`

Instead of writing everything from scratch, we copy the [`tests`](https://github.com/LedgerHQ/app-plugin-boilerplate/blob/master/tests/) folder of the boilerplate plugin repository.

Ok now for an overview of what needs to be done:

1. Install the dependencies
2. Create a folder for the contract information 
3. Build our plugin and the Ethereum app
4. Write the tests

## Installing the dependencies

This part is easy. Simply run:

```sh
cd tests && yarn install
```

This installs all the dependencies required for the tests to run correctly.

## Providing contract information

If you look carefully, you will see the `boilerplate/` folder inside the `tests/` folder. This file contains two items.

### The ABIs folder

The Application Binary Interfaces of your contracts are stored in the `abis/` folder. In our example, the contract address of `Uniswapv2` is `0x7a250d5630b4cf539739df2c5dacb4c659f2488d`. Its ABI is obtained on [Etherscan](https://etherscan.io/address/0x7a250d5630b4cf539739df2c5dacb4c659f2488d#code). We simply need to take the ABI, put it in a file named after the contract address, and append the extension ".json".
So `0x7a250d5630b4cf539739df2c5dacb4c659f2488d.json` is the name of the file, and its content is the ABI of this contract.
You can add as many contracts as you like here. They must be named appropriately (in lowercase, ending with `.json`) and must contain a valid ABI.

### The b2c.json file

Plugin and selector declarations are in `b2c.json`.

```js
{
    "chainId": 1, // chainID, no need to change
    "contracts": [
        {
            "address": "0x7a250d5630b4cf539739df2c5dacb4c659f2488d", // address of the contract address to interact with
            "contractName": "UniswapV2", // contract name is not used so feel free name as you wish.
            "selectors": { // list of selectors 
                "0x7ff36ab5": { // bytes of the selector
                    "erc20OfInterest": [ // more information down below
                        "path.1"
                    ],
                    "method": "swapExactETHForTokens", // method name: feel free to user whichever name you want to use
                    "plugin": "Boilerplate" // plugin name
                }
            }
        }
    ],
  "name": "Boilerplate" // plugin name
}
```

Fields are self-explanatory. Remember, you can add as many contracts as you want (`contracts` is an array) and as many `selectors` per contract.

The only tricky field is `erc20OfInterest`, which `selectors` have.

In our example, we trade ETH for ERC20 (tokens). To display token tickers (such as DAI, UBI, etc.), the js library needs to know which transaction field is the token address.

For the `swapExactETHForTokens` method, the token to swap is in the element `path.1`, meaning the second element of `path` (similar in syntax to `path[1]` in most languages). If the token address is `to`, then we simply use `to`.

The js library uses the ABI given in `abis/`, parse the transaction data, and extract the address of the ERC20. It will then look in its database, find the ERC20 ticker (DAI, for example), and send it to the device for later use.

Note that only two `erc20OfInterest` can be added. This is a memory constraint on Nano S.

### Build the plugin and the Ethereum app

To build the plugin, go back to your docker setup. 
Open a new terminal window, and in the `plugin-tools/` folder, run`./start.sh`.

In the container, go to the plugin repository, then to the `tests/` folder.

```sh
cd app-plugin-boilerplate/tests
```

If you have followed this tutorial, you simply need to run `./build_local_test_elfs.sh`. If you have your own setup (without using docker), then open this file and change `NANOS_SDK`, `NANOX_SDK` and `ETHEREUM_APP`.

### Writing the tests

We use the [Zondax Zemu framework](https://github.com/zondax/zemu) for testing. It allows us to:
- Create transactions either from a handcrafter transaction, or a raw transaction hash from Etherscan
- Run the transaction using Speculos, the Ledger Nano emulator
- Navigate through the transaction and accept it
- Compare the generated display snapshots with expected ones (useful for regression.)

The `tests/` folder contains these subfolders:
- `elfs` with the ELFs made by `build_local_test_elfs.sh`.
- `snapshots` contains the expected display snapshots.
- `src` with the source code for our tests.

As you might expect, we will mainly work in the `src/` folder.

The first thing is to change `src/generate_plugin_config.js` in line 5:

```js
const pluginFolder = "boilerplate";
```

Change `boilerplate` to the folder name created earlier, containing `abis/` and `b2c.json`.

Next, change `test.fixture.js`: 
```js
const NANOS_PLUGIN = { "Boilerplate": NANOS_PLUGIN_PATH };
const NANOX_PLUGIN = { "Boilerplate": NANOX_PLUGIN_PATH };
```

Once again, change `Boilerplate` to the name of your plugin (the one added to the Makefile at the beginning of this guide).

Now for the actual testing:

Tests must be in the `src/` folder and end in `.test.js`. We recommend having at least one test file per selector.
We also recommend combining Nano S and X tests in the same file.

There are two main ways to create a test transaction:
1. Build your transaction based on the contract ABI and `populateTransaction`: see the first test for NanoS in `swap_exact_eth_for_tokens.test.js`.
2. Replay a transaction from Etherscan: see the NanoX test in `swap_exact_eth_for_tokens.test.js`.

Option two is faster and easier: however, option one will give you more control and flexibility on your test transactions.

If you choose the second option, you obtain the raw transaction hex by looking up the [Etherscan transaction](https://etherscan.io/tx/0x0160b3aec12fd08e6be0040616c7c38248efb4413168a3372fc4d2db0e5961bb), then click in on the three dots in the upper right corner and click `Get Raw Tx Hex`. This opens a page like [this one](https://etherscan.io/getRawTx?tx=0x0160b3aec12fd08e6be0040616c7c38248efb4413168a3372fc4d2db0e5961bb). Voila!

Then rename `swap_exact_eth_for_tokens.test.js`, open it and edit it.
Comments are included throughout the code to help you along.

Most importantly, you need to change:
1. `contractAddr` 
2. `pluginName` 
3. The name of the test
4. Decide if you create your own transaction (and use the populateTransaction call just like the first test) or use a raw transaction from Etherscan (like in the second test).

The second argument of `navigateAndCompareSnapshots` is the folder name of the expected display snapshots. The Testing Framework takes generated display snapshots for the current test and stores them in `snapshots-tmp/test_name`. It then compares them to the expected ones in `snapshots/test_name/`. Here, `test_name` the second string passed to `navigateAndcompareSnapshots`. For a new test, simply create a folder in `snapshots/` with an appropriate name. As it doesn’t yet contain snapshots, tests will run and fail. But we will get to that later.

To start the test, simply run `yarn test` (from your computer, not the docker container). This prints a lot of debug information on your terminal and opens a Speculos window. If your plugin works and tests run correctly, Speculos displays the transaction and automatically presses the correct buttons for you.

The first time you run the tests, you probably still have the snapshots of the boilerplate plugin, or may have removed them and the folder is empty. You will get an error in both cases because the generated and expected display snapshots won't match.

Copy over the generated display snapshots from `snapshots-tmp/test_name/` to `snapshots/test_name/` to start building the expected snapshots. Make sure they are correct and that they are expected for that test!
