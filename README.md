# 蜜源风控开放接口文档

---


## 1.对接准备
 > * step1 向机蜜申请接口调用凭证信息(appId、secret、api_domain)；
 > * step2 向机蜜申请获取调试环境；（注意：测试环境为http协议，而生产环境为https协议）
 > * step3 根据接口文档开发对接接口；
 > * step4 提供服务器IP给机蜜配置白名单；
 > * step5 发布上线。

## 2.调用约定

### 2.1 请求
 > * 接口协议：http
 > * 请求方式：post
 > * 消息格式：application/json
 > * 消息编码：UTF-8
 
请求参数样例：
```
{
    "id":"517080601011",
    "name":"xx"
}
```

### 2.2 响应
 > * 消息格式：application/json
 > * 消息编码：UTF-8

成功响应参数结构：
```
{
    "code":"200",
    "data":{
        "age":"20"
    }
}
```
失败响应参数结构：
```
{
    "code":"501",
    "message":"错误消息详情"
}
```
 
### 2.3 签名算法
为确保接口访问安全，接口的请求和响应应对所有参数(header+body)使用对称加密算法做签名校验。请求和响应均应在header中加入签名参数。
    
Header参数：

| 名称    | 含义   |  类型  | 是否必填 | 备注            |
| :----   | :----  | :----  | :--      | :-------------  |
| appId |    应用唯一标识    |  varchar(15)  | Y | - |
| timestamp |    时间戳    |  varchar(15)  | Y | 时区GMT+8以秒为单位的时间戳 |
| sign    |    签名    |  varchar(15)  | Y | - |

Body参数：

指http请求的消息体。

签名算法：
 > * step1：把所有参数（包括appId、secret、timestamp、body）的key和值拼成字符串放入到数组，得到 array = ['key2=value2','key1=value1']
 > * step2：把数组按照ascii码进行升序排序，得到 array = ['key1=value1','key2=value2']
 > * step3：把数组的元素用&拼成一个字符串，得到 source = 'key1=value1&key2=value2'
 > * step4：根据step3得到的source生成MD5加密值，并转成大写，生成签名。sign=toUpperCase(Md5(source))
 
 
### 2.4 其它
 > * 所有请求和响应参数字段均为字符串格式
 > * 所有请求和响应金额参数的单位均为分

## 3. 接口列表

### 3.1 刷新密钥接口

为避免密钥泄露导致安全事故，对接方应定期调用此接口更新密钥。

接口地址：${api_domain}/api/isv/secret/refresh/v1

请求参数：无

请求示例：
```
{
}
```
响应参数：

| 名称    | 含义   |  类型  | 是否必填 | 备注            |
| :----   | :----  | :----  | :--      | :-------------  |
| secret| 新的密钥 |   varchar(32) | Y | - |
| upTime| 生成时间 |   varchar(19) | Y | - |
| expireTime| 到期时间 |   varchar(19) | Y | - |

响应示例：
```javascript
{
    "code":"200",
    "data":{
        "secret":"AAAAAAAAAAAAAAAAAA"
        "upTime":"2020-01-01 11:11:11"
        "expireTime":"2020-01-02 11:11:11"
    }
}
```

###  3.2 确认新密钥接口

刷新密钥后，对接方确认收到新密钥后，应调用此接口确认更新。

接口地址：${api_domain}/api/isv/secret/confirm/v1

请求参数：无

请求示例：
```javascript
{
}
```

响应参数：无

响应示例：
```
{
    "code": "200",
    "data": "成功"
}
```



###  3.3获取风控数据

对接方通过此接口获取用户的风险指标数据。

接口地址：${api_domain}/api/risk/data/get/v1

请求参数：

| 名称    | 含义   |  类型  | 是否必填 | 备注            |
| :----   | :----  | :----  | :--      | :-------------  |
| requestId | 请求标识 | varchar(32) | Y | 用来标识请求唯一，生成方式对接方自定义，不重复即可 |
| idcardNum | 用户身份证号 | varchar(20) | N | md5(身份证号码) |
| idcardName | 用户身份证姓名 | varchar(20) | N | md5(身份证姓名)，当身份证号不为空时，姓名也不能为空 |
| phone | 用户手机号 | varchar(20) | N | md5(手机号) |
| creLat | 下单地址纬度 | varchar(20) | N | - |
| creLng | 下单地址经度 | varchar(20) | N | - |
| recLat | 收货地址纬度 | varchar(20) | N | - |
| recLng | 收货地址经度 | varchar(20) | N | - |
| mac | 用户设备物理地址 | varchar(30) | N | md5(mac) |
| wifimac | 用户WiFi设备的物理地址 | varchar(30) | N | md5(wifimac) |
| extend | 扩展参数 | varchar(1000) | N | 扩展数据 |

请求示例：
```javascript
{
    "requestId":"10001",
    "idcardName":"AAAAAAAAAAAAAAA",
    "idcardNum":"AAAAAAAAAAAAAAA",
    "phone":"AAAAAAAAAAAAAAA",
    "creLat":"39.916527",
    "creLng":"116.397128",
    "recLat":"39.916527",
    "recLng":"116.397128",
    "mac":"AB:E2:D3",
    "wifimac":"AB:E2:D3"
}
```

响应参数：

| 名称    | 含义   |  类型  | 是否必填 | 备注            |
| :----   | :----  | :----  | :--      | :-------------  |
| id   | 风险指标标识 | varchar(32) | Y | - |
| name| 风险指标名称 |   varchar(50) | Y | - |
| value| 风险指标值 |   varchar(50) | Y | - |

响应示例：
```
{
    "code":"200",
    "data":[
        {
            "id":"my_blocklist",
            "name":"是否命中黑名单",
            "value":"是"
        },
        {
            "id":"my_risklist",
            "name":"是否命中灰名单",
            "value":"是"
        },
        {
            "id":"my_week_order_amount",
            "name":"7天内下单总金额",
            "value":"100000"
        },
        {
            "id":"my_month_order_amount",
            "name":"30天内下单总金额",
            "value":"200000"
        }
    ]
}
```
指标说明表：
| id    | 名称   |  值  | 备注            |
| :---- | :---- |:---  | :-------------  |
| my_blocklist   | 是否命中黑名单 | 是/否 |  - |
| my_risklist| 是否命中灰名单 |  是/否  | - |
| my_week_order_amount | 7天内下单总金额（分） | 整数 | - |
| my_month_order_amount | 30天内下单总金额（分） | 整数 | - |


## 4.错误代码

| 代码编码    | 代码含义   |
| :----  | :-------------  |
| 401    | 访问权限不足或签名异常 |
| 415    | 参数验证异常 |
| 500    | 未知系统异常 |

