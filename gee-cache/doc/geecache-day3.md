---
title: 动手写分布式缓存 - GeeCache第三天 HTTP 服务端
date: 2020-02-12 22:00:00
description: 7天用 Go语言/golang 从零实现分布式缓存 GeeCache 教程(7 days implement golang distributed cache from scratch tutorial)，动手写分布式缓存，参照 groupcache 的实现。本文介绍了如何使用标准库 http 搭建 HTTP Server，为 GeeCache 单机节点搭建 HTTP 服务，并进行相关的测试。
tags:
- Go
nav: 从零实现
categories:
- 分布式缓存 - GeeCache
keywords:
- Go语言
- 从零实现
- 分布式缓存
- HTTP Server
image: post/geecache-day3/http.jpg
github: https://github.com/geektutu/7days-golang
---

本文是[7天用Go从零实现分布式缓存GeeCache](https://geektutu.com/post/geecache.html)的第三篇。

- 介绍如何使用 Go 语言标准库 `http` 搭建 HTTP Server
- 并实现 main 函数启动 HTTP Server 测试 API，**代码约60行**

## 1 http 标准库

Go 语言提供了 `http` 标准库，可以非常方便地搭建 HTTP 服务端和客户端。比如我们可以实现一个服务端，无论接收到什么请求，都返回字符串 "Hello World!"

```go
package main

import (
	"log"
	"net/http"
)

type server int

func (h *server) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	log.Println(r.URL.Path)
	w.Write([]byte("Hello World!"))
}

func main() {
	var s server
	http.ListenAndServe("localhost:9999", &s)
}
```

- 创建任意类型 server，并实现 `ServeHTTP` 方法。
- 调用 `http.ListenAndServe` 在 9999 端口启动 http 服务，处理请求的对象为 `s server`。

接下来我们执行 `go run .` 启动服务，借助 curl 来测试效果：

```bash
$ curl http://localhost:9999  
Hello World!
$ curl http://localhost:9999/abc
Hello World!
```

Go 程序日志输出

```bash
2020/02/12 22:56:32 /
2020/02/12 22:56:34 /abc
```

> `http.ListenAndServe` 接收 2 个参数，第一个参数是服务启动的地址，第二个参数是 Handler，任何实现了 `ServeHTTP` 方法的对象都可以作为 HTTP 的 Handler。

## 2 GeeCache HTTP 服务端

分布式缓存需要实现节点间通信，建立基于 HTTP 的通信机制是比较常见和简单的做法。如果一个节点启动了 HTTP 服务，那么这个节点就可以被其他节点访问。今天我们就为单机节点搭建 HTTP Server。

不与其他部分耦合，我们将这部分代码放在新的 `http.go` 文件中，当前的代码结构如下：

```bash
geecache/
    |--lru/
        |--lru.go  // lru 缓存淘汰策略
    |--byteview.go // 缓存值的抽象与封装
    |--cache.go    // 并发控制
    |--geecache.go // 负责与外部交互，控制缓存存储和获取的主流程
	|--http.go     // 提供被其他节点访问的能力(基于http)
```

首先我们创建一个结构体 `HTTPPool`，作为承载节点间 HTTP 通信的核心数据结构（包括服务端和客户端，今天只实现服务端）。

[day3-http-server/geecache/http.go - github](https://github.com/geektutu/7days-golang/tree/master/gee-cache/day3-http-server/geecache)

```go
package geecache

import (
	"fmt"
	"log"
	"net/http"
	"strings"
)

const defaultBasePath = "/_geecache/"

// HTTPPool implements PeerPicker for a pool of HTTP peers.
type HTTPPool struct {
	// this peer's base URL, e.g. "https://example.net:8000"
	self     string
	basePath string
}

// NewHTTPPool initializes an HTTP pool of peers.
func NewHTTPPool(self string) *HTTPPool {
	return &HTTPPool{
		self:     self,
		basePath: defaultBasePath,
	}
}
```

- `HTTPPool` 只有 2 个参数，一个是 self，用来记录自己的地址，包括主机名/IP 和端口。
- 另一个是 basePath，作为节点间通讯地址的前缀，默认是 `/_geecache/`，那么 http://example.com/_geecache/ 开头的请求，就用于节点间的访问。因为一个主机上还可能承载其他的服务，加一段 Path 是一个好习惯。比如，大部分网站的 API 接口，一般以 `/api` 作为前缀。

接下来，实现最为核心的 `ServeHTTP` 方法。

```go

```


