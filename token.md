# JWT token 获取
业务方通过HTTP请求完成AI服务的调用，使用 JWT 进行鉴权。业务方使用 AK,SK 等信息构建请求，向服务器申请签发 JWT。

# 申请签发JWT token
业务方申请服务后，将得到 AK,SK，可用模型名等信息。

如：
```
AK: 24CvJwHsEFg8pTXfkHf4xG5Y
SK: 09xrudCm4oM+ntTbcoBXQxCVbz1r7ERG
Models: change-face,id-seg （可使用两个模型,为空可以调用AK下所有模型）
```

HTTP请求：
```
POST https://ai-serving.camera360.com/v1/token
Content-Type: application/x-www-form-urlencoded
```

参数:

|参数名|说明|
| -- | -- | 
|token	|用于 JWT 签发的token字符串，计算方式见下|

## 请求token计算方式
请求token是一个以冒号（:）分隔的字符串值，其基本形式为：

> sig:AK:时间戳:JWT可用时长:JWT可用模型标识

其中：

- sig 为信息摘要，用于校验请求数据是否被修改
- 时间戳为当前时间的秒级时间戳，若请求时间戳与服务器时间相差过大将拒绝签发JWT
- JWT可用时长标识签发的JWT可以在多少秒内使用(目前最长支持3天)
- JWT可用模型标识限定了该JWT允许调用的模型，多个模型标识用逗号分隔

### 拼接基本信息
将 AK，当前时间戳，JWT 可用时长（单位为秒），JWT 可调用的模型进行字符串拼接，以冒号连接多个值，得到待签名数据，如：

```
info = 24CvJwHsEFg8pTXfkHf1xG5Y:1623911084:7200:change-face
```

### 签名计算
使用 sha256 算法计算基本信息的摘要作为请求签名，其中 SK 作为 sha256 计算的 key。
```
hash_hmac("sha256", info, SK) // 输出16进制字符串
```
示例中的内容计算出的值为：
> ce66627882f21f6b910af947804d0998f9b17c260a71f67d3b3865539afb6596

### 完整的请求 token
连接签名字符串和待签名的数据，以得到完整的请求 token

>ce66627882f21f6b910af947804d0998f9b17c260a71f67d3b3865539afb6596:24CvJwHsEFg8pTXfkHf1xG5Y:1623911084:7200:change-face

###  响应
```
{
"data": {
"token": "jwt token"
    },
"status": 0, // 0-为成功 其他为失败
"message": "ok"
}
```

# 附录
**token 计算代码示例**

PHP
```php
const AK = '';
const SK = '';
const MODELS = ''; // 多个模型标识使用","分隔

function generateRequestToken($ak, $sk, $models, $expire) {
    $info = implode(':', $ak, time(), $expire, $models);
    $sig = hash_hmac('sha256', $info, $sk);
    return $info . ':' . $sig;
}
// 调用
generateRequestToken(AK, SK, MODELS, 7200);
```

Go
```go
func generateRequestToken(ak, sk, models string, expire int) (string, error) {
    info := fmt.Sprintf("%s:%d:%d:%s", ak, time.Now().Unix(), expire, models)
    
    sha := hmac.New(sha256.New, []byte(sk))
    if _, err := sha.Write(info); err != nil {
        return "", err
    }
    
    sig := sha.Sum(nil)
    
    return hex.EncodeToString(sig) + ":" + info, nil
}
```
