# URL 路由

<p align="center">
    <a href="http://perfect.org/get-involved.html" target="_blank">
        <img src="http://perfect.org/assets/github/perfect_github_2_0_0.jpg" alt="Get Involed with Perfect!" width="854" />
    </a>
</p>

<p align="center">
    <a href="https://github.com/PerfectlySoft/Perfect" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_1_Star.jpg" alt="Star Perfect On Github" />
    </a>  
    <a href="https://gitter.im/PerfectlySoft/Perfect" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_2_Git.jpg" alt="Chat on Gitter" />
    </a>  
    <a href="https://twitter.com/perfectlysoft" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_3_twit.jpg" alt="Follow Perfect on Twitter" />
    </a>  
    <a href="http://perfect.ly" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_4_slack.jpg" alt="Join the Perfect Slack" />
    </a>
</p>

<p align="center">
    <a href="https://developer.apple.com/swift/" target="_blank">
        <img src="https://img.shields.io/badge/Swift-3.0-orange.svg?style=flat" alt="Swift 3.0">
    </a>
    <a href="https://developer.apple.com/swift/" target="_blank">
        <img src="https://img.shields.io/badge/Platforms-OS%20X%20%7C%20Linux%20-lightgray.svg?style=flat" alt="Platforms OS X | Linux">
    </a>
    <a href="http://perfect.org/licensing.html" target="_blank">
        <img src="https://img.shields.io/badge/License-Apache-lightgrey.svg?style=flat" alt="License Apache">
    </a>
    <a href="http://twitter.com/PerfectlySoft" target="_blank">
        <img src="https://img.shields.io/badge/Twitter-@PerfectlySoft-blue.svg?style=flat" alt="PerfectlySoft Twitter">
    </a>
    <a href="https://gitter.im/PerfectlySoft/Perfect?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge" target="_blank">
        <img src="https://img.shields.io/badge/Gitter-Join%20Chat-brightgreen.svg" alt="Join the chat at https://gitter.im/PerfectlySoft/Perfect">
    </a>
    <a href="http://perfect.ly" target="_blank">
        <img src="http://perfect.ly/badge.svg" alt="Slack Status">
    </a>
</p>

本程序展示了如何设置服务器URL路由并定向到您自己定义的路由句柄。

请使用 SPM 软件包管理器编译本工程，即在工程目录下执行命令 ```swift build``` 后运行 ``` .build/debug/URLRouting```。

如果要使用 Xcode 编译，请将编译目标（target）设置为 **URL Routing** 。这样做可以启动 Perfect 服务器。

启动后请用浏览器查看 [http://localhost:8181/](http://localhost:8181/)以检查运行结果。您可以尝试自行增加不同的 URL 路由来试验效果。

## 问题报告

目前我们已经把所有错误报告合并转移到了JIRA上，因此github原有的错误汇报功能不能用于本项目。

您的任何宝贵建意见或建议，或者发现我们的程序有问题，欢迎您在这里告诉我们。 [http://jira.perfect.org:8080/servicedesk/customer/portal/1](http://jira.perfect.org:8080/servicedesk/customer/portal/1)。

目前问题清单请参考以下链接： [http://jira.perfect.org:8080/projects/ISS/issues](http://jira.perfect.org:8080/projects/ISS/issues)


## 注册并登记 URL 路由

以下代码是示例工程的程序片段，展示了如何向服务器创建并增加路由。

```swift
var routes = Routes()

routes.add(method: .get, uris: ["/", "index.html"], handler: indexHandler)
routes.add(method: .get, uri: "/foo/*/baz", handler: echoHandler)
routes.add(method: .get, uri: "/foo/bar/baz", handler: echoHandler)
routes.add(method: .get, uri: "/user/{id}/baz", handler: echo2Handler)
routes.add(method: .get, uri: "/user/{id}", handler: echo2Handler)
routes.add(method: .post, uri: "/user/{id}/baz", handler: echo3Handler)

// 如果用curl 可以用下面的命令行进行测试
// curl --data "{\"id\":123}" http://0.0.0.0:8181/raw --header "Content-Type:application/json"
routes.add(method: .post, uri: "/raw", handler: rawPOSTHandler)

// 使用通配符将剩下的其他的路由都统一到一个处理器句柄来管理
routes.add(method: .get, uri: "**", handler: echo4Handler)

// 在基本 URI 上增加路由。

// 从版本 v1 的API上增加路由
var api = Routes()
api.add(method: .get, uri: "/call1", handler: { _, response in
	response.setBody(string: "API 调用 1")
	response.completed()
})
api.add(method: .get, uri: "/call2", handler: { _, response in
	response.setBody(string: "API 调用 2")
	response.completed()
})

// API 版本 v1
var api1Routes = Routes(baseUri: "/v1")
// API 版本 v2
var api2Routes = Routes(baseUri: "/v2")

// 为版本 v1 增加主调 API
api1Routes.add(routes: api)
// 为版本 v2 增加主调 API
api2Routes.add(routes: api)
// 更新第二个调用的API
api2Routes.add(method: .get, uri: "/call2", handler: { _, response in
	response.setBody(string: "API 版本 v2 调用 2")
	response.completed()
})

// 将所有版本的路由都注册到服务器路由表上
routes.add(routes: api1Routes)
routes.add(routes: api2Routes)

// 将路由表逻辑结构输出到终端控制台，查看上面的操作都注册了什么新内容
print("\(routes.navigator.description)")

// 创建服务器
let server = HTTPServer()

// 监听8181端口
server.serverPort = 8181

// 将路由表保存到服务器
server.addRoutes(routes)

do {
    // 启动 HTTP 服务器
    try server.start()
} catch PerfectError.networkError(let err, let msg) {
    print("网络异常：\(err) \(msg)")
}
```
## 处理响应

示例 `EchoHandler` 还包含了以下代码。

```swift
func echoHandler(request: HTTPRequest, _ response: HTTPResponse) {
	response.appendBody(string: "“回声句柄” 您已经访问了路径 \(request.path) ，并包括以下变量： \(request.urlVariables)")
	response.completed()
}
```

## 使用 Apache
以下 Apache 配置文件的片段示范了如何将 HTTP 请求的、并不以静态文件方式存在的逻辑路由，将其HTTP的请求和响应重新定向到 Perfect服务器的 URL 路由上。

```apacheconf
	RewriteEngine on
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteRule (.*) - [L,NS,H=perfect-handler]
```


## 更多内容
关于 Perfect 工程的更多内容，请参考官网 [perfect.org](http://perfect.org).
