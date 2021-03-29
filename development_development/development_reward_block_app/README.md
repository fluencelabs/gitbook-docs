# Building The Reward Block Application

Our project aims to 

* retrieve the latest block height from the Ethereum mainnet,
* use that result to retrieve the associated reward block data and 
* store the result in a SQlite database 

In order to simplify Ethereum data access, we will be using the [Etherscan API](https://etherscan.io/apis). Make sure you got yours ready as we are using two Etherscan endpoints:

* [eth\_blockNumber](https://api.etherscan.io/api?module=proxy&action=eth_blockNumber&apikey=YourApiKeyToken), which returns the most recent block height as a hex string and 
* [getBlockReward](https://api.etherscan.io/api?module=block&action=getblockreward&blockno=2165403&apikey=YourApiKeyToken), which returns the block and uncle reward by block height

Both _eth\_blockNumber_ and _getBlockReward_ need access to remote endpoints and for our purposes, a cUrl service will do just fine. Hence, we ned to implement a curl adapter to access the curl binaries of the node. Moreover, as _eth\_blockNumber_ returns a hex string and _getBlockReward_ requires an integer, we need a hex to int conversion, which we are going to implement as a stand-alone service. Finally, a SQLite adapter is also required, although the sqlite3.wasm module is [readily available](https://github.com/fluencelabs/sqlite/releases) from the Fluence repo.

The high-level workflow of our application is depicted in Figure 1.

![Figure 1: Stylized Workflow](../../.gitbook/assets/image%20%282%29.png)

