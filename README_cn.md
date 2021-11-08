# SimpleWallet 协议文档

版本：2.0

最后更新：2021.9.7

## 简介
SimpleWallet是一个数字资产钱包和移动应用（如：区块链游戏）的通用对接协议，支持Ethereum等EVM兼容区块链。

## SDK & Demo

iOS SDK
https://github.com/mathwallet/MathWallet5SDK-iOS

Android SDK
https://github.com/mathwallet/MathWallet5SDK-Android

## 功能列表

- 登录

场景：dapp的移动端APP拉起钱包APP，请求登录授权


- 支付

场景：dapp的移动端拉起钱包APP请求支付授权

- 发送交易体上链

场景：dapp的移动端拉起钱包App，执行Transaction调用智能合约接口

- 打开 DApp URL

场景：dapp的移动端拉起钱包App，打开对应DApp URL

## 协议内容

### 1. 钱包APP在系统注册拦截协议

钱包APP应在操作系统内注册拦截协议（URL Scheme、appLink），以便dapp的APP拉起钱包应用。

以下为协议接入方法：
> mathwallet://mathwallet.org?sw={json数据}

协议基础结构：

```

// 请求数据包结构
{
    protocol	string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
    version     string   // 协议版本信息，如2.0
    chain  	object   // 见下面，公链数据结构
    dapp        object   // 见下面，dapp 信息数据结构
    id          string   // dapp生成的，用于作为请求id
    action      string   // 具体操作(如: login\transaction\openURL\signMessage)
    data        object   // 详情见下面场景
    callback    string   // 用户完成操作后，钱包回调拉起dapp移动端的回调URL，可选
			 // 如appABC://abc.com?response={response}
    		         // response 见下面，响应数据包结构
}

// 公链数据结构(公链标识表格: https://github.com/mathwallet/SimpleWallet)
{
    type    string   // 公链类型(如：EVM)，必须
    id      string   // 公链区分id(如：1)，必须
}

// dapp 信息数据结构
{
    name    string   // dapp名字，用于在钱包APP中展示，必须
    icon    string   // dapp图标Url，用于在钱包APP中展示，可选
}


// 回调数据包结构
{
    id	    string    // dapp 生成的请求id
    code    number    // 0为用户取消，1为成功, 2为失败
    result  Object    // 具体数据，见下面使用场景，可选
    message string    // 错误信息，可选
}

```

### 2. 登录

#### 场景：dapp的移动端应用拉起钱包App，请求登录授权

> 	适合dapp的移动端(iOS或安卓端）接入。业务流程图如下：

![image](http://qiniu.eth.fm/2021-09-03-flow.jpg)

- dapp的移动端拉起钱包APP要求登录授权，并传递给钱包App如下的数据，数据格式为json：

```
// dapp传递给钱包APP的数据包结构

{
    ...
    "id": "...",
    "action": "login",
    "data":{}
}
```

- 钱包App 响应数据包结构（成功）
```
// 成功
{
    "id": "...",
    "code": 1,
    "result": {
        "name": "Ben", 			// 账户名称
        "address": "0x000000..."	// 账户地址
    }
}

// 取消或错误
{
    "id": "...",
    "code": 0,
    "message": "用户取消"
}
```

### 3. 支付

#### 场景：dapp的移动端拉起钱包App，请求支付授权

> 业务流程图如下：

![image](http://qiniu.eth.fm/2021-09-03-flow.jpg)

```
// dapp传递给钱包APP的数据包结构
// EVM 系
{
    ...
    "id": "...",
    "action": "transaction",
    "data":{
	"from": "0x00000...", 			// 付款人的地址，必须
	"to": "0x00000...",   			// 付款人的地址，必须
	"value": "1000000000000",   		// 转账数量(1 ETH = 100000000000000000)，必须
	"data": "0x", 				// 可选
    }
    ...
}
// SOLANA 系
{
    ...
    "id": "...",
    "action": "transaction",
    "data":{
	"recentBlockhash": "0x00000...", 					// BlockHash，可选
	"instructions": [
	  {
	    "keys": [
	    	{
		  "pubkey":"GNutLCXQEEcmxkJH5f5rw51bTW2QcLGXqitmN3EaVPoV",	// An account's public key，必须
	    	  "isSigner": true,						// True if an instruction requires a transaction signature matching `pubkey`.必须
	    	  "isWritable": true,						// True if the `pubkey` can be loaded as a read-write account.必须
		}
	    ],
	    "programId": "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA",		// Program Id，必须
	    "data": "0x0000000"							// Program input，必须
	  }
	]
    }
    ...
}
// 其它公链（待补充）
```

- 钱包App 响应数据包结构（成功）
```
// 成功
{
    "id": "...",
    "code": 1,
    "result": {
        "hash": "0x000000"
    }
}

// 取消或错误
{
    "id": "...",
    "code": 0,
    "message": "用户取消"
}
```

- 钱包组装上述数据，生成一笔Ethereum的transaction，用户授权此笔转账后，提交转账数据到Ethereum主网；如果有callback，则回调拉起dapp的应用
- dapp可根据callback里的txHash去主网查询此笔交易（不能完全依赖此方式来确认用户的付款）；或自行搭建节点监控Ethereum（或EOS）主网同步节点，检查代币是否到账


### 4. 消息签名

#### 场景：dapp的移动端拉起钱包进行签名

```
// dapp传递给钱包APP的数据包结构

{
    ...
    "id": "...",
    "action": "signMessage",
    "data":{
	"address": "0x0000000", 			// 签名地址
	"message": "hello world", 			// 签名数据，格式 utf-8、hex（16进制数据必须0x开头），必须
    }
    ...
}
```

- 钱包App 响应数据包结构（成功）
```
// 成功
{
    "id": "...",
    "code": 1,
    "result": {
        "signature": "0x000000"
     }
}

// 取消或错误
{
    "id": "...",
    "code": 0,
    "message": "用户取消"
}
```

### 5. 打开 DApp URL

#### 场景：dapp的移动端拉起钱包App，并在钱包App中打开指定 URL


```
// dapp传递给钱包APP的数据包结构

{
    ...
    "id": "...",
    "action": "openURL",
	"data":{
            "link": "https://mathwallet.org", 			// 链接地址，必须
	}
    ...
}
```

- 钱包App 响应数据包结构（成功）
```
// 成功
{
    "id": "...",
    "code": 1,
    "result": {}
}

// 取消或错误
{
    "id": "...",
    "code": 0,
    "message": "用户取消"
}
```
