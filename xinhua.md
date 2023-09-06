# 新华社图片处理
**注：以下所有接口请求均需要在Header中设置Authorization，其值可通过[JWT Token接口](123)获得**

![时序图](xinhua001.png)

## 获取七牛文件上传Token
GET /model/xinhua/sync/open/v1/upload/token

header
```
"Authorization":"Bearer 获取的jwttoken"
```

**请求参数**

无

**响应结果**
```json
{
    "token":"IAM-Ej1TW2KrVqRIOkIqZVEPta_yLmnUc8-1LySvtWCb:rExyb9Nd4uxcKfi3XrsiLTHsghc=:eyJzY29wZSI6InFhLWMzNjA6YWlnYy8xMjMzMzEyMyIsImRlYWRsaW5lIjoxNjgwMDgwNjQwLCJyZXR1cm5Cb2R5Ijoie1wibWltZVR5cGVcIjokKG1pbWVUeXBlKSxcIndpZHRoXCI6JChpbWFnZUluZm8ud2lkdGgpLFwiaGVpZ2h0XCI6JChpbWFnZUluZm8uaGVpZ2h0KSxcImtleVwiOlwiJChrZXkpXCIsXCJoYXNoXCI6XCIkKGV0YWcpXCIsXCJzaXplXCI6JChmc2l6ZSksXCJidWNrZXRcIjpcIiQoYnVja2V0KVwiLFwibmFtZVwiOlwiJCh4Om5hbWUpXCJ9IiwiZm9yY2VTYXZlS2V5Ijp0cnVlLCJzYXZlS2V5IjoiYWlnYy8xMjMzMzEyMyJ9",
    "baseURL":"https://cdn-qa-all.c360dn.com",
}
```
|字段|类型|描述|
|-|-|-|
|token|string|qiniu文件上传token|
|baseURL|string|图片访问地址=baseURL/图片key|



## 创建图片处理任务
POST /model/xinhua/sync/open/v1/image

header
```
"Authorization":"Bearer 获取的jwttoken"
"Content-Type":"application/json"
```


**请求参数 body json**
```json
{
    "imgKeys":["img1.jpg","img2.jpg"],
    "async":true
}
```

|字段|类型|描述|
|-|-|-|
|imgKeys|stringArray|图片集合（七牛的图片key）|
|async|bool|是否异步处理 不传=false:同步处理、true:异步处理(异步方式可通过下方接口获取任务进度)|


**响应结果 body json**
```json
{
    "id":"d5057770794b317f42aef0f872cfcbd5",
    "count":2,
    "processing":0,
    "items":[
        {
            "id":"5fb00053cfd3e3e234873628ed19c8bf",
            "imgKey":"img1.jpg",
            "rimgKey":"rimg1.jpg",
            "status":"failed",
            "msg":[
                "阴阳脸",
                "照片水印/文字"
            ]
        },
         {
            "id":"4ac0b482ff75ea2dcbbfd73c1d7f2999",
            "imgKey":"img1.jpg",
            "rimgKey":"rimg1.jpg",
            "status":"failed",
            "msg":[
                "阴阳脸",
                "照片水印/文字"
            ]
        }
    ]
}
```

|字段|类型|描述|
|-|-|-|
|id|string|处理任务的批次ID|
|count|int|当前批次处理的图片总量|
|processing|int|当前批次处理中的图片数量|
|**items**|objectArray|每张图片处理结果集|
|id|string|图片处理任务的唯一标识ID（可通过此ID获取不带水印的结果图）|
|imgKey|string|原图（七牛的图片key）|
|rimgKey|string|结果图（七牛的图片key）|
|status|enumeration|Processing:处理中、Succeed:成功、Error:未知错误、 DetectFailed:检测不通过、DetectSuspected:检测存疑|
|msg|stringArray|处理消息集合（与status关联的消息）|


## 获取异步任务处理结果

POST /model/xinhua/sync/open/v1/image/task/{id:string}

header
```
"Authorization":"Bearer 获取的jwttoken"
```

**请求参数 路径参数**

|字段|类型|描述|
|-|-|-|
|id|string|处理任务的批次ID|


**响应结果 body json**

*与创建任务的返回结果一致*


## 获取不带水印的结果图

POST /model/xinhua/sync/open/v1/image/unwatermark/{id:string}


header
```
"Authorization":"Bearer 获取的jwttoken"
```

**请求参数 路径参数**

|字段|类型|描述|
|-|-|-|
|id|string|图片处理任务的唯一标识ID|



**响应结果 body json**

```json
{
    "imgKey":"rimg1.jpg"
}
```
