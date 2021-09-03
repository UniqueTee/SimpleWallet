# SimpleWallet 协议文档

版本：1.1

最后更新：2021.9.3

## 简介
SimpleWallet是一个数字资产钱包和移动应用（如：区块链游戏）的通用对接协议，支持Ethereum等EVM兼容区块链。

## SDK & Demo

iOS SDK
https://github.com/mathwallet/MathWalletSDK-iOS

Android SDK
https://github.com/mathwallet/MathWalletSDK-Android

## 功能列表

- 登录

场景：dapp的移动端APP拉起钱包APP，请求登录授权


- 支付

场景：dapp的移动端拉起钱包APP请求支付授权

- 交易体

场景：dapp的移动端拉起钱包App，执行Transaction调用智能合约接口

- 打开 DApp URL

场景：dapp的移动端拉起钱包App，打开对应DApp URL

## 协议内容

### 1. 钱包APP在系统注册拦截协议

钱包APP应在操作系统内注册拦截协议（URL Scheme、appLink），以便dapp的APP拉起钱包应用。

以下为协议接入方法：
> mathwallet://mathwallet.org?param={json数据}

### 2. 登录

#### 场景：dapp的移动端应用拉起钱包App，请求登录授权

> 	适合dapp的移动端(iOS或安卓端）接入。业务流程图如下：

![image](http://qiniu.eth.fm/2021-09-03-flow.jpg)

- dapp的移动端拉起钱包APP要求登录授权，并传递给钱包App如下的数据，数据格式为json：

```
// dapp传递给钱包APP的数据包结构
{
    protocol	string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
    version     string   // 协议版本信息，如1.1
    blockchain  string   // 公链标识（如:ethereum）公链标识表格: https://github.com/mathwallet/SimpleWallet
    dappName    string   // dapp名字，用于在钱包APP中展示
    dappIcon    string   // dapp图标Url，用于在钱包APP中展示
    action      string   // 赋值为login
    uuID        string   // dapp生成的，用于dapp登录验证唯一标识
    loginUrl    string   // dapp server生成的，用于接受此次登录验证的URL
    expired	number   // 登录过期时间，unix时间戳
    loginMemo	string   // 登录的备注信息，钱包用来展示，可选
    callback    string   // 用户完成操作后，钱包回调拉起dapp移动端的回调URL,如appABC://abc.com?action=login，可选
    		         // 钱包回调时在此URL后加上操作结果(&result)，如：appABC://abc.com?action=login&result=1,
			 // result的值为：0为用户取消，1为成功,  2为失败
}
```

- dapp server收到数据，验证sign签名数据，返回success == true或false；若验证成功，则在dapp的业务逻辑中，将该用户设为已登录状态

### 3. 支付

#### 场景：dapp的移动端拉起钱包App，请求支付授权

> 业务流程图如下：

![image](http://qiniu.eth.fm/2021-09-03-flow.jpg)

```
// 传递给钱包APP的数据包结构
{
	protocol    string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
	version     string   // 协议版本信息，如1.1
	blockchain  string   // 公链标识（如:ethereum）公链标识表格: https://github.com/mathwallet/SimpleWallet
	action      string   // 支付时，赋值为transfer
	dappName    string   // dapp名字，用于在钱包APP中展示，可选
	dappIcon    string   // dapp图标Url，用于在钱包APP中展示，可选
	from        string   // 付款人的Ethereum地址，可选
	to          string   // 收款人的Ethereum地址，必须
	amount      number   // 转账数量(带精度，如1.0000)，必须
	contract    string   // 转账的token所属的contract账号名或地址
	symbol      string   // 转账的token名称，必须
	precision   number   // 转账的token的精度，小数点后面的位数，必须
	dappData    string   // 由dapp生成的业务参数信息，需要钱包在转账时附加在memo或data中发出去，可选
			     // Ethereum 钱包转账时，dappData会附加到交易体的data字段中（16 进制数据）
	desc	    string   // 交易的说明信息，钱包在付款UI展示给用户，最长不要超过128个字节，可选
	expired	    number   // 交易过期时间，unix时间戳
        callback    string   // 用户完成操作后，钱包回调拉起dapp移动端的回调URL,如appABC://abc.com?action=transfer，可选
    		             // 钱包回调时在此URL后加上操作结果(result、txID)，如：appABC://abc.com?action=transfer&result=1&txID=xxx,
			     // result的值为：0为用户取消，1为成功,  2为失败；txID为EOS主网上该笔交易的id（若有）
}
```

- 钱包组装上述数据，生成一笔Ethereum的transaction，用户授权此笔转账后，提交转账数据到Ethereum主网；如果有callback，则回调拉起dapp的应用
- dapp可根据callback里的txID去主网查询此笔交易（不能完全依赖此方式来确认用户的付款）；或自行搭建节点监控Ethereum（或EOS）主网同步节点，检查代币是否到账

### 4. 交易体

#### 场景：dapp的移动端拉起钱包App，执行Transaction

> Ethereum(以太坊)

	参考支付场景，dappData用来存放交易的Data数据。

### 5. 消息签名

#### 场景：dapp的移动端拉起钱包进行签名

 ```
// 传递给钱包APP的数据包结构
{
	protocol    string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
	version     string   // 协议版本信息，如1.1
	blockchain  string   // 公链标识（如:ethereum）公链标识表格: https://github.com/mathwallet/SimpleWallet
	action      string   // 跳转时，赋值为signMessage
	dappName    string   // dapp名字，用于在钱包APP中展示，可选
	dappIcon    string   // dapp图标Url，用于在钱包APP中展示，可选
	desc        string   // 跳转的说明信息，钱包在付款UI展示给用户，最长不要超过128个字节，可选
	message     string   // 要签名的数据
	isHash       bool     // 是否是sha256 hash
	callback    string   // 用户完成操作后，钱包回调拉起dapp移动端的回调URL,
				// 可选,如appABC://abc.com?action=signMessage，
				// 钱包回调时在此URL后加上操作结果(result)，
				// 如：appABC://abc.com?action=signMessage&result=1&account=xxx&pubKey=xxx&signedMessage=xxx
				//result的值为：0为用户取消，1为成功,  2为失败；signedMessage被签名后的数据
}

```

### 6. 打开 DApp URL

#### 场景：dapp的移动端拉起钱包App，并在钱包App中打开某DApp URL

 ```
// 传递给钱包APP的数据包结构
{
	protocol    string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
	version     string   // 协议版本信息，如1.1
	blockchain  string   // 公链标识（如:ethereum）公链标识表格: https://github.com/mathwallet/SimpleWallet
	action      string   // 跳转时，赋值为openUrl
	dappName    string   // dapp名字，用于在钱包APP中展示，可选
	dappIcon    string   // dapp图标Url，用于在钱包APP中展示，可选
	desc        string   // 跳转的说明信息，钱包在付款UI展示给用户，最长不要超过128个字节，可选
	dappUrl     string   // 要跳转的DApp URL链接
	callback    string   // 用户完成操作后，钱包回调拉起dapp移动端的回调URL,
				// 可选,如appABC://abc.com?action=openUrl，
				// 钱包回调时在此URL后加上操作结果(result)，
				// 如：appABC://abc.com?action=openUrl&result=0,
				// result的值为：0为用户取消,  2为失败；成功不回调；
				// 该回调只会在打开URL失败或取消时触发
}
```

## 错误处理

### 登录和转账接口回调接口错误处理

- code不等于0则请求失败
```
// 错误返回

{
    code number     //错误符，等于0是成功，大于0说明请求失败，dapp返回具体的错误码
    error string    //返回的提示信息
}
```

### 打开 DApp URL 错误处理

- result 不等与 1 表示取消或失败
```
// 错误返回
errorMesssge string    	//返回的提示信息
```
