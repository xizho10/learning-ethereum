# Web3 Secret Storage Definition

> 资料来源：https://github.com/ethereum/wiki/wiki/Web3-Secret-Storage-Definition

## 简介

To make your app work on Ethereum, you can use the `web3` object provided by the `web3.js` library. 

Under the hood it communicates to a local node through `RPC calls`.

`web3` works with any Ethereum node, which exposes an RPC layer.

在`web3`中：

- `web3.eth`: 与Ethereum blockchain进行交互；
- `web3.shh`: 基于Whisper协议（以太坊DApps之间的通信协议，一套p2p节点间的异步广播系统）进行交互；


## 密钥文件存储

- Unix-like systems: `~/.web3/keystore`
- Windows: `~/AppData/Web3/keystore`

## 密钥文件命名

They may be named anything, but a good convention is `<uuid>.json`, where `uuid` is the `128-bit UUID` given to the secret key (a privacy-preserving proxy for the secret key's address).

## KDF

All such files have an associated password.

KDF-dependent static and dynamic parameters to the KDF function are described in `kdfparams` key.

PBKDF2 must be supported by all minimally-compliant implementations, denoted though:

- `kdf`: `pbkdf2`

For `PBKDF2`, the kdfparams include:

- `prf`: Must be `hmac-sha256` (may be extended in the future);
- `c`: number of iterations;
- `salt`: salt passed to PBKDF;
- `dklen`: length for the derived key. Must be >= 32.

## AES

All minimally-compliant implementations must support the AES-128-CTR algorithm, denoted through:

- `cipher`: `aes-128-ctr`

This cipher takes the following parameters, given as keys to the `cipherparams` key:

- `iv`: 128-bit initialisation vector for the cipher.

The key for the cipher is the leftmost 16 bytes of the derived key, i.e. `DK[0..15]`

The creation/encryption of a secret key should be essentially the reverse of these instructions. Make sure the `uuid`, `salt` and `iv` are actually random.

## Example

- Address: `008aeeda4d805471df9b2a5b0f38a0c3bcba786b`
- ICAP: `XE542A5PZHH8PYIZUBEJEO0MFWRAPPIL67`
- UUID: `3198bc9c-6672-5ab3-d9954942343ae5b6`
- Password: `testpassword`
- Secret: `7a28b5ba57c53603b0b07b56bba752f7784bf506fa95edc395f5cf6c7514fe9d`