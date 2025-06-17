---
title: ThinkPHP 8.1 + think-swoole 4.1 的 WebSocket 实时通信
date: 2025-6-6 23:49:18
updated: 2025-6-6 23:49:18
description: 'ThinkPHP 8.1 框架和 think-swoole 4.1 扩展，完整配置 WebSocket 服务端 + 客户端 HTML 页面 Demo'
---
---

# 🌐 使用 Swoole 开启 WebSocket 服务（含连接/消息/断开监听）

本文将通过一个完整的示例，介绍如何基于 Swoole 和 ThinkPHP 8 配置 WebSocket 服务，包含服务启动、连接事件监听、消息处理、断开处理及前端示例，适合初学者快速上手。


框架：thinkphp:8.1.1

扩展：think-swoole:4.1

---

## 📦 一、Swoole 配置文件（支持 WebSocket）

创建或修改你的 `config/swoole.php` 配置文件：

```php
<?php

return [
    'http' => [
        'enable'     => true,
        'host'       => '0.0.0.0',
        'port'       => 9501, // 修改为你的端口
        'worker_num' => swoole_cpu_num(),
        'options'    => [
            'package_max_length' => 10 * 1024 * 1024, // 10MB
            'max_request'        => 3000,
        ],
    ],

    'websocket' => [
        'enable'        => true, // 开启 WebSocket
        'route'         => false,
        'handler'       => \think\swoole\websocket\Handler::class,
        'ping_interval' => 25000, // 心跳间隔
        'ping_timeout'  => 60000, // 超时时间
        'room' => [
            'type'  => 'table',
            'table' => [
                'room_rows'   => 8192,
                'room_size'   => 2048,
                'client_rows' => 4096,
                'client_size' => 2048,
            ],
        ],

        // WebSocket 事件监听类
        'listen' => [
            'open'  => \app\webscoket\Connect::class,
            'event' => \app\webscoket\Event::class,
            'close' => \app\webscoket\Close::class,
        ],

        'subscribe' => [],
    ],

    // 其他服务配置（可按需启用）
    'rpc' => [
        'server' => [
            'enable'     => false,
            'host'       => '0.0.0.0',
            'port'       => 9000,
            'worker_num' => swoole_cpu_num(),
            'services'   => [],
        ],
        'client' => [],
    ],

    'queue' => [
        'enable'  => false,
        'workers' => [],
    ],

    'hot_update' => [
        'enable'  => env('APP_DEBUG', false),
        'name'    => ['*.php'],
        'include' => [app_path()],
        'exclude' => [],
    ],

    'pool' => [
        'db' => [
            'enable'        => true,
            'max_active'    => 3,
            'max_wait_time' => 5,
        ],
        'cache' => [
            'enable'        => true,
            'max_active'    => 3,
            'max_wait_time' => 5,
        ],
    ],

    'ipc' => [
        'type' => 'unix_socket',
    ],

    'lock' => [
        'enable' => false,
        'type'   => 'table',
    ],

    'tables'    => [],
    'concretes' => [],
    'resetters' => [],
    'instances' => [],
    'services'  => [],
];
```

---

## 🧩 二、WebSocket 事件监听类

在 `app/webscoket/` 目录下新建以下三个类，分别对应连接、消息、断开三个事件。

### 1. `Connect.php` — 连接事件

```php
<?php
namespace app\webscoket;

use think\swoole\Websocket;

class Connect
{
    protected $websocket;

    public function __construct()
    {
        $this->websocket = app(Websocket::class);
    }

    public function handle($event)
    {
        $fd = $this->websocket->getSender();
        $this->websocket->push(json_encode([
            'type' => 'xdebug',
            'data' => "client {$fd} Connect",
            'time' => time()
        ]));
    }
}
```

---

### 2. `Event.php` — 消息事件

```php
<?php
namespace app\webscoket;

use think\swoole\Websocket;

class Event
{
    protected $websocket;

    public function __construct()
    {
        $this->websocket = app(Websocket::class);
    }

    public function handle($event)
    {
        $fd = $this->websocket->getSender();
        echo "client {$fd} Event\n";
        echo '接收到事件: ' . $event->type . ' --- ' . $event->data . "\n";
    }
}
```

---

### 3. `Close.php` — 断开事件

```php
<?php
namespace app\webscoket;

use think\swoole\Websocket;

class Close
{
    protected $websocket;

    public function __construct()
    {
        $this->websocket = app(Websocket::class);
    }

    public function handle($event)
    {
        $fd = $this->websocket->getSender();
        echo "client {$fd} Close\n";
    }
}
```

---

## 🖥️ 三、前端 HTML 测试页面

新建一个 HTML 文件用于调试 WebSocket 通信：

```html
<html>
<head>
<title>websocket</title>
</head>
 
<body>
<h1>websocket功能</h1>
 
<input id="msg" type="text"/>
<button onclick="send()">发送</button>
 
<script>
    // wss://echo.websocket.org 公共测试服务器
    var ws = new WebSocket("ws://你的ip地址:9501");
 
    ws.onopen = function (){
        console.log("连接成功");
        var sendObj = {};
        sendObj.type = 'connect';
        sendObj.data = 'connect success';
        console.log('msg',JSON.stringify(sendObj));
        ws.send(JSON.stringify(sendObj));
    }
    
    ws.onclose = function () {
        console.log("连接失败")
    }
 
    ws.onmessage = function (evt) {
        console.log("数据已接收",evt);
    }
 
    function send(){
        console.log('运行到这里了');
        var msg = document.getElementById('msg').value;
        var sendObj = {};
        sendObj.type = 'mtest';
        sendObj.data = msg;
 
        console.log('msg',JSON.stringify(sendObj));
        ws.send(JSON.stringify(sendObj));
    }
</script>
</body>
</html>
```

---

## 🔗 参考资料

* [ThinkPHP + Swoole WebSocket 配置教程（CSDN）](https://blog.csdn.net/quweiie/article/details/146481115)
* [Swoole 官方文档：WebSocket](https://wiki.swoole.com/zh-cn/#/websocket_server?id=open_websocket_ping_frame)
* [知乎：ThinkPHP + Swoole WebSocket 使用方法](https://zhuanlan.zhihu.com/p/142404854)

---

## ✅ 总结

通过本文，你可以快速搭建一个基于 Swoole 的 WebSocket 服务，完成连接、消息接收、断开监听等基础功能。结合 ThinkPHP 的事件机制，让 WebSocket 集成更清晰、更易扩展。

如果你想继续深入，可以加入用户身份绑定、Redis 聊天室中转、消息广播等高级功能。