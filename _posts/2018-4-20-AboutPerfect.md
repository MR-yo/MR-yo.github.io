---
layout: post
author: syea
title: Perfect 入坑
# subtitle:
# header-img: 
# header-mask:  
# catalog: 
# multilingual: 
tags:
    - Swift
---

# Perfect 入坑

一直想尝试一下写服务器，然而对服务器一直有种望而生畏的感觉。处理各种请求，查询各种数据，还有什么多表查询、分布式数据库、redis缓存...等等等等。就感觉门槛应该挺高吧。

其实对 Perfect 早有耳闻，但是那时并没有 swift 基础，加上最近工作比较轻松，所以又开始研究起来。

### 自己的理解

服务器是不是只是对外开放 API，内部操作数据库? (持怀疑态度，但是现在是有点这样认为的，应该是认知深度还不够)。

### 实践

1. **运行 Demo** <br>
按照 Perfect github 上指导的一步步进行，第一次没有装数据库，服务器运行起来很 ok，也能测试接口。突然就感觉离服务器的世界近了很多。
```
// 示例接口的 handle
func handler(request: HTTPRequest, response: HTTPResponse) {
	// Respond with a simple message.
	response.setHeader(.contentType, value: "text/html")
	response.appendBody(string: "<html><title>Hello, world!</title><body>Hello, world!</body></html>")
	// Ensure that response.completed() is called when your processing is done.
	response.completed()
}
```
![](http://owlvwomsh.bkt.clouddn.com/test1.png)

2. **安装数据库**<br>
算是入了一下门，接下来安装数据库。我比对了一下各种数据库，最后选择了 Mysql。<br>
从官网装的 Mysql，设置用户，创建数据库，创建表，Navicat Permium 连接 Mysql 数据库。ok 很顺利（其实不是，坑1🙃）。
接下来在 Perfect 项目的 `Package.swift` 文件中添加相关依赖
```
.Package(url: "https://github.com/PerfectlySoft/Perfect-MySQL.git", majorVersion: 3)
```
然后重新build，终端`swift build`。ok，build 成功（其实没有，坑2🙃）。

ok, 仿佛一切准备就绪，接下来搞一个用户的增删改查！

### 还是实践

1. 首先我们建一张用户表
![](http://owlvwomsh.bkt.clouddn.com/建表.png)

2. 编写接口

### 踩过的坑，记得更深

1. 连接数据库的时候，root 用户密码用一个`caching_sha2_password`加密，而 Navicat 并不支持，导致我死活连不上数据库。而新建用户的时候，密码采用`mysql_native_password`加密，用这种加密方式的密码，Navicat 可以连接上。

2. 从官网下的 Mysql，在 Perfect 项目添加依赖并重新 build 的时候，会报错，**can't find main.swift balabala...** 具体内容忘了，我猜测应该是路径不对，反正各种 build 失败。最后是删除了先前装的 Mysql，用`Homebrew`重新下的，`brew install mysql`。再次 build 的时候就 ok 了。