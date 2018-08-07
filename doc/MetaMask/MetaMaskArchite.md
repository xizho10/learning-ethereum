# MetaMask Architecture

## 架构

![Alt text](../../img/MetaMask/Architecture.png)

> https://github.com/MetaMask/metamask-extension

The `metamask-background` describes the file at [`app/scripts/background.js`](https://github.com/MetaMask/metamask-extension/blob/develop/app/scripts/background.js), which is the web extension singleton. 

The `metamask-background` instantiates an instance of the `MetaMask Controller`, which represents the user's accounts, a connection to the blockchain, and the interaction with new Dapps.

When a new site is visited, the WebExtension creates a new `ContentScript` in that page's context, which can be seen at `app/scripts/contentscript.js`.

MetaMask's most unique feature isn't being a wallet, it's providing an Ethereum-enabled JavaScript context to websites.

## Ports, streams, and Web3!

MetaMask has two kinds of [duplex stream APIs](https://github.com/substack/stream-handbook#duplex) that it exposes:

- [metamask.setupTrustedCommunication(connectionStream, originDomain)](https://github.com/MetaMask/metamask-extension/blob/master/app/scripts/metamask-controller.js#L352): 用于通过一个远程端口连接用户界面。

- [metamask.setupUntrustedCommunication(connectionStream, originDomain)](https://github.com/MetaMask/metamask-extension/blob/master/app/scripts/metamask-controller.js#L337)：用于将一个新的`web`站点的`web3 API`连接到`MetaMask`的区块链链接（originDomain用于阻止钓鱼网站）。