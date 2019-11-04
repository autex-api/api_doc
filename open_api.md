# REST 行情、交易 API

[TOC]



### 签名认证

API 请求在通过 internet 传输的过程中极有可能被篡改，为了确保请求未被更改，除公共接口（基础信息，行情数据）外的私有接口均必须使用您的 API Key 做签名认证，以校验参数或参数值在传输途中是否发生了更改。每一个API Key需要有适当的权限才能访问相应的接口。每个新创建的API Key都需要分配权限。权限类型分为：读取，交易，提币。在使用接口前，请查看每个接口的权限类型，并确认你的API Key有相应的权限。

一个合法的请求由以下几部分组成：

- 方法请求地址：即访问服务器地址 ex-admin.nxwise.io，比如  ex-admin.nxwise.io/api/order/orders。
- API 访问密钥（AccessKeyId）：您申请的 API Key 中的 Access Key。
- 签名值（Sign）：签名计算得出的值；计算签名的基于哈希的协议，此处使用 HmacSHA256。
- 时间戳（Timestamp）：您发出请求的时间 (UTC 时间) 
- 必选和可选参数：每个方法都有一组用于定义 API 调用的必需参数和可选参数。可以在每个方法的说明中查看这些参数及其含义。

### 创建 API Key

您可以在 [这里 ]([https://ex-admin.nxwise.io](https://ex-admin.nxwise.io/))创建 API Key。

API Key 包括以下两部分

- `Access Key` API 访问密钥即下文中的AccessKeyId
- `Secret Key` 签名认证加密所使用的密钥（仅申请时可见）



### 签名步骤

规范要计算签名的请求 因为使用 HMAC 进行签名计算时，使用不同内容计算得到的结果会完全不同。所以在进行签名计算前，请先对请求进行规范化处理。下面以查询某订单详情请求为例进行说明：

获取某个订单详情 https://ex-admin.nxwise.io/api/order/order?AccessKeyId=8be4cb2d4dd45c9818358b5941750c269af5e4a3&order_code=sellmm2019100818260784463715&Timestamp=1570764868

**1、按照ASCII码的顺序对参数名进行升序排序。**

**例如，按url后参数名的原始顺序提取参数名和参数值拼接的字符串如下：**

AccessKeyId=8be4cb2d4dd45c9818358b5941750c269af5e4a3

order_code=sellmm2019100818260784463715

Timestamp=1570764868

**2、按参数名升序后为:**

AccessKeyId=8be4cb2d4dd45c9818358b5941750c269af5e4a3

Timestamp=1570764868

order_code=sellmm2019100818260784463715

**3、转为小写后为：**

accesskeyid=8be4cb2d4dd45c9818358b5941750c269af5e4a3

timestamp=1570764868

order_code=sellmm2019100818260784463715

**4、用上一步里生成的 “字符串” 和你的密钥 (Secret Key) 生成一个数字签名**

以js语言为例：

```javascript
import CryptoJS from 'crypto-js';
var sign = CryptoJS.HmacSHA256("string", "Secret Key");
//签名结果为：4201d73f8b3a1ddb6e2cc63025a1353888af7aa37120aefab97b94e8b65fec42
```

**5、最后请求的url 为**

https://ex-admin.nxwise.io/api/order/order?AccessKeyId=8be4cb2d4dd45c9818358b5941750c269af5e4a3&order_code=sellmm2019100818260784463715&Timestamp=1570764868&Sign=4201d73f8b3a1ddb6e2cc63025a1353888af7aa37120aefab97b94e8b65fec42



### 错误信息

注：当借口返回字段**success**的值为**false**时，会返回错误信息**error_code**

| **错误码**                  | 描述                              |
| --------------------------- | --------------------------------- |
| invalid-type                | 类型错误                          |
| error-parameter             | 参数错误                          |
| parameter-limit-error       | limit 参数最大值为100             |
| order-not-found             | 无订单记录                        |
| no-permissions-error        | 无权限操作                        |
| orderId-not-empty           | 订单号不能为空                    |
| user-fails-to-transact      | 用户状态异常                      |
| identity-not-been-examined  | 身份验证未审核，无法交易          |
| market-error                | 市场错误                          |
| price-cannot-be-short       | 价格参数错误                      |
| num-cannot-be-empty         | 数量参数错误                      |
| type-cannot-be-empty        | type 不能为空                     |
| parameter-page-error        | page 参数错误或缺失               |
| orderId-empty-error         | 订单id不能为空                    |
| sign-verification-failed    | 签名验证失败                      |
| parameter-timestamp-invalid | 时间戳参数错误                    |
| bad-argument                | 签名缺失                          |
| illegal-ip-access           | 非法id                            |
| api-key-expired             | ApiKey已过期                      |
| invalid-accessKeyId         | 非法的ApiKey(accessKeyId即ApiKey) |
|                             |                                   |
|                             |                                   |
|                             |                                   |
|                             |                                   |



### 获取K线

请求：GET `/api/viewt_history`

参数：

| 字段       | 类型   | 必须 | 说明                                    |
| ---------- | ------ | ---- | --------------------------------------- |
| market     | string | 是   | 市场名字 eth_usdt,btc_usdt....等        |
| default    | bool   | 是   | true 表示默认第一次，false 获取历史记录 |
| resolution | string | 是   | 区间 1min 5 min 15min                   |
| addtime    | date   | 是   | 开始时间（格式 ：2019-10-14 11:47:45）  |
| endtime    | date   | 否   | 结束时间 没有就发空 `''`                |

响应如下：

```json
 {
              "success": true,
              "status_code": 1,
              "message": "获取成功",
              "data": {
                  "noData": {
                      "noData": false
                  },
                  "data": [
                      {
                                      "id": 15774,
                                      "market": "ethusdt", // 市场
                                      "type": "1min",       //时期 
                                      "open": "172.130000",
                                      "close": "162.160000",  //收盘价格
                                      "low": "159.210000",    //最低价格
                                      "high": "175.590000",
                                      "count": "613088.000000",
                                      "amount": "1543904.162010",
                                      "vol": "260062239.631320",
                                      "time": 1555776000
                                  },
                     .......
                  ]
              }
          }
```

无数时的返回：

```json
{
                              "success": true,
                              "status_code": 1,
                              "message": "没有多余的数据",
                              "data": {
                                  "noData": {
                                      "noData": true
                                  },
                                  "data": []
                              }
                          }
```





### 获取盘口数据



请求：GET `/api/handicap`

请求参数：

| 字段   | 类型   | 说明                          |
| ------ | ------ | ----------------------------- |
| market | string | 市场 可选择的值 eth_usdt,.... |

返回示例：

```json
  "bid":[                               //买盘
   {"id":1,"price":1,"quantity":1},
   {"id":2,"price":1.5,"quantity":1}
    
   ],
   "ask":[                               //卖盘
   {"id":1,price":11,"quantity":8},
   {"id":2",price":10,"quantity":40},
   {"id":3","price":9,"quantity":24},
   {"id":4","price":7,"quantity":3},
   {"id":5","price":4,"quantity":8},
   {"id":6","price":3,"quantity":8},
   {"id":7","price":2,"quantity":5}]
   }
```





### 深度图

请求方法GET：`/api/depth` 

请求参数：

| 字段         | 类型   | 说明                          |
| ------------ | ------ | ----------------------------- |
| currencyPair | string | 市场 可选择的值 eth_usdt,.... |
| depth        | int    | 获取总条数                    |
|              |        |                               |

返回示例：



```json
{
    "bids": [
        [
            11996,  //价格
            "1"     //数据量
        ],
        [
            11995,
            "2"
        ],
        [
            11994,
            "44"
        ],
        [
            11993,
            "192"
        ]
    ],
    "asks": [
        [
            12009,
            "1125.5"
        ],
        [
            12008,
            "914.5"
        ],
        [
            12007,
            "698.5"
        ],
        [
            12006,
            "539.5"
        ],
        [
            12005,
            "388.5"
        ],
        [
            12004,
            "252.5"
        ],
        [
            12003,
            "119.5"
        ],
        [
            12002,
            "33.5"
        ],
        [
            12001,
            "0"
        ]
    ]
}
```





### 获取历史成交

请求：GET  `/api/historyComplete`

参数：

| 字段   | 类型   | 是否必须 | 描述 | 取值范围             |
| ------ | ------ | -------- | ---- | -------------------- |
| market | string | 是       |      | eth_usdt,btc_usdt... |

默认获取最近成交100条

响应数据：

| 字段     | 类型   | 是否必须 | 描述     | 取值范围       |
| -------- | ------ | -------- | -------- | -------------- |
| price    | string | 是       | 成交价格 |                |
| num      | string | 是       | 成交数量 |                |
| type     | int    | 是       | 成交类型 | 1是买入 2 卖出 |
| add_time | int    | 是       | 成交时间 |                |



响应示例：

```json
{
    "success": true,
    "message": "获取成功",
    "data": [
        {
            "price": "10005.31",  
            "num": "1.490200",     
            "type": 1,   //1 买  2卖
            "add_time": 1569467195
        },
        {
            "price": "10005.31",
            "num": "1.490200",
            "type": 2,
            "add_time": 1569467195
        },
        ]
}
```













### 获取账户信息

请求：POST  `/api/accounts/balance`  

参数

| 字段        | 类型   | 是否必须 | 描述                | 取值范围 |
| ----------- | ------ | -------- | ------------------- | -------- |
| AccessKeyId | string | 是       |                     |          |
| Timestamp   | int    | 是       | 时间戳 （UTC 时区） |          |
| Sign        | string | 是       | 签名                |          |



请求响应示例：

```json
{
    "success": true,
    "user_id":2314                 //用户id
    "data": [
        {
            "type": "trade",
            "currency": "eth",
            "balance": "50213.308800", //可用
            "frozen": "213.308800"   //冻结
        },
        {
            "type": "trade",
            "currency": "btc",
            "balance": "100007.105532",
            "frozen": "213.308800"
        },
        {
            "type": "trade",
            "currency": "usdt",
            "balance": "10000045.846466",
            "frozen": "213.308800"
        },
        {
            "type": "trade",
            "currency": "ltc",
            "balance": "3.111444",
            "frozen": "213.308800"
        },
        {
            "type": "trade",
            "currency": "jrb",
            "balance": "0.00000000",
            "frozen": "213.308800"
        }
    ]
}
```



### 委托列表



请求：POST  `/api/order/orders`

参数：

| 字段        | 类型   | 是不必须 | 描述     | 取值范围                     |
| ----------- | ------ | -------- | -------- | ---------------------------- |
| AccessKeyId | string | 是       |          |                              |
| market      | string | 是       | 市场     | eth_usdt,ltc_btc...等        |
| type        | int    | 否       | 买卖类型 | 1 买入 2是卖出               |
| status      | int    | 是       | 委托状态 | 状态 0进行中 1已成交 2已撤销 |
| page        | int    | 是       | 分页     | 1,2,3.....                   |
| Timestamp   | int    | 是       | 时间戳   |                              |
| Sign        | string | 是       | 签名     |                              |



响应数据：

| id         | Id                              |
| ---------- | ------------------------------- |
| userid     | Uid                             |
| market     | 市场名字即货币对                |
| order_code | 订单编号                        |
| price      | 价格                            |
| num        | 交易数量                        |
| total      | 交易总额                        |
| deal       | 已成交量                        |
| deal_total | 已成交总额                      |
| match_type | 交易类型 limit 限价 market 市价 |
| type       | 交易方向 1买入 2 卖出           |
| add_time   | 添加时间                        |
| end_time   | 修改时间                        |
| status     | 状态 0进行中 1已成交 2已撤销    |
|            |                                 |



响应示例

```json
{
    "success": true,
    "message": "获取成功",
    "data": {
        "page": "1",
        "count": 7,
        "data": [
            {
                "id": 17,
                "userid": 2025,
                "market": "eth_usdt",
                "order_code": "buykg2019092516235650219718",
                "price": "201.450000",   //价格
                "avg_price":"201.44000",  //成交均价
                "num": "1.000000",    //数量
                "total": "201.450000", //总额
                "deal": "1.000000",    //成交量
                "deal_total": "201.450000", //成交额
                "match_type": "limit",  //交易类型  limit 限价  market 市价
                "type": 1,             // 1 买 2 卖
                "add_time": 1569399836,
                "end_time": null,
                "status": 1     //0 进行中 1已成交 2已撤销
            },
            {
                "id": 22116,
                "userid": 2025,
                "market": "jrb_usdt",
                "order_code": "sellmm2019100818260784463715",
                "price": "15.000000",
                "num": "2.000000",
                "total": "30.000000",
                "deal": "0.000000",
                "deal_total": "0.000000",
                "match_type": "limit",
                "type": 2,
                "add_time": 1570530367,
                "end_time": null,
                "status": 0
            },
        ]
    }
}
```





### 获取订单详情

请求 GET  `/api/order/order`

参数：

| 字段        | 类型   | 必须 | 说明       | 取值范围 |
| ----------- | ------ | ---- | ---------- | -------- |
| AccessKeyId | string | 是   |            |          |
| order_code  | string | 是   | 订单编号， |          |
| Timestamp   | int    | 是   | 时间戳     |          |
| Sign        | string | 是   | 签名       |          |

返回：

```json
{
    "success": true,
    "message": "获取成功",
    "data": {
        "id": 1,
        "userid": 2025,
        "market": "eth_usdt",
        "order_code": "sellss2019092516050258656622",   //订单号
        "price": "201.440000",   //价格
        "avg_price":"201.44000",  //成交均价
        "num": "2.000000",       //数量
        "total": "402.880000",   //总额
        "deal": "2.000000",      //成交量
        "deal_total": "402.900000", //成交额
        "match_type": "limit",      //交易类型  limit 限价  market 市价
        "type": 2,                  // 1 买 2 卖
        "add_time": 1569398702,
        "end_time": null,
        "status": 1                //0 进行中 1已成交 2已撤销
    }
}

```



### 下单

请求：POST `/api/order/placePro`

参数：

| 字段        | 类型   | 必须 | 说明                                                       | 取值范围                                                     |
| ----------- | ------ | ---- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| AccessKeyId | string | 是   |                                                            |                                                              |
| market      | string | 是   | 市场名字，                                                 | 可选择的值可选择的值`eth_usdt`,`btc_usdt`.....               |
| matchType   | string | 是   | 交易类型，                                                 | 值为 `limit `（限价）或`market`（市价）`sp_sl`(止盈止损)。注：止盈止损为预留接口，暂无需处理 |
| type        | int    | 是   | 交易方向，                                                 | 可选择的值：`1` 买入 `2` 卖出                                |
| num         | double | 否   | 买入卖出数量，                                             | **情形一、matchType的值为`limit`时必填，情形二、matchType的值为`market`且type值为`2`时必填** |
| price       | string | 否   | **当matchType值为`limit`时  必填**                         |                                                              |
| total       | string | 否   | **交易总额。当matchType的值为`market`且type值为`1`时必填** |                                                              |
| Timestamp   | int    | 是   | 时间戳                                                     |                                                              |
| Sign        | string | 是   | 签名                                                       |                                                              |

 响应：

```json
{
    "success": true,
    "message": "submit successfully",
    "data": {
        "id": 216,
        "order_code": "buyuy2019102806375016696980"
    }
}

```



### 撤销订单 

请求：POST `/api/order/cancel`

参数：

| 字段        | 类型   | 必须 | 说明     |
| ----------- | ------ | ---- | -------- |
| AccessKeyId | string | 是   |          |
| order_code  | string | 是   | 订单编码 |
| Timestamp   | int    | 是   | 时间戳   |
| Sign        | string | 是   | 签名     |

响应：

```json
 {
           "success": true,
           "message": "撤销成功",         
 }

```









### 成交明细

请求：POST `/api/order/matchresults`

参数：

| 字段        | 类型   | 必须 | 说明                                                   |
| ----------- | ------ | ---- | ------------------------------------------------------ |
| AccessKeyId | string | 是   | token                                                  |
| page        | int    | 是   | 页码                                                   |
| addtime     | date   | 否   | 开始时间                                               |
| endtime     | date   | 否   | 结束时间                                               |
| market      | string | 否   | 要是没有选择市场的的话就会查询所以市场                 |
| type        | int    | 否   | 1：买入 2：卖出 要是没有选择类型的话就是会包括两种类型 |
| limit       | int    | 是   | 查询条数 可以选择的值 1--100                           |
| Timestamp   | int    | 是   | 时间戳                                                 |
| Sign        | string | 是   | 签名                                                   |

响应说明：

| 名称       | 数据类型 | 是否必须 | 描述                            | 取值范围 |
| ---------- | -------- | -------- | ------------------------------- | -------- |
| page       | int      | 是       | 页码                            |          |
| count      | int      | 是       | 总条数                          |          |
| id         | int      | 是       | 系统订单ID                      |          |
| market     | string   | 是       | 市场                            |          |
| order_code | string   | 是       | 订单编号                        |          |
| user_id    | int      | 是       | 用户id                          |          |
| fee_buy    | string   | 是       | 买入手续费                      |          |
| fee_sell   | string   | 是       | 卖出手续费                      |          |
| price      | string   | 是       | 成交价格                        |          |
| num        | string   | 是       | 成交数量                        |          |
| total      | string   | 是       | 成交总额                        |          |
| type       | int      | 是       | 1是买 2是卖出                   |          |
| match_type | strin    | 是       | 交易类型 limit 限价 market 市价 |          |
| add_time   | string   | 是       | 成交时间                        |          |
| status     | int      | 是       | 1表示已经成交                   | 1        |

响应：

```json
 {
       "success": true,
       "message": "获取成功",
       "data": {
           "page": 1,     
           "count": 44,
           "data": [
                {
                "id": 28,
                "market": "eth_usdt",
                "order_code": "buylz2019092517274766303102",
                "user_id": 2025,
                "fee_buy": "0.000000",
                "fee_sell": "0.000000",
                "price": "200.12",
                "avg_price": "200.12",
                "num": "2.000000",
                "total": "400.240000",
                "type": 1,
                "match_type": "limit",
                "add_time": 1569403667,
                "status": 1
            },
           
           ]
       }
   }

```

 

## webSocket推送

### wesocket 链接

  ws://ex-admin.nxwise.io:7272
   或者
   wss://ex-admin.nxwise.io:7272

### 登录(绑定用户)

请求：

```json
{
  "type":"login",
  "uid":"2025",
  "otcToken":"p9dnbJTY#SfJ@TyDOTc",
  "password":"yEQfTMX9xqt66zZdWOtc"
}
    

```

响应：

```json
  正确返回：
    {
        "type":"login",
        "msg":"登录成功"}
  错误返回：
   {
       "err":"Error in username or password"
   }   

```

  

### 发送心跳

请求：

```json
{"type":"ping"}

```

响应：

```json
{"type":"ping"}

```

### 当前委托推送

说明: 订单在提交后的时候,服务器主动推送撮合结果给客户端

响应：

```json
{
 "uid":2025,
 "market":"eth_usdt",  //市场
 "price":0.1,  //价格
 "order_code":"buydc2019012416533653203695",  //(发生交易的的时候修改需要用到它)
 "num":0.1,   //数量
 "total":0.01, //成交额
 "fee":0,
 "type":"newentrust",   //类型=newentrust  是我当前委托
 "type_trade":   1      , //订单方向 1：买单 2:卖单
 "deal":0,              //成交量
 "add_time":1548320016,
 "status":0
 }

```

### 委托更新推送

未成交的委托单在部分成交时会全部成交时服务器会主动推送本条消息

响应：(主动推送)

```json
{
"order_id":"buyhl2019030114300536512938",   //通过订单号去修改对于的当前委托数量
"quantity":1,                               //需要扣除发布的数量  （也就是已成交量 要累加）
"sellout":0,                                //是否已售完 1.已售完 0.未售完
"market":"eth_usdt",                        //对应市场
"uid":2025,                                 //只有该用户才收到该条信息
"type":"upentrust"                          //固定的类型 upentrust  代表需更新委托信息 
}

```

### 盘口数据推送

  说明：盘口数据更新是服务器会主动推送

   响应：json（主动推送）

```json
{
   "market": "ethusdt",  //市场    eth_usdt 变成 ethusdt
   "type": "handicap",     //类型 类型等于handicap就是盘口数据
   "bid":[                               //买入
   {"id":1,"price":1,"quantity":1,"total":1},
   {"id":2,"price":1.5,"quantity":1,"total":1}
   
   ],
   "ask":[                               //卖出
   {"id":1,"price":11,"quantity":8,"total":88},
   {"id":2,"price":10,"quantity":40,"total":400},
   {"id":3,"price":9,"quantity":24,"total":216},
   {"id":4,"price":7,"quantity":3,"total":21},
   {"id":5,"price":4,"quantity":8,"total":32},
   {"id":6,"price":3,"quantity":8,"total":24},
   {"id":7,"price":2,"quantity":5,"total":10}]
   }

```

### 实时成交推送(单条推送)

当有新的成交数据时服务器主动推送

```json
{
"id":"10",  //id
"type":"tradelog",  //实时成交类
"market":"ethusdt", //市场  不在有下划线 eth_usdt 变成 ethusdt
"trade_type":1,    //交易类型 1是买入 2是卖出
"price":"3",       //成交价格
"num":1,           //成交数量
"time":1548313548  //成交时间
}

```

### 

