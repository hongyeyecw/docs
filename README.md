
# 接口文档 1.0

## 版本
> v1.0 2018-07-11

## 验证规则

接口文档参数除key之外 将参数按照字母顺序排序。用 `'&'` 符号连接,加上商户的`secret`md5加密 

> 此接口必须为服务端调用  
> 如果数据为空的参数也作为key的密文中加密

* uid : 商户平台>api接口里面的 `uid:227590393238130****`

* secret : 商户平台>api接口里面的`token: xvi7hvszwk1b182tvjzjpezi4hx9gvmk`

body内容：
``` 
goodsname=商品名&notify_url=www.google.com&orderid=201806221558103445&orderuid=6000028&pay_type=100&price=50&return_url=www.baidu.com&uid=229638810097422336&user_ip=192.168.1.1
加密规则：
```

1. 参数排序后：  
```
 goodsname=商品名&notify_url=www.google.com&orderid=201806221558103445&orderuid=6000028&pay_type=100&price=50&return_url=www.baidu.com&uid=229638810097422336&user_ip=192.168.1.1
```

2. 加上secret  
```
 goodsname=商品名&notify_url=www.google.com&orderid=201806221558103445&orderuid=6000028&pay_type=100&price=50&return_url=www.baidu.com&uid=229638810097422336&user_ip=192.168.1.1xvi7hvszwk1b182tvjzjpezi4hx9gvmk
```

3. md5:  
```
2ad19ebf430323f0f08fe63bc21be50c
```

## 接口
### 1 商户平台请求支付
#### Request:
Post /ccpay/ach/pay

| header  | 值  | 是否必须  |
|--|--|-- |
|Content-Type | application/json | 是 |


| 参数名称  | 是否必须  | 解释  |
|--|--|-- |
|uid | Y | 商户id |
|price | Y | 商品价格（分） |
|pay_type | Y | 100：支付宝；200：微信支付 |
|notify_url | Y |  用户支付成功后，我们服务器会主动发送一个post消息到这个网址。由您自定义。不要urlencode。例：http://www.your-domian.com/paysapi_notify |
|return_url | N |  跳转的链接 |
|orderid | Y |  我们会据此判别是同一笔订单还是新订单。我们回调时，会带上这个参数。例：201710192541 |
|goodsname | N |  商品名称 |
|orderuid | Y |  商户自己的用户id |
|key | Y |  必填，验证规则参考验证规则目录 |
|user_ip | Y |  商户用户的ip |

key 验证的key

### Response:
``` json
{
    "code": "1",
    "msg": "succcess",
    "err_msg": "no msg",
    "data": {
        "result": {
            "qr_url": "http://192.168.1.27:10132/1524913744.jpg", // 待支付的二维码
            "out_order_id":"xxx",                                 // 平台生成的订单id  
            "price":"xx",                                         // 订单金额（分）
            "goodsname":"商品名称",
            "orderid":"xxx",                                      // 商户自己的订单id
            "parse_url":"xxx"                                     // 解析的url
        }
    }
}
```

注： 每个订单的平台有效期为三分钟，每次获取的二维码仅支持支付该笔订单，多支付后会造成多支付的金额无法查询。
    建议客户对用户的二维码展示时长不超过2.5分钟避免用户超时支付。



## 接口
### 2 查询订单状态
#### Request:
Post /ccpay/ach/query


| header  | 值  | 是否必须  |
|--|--|-- |
|Content-Type | application/json | 是 |


| 参数名称  | 是否必须  | 解释  |
|--|--|-- |
|uid | Y | 商户id |
|out_order_id | Y | 查询订单号 |
|key | Y |  必填，验证规则参考验证规则目录 |
key 验证的key
### Response:
``` json
{
    "code": "1",
    "msg": "succcess",
    "err_msg": "no msg",
    "data": {
        "result": {
            "out_order_id":"xxx", // 平台生成的订单id
            "status":"xx"        // 1未支付 2已经支付
        }
    }
}
```

### callback:
Post  

| header  | 值  | 是否必须  |
|--|--|-- |
|Content-Type | application/json | 是 |


| 参数名称   | 解释  |
|--|--|
| user_id| 商户自己的用户id |
| goodsname| 商品名称 |
| pay_type| 支付类型 |
| orderid| 商户自己的订单id |
| price| 商品价格 |
| out_order_id| 平台订单号 |
| key|   必填，验证规则参考验证规则目录 |
### Response:

### http status  200 成功处理

> 当 http status 为200  且 code =1  平台才认为商户成功接受此回调 其他平台会在两分钟后重发此请求，总共尝试10次 如果10次之后仍然没法通知商户，商户可调用查询订单接口查询订单状态  
> 回调可能多次, 商户订单需要做重复处理

``` json
{"code":"1","msg":"处理成功"}
```

### 回调验证
回调数据:
``` json
{
    "user_id":"",
    "goodsname":"18774005563",
    "pay_type":"200",
    "orderid":"201807060939504936122",
    "key":"daf2bc680fffac7534980a29f9b4d98b",
    "price":"30000",
    "out_order_id":"201807060939585995"
}
```

验证key是否匹配:
1.密文排序：

```
goodsname=iphone 8x&orderid=54199961&out_order_id=2018062214142356&pay_type=200&price=1000&user_id=daycool
```

2.加 secret

```
goodsname=iphone 8x&orderid=54199961&out_order_id=2018062214142356&pay_type=200&price=1000&user_id=daycoolxvi7hvszwk1b182tvjzjpezi4hx9gvmk
```

3.md5

```
09aa0422dd6c161c986804aafec4ad1d
```


### 支付结果申诉

Post /ccpay/ach/order/check

| header  | 值  | 是否必须  |
|--|--|-- |
|Content-Type | application/json | 是 |


| 参数名称   | 解释  |
|--|--|
| uid| 商户自己的用户id |
| out_order_id| 平台订单号 |
| cert_img | 已支付的凭证图片地址 |
| key|   必填，验证规则参考验证规则目录 |

### Response
``` json
{
    "code":"1",
    "msg":"succcess",
    "data":{
        "result":{"status":"1"}
    }
}
```

## 错误码

| 错误码  | 注释  |
|--|--|
|-18 | 商户key 验证失败 |
|-19 | 商户号不可用 |
|-20 | 系统没有可分配的二维码,请稍后再试 |
|-32 | 商户验证失败,请确认商户是否正常或者参数是否正确 |
|-34 | 系统繁忙,当前用户获取请求太频繁 |
|-36 | 当前用户被设置成黑名单,请联系平台。|
|-37 | 当前IP不是商户的白名单|
|-38 | 商户端系统错误|
|-39 | 商户异常|
|-44 | 重复提交|




