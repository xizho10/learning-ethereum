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

- [metamask-controller(connectionStream, originDomain)](https://github.com/MetaMask/metamask-extension/blob/master/app/scripts/metamask-controller.js#L352): 用于通过一个远程端口连接用户界面。

- [metamask.setupUntrustedCommunication(connectionStream, originDomain)](https://github.com/MetaMask/metamask-extension/blob/master/app/scripts/metamask-controller.js#L337)：用于将一个新的`web`站点的`web3 API`连接到`MetaMask`的区块链链接（originDomain用于阻止钓鱼网站）。

### metamask-controller

```js
/**
   * Returns an Object containing API Callback Functions.
   * These functions are the interface for the UI.
   * The API object can be transmitted over a stream with dnode.
   *
   * @returns {Object} Object containing API functions.
   */
getApi () {
    const keyringController = this.keyringController
    const preferencesController = this.preferencesController
    const txController = this.txController
    const noticeController = this.noticeController
    const addressBookController = this.addressBookController
    const networkController = this.networkController

    return {
        // etc
        getState: (cb) => cb(null, this.getState()),
        setCurrentCurrency: this.setCurrentCurrency.bind(this),
        setUseBlockie: this.setUseBlockie.bind(this),
        setCurrentLocale: this.setCurrentLocale.bind(this),
        markAccountsFound: this.markAccountsFound.bind(this),
        markPasswordForgotten: this.markPasswordForgotten.bind(this),
        unMarkPasswordForgotten: this.unMarkPasswordForgotten.bind(this),
        getGasPrice: (cb) => cb(null, this.getGasPrice()),

        // coinbase
        buyEth: this.buyEth.bind(this),
        // shapeshift
        createShapeShiftTx: this.createShapeShiftTx.bind(this),

        // primary HD keyring management
        addNewAccount: nodeify(this.addNewAccount, this),
        placeSeedWords: this.placeSeedWords.bind(this),
        verifySeedPhrase: nodeify(this.verifySeedPhrase, this),
        clearSeedWordCache: this.clearSeedWordCache.bind(this),
        resetAccount: nodeify(this.resetAccount, this),
        removeAccount: nodeify(this.removeAccount, this),
        importAccountWithStrategy: nodeify(this.importAccountWithStrategy, this),

        // hardware wallets
        connectHardware: nodeify(this.connectHardware, this),
        forgetDevice: nodeify(this.forgetDevice, this),
        checkHardwareStatus: nodeify(this.checkHardwareStatus, this),

        // TREZOR
        unlockTrezorAccount: nodeify(this.unlockTrezorAccount, this),

        // vault management
        submitPassword: nodeify(this.submitPassword, this),

        // network management
        setProviderType: nodeify(networkController.setProviderType, networkController),
        setCustomRpc: nodeify(this.setCustomRpc, this),

        // PreferencesController
        setSelectedAddress: nodeify(preferencesController.setSelectedAddress, preferencesController),
        addToken: nodeify(preferencesController.addToken, preferencesController),
        removeToken: nodeify(preferencesController.removeToken, preferencesController),
        setCurrentAccountTab: nodeify(preferencesController.setCurrentAccountTab, preferencesController),
        setAccountLabel: nodeify(preferencesController.setAccountLabel, preferencesController),
        setFeatureFlag: nodeify(preferencesController.setFeatureFlag, preferencesController),

        // AddressController
        setAddressBook: nodeify(addressBookController.setAddressBook, addressBookController),

        // KeyringController
        setLocked: nodeify(keyringController.setLocked, keyringController),
        createNewVaultAndKeychain: nodeify(this.createNewVaultAndKeychain, this),
        createNewVaultAndRestore: nodeify(this.createNewVaultAndRestore, this),
        addNewKeyring: nodeify(keyringController.addNewKeyring, keyringController),
        exportAccount: nodeify(keyringController.exportAccount, keyringController),

        // txController
        cancelTransaction: nodeify(txController.cancelTransaction, txController),
        updateTransaction: nodeify(txController.updateTransaction, txController),
        updateAndApproveTransaction: nodeify(txController.updateAndApproveTransaction, txController),
        retryTransaction: nodeify(this.retryTransaction, this),
        getFilteredTxList: nodeify(txController.getFilteredTxList, txController),
        isNonceTaken: nodeify(txController.isNonceTaken, txController),
        estimateGas: nodeify(this.estimateGas, this),

        // messageManager
        signMessage: nodeify(this.signMessage, this),
        cancelMessage: this.cancelMessage.bind(this),

        // personalMessageManager
        signPersonalMessage: nodeify(this.signPersonalMessage, this),
        cancelPersonalMessage: this.cancelPersonalMessage.bind(this),

        // personalMessageManager
        signTypedMessage: nodeify(this.signTypedMessage, this),
        cancelTypedMessage: this.cancelTypedMessage.bind(this),

        // notices
        checkNotices: noticeController.updateNoticesList.bind(noticeController),
        markNoticeRead: noticeController.markNoticeRead.bind(noticeController),
    }
}


```

> https://github.com/MetaMask/metamask-extension/blob/master/app/scripts/metamask-controller.js

### MessageManager

> JSON RPC: https://github.com/ethereum/wiki/wiki/JSON-RPC

```js
module.exports = class MessageManager extends EventEmitter {

  /**
   * Controller in charge of managing - storing, adding, removing, updating - Messages.
   *
   * @typedef {Object} MessageManager
   * @param {Object} opts @deprecated
   * @property {Object} memStore The observable store where Messages are saved.
   * @property {Object} memStore.unapprovedMsgs A collection of all Messages in the 'unapproved' state
   * @property {number} memStore.unapprovedMsgCount The count of all Messages in this.memStore.unapprobedMsgs
   * @property {array} messages Holds all messages that have been created by this MessageManager
   *
   */
  constructor (opts) {
    super()
    this.memStore = new ObservableStore({
      unapprovedMsgs: {},
      unapprovedMsgCount: 0,
    })
    this.messages = []
  }

  /**
   * A getter for the number of 'unapproved' Messages in this.messages
   *
   * @returns {number} The number of 'unapproved' Messages in this.messages
   *
   */
  get unapprovedMsgCount () {
    return Object.keys(this.getUnapprovedMsgs()).length
  }

  /**
   * A getter for the 'unapproved' Messages in this.messages
   *
   * @returns {Object} An index of Message ids to Messages, for all 'unapproved' Messages in this.messages
   *
   */
  getUnapprovedMsgs () {
    return this.messages.filter(msg => msg.status === 'unapproved')
    .reduce((result, msg) => { result[msg.id] = msg; return result }, {})
  }

  /**
   * Creates a new Message with an 'unapproved' status using the passed msgParams. this.addMsg is called to add the
   * new Message to this.messages, and to save the unapproved Messages from that list to this.memStore.
   *
   * @param {Object} msgParams The params for the eth_sign call to be made after the message is approved.
   * @returns {number} The id of the newly created message.
   *
   */
  addUnapprovedMessage (msgParams) {
    msgParams.data = normalizeMsgData(msgParams.data)
    // create txData obj with parameters and meta data
    var time = (new Date()).getTime()
    var msgId = createId()
    var msgData = {
      id: msgId,
      msgParams: msgParams,
      time: time,
      status: 'unapproved',
      type: 'eth_sign',
    }
    this.addMsg(msgData)

    // signal update
    this.emit('update')
    return msgId
  }

  /**
   * Adds a passed Message to this.messages, and calls this._saveMsgList() to save the unapproved Messages from that
   * list to this.memStore.
   *
   * @param {Message} msg The Message to add to this.messages
   *
   */
  addMsg (msg) {
    this.messages.push(msg)
    this._saveMsgList()
  }

  /**
   * Returns a specified Message.
   *
   * @param {number} msgId The id of the Message to get
   * @returns {Message|undefined} The Message with the id that matches the passed msgId, or undefined if no Message has that id.
   *
   */
  getMsg (msgId) {
    return this.messages.find(msg => msg.id === msgId)
  }

  /**
   * Approves a Message. Sets the message status via a call to this.setMsgStatusApproved, and returns a promise with
   * any the message params modified for proper signing.
   *
   * @param {Object} msgParams The msgParams to be used when eth_sign is called, plus data added by MetaMask.
   * @param {Object} msgParams.metamaskId Added to msgParams for tracking and identification within MetaMask.
   * @returns {Promise<object>} Promises the msgParams object with metamaskId removed.
   *
   */
  approveMessage (msgParams) {
    this.setMsgStatusApproved(msgParams.metamaskId)
    return this.prepMsgForSigning(msgParams)
  }

  /**
   * Sets a Message status to 'approved' via a call to this._setMsgStatus.
   *
   * @param {number} msgId The id of the Message to approve.
   *
   */
  setMsgStatusApproved (msgId) {
    this._setMsgStatus(msgId, 'approved')
  }

  /**
   * Sets a Message status to 'signed' via a call to this._setMsgStatus and updates that Message in this.messages by
   * adding the raw signature data of the signature request to the Message
   *
   * @param {number} msgId The id of the Message to sign.
   * @param {buffer} rawSig The raw data of the signature request
   *
   */
  setMsgStatusSigned (msgId, rawSig) {
    const msg = this.getMsg(msgId)
    msg.rawSig = rawSig
    this._updateMsg(msg)
    this._setMsgStatus(msgId, 'signed')
  }

  /**
   * Removes the metamaskId property from passed msgParams and returns a promise which resolves the updated msgParams
   *
   * @param {Object} msgParams The msgParams to modify
   * @returns {Promise<object>} Promises the msgParams with the metamaskId property removed
   *
   */
  prepMsgForSigning (msgParams) {
    delete msgParams.metamaskId
    return Promise.resolve(msgParams)
  }

  /**
   * Sets a Message status to 'rejected' via a call to this._setMsgStatus.
   *
   * @param {number} msgId The id of the Message to reject.
   *
   */
  rejectMsg (msgId) {
    this._setMsgStatus(msgId, 'rejected')
  }

  /**
   * Updates the status of a Message in this.messages via a call to this._updateMsg
   *
   * @private
   * @param {number} msgId The id of the Message to update.
   * @param {string} status The new status of the Message.
   * @throws A 'MessageManager - Message not found for id: "${msgId}".' if there is no Message in this.messages with an
   * id equal to the passed msgId
   * @fires An event with a name equal to `${msgId}:${status}`. The Message is also fired.
   * @fires If status is 'rejected' or 'signed', an event with a name equal to `${msgId}:finished` is fired along with the message
   *
   */
  _setMsgStatus (msgId, status) {
    const msg = this.getMsg(msgId)
    if (!msg) throw new Error('MessageManager - Message not found for id: "${msgId}".')
    msg.status = status
    this._updateMsg(msg)
    this.emit(`${msgId}:${status}`, msg)
    if (status === 'rejected' || status === 'signed') {
      this.emit(`${msgId}:finished`, msg)
    }
  }

  /**
   * Sets a Message in this.messages to the passed Message if the ids are equal. Then saves the unapprovedMsg list to
   * storage via this._saveMsgList
   *
   * @private
   * @param {msg} Message A Message that will replace an existing Message (with the same id) in this.messages
   *
   */
  _updateMsg (msg) {
    const index = this.messages.findIndex((message) => message.id === msg.id)
    if (index !== -1) {
      this.messages[index] = msg
    }
    this._saveMsgList()
  }

  /**
   * Saves the unapproved messages, and their count, to this.memStore
   *
   * @private
   * @fires 'updateBadge'
   *
   */
  _saveMsgList () {
    const unapprovedMsgs = this.getUnapprovedMsgs()
    const unapprovedMsgCount = Object.keys(unapprovedMsgs).length
    this.memStore.updateState({ unapprovedMsgs, unapprovedMsgCount })
    this.emit('updateBadge')
  }

}

/**
 * A helper function that converts raw buffer data to a hex, or just returns the data if it is already formatted as a hex.
 *
 * @param {any} data The buffer data to convert to a hex
 * @returns {string} A hex string conversion of the buffer data
 *
 */
function normalizeMsgData (data) {
  if (data.slice(0, 2) === '0x') {
    // data is already hex
    return data
  } else {
    // data is unicode, convert to hex
    return ethUtil.bufferToHex(new Buffer(data, 'utf8'))
  }
}

```
> https://github.com/MetaMask/metamask-extension/blob/master/app/scripts/lib/message-manager.js

### Identity Management (signature operations)

```js
// eth_sign methods:

/**
* Called when a Dapp uses the eth_sign method, to request user approval.
* eth_sign is a pure signature of arbitrary data. It is on a deprecation
* path, since this data can be a transaction, or can leak private key
* information.
*
* @param {Object} msgParams - The params passed to eth_sign.
* @param {Function} cb = The callback function called with the signature.
*/
newUnsignedMessage (msgParams, cb) {
const msgId = this.messageManager.addUnapprovedMessage(msgParams)
this.sendUpdate()
this.opts.showUnconfirmedMessage()
this.messageManager.once(`${msgId}:finished`, (data) => {
    switch (data.status) {
    case 'signed':
        return cb(null, data.rawSig)
    case 'rejected':
        return cb(cleanErrorStack(new Error('MetaMask Message Signature: User denied message signature.')))
    default:
        return cb(cleanErrorStack(new Error(`MetaMask Message Signature: Unknown problem: ${JSON.stringify(msgParams)}`)))
    }
})
}

/**
* Signifies user intent to complete an eth_sign method.
*
* @param  {Object} msgParams The params passed to eth_call.
* @returns {Promise<Object>} Full state update.
*/
signMessage (msgParams) {
log.info('MetaMaskController - signMessage')
const msgId = msgParams.metamaskId

// sets the status op the message to 'approved'
// and removes the metamaskId for signing
return this.messageManager.approveMessage(msgParams)
.then((cleanMsgParams) => {
    // signs the message
    return this.keyringController.signMessage(cleanMsgParams)
})
.then((rawSig) => {
    // tells the listener that the message has been signed
    // and can be returned to the dapp
    this.messageManager.setMsgStatusSigned(msgId, rawSig)
    return this.getState()
})
}

/**
* Used to cancel a message submitted via eth_sign.
*
* @param {string} msgId - The id of the message to cancel.
*/
cancelMessage (msgId, cb) {
const messageManager = this.messageManager
messageManager.rejectMsg(msgId)
if (cb && typeof cb === 'function') {
    cb(null, this.getState())
}
}
```

### personal_sign methods

```js
/**
* Called when a dapp uses the personal_sign method.
* This is identical to the Geth eth_sign method, and may eventually replace
* eth_sign.
*
* We currently define our eth_sign and personal_sign mostly for legacy Dapps.
*
* @param {Object} msgParams - The params of the message to sign & return to the Dapp.
* @param {Function} cb - The callback function called with the signature.
* Passed back to the requesting Dapp.
*/
newUnsignedPersonalMessage (msgParams, cb) {
if (!msgParams.from) {
    return cb(cleanErrorStack(new Error('MetaMask Message Signature: from field is required.')))
}

const msgId = this.personalMessageManager.addUnapprovedMessage(msgParams)
this.sendUpdate()
this.opts.showUnconfirmedMessage()
this.personalMessageManager.once(`${msgId}:finished`, (data) => {
    switch (data.status) {
    case 'signed':
        return cb(null, data.rawSig)
    case 'rejected':
        return cb(cleanErrorStack(new Error('MetaMask Message Signature: User denied message signature.')))
    default:
        return cb(cleanErrorStack(new Error(`MetaMask Message Signature: Unknown problem: ${JSON.stringify(msgParams)}`)))
    }
})
}

/**
* Signifies a user's approval to sign a personal_sign message in queue.
* Triggers signing, and the callback function from newUnsignedPersonalMessage.
*
* @param {Object} msgParams - The params of the message to sign & return to the Dapp.
* @returns {Promise<Object>} - A full state update.
*/
signPersonalMessage (msgParams) {
log.info('MetaMaskController - signPersonalMessage')
const msgId = msgParams.metamaskId
// sets the status op the message to 'approved'
// and removes the metamaskId for signing
return this.personalMessageManager.approveMessage(msgParams)
.then((cleanMsgParams) => {
    // signs the message
    return this.keyringController.signPersonalMessage(cleanMsgParams)
})
.then((rawSig) => {
    // tells the listener that the message has been signed
    // and can be returned to the dapp
    this.personalMessageManager.setMsgStatusSigned(msgId, rawSig)
    return this.getState()
})
}

/**
* Used to cancel a personal_sign type message.
* @param {string} msgId - The ID of the message to cancel.
* @param {Function} cb - The callback function called with a full state update.
*/
cancelPersonalMessage (msgId, cb) {
const messageManager = this.personalMessageManager
messageManager.rejectMsg(msgId)
if (cb && typeof cb === 'function') {
    cb(null, this.getState())
}
}

```

### stream-handbook

> [This document](https://github.com/substack/stream-handbook) covers the basics of how to write node.js programs with streams.