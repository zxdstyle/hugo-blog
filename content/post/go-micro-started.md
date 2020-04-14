---
title: "GoLang 微服务 go-micro 之启蒙篇"
date: 2020-04-14
lastmod: 2020-04-14
draft: false
tags: ["微服务", "服务注册", "GoLang", "Go Micro"]
categories: ["GoLang", "Go Micro"]
author: "zxdstyle"
contentCopyright: '本站所有文章无特殊说明都是原创，转载请注明出处<a rel="license noopener" href="https://www.zxdstyle.cn" target="_blank">www.zxdstyle.cn</a>'

---

# Get Started
上来直接一把梭，上代码：
```go
// 直接引入我们今天的主角
import (
	"fmt"
	"github.com/micro/go-micro"
)

func main() {
	// 创建一个新的服务
	service := micro.NewService()
	// 初始化
	service.Init()
	// 启动服务
	if err := service.Run(); err != nil {
		fmt.Println(err)
	}
}
```
然后执行 `go run main.go`启动服务，不出意外的话你会看到一下输出（出意外我也没辙:joy:）
```go
2020-04-13 11:18:47.658403 I | Transport [http] Listening on [::]:55351
2020-04-13 11:18:47.658510 I | Broker [http] Connected to [::]:55352
2020-04-13 11:18:47.658929 I | Registry [mdns] Registering node: go.micro.server-7b38a9bc-a20b-4c89-b5b7-fd40cafe22a2
```
看到这些输出意味着服务正常启动，并且还将服务注册到 mdns 中。这个时候有的人会问了， mdns 是个什么东西。其实 mdns 就是传说中的服务注册中心啦，接下来就讲讲服务注册中心到底是个啥。

# 服务注册中心
服务注册中心到底是个啥？举个栗子，一个服务就相当于一个手机号码，你可能记得你家人的手机号码，但是你的朋友老板客户的手机号码你记得么？有的人还有可能有多个手机号码，所以你的手机里有一个通讯录。

微服务也是，一个两个服务你可以直接告诉调用方有这么个服务，直接调用就好了。那如果有几百几千个服务呢？所以微服务注册中心应运而生。

注册中心就相当于通讯录，存储了服务与对应服务地址的映射关系。通常一个服务器可能部署多个节点进行负载均衡。比如说现在有一个获取最新文章的接口，因为请求量比较大，我们将这个接口微服务化，并且部署多个节点。那这个时候问题来了,比如说我现在部署了三个节点,分别是`127.0.0.1:8001`、`127.0.0.1:8002`、`127.0.0.1:8003`，请求任意一个节点都可以正常获取数据。
1. 那客户端怎么知道该请求哪个节点你呢？
2. 如果某个节点出问题不可用了，该怎么让客户端自动切换到可用节点呢？
3. 怎么让请求合理的分配到各个节点呢？

这些问题都可以用服务注册中心来解决，下面举个例子来一一说明：

# 安装 etcd
上文启动服务时，go micro 默认使用了 mdns 作为注册中心。我们这里使用 etcd 作为注册中心。

Mac 可以使用 `brew install etcd` 安装，用 window 的同学可以自行去 [github](https://github.com/etcd-io/etcd/releases) 下载安装。

安装完成后直接执行 `etcd` 启动即可。

然后安装一下 go micro 提供的命令行工具 micro:

```
go get github.com/micro/micro/v2
```

执行 `MICRO_REGISTRY=etcd micro web` 即可启动 web 界面，访问`127.0.0.1:8082` 就可以查看到 etcd 中注册了哪些服务。

# 业务接口
上文开启的服务中未实现任何接口，为了方便演示，我们小小改造一下，先使用常用的 gin 框架编写一个 HTTP 接口，并整合到 micro 里。

```
import (
	"github.com/gin-gonic/gin"
	"github.com/micro/go-micro/registry"
	"github.com/micro/go-micro/registry/etcd"
	"github.com/micro/go-micro/web"
)

func main() {
	// 指定注册中心
	etcdReg := etcd.NewRegistry(registry.Addrs("127.0.0.1:2379"))

	router := gin.Default()

	router.Handle("GET", "/hello", func(context *gin.Context) {
		context.String("hello world")
	})

	server := web.NewService(
		// 为我们的服务取个名字 
		web.Name("product"),
		// 指定 handler
		web.Handler(router),
		// 将服务注册到 etcd
		web.Registry(etcdReg),
	)

	server.Init()

	server.Run()
}
```

然后执行 `go run main.go` 启动服务：

![GoLang 微服务 go-micro 之启蒙篇](https://cdn.learnku.com/uploads/images/202004/13/39723/eO9PKVErma.png!large)
