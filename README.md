# Setup

* Setup Geth Node (independent of MIST or Parity) as an Ethereum client using Ethereum Protocol that allows for interaction with the Private Network blockchain.

* Make a directory
  ```
  mkdir /Users/Ls/code/blockchain/geth-node;
  cd /Users/Ls/code/blockchain/geth-node
  ```

* Create genesis.json file and chaindata/ folder, where:
  * `coinbase` is where mining goes to
  * `difficulty` is difficulty to mine a new block (`0x20000` is fast)
  * `gasLimit` is upper limit to the amount of assembler OPCODEs a miner is allowed to perform for transactions on blockchain that require payment of gas to miners
  * `timestamp` - defines the time when this block was generated (i.e. `0x00` is ~1970 epoch)
  * `alloc` - used to add Ether to pre-allocated accounts
  * `config` - 
    * `chainID` - do not set to `1` since this is the main net

* Initial genesis.json

```
{
  "coinbase"   : "0x0000000000000000000000000000000000000001",
  "difficulty" : "0x20000",
  "extraData"  : "",
  "gasLimit"   : "0x8000000",
  "nonce"      : "0x0000000000000042",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x00",
  "alloc": {},
  "config": {
        "chainId": 15,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    }
}
```

* Terminal Tab #1 - Initialise Private Network and configure Geth to save blockchain data with Genesis Block inside the chaindata/ folder, informing Geth where genesis.json blockchain configuration is located (cannot be in chaindata/)

```
geth --datadir=./chaindata/ init ./genesis.json
```

* Terminal Tab #1 - Start Geth Node using ./chaindata/ folder. It starts the network with same Chain ID as defined in the genesis.json file. Geth listens for incoming connections on port 30303 using an IPC file geth.ipc that is created when it is running to allow processes to connect to Geth (i.e. with `geth attach`, MIST / Ethereum Wallet). With the Private Network it has no other Nodes and nothing to synchronise. Note: Verbosity of 3 is recommended.
```
geth --datadir=./chaindata/ --verbosity 5
```

* Terminal Tab #2 - Use MIST to run Ethereum Wallet Dapp in MIST browser at http://wallet.ethereum.org. MIST binds Web3 to `window` object to inject it into the browser so Web3.js may be used to manage accounts (i.e. allows selecting one account from `web3.eth.accounts` to be visible in MIST via localhost:8545 by clicking the Manage Identity button), deploy or interact with contracts

* Add MIST to PATH - https://trello.com/c/XaTEDu12/97-mist-setup-macos

* Terminal Tab #2 - Start MIST after Geth running. Ensure to identify where the IPC file location of the Private Chain is to MIST (to avoid error `Fatal: Error starting protocol stack: listen udp :30303: bind: address already in use`).
  
  * Warning: Using loglevel "trace" uses creates a very large geth.log file quickly

```
cd /Users/Ls/code/blockchain/geth-node;

/Applications/Mist.app/Contents/MacOS/mist --help

open -a mist --args --mode "mist" \
  --node "geth" \
  --gethpath "/usr/local/bin/geth" \
  --rpc "/Users/Ls/code/blockchain/geth-node/chaindata/geth.ipc" \
  --node-ipcpath "/Users/Ls/code/blockchain/geth-node/chaindata/geth.ipc" \
  --node-datadir "/Users/Ls/code/blockchain/geth-node/chaindata/geth.ipc" \
  --logfile "/Users/Ls/code/blockchain/geth-node/geth.log" \
  --loglevel "trace" \
  --skiptimesynccheck true
```

* Terminal Tab #3 - View Logs

  ```
  cd /Users/Ls/code/blockchain/geth-node;
  tail -f ./geth.log;
  ```

* In MIST click "LAUNCH APPLICATION" button shown in the popup

* Terminal Tab #2 - Identify the IPC file location to Geth after MIST launches and Launch Application has been clicked, to load the JavaScript RPC Geth Console (JSRE REPL) on the already running Geth instance (communication with Geth through RPC on port 8545).
  * Note: MIST's default is `geth attach ipc:/Users/Ls/Library/Ethereum/geth.ipc`
  * Reference: https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console

  * Custom Private Chain

```
geth attach --help;
geth attach "/Users/Ls/code/blockchain/geth-node/chaindata/geth.ipc" 
```

    * Outputs the following

``` 
Welcome to the Geth JavaScript console!

instance: Geth/v1.7.3-stable/darwin-amd64/go1.9.2
 modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

> web3.eth.accounts
[]
```

* In MIST go to the Wallet Tab
  * Click "Add Account"
  * Click "Create new account"
  * Enter a password (i.e. test123456)
    * Note: This shows a popup: 

      > "Make sure you backup your keyfiles AND password! You can find your keyfiles folder using the main menu -> Accounts -> Backup -> Accounts. Keep a copy of the "keystore" folder where you can't lose it!"

    * Note: It will create the Wallet file in the directory /Users/Ls/code/blockchain/geth-node/chaindata/keystore/UTC--2017-12-29xxxxxxxxxxx--<ACCOUNT_NUMBER_WITHOUT_0x_PREFIX>
  * Click the new account "Main account (Etherbase) 0x<ACCOUNT_NUMBER>" and 
  then click "AUTHORIZE" to share the identity with Ethereum Wallet DApp (giving it permission to view public account information including accounts and balances)
  * Now in the Wallet Tab under the heading "WALLET CONTRACTS" it says: 

    > "Once you have more than 1 Ether you can create more sophisticated contracts. Wallets are smart contracts that allow your funds to be controlled by multiple accounts. They can have an optional daily limit on withdrawals to increase security. Create your own custom contracts on the Contracts tab."

* Terminal Tab #2 - Use Geth JavaScript console to show the Accounts using Web3.js:

```
> web3.eth.accounts
["0x<ACCOUNT_NUMBER>"]
```

* Terminal Tab #2 - Start Mining in the Private Network to earn Ether so we may create custom Contracts in the Contracts Tab of MIST. Enter the amount of threads it should start mining with:

```
> miner.start(1); 
```
  
* Optional Alternative: Try downloading EthMiner, which also supports GPU mining with `ethminer -G`. 
  * Reference: https://github.com/ethereum-mining/ethminer/releases

* Wait about 5 minutes for the DAG to generate before CPU mining starts, since it must first generate the DAG (see `Generating DAG in progress` in Geth logs with a percentage complete shown i.e. `epoch=1 percentage=42`). Reference: https://github.com/ethereum/wiki/wiki/Mining

* In MIST and Terminal Tab #1 - Check latest Block being mined until Account has an Ether balance of at least 1 ETH

* Terminal Tab #2 - Stop Mining

```
> miner.stop();
```

* In MIST, Deploy the FixedSupplyContract.sol code to the Private Network.
  * Check how much Ether we have CPU mined (say 50 ETH)
  * Go to "Contracts" tab
    * Click "Deploy New Contract" button
      * Select the new Account for the "FROM" field value
      * Do not send any Ether
      * Copy/Paste the Sample FixedSupplyContract.sol code into "SOLIDITY CONTRACT SOURCE CODE" text field value
      * MIST compiles the Solidity Contract to Bytecode
      * Select the "Fixed Supply Token" interface as the Contract to Deploy
      * Click "DEPLOY"
      * Click "SEND TRANSACTION"
      * Go to Wallet Tab > View "LATEST TRANSACTIONS"
      * Re-start Mining
        * Note: Each block mined for the transaction gives a block confirmation. After ~12 confirmations the contract is published on the Private Network

* In MIST, 
  * Go to "Contracts" tab
    * Select the Deployed Contract that was saved in the Wallet
    * View the options available:
      * Copy Contract Address
      * Show Contract QR Code
      * Show Contract JSON Interface (allows execution of the contract)
      * "Call" Functions are under heading "READ FROM CONTRACT"
      * "Transaction" Functions are under heading "WRITE TO CONTRACT" (i.e. `transfer`, `transferFrom`, `approve`)
      * Copy the Transaction Hash

* Terminal Tab #2 - JavaScript RPC Geth Console
  * Reference: https://github.com/ethereum/wiki/wiki/JavaScript-API

  * Show Transaction
```
var tx = web3.eth.getTransaction('<INSERT_TRANSACTION_HASH_OF_DEPLOYED_CONTRACT>');
```

  * Estimate Gas
```
web3.eth.estimateGas(tx);
```

  * Transaction Receipt
```
web3.eth.getTransactionReceipt('<INSERT_TRANSACTION_HASH_OF_DEPLOYED_CONTRACT>', function(err, res) { console.log(JSON.stringify(res, null, 2)) });
```

* Terminal Tab #2 - JavaScript Node.js script using Web3.js
  * Reference: 
    * Web3.js API 0.2x.x Docs - https://github.com/ethereum/wiki/wiki/JavaScript-API
    * Web3.js API 1.0.0-beta.xx Docs - https://web3js.readthedocs.io/en/1.0/web3.html
    * Web3.js Forums - https://forum.ethereum.org/categories/ethereum-js

  * Install Node.js dependencies

```
rm -rf node_modules

npm install --save-dev \
  web3@1.0.0-beta.27 \
  net@1.0.2 \
  solc@0.4.19 \
  fs@0.0.1-security

npm install
```

  * Deploy New Contract using Web3.js instead of via MIST

```
node scripts/main.js
```

  * IMPORTANT NOTE: Errors with `authentication needed: password or unlock` since Web3.js 1.0.0-beta has not finished implementing Unlock - http://web3js.readthedocs.io/en/1.0/web3-eth-personal.html?highlight=unlock

  * Deplot New Contract using Web3.py instead of MIST

    * References:
      * https://github.com/pyenv/pyenv
      * https://pypi.python.org/pypi/ethereum/2.1.5
      * https://pypi.python.org/pypi/web3/4.0.0b5
      * https://pypi.python.org/pypi/py-solc/2.1.0

```
pyenv install 3.6.4rc1
pyenv versions
pyenv global 3.6.4rc1
python -m pip install py-solc==2.1.0
python -m solc.install v0.4.18
python -m pip install web3==4.0.0b5 
python -m pip install ethereum==2.1.5

python scripts/main.py
```

* Experimental code snippets. 
  
  * Install dependencies included for each file first 

```
npm install ethers
npm install ethjs-query
npm install ethjs-account
npm install ethereumjs-tx
npm install ethjs-provider-signer
```
  
  * Run each experiment
  
```
node scripts/experiments/ethers-lib.js;
node scripts/experiments/ethjs-lib.js
```

* References:
  * Udemy Ethereum Masterclass - https://www.udemy.com/ethereum-masterclass