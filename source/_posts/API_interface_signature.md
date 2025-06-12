---
title: API 接口签名验证机制
date: 2025-5-24 08:35:14
updated: 2025-5-27 11:20:30
description: 接口签名验证机制详解（含 JS + PHP 示例）
---


## ✨ 一、什么是 API 接口签名验证？

API 签名是一种保障接口安全性的机制。其核心目的是：

* 防篡改**确保请求内容未被第三方修改**
* 防伪造**验证请求来源合法性**
* 时效性**防止历史请求重复提交**
* 完整性**保证传输过程数据完整**


签名的基本原理是：客户端将约定参数+请求体 + 密钥做加密签名，服务器根据相同逻辑重新生成签名进行比对，从而校验请求合法性。

---

## 📐 二、签名参数设计

一般包括以下几个字段：

| 参数名           | 含义                 |
| ------------- | ------------------ |
| `X-App-Id`    | 应用标识（客户端ID）        |
| `X-Version`   | API版本号             |
| `X-Timestamp` | 请求时间戳（单位秒）         |
| `X-Nonce`     | 随机字符串（防重放）         |
| `X-Signature` | 签名值（HMAC-SHA256生成） |

请求体（body）也会作为签名的一部分参与计算。

### 签名生成步骤

1. **参数收集**: 收集所有参与签名的参数
2. **参数排序**: 按参数名进行字典序排列
3. **字符串拼接**: 按`key=value&`格式拼接
4. **HMAC签名**: 使用HMAC-SHA256算法计算签名

### 算法示例


```
原始参数：
appid=appId
version=1.0.0
timestamp=1716456789
nonce=abc123
body={"userId":123,"action":"testAction"}
secretKey=secretKey

注意：body如果为空就赋值"{}",body={}

将请求体（如 JSON）序列化为字符串，忽略undefined、function等非法类型

按参数名字母序排序（如appid→body→nonce→timestamp→version）

生成签名字符串

拼接为key1=value1&key2=value2&...格式

排序后拼接：
appid=appId&body={"userId":123,"action":"testAction"}&nonce=abc123&timestamp=1716456789&version=1.0.0

HMAC-SHA256签名：
signature = HMAC-SHA256(拼接字符串, secretKey)

signature示例: c8313008af78b064d700de24c5d15cee2ab6a14e6f506930656faa73668419f4

```

---

## 🔐 三、代码示例

### 前端加签逻辑（JavaScript）

```html
<!DOCTYPE html>
<html>
<head>
    <title>签名测试</title>
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/4.1.1/crypto-js.min.js"></script>
</head>
<body>
<button onclick="sendRequest()">发送请求</button>

<script>

    const config = {
        appId: "appId",
        secretKey: "secretKey",
        version: "1.0.0",
        apiUrl: "https://wds7788.github.io/api/v1/test/test"
    };

    function generateSignature(config, data) {
        const timestamp = Math.floor(Date.now() / 1000);
        const nonce = generateUniqueId();

        // 保持原始JSON顺序
        const bodyStr = data ? JSON.stringify(data, jsonSafeReplacer) : '';


        const signParams = {
            appid: config.appId,
            version: config.version,
            timestamp: timestamp,
            nonce: nonce,
            body: bodyStr
        };

        // 调试输出（完成后可移除）
        console.log('参与签名参数:', JSON.stringify(signParams, null, 2));

        const sortedParams = Object.keys(signParams)
            .sort()
            .map(key => `${key}=${signParams[key]}`)
            .join('&');

        // 调试输出
        console.log('签名字符串:', sortedParams);

        const signature = CryptoJS.HmacSHA256(
            sortedParams,
            config.secretKey
        ).toString(CryptoJS.enc.Hex);

        console.log('signature:', signature);

        return {
            headers: {
                'X-App-Id': config.appId,
                'X-Version': config.version,
                'X-Timestamp': timestamp,
                'X-Nonce': nonce,
                'X-Signature': signature
            },
            body: bodyStr
        };
    }

    function generateUniqueId() {
        var length = 12;
        const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
        let result = '';
        const charactersLength = characters.length;
        for (let i = 0; i < length; i++) {
            result += characters.charAt(Math.floor(Math.random() * charactersLength));
        }
        return result;
    }

    function jsonSafeReplacer(key, value) {
        if (typeof value === 'bigint') {
            return value.toString(); // BigInt 转为字符串
        }
        if (typeof value === 'symbol' || typeof value === 'function' || typeof value === 'undefined') {
            return undefined; // 忽略 Symbol、Function、undefined
        }
        return value;
    }

    async function sendRequest() {
        try {

            const requestData = {
                userId: 123,
                action: 'testAction'
            };

            const {headers, body} = generateSignature(config, requestData);

            // 调试输出
            console.log('最终请求头:', headers);
            console.log('请求体:', body);

            const response = await axios.post(config.apiUrl, body, {
                headers: {
                    ...headers,
                    'Content-Type': 'application/json'
                }
            });

            console.log('请求成功:', response.data);
        } catch (error) {
            console.log('请求失败:', error);
        }
    }

    sendRequest();
</script>
</body>
</html>
```

---

### 后端验签逻辑（PHP）

```php
<?php
/**
 * Created by PhpStorm.
 * User: wds
 * Date: 2025/5/23
 * Time: 16:33
 */

namespace utils;

class Signature
{
    private $expireTime = 600;
    private $appId      = "appId";
    private $secretKey  = "secretKey";
    private $success    = 200;
    private $error      = 400;

    //检验sign是否正确
    public function verifySign(array $header, string $body)
    {
        // 获取请求头参数
        $appid     = $header['X-App-Id'] ?? '';
        $version   = $header['X-Version'] ?? '';
        $nonce     = $header['X-Nonce'] ?? '';
        $timestamp = $header['X-Timestamp'] ?? 0;
        $signature = $header['X-Signature'] ?? '';

        // 基础验证
        if (!$appid || !$version || !$nonce || !$timestamp || !$signature) {
            return ['code' => $this->error, 'msg' => 'Missing authentication headers'];
        }
        // 应用ID有效性验证
        if ($appid !== $this->appId) {
            return ['code' => $this->error, 'msg' => 'Invalid appid'];
        }
        // 时间有效性验证
        if (time() - $timestamp > $this->expireTime) {
            return ['code' => $this->error, 'msg' => 'Request expired'];
        }

        // Nonce防重放
        // if (Cache::has('api_nonce:'.$nonce)) {
        //     throw new \Exception('Duplicate request', 401);
        // }
        //Cache::set('api_nonce:'.$nonce, 1, $this->expireTime);

        
        if (empty($body)) {
            $body = "{}";
        }


        // 生成服务端签名
        $serverSign = $this->generateSign([
            'appid'     => $appid,
            'version'   => $version,
            'timestamp' => $timestamp,
            'nonce'     => $nonce,
            'body'      => $body // 请求体
        ]);

        // 签名比对
        if (!hash_equals($serverSign, $signature)) {
            return ['code' => $this->error, 'msg' => 'Invalid signature'];
        }
        return ['code' => $this->success, 'msg' => 'success'];
    }

    private function generateSign(array $params): string
    {
        // 参数排序
        ksort($params);

        // 拼接字符串
        $signStr = '';
        foreach ($params as $key => $value) {
            $signStr .= "{$key}={$value}&";
        }
        $signStr = rtrim($signStr, '&');

        // 使用HMAC-SHA256
        return hash_hmac('sha256', $signStr, $this->secretKey);
    }
}
```

