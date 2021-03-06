## 功能描述

QueryBmNatGatewayProductionStatus 用于查询操作黑石NAT网关任务的执行状态

接口请求域名：bmvpc.api.qcloud.com

## 请求

语法示例：
```
GET https://bmvpc.api.qcloud.com/v2/index.php/?Action=QueryBmNatGatewayProductionStatus
    &<公共请求参数>
    &taskId=<任务ID>
```
### 请求参数
以下请求参数列表仅列出了接口请求参数，正式调用时需要加上公共请求参数，见<a href="/doc/api/372/4153" title="公共请求参数">公共请求参数</a>页面。其中，此接口的Action字段为QueryBmNatGatewayProductionStatus。

| 参数名称 | 必选  | 类型 | 描述 |
|---------|---------|---------|---------|
| taskId | 是 | string | 任务ID, 可使用该ID查询任务执行结果|

## 响应
响应示例：
```
{
    "code": "0",
    "message": "",
    "data": {
        "status": 1
    }
}
```

### 响应参数

| 参数名称 | 类型 | 描述 |
|---------|---------|---------|
| code | int | 错误码。0: 成功, 其他值: 失败|
| message | string | 错误信息|
| data.status | int | 0为执行成功，1为执行失败，2为正在执行中 |

## 实际案例

### 请求
```
GET https://bmvpc.api.qcloud.com/v2/index.php?	
	Action=QueryBmNatGatewayProductionStatus
	&SecretId=AKID1ub7R1JoyBF7nHqjk7IH8nGWaR6Yezwd
	&Nonce=56046
	&Timestamp=1507703517
	&Region=gz
	&taskId=3000
	&Signature=QKEu1BoztPhLn%2B1MRThSbNogd1s%3D
```

### 响应

```
{
    "code": "0",
    "message": "",
    "data": {
        "status": 1
    }
}
```