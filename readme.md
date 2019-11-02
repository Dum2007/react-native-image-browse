#Secret 新币种接入手册


##1.前言
Secret钱包开放允许第三方币种接入，包括私链和公链。
###1)公链
公链支持以太坊ERC20､EOS两类币种的接入，步骤如下：

+ 提交币种的合约地址
+ 提交附加币种信息到开放平台审核，需要提交的信息参看__2.币种信息__
+ 500个SIE的币种接入费用
+ 向托管帐号转帐价值500USD的对应货币


###2)私链
目前支持任意的私链接入，步骤如下：

+ 币种发行方按照下面__3.接入API__部分的说明标准，开发币种操作API并提供给Secret，我们将会以JSON RPC调用你们开发的API
+ 提交币种信息到开放平台审核，需要提交的信息参看__2.币种信息__
+ 币处在后台审核通过后，需要支付价值500SIE的币种接入费用
+ 向需向托管帐号转帐价值500USD的对应货币



##2.币种信息
接入币种需要提供币种的以下信息

| 项 | 说明 |
|---|----|
|币种名称||
|币种说明|币种的详细说明信息|
|币种图标|270*270 png格式图标的url|
|API URL|实现币种操作的API URL|
|精度|币种的金额最大小数位数|
|白皮书|币种发行的白皮书|
|官方URL|可提供官网的URL或APP下载地址|

___在开放平台后台审核通过后，才表示币种接入成功。___


##3.接入API
###1) 开发描述
如果需要在Secret钱包接入第三方发行的币种，需先实现第三方币种的Json Rpc Api Server，也就是币种发行方必须要按照Secret定义的币种操作API，实现币种的转帐、余额查询、交易查询、生成帐号等的区块链操作。

+ 币种发行方需要实现的API列表参看__3) 币种API实现标准__
+ Json Rpc Api 标准参考<https://www.jsonrpc.org/specification>
+ 所有Api以POST方式提交


###2) 认证
所有币种操作API，必须加入签名验证，验证信息使用HTTP Header提交

__标准的头如下：__

```
Content-Type: application/json; charset=utf-8
Authorization: sign 801da1307e60824914991808f050951d
Timestamp: 1572340980

```

说明：

+ Authorization 为验证信息
+ Authorization 头格式为:  Authorization: sign <签名串>
+ Authorization 中的签名串格式为： md5(body+Timestamp+APIKEY)  //body为POST的json串，Timestamp为当前时间戳，APIKEY为币种分配的私钥
+ APIKEY私钥向开放平台索取
+ Timestamp 为当前的时间戳


###3) 币种API实现标准
####(1) 转帐
+ __Method__: send_address
+ __Parameters__: (String *fromAddress*, String *toAddress*, String *amount*, String *exSignKey*)
+ __Return__: String，成功返回转帐的结果hash
+ __Description__: 实现币种的转帐操作，由fromAddress向toAddress转帐amount，exSignKey为sha256签名串，格式为：sha256(fromAddress+toAddress+amount+privateKey)，__注意privateKey为用户的私钥__
+ __Example__:

___[Request]___

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "send_address",
    "params": ["0xe7c3347dc1ab20fa57b12ec660c6190c", "0x6078f15ace556c12ff84246ffddb3caa", "0.123456", "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08"]
}

```

___[Response]___

```
{
    "jsonrpc": "2.0",
    "id":"1",
    "result":"0xa353c2222ee17b2becccad1037b14c227af7f6b51bec00fa7cfe1c664a081111"
}
```


####(2) 获取余额
+ __Method__: get_balance
+ __Parameters__: (String *address*)
+ __Return__: String，成功返回地址帐户的余额
+ __Description__: 查询区块链上币种余额
+ __Example__:

___[Request]___

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "get_balance",
    "params": ["0xe7c3347dc1ab20fa57b12ec660c6190c"]
}

```

___[Response]___

```
{
    "jsonrpc": "2.0",
    "id":"1",
    "result":"0.0111123"
}
```

####(3) 创建帐号

+ __Method__: new_account
+ __Parameters__: ()
+ __Return__: JsonObject, 成功返回一个对象，里面包括帐号的address(公钥)和private_key(私钥)
+ __Description__: 
+ __Example__:

___[Request]___

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "new_account",
    "params": []
}

```

___[Response]___

```
{
    "jsonrpc": "2.0",
    "id":"1",
    "result":
    {
        "address":"0xe7c3347dc1ab20fa57b12ec660c6190c",
        "private_key":"123"
    }
}
```

####(4) 查询交易
+ __Method__: get_transation
+ __Parameters__: (String *hash*)
+ __Return__: JsonObject,成功返回该交易的相关信息，__注意：交易的返回数据可以根据自己区块链上的标准返回不同的数据，但[Response]示例中的所有字段必须有。__
+ __Description__: 根据hash查询交易的信息
+ __Example__:

___[Request]___

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "get_transation",
    "params": ["0xa353c2222ee17b2becccad1037b14c227af7f6b51bec00fa7cfe1c664a081111"]
}

```

___[Response]___

```
{
   "jsonrpc": "2.0",
   "id":1,
   "result": {
       "number": "1234", //区块号
       "hash": "0xa353c2222ee17b2becccad1037b14c227af7f6b51bec00fa7cfe1c664a081111",//交易hash
       "amount": "22",//交易金额
       "fee" : "0.1",//交易手续费
       "from_address",//转帐者
       "receive_address",//接收者
       "timestamp": "",//交易时间
       "confirm": "1",//交易是否确认，1已确认，0未确认
   }
}
```

####(5) 汇率查询
+ __Method__: get_rate
+ __Parameters__: ()
+ __Return__: string，返回币种兑换USD的汇率值
+ __Description__: 查询币种兑换USD的汇率
+ __Example__:

___[Request]___

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "get_rate",
    "params": []
}

```

___[Response]___

```
{
   "jsonrpc": "2.0",
   "id":1,
   "result": "0.0112"
}
```


##4.开放平台通知API
当第三方币种是通过区块链转帐时（不管是在哪个平台），在成功后，需要币种发行方通过调用Secret提供的API，以通知Secret币种有区块链的交易。

Secret的通知API为非标准的JSON API RPC，但Authorization校验与上面相同

Http Request Header如下：

```
Content-Type: application/json; charset=utf-8
Authorization: sign 801da1307e60824914991808f050951d
Timestamp: 1572340980
```

Http Request Body 部分json如下：

```
{
   "hash" : "0xa353c2222ee17b2becccad1037b14c227af7f6b51bec00fa7cfe1c664a081111",//交易hash
   "symbol" : "STE",//币种名称
   "from_address" : "0xe7c3347dc1ab20fa57b12ec660c6190c",//转帐者
   "receive_address" : "0x6078f15ace556c12ff84246ffddb3caa",//接收者
   "amount" : "0.12",//金额
   "blockNumber" : ""//区块号
}

```

Http Response Body 部分json如下

```
{
   "code" : 200,//200表示通知成功
   "msg" : "",
   "data": null
}
```


##5.附1

| 项 | 说明 |
|---|----|
|签名key|向Secret索取|
|通知API|测试:http://test.isecret.im/notice_transation|


