# 新华社图片处理
![时序图](xinhua001.png)

生产环境接口地址:https://xinhua-api.camera360.com/

## 获取JWTToken
**注：以下所有接口请求均需要在Header中设置Authorization:Bearer JWTToken**

POST /v1/token/jwt

**请求参数**
```json
{
    "token":"b49dadca020b4a502aecd9e439290b8ffe21b9af1146c32abd17875b255528a8:wolqCUc8jHaeJdmgWnqrmJ2H:1694678794:72000:pg-xhs-go"
}
```
|字段|类型|描述|
|-|-|-|
|token|string|用于 JWT 签发的token字符串，计算方式见下|

### 请求token计算方式

请求token是一个以冒号（:）分隔的字符串值，其基本形式为：

> sig:AK:时间戳:JWT可用时长:JWT可用模型标识

其中：

- sig 为信息摘要，用于校验请求数据是否被修改
- 时间戳为当前时间的秒级时间戳，若请求时间戳与服务器时间相差过大将拒绝签发JWT
- JWT可用时长标识签发的JWT可以在多少秒内使用(目前最长支持3天)
- JWT可用模型标识限定了该JWT允许调用的模型，多个模型标识用逗号分隔

#### 拼接基本信息

将 AK，当前时间戳，JWT 可用时长（单位为秒），JWT 可调用的模型进行字符串拼接，以冒号连接多个值，得到待签名数据，如：

```
info = wolqCUc8jHaeJdmgWnqrmJ2H:1694678794:72000:pg-xhs-go
```

#### 签名计算

使用 sha256 算法计算基本信息的摘要作为请求签名，其中 SK 作为 sha256 计算的 key。
```
hash_hmac("sha256", info, SK) // 输出16进制字符串
```
示例中的内容计算出的值为：
> b49dadca020b4a502aecd9e439290b8ffe21b9af1146c32abd17875b255528a8

#### 完整的请求 token

连接签名字符串和待签名的数据，以得到完整的请求 token

>b49dadca020b4a502aecd9e439290b8ffe21b9af1146c32abd17875b255528a8:wolqCUc8jHaeJdmgWnqrmJ2H:1694678794:72000:pg-xhs-go

**响应结果**

```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhayI6IndvbHFDVWM4akhhZUpkbWdXbnFybUoySCIsIm1zIjpbeyJtIjoicGcteGhzLWdvIiwicnMiOjEwLCJybSI6MH1dLCJ2dCI6IjE2OTQ1MzQ0MDAtMTg5OTA0MzIwMCIsImV4cCI6MTY5NDc1MDgxNSwiaWF0IjoxNjk0Njc4ODE1LCJpc3MiOiJwaW5ndW8uYWktcGxhdGZvcm0ifQ.6ID3yx-jAF-D-Gyujvxc8GmpAIjdOIe0g1WHwkyegp0"
}
```




具体算法参考[JWT Token接口](./token.md)，请求需替换当前



## 获取七牛文件上传Token

GET /v1/upload/token

header
```
"Authorization":"Bearer 获取的jwttoken"
```

**请求参数**

```json
{
    "keys":["test001.jpg"]
}
```

**响应结果**

```json
{
    "tokens": [
        {
            "key": "test001.jpg",
            "token": "IAM-Ej1TW2KrVqRIOkIqZVEPta_yLmnUc8-1LySvtWCb:JiXTaUGjyzBGjlRwOSY0J8s8U3M=:eyJzY29wZSI6InFhLWMzNjA6dGVzdDAwMS5qcGciLCJkZWFkbGluZSI6MTY5NDY4MDY5OCwiaW5zZXJ0T25seSI6MSwicmV0dXJuQm9keSI6IntcIm1pbWVUeXBlXCI6JChtaW1lVHlwZSksXCJ3aWR0aFwiOiQoaW1hZ2VJbmZvLndpZHRoKSxcImhlaWdodFwiOiQoaW1hZ2VJbmZvLmhlaWdodCksXCJrZXlcIjpcIiQoa2V5KVwiLFwiaGFzaFwiOlwiJChldGFnKVwiLFwic2l6ZVwiOiQoZnNpemUpLFwiYnVja2V0XCI6XCIkKGJ1Y2tldClcIixcIm5hbWVcIjpcIiQoeDpuYW1lKVwifSIsImZzaXplTGltaXQiOjEwNDg1NzYsIm1pbWVMaW1pdCI6ImltYWdlL2pwZWcifQ=="
        }
    ],
    "baseURL": "https://cdn-qa-all.c360dn.com"
}
```
|字段|类型|描述|
|-|-|-|
|token|string|qiniu文件上传token（只能上传jpg、1MB以内的的图片、30分钟有效期）|
|baseURL|string|图片访问地址=baseURL/图片key|



## 创建图片处理任务

POST /v1/image/task

header
```
"Authorization":"Bearer 获取的jwttoken"
"Content-Type":"application/json"
```


**请求参数 body json**

```json
{
    "keys":["test001.jpg"]
}
```

|字段|类型|描述|
|-|-|-|
|keys|stringArray|图片key集合（七牛的图片key）|


**响应结果 body json**

```json
{
    "data": [
        {
            "taskID": "6502c60b519e34746a3089a1",
            "key": "test001.jpg"
        }
    ]
}
```

|字段|类型|描述|
|-|-|-|
|taskID|string|图片处理的任务ID|
|key|string|图片key|


## 获取异步任务处理结果（有水印图）

POST /v1/image/watermark/{id:string}

header
```
"Authorization":"Bearer 获取的jwttoken"
```

**请求参数 路径参数**

|字段|类型|描述|
|-|-|-|
|id|string|图片处理的任务ID|


**响应结果**

```json
{
    "taskStatus": "done",
    "taskMessage": "完成",
    "image": "https://cdn-qa-all.c360dn.com/watermark/f485b7e80593435dd75aefd2163a7eef.jpg",
    "detectStatus": "pass",
    "detectMessage": []
}
```

|字段|类型|描述|
|-|-|-|
|taskStatus|enumeration|任务状态 init:初始化、running:处理中、done:处理完成、failed:处理失败|
|taskMessage|string|任务状态所对应的消息|
|image|string|图片key|
|detectStatus|enumeration|检测状态 pass:通过、failed：不通过、suspected:存疑|
|detectMessage|stringArray|检测状态所对应的纤细|


## 获取异步任务处理结果（无水印图）

> 此接口返回无水印图后会执行扣次数逻辑（同一个id只会扣一次）
POST /v1/image/unwatermark/{id:string}


header
```
"Authorization":"Bearer 获取的jwttoken"
```

**请求参数 路径参数**

|字段|类型|描述|
|-|-|-|
|id|string|图片处理任务的唯一标识ID|



**响应结果**

*与获取水印图的响应结果一致*
