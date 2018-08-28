# Vapor

Vapor 是可以运行在 macOS 和 Ubuntu 上的 Swift Web 框架, Vapor 3 需要 Swift 4.1 支持, 本文选择 macOS 作为开发平台, 并默认你已经安装好了最新的 Xcode, 

## Vapor 入门

- 安装 Vapor

> Vapor 通过 Homebrew 安装, 如果没有安装 Homebrew, 请前往 [https://brew.sh/](https://brew.sh/), 跟随指引安装. 

```
brew install vapor/tap/vapor
```

 - 通过模板创建项目

```
vapor new HelloVapor
```

 - 编译

```
vapor build
```

 - 运行

```
vapor run
```

> 终端输出: Server starting on http://localhost:8080, 则表示成功运行
> 
> 停止运行按 ctrl + c

 - 在浏览器访问: `http://localhost:8080/hello`, 网页显示

```
Hello, world!
```

 - 生成 Xcode 项目
 
```
vapor xcode -y
```

 - 自定义路由
 
打开 `Sources/App/routes.swift`

输入

```
public func routes(_ router: Router) throws { 
    //响应 GET 请求, 路径是 http://localhost:8080/hello/test, 闭包回调 req, 返回值是 String.
    router.get("hello", "test") { req -> String in
        return "Router test"
    }
    //响应 GET 请求, 路径是 http://localhost:8080/hello/[用户输入String], 闭包回调 req, 通过 req 获取用户输入的字符串, 并返回.
    router.get("hello", String.parameter) { req -> String in
        let name = try req.parameters.next(String.self)
        return "Hello! \(name)"
    }
    // 响应 POST 请求, 路径是http://localhost:8080/info, Content type 是 InfoData, 返回 InfoResponse. 
    router.post(InfoData.self, at: "info") { (req, data) -> InfoResponse in
        return InfoResponse(request: data)
    }
}

struct InfoData: Content {
    let name: String
}

struct InfoResponse: Content {
    let request: InfoData
}
```

编译运行, 通过浏览器访问: `http://localhost:8080/hello/test`, 网页显示

```
Router test
```

通过浏览器访问: `http://localhost:8080/hello/Corgi`, 网页显示

```
Hello! Corgi
```

配置 Rested: 

路径: `http://localhost:8080/info`

请求类型: POST

请求体: {"name": "Corgi"}

响应体: {"request": {"name": "Corgi"}} 

 - 问题处理
	- 重新生成项目 `vapor xcode -y`
	- 更新依赖 `vapor update`
	- 清理项目重新编译 `rm -rf .build`
	- Vapor 版本不一致