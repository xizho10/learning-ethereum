# MetaMask Javascript API(Web3.js)

由于[`web3.js 1.0`](http://web3js.readthedocs.io/en/1.0/)尚未发布，因此该文档仍然以实际使用的[`web3.js 0.x.x`](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3)为准。

![Alt text](../../img/MetaMask/web3_1_0.png)

<!-- TOC -->

- [MetaMask Javascript API(Web3.js)](#metamask-javascript-apiweb3js)
    - [使用](#%E4%BD%BF%E7%94%A8)
        - [同步调用](#%E5%90%8C%E6%AD%A5%E8%B0%83%E7%94%A8)
        - [异步调用](#%E5%BC%82%E6%AD%A5%E8%B0%83%E7%94%A8)
    - [Detecting MetaMask](#detecting-metamask)
    - [Batch requests](#batch-requests)
    - [MetaMask RPC Settings](#metamask-rpc-settings)
    - [权限](#%E6%9D%83%E9%99%90)
    - [常用接口](#%E5%B8%B8%E7%94%A8%E6%8E%A5%E5%8F%A3)
        - [web3.version.network](#web3versionnetwork)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.version.ethereum](#web3versionethereum)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.version.whisper](#web3versionwhisper)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.isConnected](#web3isconnected)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.setProvider](#web3setprovider)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.currentProvider](#web3currentprovider)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.reset](#web3reset)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.getBalance](#web3ethgetbalance)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.getStorageAt](#web3ethgetstorageat)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.getCode](#web3ethgetcode)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.getBlock](#web3ethgetblock)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.getBlockTransactionCount](#web3ethgetblocktransactioncount)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.getUncle](#web3ethgetuncle)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.getTransaction](#web3ethgettransaction)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.getTransactionFromBlock](#web3ethgettransactionfromblock)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.getTransactionReceipt](#web3ethgettransactionreceipt)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.getTransactionCount](#web3ethgettransactioncount)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.sendTransaction](#web3ethsendtransaction)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.sendRawTransaction](#web3ethsendrawtransaction)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.sign](#web3ethsign)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.contract](#web3ethcontract)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)
        - [web3.eth.compile.solidity](#web3ethcompilesolidity)
            - [Parameters](#parameters)
            - [Returns](#returns)
            - [Example](#example)

<!-- /TOC -->

## 使用

### 同步调用

```js
web3.net.getPeerCount()
```

### 异步调用

```js
web3.net.getPeerCount(function(error, result){
    if(!error)
        console.log(result)
    else
        console.error(error);
})
```

![Alt text](../../img/MetaMask/async_call.png)

## Detecting MetaMask

Metamask/Mist currently inject their interface into page as global `web3` object.

```js
window.addEventListener('load', function() {

  // Checking if Web3 has been injected by the browser (Mist/MetaMask)
  if (typeof web3 !== 'undefined') {

    // Use the browser's ethereum provider
    var provider = web3.currentProvider

  } else {
    console.log('No web3? You should consider trying MetaMask!')
  }

})
```

The `provider` object is a low-level object with just one supported method: `provider.sendAsync(options, callback)`.

The `options` object has several useful fields: `method` (the method name), `from` (the sender address), `params` (an array of parameters to send to that method).

These methods all correspond directly to the options in [the RPC provider spec](https://github.com/ethereum/wiki/wiki/JSON-RPC).

To see if the injected provider is from MetaMask, you can check `web3.currentProvider.isMetaMask`.

## Batch requests

Batch requests allow queuing up requests and processing them at once.

Note Batch requests are not faster! In fact making many requests at once will in some cases be faster, as requests are processed asynchronously. Batch requests are mainly useful to ensure the serial processing of requests.

```js
var batch = web3.createBatch();
batch.add(web3.eth.getBalance.request('0x0000000000000000000000000000000000000000', 'latest', callback));
batch.add(web3.eth.contract(abi).at(address).balance.request(address, callback2));
batch.execute();
```

## MetaMask RPC Settings

![Alt text](../../img/MetaMask/new_rpc.png)

## 权限

![Alt text](../../img/MetaMask/MetaMaskPermissions_1.png)

```json
"permissions": [
    "storage",
    "unlimitedStorage",
    "clipboardWrite",
    "http://localhost:8545/",
    "https://*.infura.io/",
    "activeTab",
    "webRequest",
    "*://*.eth/",
    "*://*.test/",
    "notifications"
]
```

> https://github.com/MetaMask/metamask-extension/blob/1540b5e256634dbbbc01627ca45afe82329d39a9/app/manifest.json

![Alt text](../../img/MetaMask/SendTransaction_1.png)

![Alt text](../../img/MetaMask/SendTransaction_2.png)

![Alt text](../../img/MetaMask/SendTransaction_3.png)

```js
var sendTransaction = new Method({
        name: 'sendTransaction',
        call: 'eth_sendTransaction',
        params: 1,
        inputFormatter: [formatters.inputTransactionFormatter]
    });
```

> https://github.com/ethereum/web3.js/blob/develop/lib/web3/methods/eth.js

```js
var Method = function (options) {
    this.name = options.name;
    this.call = options.call;
    this.params = options.params || 0;
    this.inputFormatter = options.inputFormatter;
    this.outputFormatter = options.outputFormatter;
    this.requestManager = null;
};
```

> https://github.com/ethereum/web3.js/blob/develop/lib/web3/method.js

***

**chrome.notifications**

| 描述 | 使用 chrome.notifications API 通过模板创建丰富通知，并在系统托盘中向用户显示这些通知  |
| ---------|----------|
| 可用版本 | 从 Chrome 28 开始支持 |
| 权限 | "notifications"  |

> https://crxdoc-zh.appspot.com/apps/notifications

***

```js
showTransactionNotification (txMeta) {

const status = txMeta.status
if (status === 'confirmed') {
    this._showConfirmedTransaction(txMeta)
} else if (status === 'failed') {
    this._showFailedTransaction(txMeta)
}
}

_showConfirmedTransaction (txMeta) {

this._subscribeToNotificationClicked()

const url = explorerLink(txMeta.hash, parseInt(txMeta.metamaskNetworkId))
const nonce = parseInt(txMeta.txParams.nonce, 16)

const title = 'Confirmed transaction'
const message = `Transaction ${nonce} confirmed! View on EtherScan`
this._showNotification(title, message, url)
}

_showFailedTransaction (txMeta) {

const nonce = parseInt(txMeta.txParams.nonce, 16)
const title = 'Failed transaction'
const message = `Transaction ${nonce} failed! ${txMeta.err.message}`
this._showNotification(title, message)
}

_showNotification (title, message, url) {
extension.notifications.create(
    url,
    {
    'type': 'basic',
    'title': title,
    'iconUrl': extension.extension.getURL('../../images/icon-64.png'),
    'message': message,
    })
  }

```

> https://github.com/MetaMask/metamask-extension/blob/fed9ae0deed5853014cfc76b4314195d477f14f4/app/scripts/platforms/extension.js

## 常用接口

### web3.version.network

    web3.version.network
    // or async
    web3.version.getNetwork(callback(error, result){ ... })


#### Returns

`String` - The network protocol version.

#### Example

```js
var version = web3.version.network;
console.log(version); // 54
```

***

### web3.version.ethereum

    web3.version.ethereum
    // or async
    web3.version.getEthereum(callback(error, result){ ... })


#### Returns

`String` - The ethereum protocol version.

#### Example

```js
var version = web3.version.ethereum;
console.log(version); // 60
```

***

### web3.version.whisper

    web3.version.whisper
    // or async
    web3.version.getWhisper(callback(error, result){ ... })


#### Returns

`String` - The whisper protocol version.

#### Example

```js
var version = web3.version.whisper;
console.log(version); // 20
```

***

### web3.isConnected

    web3.isConnected()

Should be called to check if a connection to a node exists

#### Parameters
none

#### Returns

`Boolean`

#### Example

```js
if(!web3.isConnected()) {
  
   // show some dialog to ask the user to start a node

} else {
 
   // start web3 filters, calls, etc
  
}
```

***

### web3.setProvider

    web3.setProvider(provider)

Should be called to set provider.

#### Parameters
none

#### Returns

`undefined`

#### Example

```js
web3.setProvider(new web3.providers.HttpProvider('http://localhost:8545')); // 8080 for cpp/AZ, 8545 for go/mist
```

***

### web3.currentProvider

    web3.currentProvider

Will contain the current provider, if one is set. This can be used to check if mist etc. has set already a provider.


#### Returns

`Object` - The provider set or `null`;

#### Example

```js
// Check if mist etc. already set a provider
if(!web3.currentProvider)
    web3.setProvider(new web3.providers.HttpProvider("http://localhost:8545"));

```

***

### web3.reset

    web3.reset(keepIsSyncing)

Should be called to reset state of web3. Resets everything except manager. Uninstalls all filters. Stops polling.

#### Parameters

1. `Boolean` - If `true` it will uninstall all filters, but will keep the [web3.eth.isSyncing()](#web3ethissyncing) polls

#### Returns

`undefined`

#### Example

```js
web3.reset();
```

***

### web3.eth.getBalance

    web3.eth.getBalance(addressHexString [, defaultBlock] [, callback])

Get the balance of an address at a given block.

#### Parameters

1. `String` - The address to get the balance of.
2. `Number|String` - (optional) If you pass this parameter it will not use the default block set with [web3.eth.defaultBlock](#web3ethdefaultblock).
3. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.

#### Returns

`String` - A BigNumber instance of the current balance for the given address in wei.

See the [note on BigNumber](#a-note-on-big-numbers-in-web3js).

#### Example

```js
var balance = web3.eth.getBalance("0x407d73d8a49eeb85d32cf465507dd71d507100c1");
console.log(balance); // instanceof BigNumber
console.log(balance.toString(10)); // '1000000000000'
console.log(balance.toNumber()); // 1000000000000
```

***

### web3.eth.getStorageAt

    web3.eth.getStorageAt(addressHexString, position [, defaultBlock] [, callback])

Get the storage at a specific position of an address.

***

> 由于状态变量是存储在区块链上的，所以存储空间需要预先分配，但映射的存储值是可以动态增改的。因此，在以太坊的状态存储模型中，实际存储是以哈希键值对的方式实现的。

> 其中，哈希是由键值和映射的存储槽位序号拼接后计算的哈希值（映射只占一个槽位序号），也就是说值是存到由keccak256(k . p)计算的哈希串里，这里的k表示的是映射要查找的键，p表示映射在整个合约中相对序号位置。

```solidity
pragma solidity ^0.4.0;

contract MappingLayout{
  //position: 0
  mapping(string=>string) strMapping;

  function setString() {
      //"aaa" -> hex: "616161"
      strMapping["aaa"] = "aaa";
  }
}
```

```js
function(err, result){
    var key = "616161"
    var pos = "0000000000000000000000000000000000000000000000000000000000000000"
    let hash = web3.sha3(key + pos, {"encoding":"hex"})

    var state = web3.eth.getStorageAt(myContract.address, hash);
    //0x6161610000000000000000000000000000000000000000000000000000000006
    console.log(state);
}
```

***

#### Parameters

1. `String` - The address to get the storage from.
2. `Number` - The index position of the storage.
3. `Number|String` - (optional) If you pass this parameter it will not use the default block set with [web3.eth.defaultBlock](#web3ethdefaultblock).
4. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.


#### Returns

`String` - The value in storage at the given position.

#### Example

```js
var state = web3.eth.getStorageAt("0x407d73d8a49eeb85d32cf465507dd71d507100c1", 0);
console.log(state); // "0x03"
```

***

### web3.eth.getCode

    web3.eth.getCode(addressHexString [, defaultBlock] [, callback])

Get the code at a specific address.

#### Parameters

1. `String` - The address to get the code from.
2. `Number|String` - (optional) If you pass this parameter it will not use the default block set with [web3.eth.defaultBlock](#web3ethdefaultblock).
3. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.

#### Returns

`String` - The data at given address `addressHexString`.

#### Example

```js
var code = web3.eth.getCode("0xd5677cf67b5aa051bb40496e68ad359eb97cfbf8");
console.log(code); // "0x600160008035811a818181146012578301005b601b6001356025565b8060005260206000f25b600060078202905091905056"
```

***

### web3.eth.getBlock

     web3.eth.getBlock(blockHashOrBlockNumber [, returnTransactionObjects] [, callback])

Returns a block matching the block number or block hash.

#### Parameters

1. `String|Number` - The block number or hash. Or the string `"earliest"`, `"latest"` or `"pending"` as in the [default block parameter](#web3ethdefaultblock).
2. `Boolean` - (optional, default `false`) If `true`, the returned block will contain all transactions as objects, if `false` it will only contains the transaction hashes.
3. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.

#### Returns

`Object` - The block object:

  - `number`: `Number` - the block number.
  - `hash`: `String`, 32 Bytes - hash of the block. `null` when its pending block.
  - `parentHash`: `String`, 32 Bytes - hash of the parent block.
  - `nonce`: `String`, 8 Bytes - hash of the generated proof-of-work. `null` when its pending block.
  - `sha3Uncles`: `String`, 32 Bytes - SHA3 of the uncles data in the block.
  - `logsBloom`: `String`, 256 Bytes - the bloom filter for the logs of the block. `null` when its pending block.
  - `transactionsRoot`: `String`, 32 Bytes - the root of the transaction trie of the block
  - `stateRoot`: `String`, 32 Bytes - the root of the final state trie of the block.
  - `miner`: `String`, 20 Bytes - the address of the beneficiary to whom the mining rewards were given.
  - `difficulty`: `BigNumber` - integer of the difficulty for this block.
  - `totalDifficulty`: `BigNumber` - integer of the total difficulty of the chain until this block.
  - `extraData`: `String` - the "extra data" field of this block.
  - `size`: `Number` - integer the size of this block in bytes.
  - `gasLimit`: `Number` - the maximum gas allowed in this block.
  - `gasUsed`: `Number` - the total used gas by all transactions in this block.
  - `timestamp`: `Number` - the unix timestamp for when the block was collated.
  - `transactions`: `Array` - Array of transaction objects, or 32 Bytes transaction hashes depending on the last given parameter.
  - `uncles`: `Array` - Array of uncle hashes.

#### Example

```js
var info = web3.eth.getBlock(3150);
console.log(info);
/*
{
  "number": 3,
  "hash": "0xef95f2f1ed3ca60b048b4bf67cde2195961e0bba6f70bcbea9a2c4e133e34b46",
  "parentHash": "0x2302e1c0b972d00932deb5dab9eb2982f570597d9d42504c05d9c2147eaf9c88",
  "nonce": "0xfb6e1a62d119228b",
  "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
  "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "transactionsRoot": "0x3a1b03875115b79539e5bd33fb00d8f7b7cd61929d5a3c574f507b8acf415bee",
  "stateRoot": "0xf1133199d44695dfa8fd1bcfe424d82854b5cebef75bddd7e40ea94cda515bcb",
  "miner": "0x8888f1f195afa192cfee860698584c030f4c9db1",
  "difficulty": BigNumber,
  "totalDifficulty": BigNumber,
  "size": 616,
  "extraData": "0x",
  "gasLimit": 3141592,
  "gasUsed": 21662,
  "timestamp": 1429287689,
  "transactions": [
    "0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b"
  ],
  "uncles": []
}
*/
```

***

### web3.eth.getBlockTransactionCount

    web3.eth.getBlockTransactionCount(hashStringOrBlockNumber [, callback])

Returns the number of transaction in a given block.

#### Parameters

1. `String|Number` - The block number or hash. Or the string `"earliest"`, `"latest"` or `"pending"` as in the [default block parameter](#web3ethdefaultblock).
2. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.

#### Returns

`Number` - The number of transactions in the given block.

#### Example

```js
var number = web3.eth.getBlockTransactionCount("0x407d73d8a49eeb85d32cf465507dd71d507100c1");
console.log(number); // 1
```

***

### web3.eth.getUncle

    web3.eth.getUncle(blockHashStringOrNumber, uncleNumber [, returnTransactionObjects] [, callback])

Returns a blocks uncle by a given uncle index position.

#### Parameters

1. `String|Number` - The block number or hash. Or the string `"earliest"`, `"latest"` or `"pending"` as in the [default block parameter](#web3ethdefaultblock).
2. `Number` - The index position of the uncle.
3. `Boolean` - (optional, default `false`) If `true`, the returned block will contain all transactions as objects, if `false` it will only contains the transaction hashes.
4. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.


#### Returns

`Object` - the returned uncle. For a return value see [web3.eth.getBlock()](#web3ethgetblock).

**Note**: An uncle doesn't contain individual transactions.

#### Example

```js
var uncle = web3.eth.getUncle(500, 0);
console.log(uncle); // see web3.eth.getBlock

```

***


### web3.eth.getTransaction

    web3.eth.getTransaction(transactionHash [, callback])

Returns a transaction matching the given transaction hash.

#### Parameters

1. `String` - The transaction hash.
2. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.

#### Returns

`Object` - A transaction object its hash `transactionHash`:

- `hash`: `String`, 32 Bytes - hash of the transaction.
- `nonce`: `Number` - the number of transactions made by the sender prior to this one.
- `blockHash`: `String`, 32 Bytes - hash of the block where this transaction was in. `null` when its pending.
- `blockNumber`: `Number` - block number where this transaction was in. `null` when its pending.
- `transactionIndex`: `Number` - integer of the transactions index position in the block. `null` when its pending.
- `from`: `String`, 20 Bytes - address of the sender.
- `to`: `String`, 20 Bytes - address of the receiver. `null` when its a contract creation transaction.
- `value`: `BigNumber` - value transferred in Wei.
- `gasPrice`: `BigNumber` - gas price provided by the sender in Wei.
- `gas`: `Number` - gas provided by the sender.
- `input`: `String` - the data sent along with the transaction.

#### Example

```js
var transaction = web3.eth.getTransaction('0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b');
console.log(transaction);
/*
{
  "hash": "0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b",
  "nonce": 2,
  "blockHash": "0xef95f2f1ed3ca60b048b4bf67cde2195961e0bba6f70bcbea9a2c4e133e34b46",
  "blockNumber": 3,
  "transactionIndex": 0,
  "from": "0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b",
  "to": "0x6295ee1b4f6dd65047762f924ecd367c17eabf8f",
  "value": BigNumber,
  "gas": 314159,
  "gasPrice": BigNumber,
  "input": "0x57cb2fc4"
}
*/

```

***

### web3.eth.getTransactionFromBlock

    getTransactionFromBlock(hashStringOrNumber, indexNumber [, callback])

Returns a transaction based on a block hash or number and the transactions index position.

#### Parameters

1. `String` - A block number or hash. Or the string `"earliest"`, `"latest"` or `"pending"` as in the [default block parameter](#web3ethdefaultblock).
2. `Number` - The transactions index position.
3. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.

#### Returns

`Object` - A transaction object, see [web3.eth.getTransaction](#web3ethgettransaction):


#### Example

```js
var transaction = web3.eth.getTransactionFromBlock('0x4534534534', 2);
console.log(transaction); // see web3.eth.getTransaction

```

***

### web3.eth.getTransactionReceipt

    web3.eth.getTransactionReceipt(hashString [, callback])

Returns the receipt of a transaction by transaction hash.

**Note** That the receipt is not available for pending transactions.


#### Parameters

1. `String` - The transaction hash.
2. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.

#### Returns

`Object` - A transaction receipt object, or `null` when no receipt was found:

  - `blockHash`: `String`, 32 Bytes - hash of the block where this transaction was in.
  - `blockNumber`: `Number` - block number where this transaction was in.
  - `transactionHash`: `String`, 32 Bytes - hash of the transaction.
  - `transactionIndex`: `Number` - integer of the transactions index position in the block.
  - `from`: `String`, 20 Bytes - address of the sender.
  - `to`: `String`, 20 Bytes - address of the receiver. `null` when its a contract creation transaction.
  - `cumulativeGasUsed `: `Number ` - The total amount of gas used when this transaction was executed in the block.
  - `gasUsed `: `Number ` -  The amount of gas used by this specific transaction alone.
  - `contractAddress `: `String` - 20 Bytes - The contract address created, if the transaction was a contract creation, otherwise `null`.
  - `logs `:  `Array` - Array of log objects, which this transaction generated.
  - `status `:  `String` - '0x0' indicates transaction failure , '0x1' indicates transaction succeeded. 

#### Example
```js
var receipt = web3.eth.getTransactionReceipt('0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b');
console.log(receipt);
{
  "transactionHash": "0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b",
  "transactionIndex": 0,
  "blockHash": "0xef95f2f1ed3ca60b048b4bf67cde2195961e0bba6f70bcbea9a2c4e133e34b46",
  "blockNumber": 3,
  "contractAddress": "0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b",
  "cumulativeGasUsed": 314159,
  "gasUsed": 30234,
  "logs": [{
         // logs as returned by getFilterLogs, etc.
     }, ...],
  "status": "0x1"
}
```

***

### web3.eth.getTransactionCount

    web3.eth.getTransactionCount(addressHexString [, defaultBlock] [, callback])

Get the numbers of transactions sent from this address.

#### Parameters

1. `String` - The address to get the numbers of transactions from.
2. `Number|String` - (optional) If you pass this parameter it will not use the default block set with [web3.eth.defaultBlock](#web3ethdefaultblock).
3. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.

#### Returns

`Number` - The number of transactions sent from the given address.

#### Example

```js
var number = web3.eth.getTransactionCount("0x407d73d8a49eeb85d32cf465507dd71d507100c1");
console.log(number); // 1
```

***

### web3.eth.sendTransaction

    web3.eth.sendTransaction(transactionObject [, callback])

Sends a transaction to the network.

#### Parameters

1. `Object` - The transaction object to send:
  - `from`: `String` - The address for the sending account. Uses the [web3.eth.defaultAccount](#web3ethdefaultaccount) property, if not specified.
  - `to`: `String` - (optional) The destination address of the message, left undefined for a contract-creation transaction.
  - `value`: `Number|String|BigNumber` - (optional) The value transferred for the transaction in Wei, also the endowment if it's a contract-creation transaction.
  - `gas`: `Number|String|BigNumber` - (optional, default: To-Be-Determined) The amount of gas to use for the transaction (unused gas is refunded).
  - `gasPrice`: `Number|String|BigNumber` - (optional, default: To-Be-Determined) The price of gas for this transaction in wei, defaults to the mean network gas price.
  - `data`: `String` - (optional) Either a [byte string](https://github.com/ethereum/wiki/wiki/Solidity,-Docs-and-ABI) containing the associated data of the message, or in the case of a contract-creation transaction, the initialisation code.
  - `nonce`: `Number`  - (optional) Integer of a nonce. This allows to overwrite your own pending transactions that use the same nonce.
2. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.

#### Returns

`String` - The 32 Bytes transaction hash as HEX string.

If the transaction was a contract creation use [web3.eth.getTransactionReceipt()](#web3ethgettransactionreceipt) to get the contract address, after the transaction was mined.

#### Example

```js

// compiled solidity source code using https://chriseth.github.io/cpp-ethereum/
var code = "603d80600c6000396000f3007c01000000000000000000000000000000000000000000000000000000006000350463c6888fa18114602d57005b6007600435028060005260206000f3";

web3.eth.sendTransaction({data: code}, function(err, transactionHash) {
  if (!err)
    console.log(transactionHash); // "0x7f9fade1c0d57a7af66ab4ead7c2eb7b11a91385"
});
```

***

### web3.eth.sendRawTransaction

    web3.eth.sendRawTransaction(signedTransactionData [, callback])

Sends an already signed transaction. For example can be signed using: https://github.com/SilentCicero/ethereumjs-accounts

#### Parameters

1. `String` - Signed transaction data in HEX format
2. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.

#### Returns

`String` - The 32 Bytes transaction hash as HEX string.

If the transaction was a contract creation use [web3.eth.getTransactionReceipt()](#web3ethgettransactionreceipt) to get the contract address, after the transaction was mined.

#### Example

```js
var Tx = require('ethereumjs-tx');
var privateKey = new Buffer('e331b6d69882b4cb4ea581d88e0b604039a3de5967688d3dcffdd2270c0fd109', 'hex')

var rawTx = {
  nonce: '0x00',
  gasPrice: '0x09184e72a000', 
  gasLimit: '0x2710',
  to: '0x0000000000000000000000000000000000000000', 
  value: '0x00', 
  data: '0x7f7465737432000000000000000000000000000000000000000000000000000000600057'
}

var tx = new Tx(rawTx);
tx.sign(privateKey);

var serializedTx = tx.serialize();

//console.log(serializedTx.toString('hex'));
//f889808609184e72a00082271094000000000000000000000000000000000000000080a47f74657374320000000000000000000000000000000000000000000000000000006000571ca08a8bbf888cfa37bbf0bb965423625641fc956967b81d12e23709cead01446075a01ce999b56a8a88504be365442ea61239198e23d1fce7d00fcfc5cd3b44b7215f

web3.eth.sendRawTransaction('0x' + serializedTx.toString('hex'), function(err, hash) {
  if (!err)
    console.log(hash); // "0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385"
});
```

***

### web3.eth.sign

    web3.eth.sign(dataToSign, address, [, callback])

Signs data from a specific account. This account needs to be unlocked.

#### Parameters

1. `String` - Data to sign.
2. `String` - Address to sign with.
3. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.

#### Returns

`String` - The signed data.

After the hex prefix, characters correspond to ECDSA values like this:
```
r = signature[0:64]
s = signature[64:128]
v = signature[128:130]
```

Note that if you are using `ecrecover`, `v` will be either `"00"` or `"01"`. As a result, in order to use this value, you will have to parse it to an integer and then add `27`. This will result in either a `27` or a `28`.

#### Example

```js
var result = web3.eth.sign(
    "0x9dd2c369a187b4e6b9c402f030e50743e619301ea62aa4c0737d4ef7e10a3d49", // first argument is web3.sha3("xyz")
    "0x135a7de83802408321b74c322f8558db1679ac20"); 
console.log(result); // "0x30755ed65396facf86c53e6217c52b4daebe72aa4941d89635409de4c9c7f9466d4e9aaec7977f05e923889b33c0d0dd27d7226b6e6f56ce737465c5cfd04be400"
```

***

### web3.eth.contract

    web3.eth.contract(abiArray)

Creates a contract object for a solidity contract, which can be used to initiate contracts on an address.
You can read more about events [here](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI#example-javascript-usage).

#### Parameters

1. `Array` - ABI array with descriptions of functions and events of the contract.

#### Returns

`Object` - A contract object, which can be initiated as follows:

```js
var MyContract = web3.eth.contract(abiArray);

// instantiate by address
var contractInstance = MyContract.at(address);

// deploy new contract
var contractInstance = MyContract.new([constructorParam1] [, constructorParam2], {data: '0x12345...', from: myAccount, gas: 1000000});

// Get the data to deploy the contract manually
var contractData = MyContract.new.getData([constructorParam1] [, constructorParam2], {data: '0x12345...'});
// contractData = '0x12345643213456000000000023434234'
```

And then you can either initiate an existing contract on an address,
or deploy the contract using the compiled byte code:

```js
// Instantiate from an existing address:
var myContractInstance = MyContract.at(myContractAddress);


// Or deploy a new contract:

// Deploy the contract asynchronous from Solidity file:
...
const fs = require("fs");
const solc = require('solc')

let source = fs.readFileSync('nameContract.sol', 'utf8');
let compiledContract = solc.compile(source, 1);
let abi = compiledContract.contracts['nameContract'].interface;
let bytecode = compiledContract.contracts['nameContract'].bytecode;
let gasEstimate = web3.eth.estimateGas({data: bytecode});
let MyContract = web3.eth.contract(JSON.parse(abi));

var myContractReturned = MyContract.new(param1, param2, {
   from:mySenderAddress,
   data:bytecode,
   gas:gasEstimate}, function(err, myContract){
    if(!err) {
       // NOTE: The callback will fire twice!
       // Once the contract has the transactionHash property set and once its deployed on an address.

       // e.g. check tx hash on the first call (transaction send)
       if(!myContract.address) {
           console.log(myContract.transactionHash) // The hash of the transaction, which deploys the contract
       
       // check address on the second call (contract deployed)
       } else {
           console.log(myContract.address) // the contract address
       }

       // Note that the returned "myContractReturned" === "myContract",
       // so the returned "myContractReturned" object will also get the address set.
    }
  });

// Deploy contract syncronous: The address will be added as soon as the contract is mined.
// Additionally you can watch the transaction by using the "transactionHash" property
var myContractInstance = MyContract.new(param1, param2, {data: myContractCode, gas: 300000, from: mySenderAddress});
myContractInstance.transactionHash // The hash of the transaction, which created the contract
myContractInstance.address // undefined at start, but will be auto-filled later
```

#### Example

```js
// contract abi
var abi = [{
     name: 'myConstantMethod',
     type: 'function',
     constant: true,
     inputs: [{ name: 'a', type: 'string' }],
     outputs: [{name: 'd', type: 'string' }]
}, {
     name: 'myStateChangingMethod',
     type: 'function',
     constant: false,
     inputs: [{ name: 'a', type: 'string' }, { name: 'b', type: 'int' }],
     outputs: []
}, {
     name: 'myEvent',
     type: 'event',
     inputs: [{name: 'a', type: 'int', indexed: true},{name: 'b', type: 'bool', indexed: false}]
}];

// creation of contract object
var MyContract = web3.eth.contract(abi);

// initiate contract for an address
var myContractInstance = MyContract.at('0xc4abd0339eb8d57087278718986382264244252f');

// call constant function
var result = myContractInstance.myConstantMethod('myParam');
console.log(result) // '0x25434534534'

// send a transaction to a function
myContractInstance.myStateChangingMethod('someParam1', 23, {value: 200, gas: 2000});

// short hand style
web3.eth.contract(abi).at(address).myAwesomeMethod(...);

// create filter
var filter = myContractInstance.myEvent({a: 5}, function (error, result) {
  if (!error)
    console.log(result);
    /*
    {
        address: '0x8718986382264244252fc4abd0339eb8d5708727',
        topics: "0x12345678901234567890123456789012", "0x0000000000000000000000000000000000000000000000000000000000000005",
        data: "0x0000000000000000000000000000000000000000000000000000000000000001",
        ...
    }
    */
});
```

***

### web3.eth.compile.solidity

    web3.eth.compile.solidity(sourceString [, callback])

Compiles solidity source code.

#### Parameters

1. `String` - The solidity source code.
2. `Function` - (optional) If you pass a callback the HTTP request is made asynchronous. See [this note](#using-callbacks) for details.

#### Returns

`Object` - Contract and compiler info.


#### Example

```js
var source = "" + 
    "contract test {\n" +
    "   function multiply(uint a) returns(uint d) {\n" +
    "       return a * 7;\n" +
    "   }\n" +
    "}\n";
var compiled = web3.eth.compile.solidity(source);
console.log(compiled); 
// {
  "test": {
    "code": "0x605280600c6000396000f3006000357c010000000000000000000000000000000000000000000000000000000090048063c6888fa114602e57005b60376004356041565b8060005260206000f35b6000600782029050604d565b91905056",
    "info": {
      "source": "contract test {\n\tfunction multiply(uint a) returns(uint d) {\n\t\treturn a * 7;\n\t}\n}\n",
      "language": "Solidity",
      "languageVersion": "0",
      "compilerVersion": "0.8.2",
      "abiDefinition": [
        {
          "constant": false,
          "inputs": [
            {
              "name": "a",
              "type": "uint256"
            }
          ],
          "name": "multiply",
          "outputs": [
            {
              "name": "d",
              "type": "uint256"
            }
          ],
          "type": "function"
        }
      ],
      "userDoc": {
        "methods": {}
      },
      "developerDoc": {
        "methods": {}
      }
    }
  }
}
```

***