# 1. 以太坊智能合约：Remix

> https://remix.ethereum.org/

<!-- TOC -->

- [1. 以太坊智能合约：Remix](#1-以太坊智能合约remix)
    - [1.1. 主界面](#11-主界面)
    - [1.2. 支持HTTPS](#12-支持https)
    - [1.3. 智能合约调试](#13-智能合约调试)
    - [1.4. 智能合约运行](#14-智能合约运行)
    - [1.5. 智能合约分析](#15-智能合约分析)

<!-- /TOC -->

## 1.1. 主界面

![Alt text](../../img/SmartContract/Remix/RemixUi_1.png)

## 1.2. 支持HTTPS

`Remix`已支持HTTPS：

> https://remix.ethereum.org/

![Alt text](../../img/SmartContract/Remix/RemixHttp.png)

## 1.3. 智能合约调试

`Remix`支持设置断点、单步调试、后退执行等多种常用的调试功能：

![Alt text](../../img/SmartContract/Remix/RemixDebug_1.png)

![Alt text](../../img/SmartContract/Remix/RemixDebug_2.png)

![Alt text](../../img/SmartContract/Remix/RemixDebug_3.png)

## 1.4. 智能合约运行

`Remix`支持3种运行环境：

- **JavaScript VM**: All the transactions will be executed in a sandbox blockchain in the browser. This means nothing will be persisted and a page reload will restart a new blockchain from scratch, the old one will not be saved.

- **Injected Provider**: Remix will connect to an injected web3 provider. Mist and Metamask are example of providers that inject web3, thus can be used with this option.

- **Web3 Provider**: Remix will connect to a remote node. You will need to provide the URL address to the selected provider: geth, parity or any Ethereum client.

> 资料来源：https://remix.readthedocs.io/en/latest/run_tab.html

![Alt text](../../img/SmartContract/Remix/RemixRunEnv_1.png)

![Alt text](../../img/SmartContract/Remix/RemixRunEnv_2.png)

`Remix`提供`JavaScript VM`运行环境，支持在

![Alt text](../../img/SmartContract/Remix/RemixDeploy_1.png)

## 1.5. 智能合约分析

`Remix`在每次编译智能合约的时候都会自动对合约代码进行分析，从而避免一些错误。

![Alt text](../../img/SmartContract/Remix/RemixAnaly_1.png)

> Here is the list of analyzers:
> - Security: Transaction origin: Warns if tx.origin is used
>   - Check effects: Avoid potential reentrancy bugs
>   - Inline assembly: Use of Inline Assembly
>   - Block timestamp: Semantics maybe unclear
>   - Low level calls: Semantics maybe unclear
>   - Block.blockhash usage: Semantics maybe unclear
> - Gas & Economy:
>   - Gas costs: Warns if the gas requirements of the functions are too high
>   - This on local calls: Invocation of local functions via this
> - Miscellaneous:
>   - Constant functions: Checks for potentially constant functions
>   - Similar variable names: Checks if variable names are too similar

***
> 资料来源：https://remix.readthedocs.io/en/latest/analysis_tab.html