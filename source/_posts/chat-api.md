---
title: chat_api
date: 2018-11-21 15:37:05
tags:
---

## 创建公共房间

#### /api/chat/public/create

#### 请求
``` js
// POST
{
    "chatId": "simi",
    "roomName": "房间名"
}
```

#### 返回
``` js
{
    "result": {
        "chatId": "simi",
        "roomName": "房间名"
    },
    "success": true
}
```

## 创建私密房间

#### /api/chat/secret/create

#### 请求
``` js
// POST
{
	"chatId": "simi",
	"key": "simi",
    "roomName": "房间名"
}
```

#### 返回
``` js
{
    "result": {
        "chatId": "simi",
        "key": "simi",
        "roomName": "房间名"
    },
    "success": true
}
```

## 统一失败返回

``` js
{
    "message": "失败，原因xxx",
    "success": false
}
```
