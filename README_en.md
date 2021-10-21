# SimpleWallet protocol document

Version: 2.0

Last updated: 2021.9.3

## Introduction

SimpleWallet is a universal protocol for connecting native blockchain game mobile app to MathWallet.

This protocol aims to reduce the development and adaption work of all parties through a low-coupling implementation of a wallet that helps to authorize login and payment of mobile dapp.

## SDK & Demo

iOS SDK
https://github.com/mathwallet/MathWalletSDK-iOS

Android SDK
https://github.com/mathwallet/MathWalletSDK-Android

## Function list

- Login
- Payment
- Sign a message
- Open Dapp URL

Scenario: The Mobile App requests payment authorization (Mobile DApp)

## Flow of the Protocol

### 1. The Wallet App registers the intercept protocol in the system

MathWallet App first registers the intercept protocol (URL Scheme, appLink) in the Operating System (OS) such that the App of dapp can pull up the wallet application. This is located at: mathwallet://mathwallet.org

Following which, the mobile terminal application of dapp calls this protocol and transfer data to the wallet App. The request format of data transfer is structured as:
mathwallet://mathwallet.org?sw={the json data}


```

// JSON Data
{
    protocol	string   // protocol name, wallet is used to distinguish different protocols, and this protocol is SimpleWallet
    version     string   // protocol version information (ex:2.0)
    chain  	object   // chain object
    dapp        object   // dapp information
    id          string   // request id
    action      string   // reqeust action, ex: login,transaction,openURL,signMessage
    data        object   // reqeust Data
    callback    string   //  after the user completes the operation, the wallet callback pulls up the callback URL of the dapp mobile terminal, optional
			 // such as appABC://abc.com?response={response}
}

// Chain Object
// Chain name table:: https://github.com/mathwallet/SimpleWallet
{
    type    string   // chain type(ex：EVM),required
    id      string   // chain id(1),required
}

// DApp Object
{
    name    string   // dapp name for display in the wallet APP
    icon    string   // dapp icon Url for display in the wallet APP
}


// Callback
{
    id	    string    // request id
    code    number    // The value of result is: 0 for user cancel, 1 for success and 2 for failure
    result  Object    // response data, optional
    message string    // error message, optional
}

```

### 2. Login

#### Scenario: The mobile App of dapp pulls up the wallet App and requests the login authorization

Suitable for dapp mobile (iOS or Android) access. Business flow chart is as follows:

- The mobile terminal of dapp pulls up the wallet App, which requires the login authorization, and transfers the following data to the wallet App in json format:
```
// the data package structure transfered by dapp to wallet APP

{
    ...
    "id": "...",
    "action": "login",
    "data":{}
}
```

- Response
```
// Success
{
    "id": "...",
    "code": 1,
    "result": {
        "name": "Ben", 			// wallet name
        "address": "0x000000..."	// wallet address
    }
}

// Cancel or Error
{
    "id": "...",
    "code": 0,
    "message": "Unknown Error"
}
```

### 3. Payment

#### Scenario: The mobile end of dapp pulls up the wallet App and requests payment authorization

Business flow chart is as follows:

![](http://qiniu.eth.fm/2021-09-03-flow.jpg)


The data package structure transfered by dapp to wallet APP
```
// EVM
{
    ...
    "id": "...",
    "action": "transaction",
    "data":{
	"from": "0x00000...", 			// Address of payer, required
	"to": "0x00000...",   			// Address of recipient, required
	"value": "100000000000000",   		// The amount of transfers(1 ETH = 10000000000000000)，required
	"data": "0x", 				// optional
    }
    ...
}
// To be added

```

- Response
```
// Success
{
    "id": "...",
    "code": 1,
    "result": {
        "hash": "0x000000"
    }
}

// Cancel or Error
{
    "id": "...",
    "code": 0,
    "message": "Unknown Error"
}
```

- The wallet assembles the above data and generates an transaction. After the user authorizes the transfer, the user submits the transfer data to the blockchain; If there is callback, pulls the dapp application
- The Dapp will either check this transaction from the mainnet according to the txHash in callback (it cannot completely rely on this method to confirm the user's payment); or the dapp will set up the node to monitor the blockchain by itself, and check whether the tokens are received

### 4. Sign a message

#### Scenario: The mobile end of dapp pulls up the wallet App and sign a specific message for validation

The data package structure transfered by dapp to wallet APP

```
{
    ...
    "id": "...",
    "action": "signMessage",
    "data":{
	"address": "0x0000000", 			// address
	"message": "hello world", 			// sign message( utf-8, hex )
    }
    ...
}

```

### 5. Open Dapp URL

#### Scenario: The mobile end of dapp pulls up the wallet App and open URL inside the wallet in-app browser

The data package structure transfered by dapp to wallet APP
```
{
    ...
    "id": "...",
    "action": "signMessage",
    "data":{
	"link": "https://mathwallet.org", 			// link
    }
    ...
}

```
