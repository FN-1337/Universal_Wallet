# Multi-Blockchain Wallet in Python

![coins](Screenshots/coins.jpg)

## Description
This project will allow you to send transactions on the BTC test net as well as your own ETH test net. If you need help setting up your own ETH network you can visit my other project(https://github.com/FN-1337/POW_BC) that will show you how to set up your own network. We are going to use HD wallet derive as well as bit, python,  and web.py to set up our wallet. The main focus of this project is to allow you to have a universal wallet that you can send transactions to any coin you want. In this project we are only using BTC and ETH to test the wallet.py file. By following the instructions below and using my wallet.py file you will also be able to send transactions with ease. 


## Dependencies

The following dependencies are required for this project to run properly. 

> **Important:**  Below, you may do so by following the instructions found in the following guides:
  > - [HD Wallet Derive Installation Guide](Resources/HD_Wallet_Derive_Install_Guide.md) 
  > - [Blockchain TX Installation Guide](Resources/Blockchain_TX_Install_Guide.md).

**Dependencies List:**
- PHP must be installed on your operating system.

- You will need to clone the [`hd-wallet-derive`](https://github.com/dan-da/hd-wallet-derive) tool.

- [`bit`](https://ofek.github.io/bit/) Python Bitcoin library.

- [`web3.py`](https://github.com/ethereum/web3.py) Python Ethereum library.


## Instructions

### 1. Project setup

- Create a project directory called `wallet` and `cd` into it.

- Clone the `hd-wallet-derive` tool into this folder and install it using the [HD Wallet Derive Installation Guide](Resources/HD_Wallet_Derive_Install_Guide.md) 

- Create a symlink called `derive` for the `hd-wallet-derive/hd-wallet-derive.php` script. This will clean up the command needed to run the script in our code, as we can call `./derive` instead of `./hd-wallet-derive/hd-wallet-derive.php`: 

  - Make sure you are in the top level project directory - in this case the directory named `wallet`. 
  
  - **Mac Users:** Run the following command: `ln -s hd-wallet-derive/hd-wallet-derive.php derive`. 
  
  - **Windows Users:** Creating symlinks is not supported by default on Windows, only reading them, so Windows users must perform the following steps:

    - Open up Git-Bash as an administrator (right-click on Git-Bash in the start menu).

    - Within `bash`, run the command `export MSYS=winsymlinks:nativestrict`.
    
    - Run the following command: `ln -s hd-wallet-derive/hd-wallet-derive.php derive`. 

  - Test that you can run the `./derive` script properly, by running the following command.  

    - ```
      ./derive --key=xprv9zbB6Xchu2zRkf6jSEnH9vuy7tpBuq2njDRr9efSGBXSYr1QtN8QHRur28QLQvKRqFThCxopdS1UD61a5q6jGyuJPGLDV9XfYHQto72DAE8 --cols=path,address --coin=ZEC --numderive=3 -g
      ```
  - The output should match what you see below:
    - ```
      +------+-------------------------------------+
      | path | address                             |
      +------+-------------------------------------+
      | m/0  | t1V1Qp41kbHn159hvVXZL5M1MmVDRe6EdpA |
      | m/1  | t1Tw6iqFY1g9dKeAqPDAncaUjha8cn9SZqX |
      | m/2  | t1VGTPzBSSYd27GF8p9rGKGdFuWekKRhug4 |
      +------+-------------------------------------+
      ```

- Create a file called `wallet.py` -- this will be your universal wallet script. You can use [this starter code](Starter-Code/wallet.py) as a starting point.

- Your directory tree should look something like this:

  ![directory-tree](Screenshots/tree.png)

### 2. Setup constants

- In a separate file, `constants.py`, set the following constants:
  - `BTC = 'btc'`
  - `ETH = 'eth'`
  - `BTCTEST = 'btc-test'`

- In `wallet.py`, import all constants: `from constants import *`

- Use these anytime you reference these strings, both in function calls, and in setting object keys.

### 3. Generate a Mnemonic

- Generate a **new** 12 word mnemonic using `hd-wallet-derive` or by using [this tool](https://iancoleman.io/bip39/).

- Set this mnemonic as an environment variable by storing it a an `.env` file and importing it into your `wallet.py`.

### 4. Derive the wallet keys

- Create a function called `derive_wallets` that does the following:

  - Use the `subprocess` library to create a shell command that calls the `./derive` script from Python. Make sure to properly wait for the process. **Windows Users** may need to prepend the `php` command in front of `./derive` like so: `php ./derive`.

  - The following flags must be passed into the shell command as variables:
    - Mnemonic (`--mnemonic`) must be set from an environment variable, or default to a test mnemonic
    - Coin (`--coin`)
    - Numderive (`--numderive`) to set number of child keys generated
    - Format (`--format=json`) to parse the output into a JSON object using `json.loads(output)`

- Create a dictionary object called `coins` that uses the `derive_wallets` function to derive `ETH` and `BTCTEST` wallets.

- When done properly, the final object should look something like this (there are only 3 children each in this image):

  ![wallet-object](Screenshots/derive_wallets.PNG
- You should now be able to select child accounts (and thus, private keys) by accessing items in the `coins` dictionary like so: `coins[COINTYPE][INDEX]['privkey']`.

### 5. Linking the transaction signing libraries

- Use `bit` and `web3.py` to leverage the keys stored in the `coins` object by creating three more functions:

  - `priv_key_to_account`:
  
    - This function will convert the `privkey` string in a child key to an account object that `bit` or `web3.py` can use to transact.
    - This function needs the following parameters:

      - `coin` -- the coin type (defined in `constants.py`).
      - `priv_key` -- the `privkey` string will be passed through here.

    - You will need to check the coin type, then return one of the following functions based on the library:

      - For `ETH`, return `Account.privateKeyToAccount(priv_key)`
          - This function returns an account object from the private key string. You can read more about this object [here](https://web3js.readthedocs.io/en/v1.2.0/web3-eth-accounts.html#privatekeytoaccount).
      - For `BTCTEST`, return `PrivateKeyTestnet(priv_key)`
          - This is a function from the `bit` libarary that converts the private key string into a WIF (Wallet Import Format) object. WIF is a special format bitcoin uses to designate the types of keys it generates. 
          - You can read more about this function [here](https://ofek.dev/bit/dev/api.html).

  - `create_tx`: 
    - This function will create the raw, unsigned transaction that contains all metadata needed to transact.
    - This function needs the following parameters:

      - `coin` -- the coin type (defined in `constants.py`).
      - `account` -- the account object from `priv_key_to_account`.
      - `to` -- the recipient address.
      - `amount` -- the amount of the coin to send.

    - You will need to check the coin type, then return one of the following functions based on the library:

      - For `ETH`, return an object containing `to`, `from`, `value`, `gas`, `gasPrice`, `nonce`, and `chainID`.
        Make sure to calculate all of these values properly using `web3.py`!
      - For `BTCTEST`, return `PrivateKeyTestnet.prepare_transaction(account.address, [(to, amount, BTC)])`

- `send_tx`:
  - This function will call `create_tx`, sign the transaction, then send it to the designated network.
  - This function needs the following parameters:

    - `coin` -- the coin type (defined in `constants.py`).
    - `account` -- the account object from `priv_key_to_account`.
    - `to` -- the recipient address.
    - `amount` -- the amount of the coin to send.

  - You may notice these are the exact same parameters as `create_tx`. `send_tx` will call `create_tx`, so it needs all of this information available.

  - You will need to check the coin, then create a `raw_tx` object by calling `create_tx`. Then, you will need to sign the `raw_tx` using `bit` or `web3.py` (hint: the account objects have a sign transaction function within).

  - Once you've signed the transaction, you will need to send it to the designated blockchain network.

    - For `ETH`, return `w3.eth.sendRawTransaction(signed.rawTransaction)`
    - For `BTCTEST`, return `NetworkAPI.broadcast_tx_testnet(signed)`

### 6. Send some transactions!

- Now, you should be able to fund these wallets using testnet faucets. 
- Open up a new terminal window inside of `wallet`.
- Then run the command `python` to open the Python shell. 
- Within the Python shell, run the command `from wallet import *`. This will allow you to access the functions in `wallet.py` interactively.
- You'll need to set the account with  `priv_key_to_account` and use `send_tx` to send transactions.

  - **Bitcoin Testnet transaction**

    - Fund a `BTCTEST` address using [this testnet faucet](https://testnet-faucet.mempool.co/).

    ![btc-test](Screenshots/btcfaucet.PNG)

    - Use a [block explorer](https://tbtc.bitaps.com/) to watch transactions on the address.

    - Send a transaction to another testnet address (either one of your own, or the faucet's).
    
    - Screenshot the confirmation of the transaction like so:

      ![btc-test](Screenshots/btctx.PNG)

  - **Local PoA Ethereum transaction**

    - Add one of the `ETH` addresses to the pre-allocated accounts in your `networkname.json`.

    - Delete the `geth` folder in each node, then re-initialize using `geth --datadir nodeX init networkname.json`.
      This will create a new chain, and will pre-fund the new account.

    - [Add the following middleware](https://web3py.readthedocs.io/en/stable/middleware.html#geth-style-proof-of-authority)
      to `web3.py` to support the PoA algorithm:

      ```
      from web3.middleware import geth_poa_middleware

      w3.middleware_onion.inject(geth_poa_middleware, layer=0)
      ```

    - Due to a bug in `web3.py`, you will need to send a transaction or two with MyCrypto first, since the
      `w3.eth.generateGasPrice()` function does not work with an empty chain. You can use one of the `ETH` address `privkey`,
      or one of the `node` keystore files.

    - Send a transaction from the pre-funded address within the wallet to another, then copy the `txid` into
      MyCrypto's TX Status, and screenshot the successful transaction like so:

      ![eth-test](Screenshots/ethtx.PNG)
### Code used to run Transactions
### BTCTESTNET
btc_acc = priv_key_to_account(BTCTEST,btc_PrivateKey) 
create_trx(BTCTEST,btc_acc,"mmUQuAvBYqVpKzayhaYvnJe9VuwkRdSWPu", 0.00001)
send_tx(BTCTEST,btc_acc,'mmUQuAvBYqVpKzayhaYvnJe9VuwkRdSWPu',0.00001)
### Accounts 
address:"mkwqRYLAfeoMJnnb5A9WegujdggyBeEPGD"
privkey:"cUMT7ptJgxrwSm9s5zSUCKUhSNVq1UC9v31vgoSusyukAciNMdDt",
pubkey:"02dcb7a11c7172966e58679c0cb207bfb95d119cc55f9fb318d0a522aaff5b9577",
address 2: "mmUQuAvBYqVpKzayhaYvnJe9VuwkRdSWPu",
privkey 2: "cVj1Q51vqgtBEYokpcb6RtjMwVbAsaCmrQRyjMTHFm5rGoziYe44",
pubkey 2: "024fca815b769b3e6920eb31081de336536e0a0dcba4955b6d7773b8acc205e0b4",
### ETH
#### eth_acc = priv_key_to_account(ETH,eth_PrivateKey) 
#### create_trx(ETH,eth_acc,"0x3dFb64A5AdAe2aDece2E252FE5E889d61Ef75122", 1000) 
#### send_tx(ETH, eth_acc,"0x3dFb64A5AdAe2aDece2E252FE5E889d61Ef75122", 1000)
### Accounts 
address:"0x02194a55DDA1B029F70220D53f099F38c629ab36"
privkey: "0xd36d0f9ff2f6dc80208cd19e0b7d939f11bd1a63d10c38cdc1c58f4dbbc95edc",  
pubkey: "03b21e2e1bc0b74c298d113c5c305c80cc76e67c4ff29e9b31028ea17328f8bba8",
address 2: "0x3dFb64A5AdAe2aDece2E252FE5E889d61Ef75122"
privkey 2 : "0x81b180541f1ae73a48976095a58a342611fcedac9c00758510c0f3c0c63e5710",
pubkey 2: "03b64da52e7b23d110002752118ecc97052e6637d916794bef1eaac7bba0a54d3b",
