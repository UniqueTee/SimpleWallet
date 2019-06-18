# SimpleWallet 协议文档

版本：1.0

协议最后更新：2019.01.04

## 简介
SimpleWallet是一个数字资产钱包和dapp的通用对接协议，支持Ethereum、EOS、EOS Force、TRON。

## SDK

iOS SDK
https://github.com/mathwallet/MathWalletSDK-iOS

Android SDK
https://github.com/mathwallet/MathWalletSDK-Android

## 功能列表
- 登录
1. 场景1：钱包App扫二维码进行登录，适用于WEB版dapp
2. 场景2：dapp的移动端APP拉起钱包APP，请求登录授权
3. 场景3：钱包APP内嵌dapp的H5页面，进行登录（暂无）

- 支付
1. 场景1：钱包扫码支付，适用于WEB版dapp
2. 场景2：dapp的移动端拉起钱包APP请求支付授权
3. 场景3：钱包APP内嵌dapp的H5页面，进行支付（暂无）

- 交易体
1. 场景1：钱包扫描二维码，执行Transaction
2. 场景2：dapp的移动端拉起钱包App，执行Transaction

- 打开 DApp URL
1. 场景1：dapp的移动端拉起钱包App，打开对应DApp URL

## 协议内容

### 1. 钱包APP在系统注册拦截协议

钱包APP应在操作系统内注册拦截协议（URL Scheme、appLink），以便dapp的APP拉起钱包应用。

以下为协议接入方法：
> mathwallet://mathwallet.org?param={json数据}

兼容之前的SimpleWallet协议
> simplewallet://eos.io?param={json数据}

### 2. 登录


#### 场景1：使用钱包扫码二维码登录
> 	适合dapp的网站接入。
>
> 业务流程图如下：

![image](http://on-img.com/chart_image/5b658d5de4b0be50eacf8f0c.png?t=1)

- dapp生成二维码，钱包扫描dapp web提供的登录二维码，此二维码的数据格式为json，包含以下数据：
```
// 登录的二维码数据格式
{
    protocol	string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
    version     string   // 协议版本信息，如1.0
    blockchain  string   // 公链标识（eosio、eosforce、ethereum,tron等）
    dappName    string   // dapp名字
    dappIcon    string   // dapp图标
    action      string   // 赋值为login
    uuID        string   // dapp server生成的，用于此次登录验证的唯一标识
    loginUrl    string   // dapp server上用于接受登录验证信息的url
    expired		number		// 二维码过期时间，unix时间戳
    loginMemo	string   // 登录备注信息，钱包用来展示，可选
}
```
- 钱包对登录相关数据进行签名
```
// 生成sign算法
let data = timestamp + account + uuID + ref     //ref为钱包名，标示来源
sign = ecc.sign(data, privateKey)
```
- 钱包将签名后的数据POST到dapp提供的loginUrl，请求登录验证
```
 // 请求登录验证的数据格式
{
    protocol   string     // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
    version    string     // 协议版本信息，如1.0
    blockchain string    // 公链标识（eosio、eosforce、ethereum,tron等）
    timestamp  number     // 当前UNIX时间戳
    sign       string     // eos、ethereum签名
    uuID       string     // dapp server生成的，用于此次登录验证的唯一标识
    account    string     // eos账户名、ethereum地址等
    ref        string     // 来源,如钱包名
}
```
- dapp server收到数据，验证sign签名数据，并返回结果code；若验证成功，则在dapp的业务逻辑中，将该用户设为已登录状态

```
// 错误返回
{
    code number     //错误符，等于0是成功，大于0说明请求失败，dapp返回具体的错误码
    error string    //返回的提示信息
}

```
#### 场景2：dapp的移动端应用拉起钱包App，请求登录授权
> 	适合dapp的移动端(iOS或安卓端）接入。业务流程图如下：

![image](http://on-img.com/chart_image/5b6591fbe4b0edb750f9a364.png?t=1)
- dapp的移动端拉起钱包APP要求登录授权，并传递给钱包App如下的数据，数据格式为json：
```
// dapp传递给钱包APP的数据包结构
{
    protocol	string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
    version     string   // 协议版本信息，如1.0
    blockchain  string   // 公链标识（eosio、eosforce、ethereum等）
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
#### 场景1：钱包扫描二维码进行支付
> 业务流程图如下:

![image](http://on-img.com/chart_image/5b6594bae4b053a09c24fa9a.png?t=1)

```
// dapp生成的用于钱包扫描的二维码数据格式
{
	protocol    string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
	version     string   // 协议版本信息，如1.0
	blockchain  string   // 公链标识（eosio、eosforce、ethereum等）
	dappName    string   // dapp名字，用于在钱包APP中展示，可选
	dappIcon    string   // dapp图标Url，用于在钱包APP中展示，可选
	action      string   // 支付时，赋值为transfer，必须
	from        string   // 付款人的EOS账号、EOSForce账号或Ethereum地址，可选
	to          string   // 收款人的EOS账号、EOSForce账号或Ethereum地址，必须
	amount      number   // 转账数量(带精度，如1.0000 EOS)，必须
	contract    string   // 转账的token所属的contract账号名或地址，必须
	symbol      string   // 转账的token名称，必须
	precision   number   // 转账的token的精度，小数点后面的位数，必须
	dappData    string   // 由dapp生成的业务参数信息，需要钱包在转账时附加在memo或data中发出去，格式为:k1=v1&k2=v2，可选
			     // 钱包转账时还可附加ref参数标明来源，如：k1=v1&k2=v2&ref=walletname
	desc	    string   // 交易的说明信息，钱包在付款UI展示给用户，最长不要超过128个字节，可选
	expired	    number   // 交易二维码过期时间，unix时间戳
        callback    string   // 用户完成操作后，钱包回调拉起dapp移动端的回调URL,如https://abc.com?action=login&qrcID=123，可选
    		             // 钱包回调时在此URL后加上操作结果(result、txID)，如：https://abc.com?action=login&qrcID=123&result=1&txID=xxx,
			     // result的值为：0为用户取消，1为成功,  2为失败；txID为EOS、EOS原力或以太坊主网上该笔交易的id（若有）
}
```
- 钱包组装上述数据，生成一笔EOS或Ethereum的transaction，用户授权此笔转账后，提交转账数据到EOS或Ethereum主网；若有callback参数，则进行回调访问
- dapp可根据callback中的txID去主网查询此笔交易（不能完全依赖此方式来确认用户的付款）；或dapp自行搭建节点监控EOS、EOS原力或Ethereum主网，检查代币是否到账
- 对于流行币种如IQ，如果二维码中给出的contract名和官方的合约名不一致，钱包方要提醒用户，做二次确认
- 钱包应该提醒用户注意辨别二维码的来源，避免被钓鱼攻击
- TRON钱包支付，请查看协议第4部分（交易体）封装


#### 场景2：dapp的移动端拉起钱包App，请求支付授权
> 业务流程图如下：

![image](http://on-img.com/chart_image/5b659391e4b0f8477da3138b.png?t=1)
```
// 传递给钱包APP的数据包结构
{
	protocol    string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
	version     string   // 协议版本信息，如1.0
	blockchain  string   // 公链标识（eosio、ethereum、eosforce等）
	action      string   // 支付时，赋值为transfer
	dappName    string   // dapp名字，用于在钱包APP中展示，可选
	dappIcon    string   // dapp图标Url，用于在钱包APP中展示，可选
	from        string   // 付款人的EOS账号或Ethereum地址，可选
	to          string   // 收款人的EOS账号或Ethereum地址，必须
	amount      number   // 转账数量(带精度，如1.0000 EOS)，必须
	contract    string   // 转账的token所属的contract账号名或地址
	symbol      string   // 转账的token名称，必须
	precision   number   // 转账的token的精度，小数点后面的位数，必须
	dappData    string   // 由dapp生成的业务参数信息，需要钱包在转账时附加在memo或data中发出去，格式为:k1=v1&k2=v2，可选
			     // 钱包转账时还可附加ref参数标明来源，如：k1=v1&k2=v2&ref=walletname
	desc	    string   // 交易的说明信息，钱包在付款UI展示给用户，最长不要超过128个字节，可选
	expired	    number   // 交易过期时间，unix时间戳
        callback    string   // 用户完成操作后，钱包回调拉起dapp移动端的回调URL,如appABC://abc.com?action=transfer，可选
    		             // 钱包回调时在此URL后加上操作结果(result、txID)，如：appABC://abc.com?action=transfer&result=1&txID=xxx,
			     // result的值为：0为用户取消，1为成功,  2为失败；txID为EOS主网上该笔交易的id（若有）
}
```
- 钱包组装上述数据，生成一笔EOS或Ethereum的transaction，用户授权此笔转账后，提交转账数据到EOS或Ethereum主网；如果有callback，则回调拉起dapp的应用
- dapp可根据callback里的txID去主网查询此笔交易（不能完全依赖此方式来确认用户的付款）；或自行搭建节点监控EOS或Ethereum主网，检查代币是否到账
- TRON支付，请查看协议第4部分（交易体）封装

### 4. 交易体
#### 场景1：钱包扫描二维码，执行Transaction

> Ethereum
 参考支付场景，dappData用来存放交易的Data数据。

> EOS、EOS原力
 ```
// 传递给钱包APP的数据包结构
{
	protocol    string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
	version     string   // 协议版本信息，如1.0
	blockchain  string   // 公链标识（eosio、ethereum、eosforce、tron等）
	action      string   // 支付时，赋值为transaction
	dappName    string   // dapp名字，用于在钱包APP中展示，可选
	dappIcon    string   // dapp图标Url，用于在钱包APP中展示，可选
	actions     string   // JSON 数组，格式：[{"code": "eosio.token","action": "transfer","binargs":"00000"}]
	from        string   // 要执行交易的账户，可选
	desc        string   // 交易的说明信息，钱包在付款UI展示给用户，最长不要超过128个字节，可选
	expired     number   // 交易过期时间，unix时间戳
	callback    string   // 用户完成操作后，钱包回调拉起dapp移动端的回调URL,
				// 如https://abc.com?action=transaction&qrcID=123，可选
				// 钱包回调时在此URL后加上操作结果(result、txID)，
				// 如：https://abc.com?action=transaction&qrcID=123&result=1&txID=xxx,
				// result的值为：0为用户取消，1为成功,  2为失败；txID为EOS主网上该笔交易的id（若有）
}

```
#### 场景2：dapp的移动端拉起钱包App，执行Transaction

> Ethereum(以太坊)

	参考支付场景，dappData用来存放交易的Data数据。

> EOS主网、EOS原力

 ```
// 传递给钱包APP的数据包结构
{
	protocol    string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
	version     string   // 协议版本信息，如1.0
	blockchain  string   // 公链标识（eosio、eosforce）
	action      string   // 支付时，赋值为transaction
	dappName    string   // dapp名字，用于在钱包APP中展示，可选
	dappIcon    string   // dapp图标Url，用于在钱包APP中展示，可选
	actions     string   // JSON 数组，格式：[{"code": "eosio.token","action": "transfer","binargs":"00000"}]
	from        string   // 要执行交易的账户，可选
	desc        string   // 交易的说明信息，钱包在付款UI展示给用户，最长不要超过128个字节，可选
	expired     number   // 交易过期时间，unix时间戳
	callback    string   // 用户完成操作后，钱包回调拉起dapp移动端的回调URL,
				// 如appABC://abc.com?action=transfer，
				// 可选钱包回调时在此URL后加上操作结果(result、txID)，
				// 如：appABC://abc.com?action=transaction&result=1&txID=xxx,
				// result的值为：0为用户取消，1为成功,  2为失败；txID为EOS主网上该笔交易的id（若有）
}

```

> TRON(波场)

 ```
// 传递给钱包APP的数据包结构
{
	protocol    string	// 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
	version     string	// 协议版本信息，如1.0
	blockchain  string	// 公链标识（tron）
	action      string	// 调用时，赋值为transaction
	dappName    string	// dapp名字，用于在钱包APP中展示，可选
	dappIcon    string	// dapp图标Url，用于在钱包APP中展示，可选
	contract    string	// JSON 数组，格式参考下面说明；
	from        string 	// 要执行交易的账户，可选
	desc        string	// 交易的说明信息，钱包在付款UI展示给用户，最长不要超过128个字节，可选
	expired     number	// 交易过期时间，unix时间戳
	callback    string	// 用户完成操作后，钱包回调拉起dapp移动端的回调URL,
				// 如appABC://abc.com?action=transfer，
				// 可选钱包回调时在此URL后加上操作结果(result、txID)，
				// 如：appABC://abc.com?action=transaction&result=1&txID=xxx,
				// result的值为：0为用户取消，1为成功,  2为失败；txID为TRON主网上该笔交易的id（若有）
}
```

// 传递给钱包APP的contract字段说明
1. TriggerSmartContract
```
[{
	"parameter": {
		"value": {
			"data": "7365870b0000000000000000000000000000000000000000000000000000000000000060",
			"owner_address": "TWXNtL6rHGyk2xeVR3QqEN9QGKfgyRTeU2",
			"contract_address": "TWXNtL6rHGyk2xeVR3QqEN9QGKfgyRTeU2",
			"call_value": 10000000
		},
		"type_url": "type.googleapis.com/protocol.TriggerSmartContract"
	},
	"type": "TriggerSmartContract"
}]
```

2. TransferAssetContract
```
[{
	"parameter": {
		"value": {
			"amount": 2,
			"asset_name": "tokenId",  // asset_name对应的值为tokenId
			"owner_address": "TWXNtL6rHGyk2xeVR3QqEN9QGKfgyRTeU2",
			"to_address": "TWXNtL6rHGyk2xeVR3QqEN9QGKfgyRTeU2"
		},
		"type_url": "type.googleapis.com/protocol.TransferAssetContract"
	},
	"type": "TransferAssetContract"
}]
```

3. TransferContract
```
[{
	"parameter": {
		"value": {
			"amount": 8,
			"owner_address": "TWXNtL6rHGyk2xeVR3QqEN9QGKfgyRTeU2",
			"to_address": "TWXNtL6rHGyk2xeVR3QqEN9QGKfgyRTeU2"
		},
		"type_url": "type.googleapis.com/protocol.TransferContract"
	},
	"type": "TransferContract"
}]
```
### 5. 消息签名
#### 场景1：扫描二维码进行签名
 ```
// 传递给钱包APP的数据包结构
{
	protocol    string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
	version     string   // 协议版本信息，如1.0
	blockchain  string   // 公链标识（eosio、ethereum、eosforce、tron等）
	action      string   // 跳转时，赋值为signMessage
	dappName    string   // dapp名字，用于在钱包APP中展示，可选
	dappIcon    string   // dapp图标Url，用于在钱包APP中展示，可选
	desc        string   // 跳转的说明信息，钱包在付款UI展示给用户，最长不要超过128个字节，可选
	message     string   // 要签名的数据
	isHash       bool     // 是否是sha256 hash
	callback    string   // 用户完成操作后，钱包回调拉起dapp移动端的回调URL,
				// 如https://abc.com?action=signMessage，可选
				// 钱包回调时在此URL后加上操作结果(signedMessage)，
				// 如：https://abc.com?action=signMessage&result=1&account=xxx&pubKey=xxx&signedMessage=xxx,
				// result的值为：0为用户取消，1为成功,  2为失败；signedMessage被签名后的数据
}
 ```

#### 场景2：dapp的移动端拉起钱包进行签名
 ```
// 传递给钱包APP的数据包结构
{
	protocol    string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
	version     string   // 协议版本信息，如1.0
	blockchain  string   // 公链标识（eosio、ethereum、eosforce、tron等）
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
#### 场景1：dapp的移动端拉起钱包App，打开对应DApp URL

 ```
// 传递给钱包APP的数据包结构
{
	protocol    string   // 协议名，钱包用来区分不同协议，本协议为 SimpleWallet
	version     string   // 协议版本信息，如1.0
	blockchain  string   // 公链标识（eosio、ethereum、eosforce、tron等）
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
### 网络回调接口错误处理
- code不等于0则请求失败
```
// 错误返回

{
    code number     //错误符，等于0是成功，大于0说明请求失败，dapp返回具体的错误码
    error string    //返回的提示信息
}
```

### 麦子钱包DApp跳转错误处理
- result不等与1 表示取消或失败
```
// 错误返回
errorMesssge string    	//返回的提示信息
```
