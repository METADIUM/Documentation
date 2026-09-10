# Metadium Blockchain: Developer Guide Document
Updated on Sep 10, 2026

## Table of contents
- [1.Introduction](#1introduction)
- [2.Metadium Blockchain Full Client (gmet)](#2metadium-blockchain-full-client-gmet)
  * [Download Source Code](#download-source-code)
  * [Install](#install)
  * [Address Format](#address-format)
  * [Coin Balance](#coin-balance)
  * [Interface (JSON RPC)](#interface-json-rpc)
  * [Network ID & Chain ID](#network-id--chain-id)
  * [Transaction Fee](#transaction-fee)
  * [EVM Version](#evm-version)
- [3.Block Explorer](#3block-explorer)
- [4.Testnet META Faucet](#4-testnet-meta-faucet)
- [5.Block Interval](#5-block-interval)
- [6.BIP32 HD wallet Rule (Reference)](#6-bip32-hd-wallet-rule-reference)
- [7.How to Connect Metamask with Metadium Testnet](#7-how-to-connect-metamask-with-metadium-testnet)
- [8.Link](#8link)
  * Source Code
  * Open API Server
  * Block Explorer
  * Test Meta Faucet
  * Gmet Install Guide
- [9.Contact](#9contact)
  * Technical Support
<br>

## 1.Introduction
This document is prepared for the Metadium Blockchain Integration as a reference document for the exchange technical team and any individual developers to support Metadium Blockchain. The current document reflects Metadium Mainnet and Testnet as of the Camellia upgrade (Cancun EVM).
<br>
<br>

## 2.Metadium Blockchain Full Client (gmet)
In order to connect with Metadium blockchain, you must install Metadium client `gmet` and finish complete block sync.
<br>

### Download Source Code
The Metadium Client is called Go-Metadium (gmet) and was made by forking Ethereum's Go-Ethereum client. The corresponding source code can be downloaded from the [Github Repository](https://github.com/METADIUM/go-metadium), and released binaries are published on the [Releases page](https://github.com/METADIUM/go-metadium/releases).
<br>

### Install
**Use the published release asset, not a source build.** Each release ships two Linux tarballs, one per database engine: `metadium-<version>-linux-leveldb.tar.gz` and `metadium-<version>-linux-rocksdb.tar.gz`. Download the one matching the engine your node uses from the [Releases page](https://github.com/METADIUM/go-metadium/releases), unpack it over the node directory, and restart. A node's engine cannot be changed by swapping the binary — the existing chain data determines it.

**Minimum OS: Ubuntu 20.04 or newer** (glibc 2.31+). Release binaries are built against that floor and will not start on anything older.

The RocksDB build needs the following shared libraries present on the host.
<pre><code>sudo apt-get install -y libjemalloc2 libsnappy1v5 liblz4-1 libzstd1</pre></code>

Building from source is for development only and needs **Go 1.22 or newer**; see the [readme](https://github.com/METADIUM/go-metadium/blob/master/README.md) for build instructions. Binaries built on your own host are not portable and should not be deployed.

After installing, run `gmet metadium new-account` to create an account on your local node. You should now be able to run gmet and connect to either Mainnet or Testnet network. Make sure to check the different options and commands with `gmet --help`

Below is the recommended command to start `gmet` node as a full data sync node.
<pre><code>## MAINNET
{data_folder}/bin/gmet --syncmode full --datadir {data_folder} --http

## TESTNET
{data_folder}/bin/gmet --metadium-testnet --syncmode full --datadir {data_folder} --http</pre></code>

The RPC listener binds to `127.0.0.1` by default and should stay that way. Expose it only behind a reverse proxy or a security group that restricts the source, and never bind `0.0.0.0` on a node holding accounts.

Full sync is the only supported mode. The Metadium network does not serve snap/fast sync, so a node started without `--syncmode full` will not complete its initial sync. Keep `--syncmode full` on every node.

Note that you don’t need to specify bootnode. Pre-defined bootnodes are embedded in the binary code of `gmet` executable. If you want add extra bootnode, please specify extra boot node manually when you start `gmet` at the command line.
<br>

### Address Format
Since the address system of Metadium is the same as that of Ethereum, it is possible to use the Ethereum address as it is (e.g., 0x0300cef240e54a14329f2dd4bf21bc5ca394eec1). In other words, the exchange that supports Metadium ERC20 can use the same address without changing the customer's cold wallet address. You can download Metadium Keepin App for both [Android](https://play.google.com/store/apps/details?id=com.coinplug.metadium&hl=en_US) and [iOS](https://itunes.apple.com/us/app/keepin-metadium-id-wallet/id1452993752?mt=8) mobile phone.
<br>

### Coin Balance
The Metadium Team has frozen the Metadium ERC20 contract at 3 AM UTC on February 22, 2019. A snapshot of the frozen ERC20 token has been recorded in the genesis block when Mainnet started. The amount is converted to at a 1: 1 ratio (i.e., 100 ERC20 Meta Token = 100 Metadium Mainnet Coin). If you are ERC20 META holder, please go to Metadium block explorer and check your balance with the same address. [Mainnet Block Explorer](https://explorer.metadium.com)
<br>

### Interface (JSON RPC)
Metadium implements the standard Ethereum JSON-RPC API ([execution-apis](https://ethereum.github.io/execution-apis/)), with Metadium additions such as fee-delegated transactions. Where an application relies on a specific method, check it against the node rather than assuming parity with a given go-ethereum release. Below is the cURL command for testing using Metadium Testnet/Mainnet Open API server.

The default RPC port for gmet is 8588 for both Mainnet and Testnet, and you can change it to 8545, which is the same as geth, using the `--http.port 8545` option when running gmet if you prefer.

Moreover, the default P2P port for gmet is 8589 for both Mainnet and Testnet, 
<pre><code>## MAINNET
curl -i -X POST -H "Content-Type: application/json" -d '{"jsonrp
c":"2.0","method":"eth_getBalance","params":["0x47f7b1714603beEBf0643e7bA5c7D7c9b7eF7D27", "latest"],"id":1}' https://api.metadium.com/prod

## TESTNET
curl -i -X POST -H "Content-Type: application/json" -d '{"jsonrp
c":"2.0","method":"eth_getBalance","params":["0x47f7b1714603beEBf0643e7bA5c7D7c9b7eF7D27", "latest"],"id":1}' https://api.metadium.com/dev
</pre></code>

### [Network ID & Chain ID](https://chainid.network)
Metadium uses the same network ID and chain ID pair. Please use 11 for the mainnet and 12 for the testnet especially when you are using Metamask like applications.
<br>

### Transaction Fee
Gmet 's default gas price is 80 Gwei, and if it is lower than this, confirmation may be delayed. When you create a withdrawal transaction on the exchange, you must read and use the current network's gas price through the `eth_gasPrice` RPC as shown below.

This is how to get gas price to use for sending transaction.
<pre><code>## MAINNET
curl -i -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":1}' https://api.metadium.com/prod

## TESTNET
curl -i -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":1}' https://api.metadium.com/dev
</pre></code>
<br>

### EVM Version
Metadium is on the **Cancun** EVM (the Camellia upgrade), active since block **117,764,000** on Mainnet and **86,449,000** on Testnet. Compile contracts with `--evm-version cancun` (or leave your toolchain's default if it targets Cancun or older). `PUSH0`, `TLOAD`/`TSTORE`, `MCOPY`, `BLOBBASEFEE` and warm `COINBASE` are available from those blocks onward; a contract compiled for a newer EVM version than Cancun will not run.
<br>
<br>

## 3.Block Explorer
Metadium Block Explorer is a web tool that provides detailed information about Metadium blocks, addresses, and transactions. It provides almost all the functions of Etherscan including contract verification, read, write, and much more. It has also unique Open API services to subscribers for free. Don't miss out – subscribe today! Please try by yourself at [Mainnet Explorer](https://explorer.metadium.com) and [Testnet Explorer](https://testnetexplorer.metadium.com).
<br>
<br>

## 4. Testnet META Faucet
Testnet META, a native coin used in the Metadium blockchain, can be obtained from the [Testnet META Faucet](https://testnetfaucet.metadium.com). Faucet sends a 10 META coin immediately to the requested address. Please note that the faucet has a limitation depending on the balance of the requested address, requester’s IP address, and the time interval of the consecutive requests for transmission.

If you need a large amount of META to use in the dev/ staging system of the exchange, you can contact tech@metadium.com to get it.
<br>
<br>

## 5. Block Interval
Metadium runs a permissioned Proof-of-Authority consensus: a fixed set of block producers registered on-chain through the governance contracts, with staking as the membership requirement. Block production rotates among them by height.

|   | Testnet | Mainnet |
| -------------------------------------------- | ----------- | ----------- |
| Block interval | 2 seconds | 2 seconds |

The interval is a governance parameter (`blockCreationTime`), not a client setting, and it is 2 seconds on both networks. A block is sealed when its slot elapses, whether or not it carries transactions — so a transaction submitted mid-slot is confirmed within about one interval on average, not immediately.

<br>
<br>

## 6. BIP32 HD wallet Rule (Reference)
Metadium Mainnet is registered in SLIP-0044 (Registered coin types for BIP-0044). The basic path of private key generated by mnemonic is as follows.
<pre><code>m/44'/916'/0'/0</pre></code>

https://github.com/satoshilabs/slips/blob/master/slip-0044.md

The basic path should be used when creating an app or web service that directly generates a private key for Metadium blockchain.
<br>
<br>

## 7. How to Connect Metamask with Metadium Testnet
Metamask can be used to send / receive META with Metadium blockchain. It is also possible to distribute and interact with contracts at http://remix.ethereum.org/ using metamask. 

Remember to set the gas price to 80 GWei or higher when you send via metamask.

* Step#1: Install Metamask Chrome Extension [Web Store](https://chrome.google.com/webstore/search/metamask?utm_source=chrome-ntp-icon)
* Step#2: Choose "Settings"
* Step#3: Choose "Networks" → Choose “Add a network”
* Step#4: Type Metadium Network Info → Save → Close Settings
  - Setting for Mainnet <pre><code>Network name: META Mainnet
  New RPC URL: https://api.metadium.com/prod
  Chain ID: 11
  Currency symbol: META
  Block explorer URL (Optional): https://explorer.metadium.com
  </pre></code>
  - Setting for Testnet<pre><code>Network name: META Testnet
  New RPC URL: https://api.metadium.com/dev
  Chain ID: 12
  Currency symbol: META
  Block explorer URL (Optional): https://testnetexplorer.metadium.com
  </pre></code>
* Step#5: You are now connected to Metadium Network (either Mainnet or Testnet)
* Step#6: Check your balance
* Step#7: Click Send → Fill To/Amount fields → Choose “Advanced Options”
* Step#8: Change both "Max priority fee (GWEI)" and "Max fee (GWEI)" to 80 at least → Save → Confirm
* Step#9: Copy source or destination address → Go [Metadium Block Explorer](https://explorer.metadium.com/) → Copy Address to search window and Enter → See [transaction lists of address](https://explorer.metadium.com/addresses/)
<br>
<br>

## 8. Link
* [Source Code](https://github.com/metadium/go-metadium)
* Open API Server
  - [MAINNET](https://api.metadium.com/prod)
  - [TESTNET](https://api.metadium.com/dev)
* Block Explorer
  - [MAINNET](https://explorer.metadium.com)
  - [TESTNET](https://testnetexplorer.metadium.com)
* [Test META Faucet](https://testnetfaucet.metadium.com)
* [Gmet Install Guide](https://github.com/METADIUM/go-metadium/blob/master/README.md)
<br>
<br>

## 9. Contact
* Technical Support tech@metadium.com
  

