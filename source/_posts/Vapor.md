# Vapor

Vapor 是可以运行在 macOS 和 Ubuntu 上的 Swift Web 框架, Vapor 3 需要 Swift 4.1 支持, 本文选择 macOS 作为开发平台, 并默认你已经安装好了最新的 Xcode, 

## 目录

- [**Vapor 入门**](#Hello Vapor)

- [**HTTP 基础**](#HTTP Basics)

- [**异步**](#Async)


## <a name="Hello Vapor"></a>Vapor 入门

### 安装 Vapor

> Vapor 通过 Homebrew 安装, 如果没有安装 Homebrew, 请前往 [https://brew.sh/](https://brew.sh/), 跟随指引安装. 

```
brew install vapor/tap/vapor
```

### 通过模板创建项目

```
vapor new HelloVapor
```

### 编译

```
vapor build
```

### 运行

```
vapor run
```

> 终端输出: Server starting on http://localhost:8080, 则表示成功运行
> 
> 停止运行按 ctrl + c

在浏览器访问: `http://localhost:8080/hello`, 网页显示

```
Hello, world!
```

### 生成 Xcode 项目
 
```
vapor xcode -y
```

### 自定义路由
 
在 `Sources/App/routes.swift` 中输入

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

### 问题处理
- 重新生成项目 `vapor xcode -y`
- 更新依赖 `vapor update`
- 清理项目重新编译 `rm -rf .build`
- Vapor 版本不一致

## <a name="HTTP Basics"></a>HTTP 基础

### HTTP request

 - request line: HTTP method + resource requested + HTTP version (比如 GET /about.html HTTP/1.1)
 - request header
 - blank line
 - optional request body

### HTTP response

 - status line: HTTP version + status code + message
 - response header
 - 空行
 - optional response body

## <a name="Async"></a>异步

### 关于异步

为什么 **异步** 很重要呢, 我们用例子来说明.

想象一下, 在只有单线程的条件下, 有这样一个需求场景:

 - 一个关于股票报价的请求, 它会指向另外一个服务器 API;
 - 一个关于 CSS 的请求, 它可以立即响应;
 - 一个关于用户配置文件的请求, 需要查询数据库之后响应;
 - 一个关于静态 HTML 的请求, 他可以立即响应.

在同步服务器中, 服务器的线程会一直阻塞直到股票报价请求得到响应, 然后服务器处理 CSS 请求, 接下来服务器在进行查询数据库的时候再次阻塞, 直到返回用户配置文件, 只有到这时,服务器返回 HTML 给客户端.

而在异步服务器中, 服务器的线程开始处理股票报价的请求, 然后把它 **放到一边** 等待处理完成. 同时处理 CSS 请求, 查询数据库和处理 HTML 请求. 当股票报价请求完成, 服务器线程将他们的结果返回给客户端.

但是有人会提出服务器不是只有单线程, 虽然你是正确的, 但是服务器可调用的线程数量是有限的. 在线程之间切换是代价昂贵的, 而且确保每个数据接口是线程安全的也是耗时而且容易出现问题的. 所以, 紧靠开启线程来解决问题是低效的错误之选.

#### Future 和 promise

为了能够将耗时任务 **放到一边** 等待响应, 你需要将任务打包进 **promise** 以便得到相应的时候能够继续工作. 在代码中的表现就是, 你要改变可以 **放到一边** 的方法的返回值类型. 在同步环境中, 有如下方法:

```
func getAllUsers() -> [User] {
  // do some database queries
}
```

在异步环境中, 在 `getAllUsers()` 方法返回的时候数据库读取可能还未完成, 因此上述方法是不可行的. `[User]` 会在未来的某个时刻返回, 但不是现在. 在 Vapor 中, 你承诺返回一个结果这就叫做 **Future**, 你需要把上述方法改为:

```
func getAllUsers() -> Future<[User]> {
  // do some database queries
}
```

### 使用 Future

一开始使用 Future 让你感到困惑, 但是 Vapor 在很多地方都使用 Future.

#### Future 的解包

Vapor 为直接使用 Future 提供了许多便利的方法. 但是仍然存在许多需要等待响应的使用场景. 比如说, 想象一下, 你有一个路由, 在返回 `HTTP 204 No Content` 之前, 通过上面提到的 `getAllUsers()` 方法从服务器获取用户列表, 然后更改第一个用户的数据.

为了能够使用返回的数据, 你需要将结果解包并且提供一个闭包在 Future 返回的时候来执行. Vapor 提供了两种主要的方法来解包数据:

 - `flatMap(to:)`: 当解包方法承诺返回的闭包返回值是 Future 的时候使用.
 - `map(to:)`: 当解包方法的承诺返回的闭包返回值不是 Future 的时候使用.

这两种方法都会生成一个新的 Future, 比如:

```
// 1
return database
		.getAllUsers()
		.flatMap(to: HTTPStatus.self) { users in
			
	// 2
	let user = users[0]
	user.name = "Corgi"
	// 3
	return user.save().map(to: HTTPStatus.self) { user in
		return HTTPStatus.noContent
	}		
}
```

代码分析: 

1. 从数据库获取所有用户数据, 返回值是 `Future <[User]>`. 解包方法闭包返回值是另一个 `Future <HTTPStatus>` (见步骤 3), 所以使用 `flatMap(to:)`. 将解包完成后的 [User] 作为闭包的参数.
2. 更改第一个用户的数据.
3. 将更改后的数据同步更新到数据库, 由于解包生成 `Future <User>` 而闭包需要返回 HTTPStatus, 所以使用 `map(to:)`
4. 返回合适的 HTTPStatus.

如你所见. 在外层的承诺操作返回值是 Future, 所以使用 `flatMap(to:)`, 里层返回值是 HTTPStatus, 使用 `map(to:)`.

#### Transform

有些时候, 我们只关心 Future 能够的成功完成, 而不关心返回值. 在上面的需求中, 我们不需要对 `save()` 结果进行解包, 我们只需要将第三步改为 `transform(to:)`.

```
return database
		.getAllUsers()
		.flatMap(to: HTTPStatus.self) { users in
			
	// 2
	let user = users[0]
	user.name = "Corgi"
	// 3
	return user.save().transform(to: HTTPStatus.noContent)
}
```
这帮助我们减少代码嵌套, 提高代码可读性并且易于维护, 后面你会看到 `transform(to:)` 的大量使用.

#### Flatten

有些时候, 我们需要等待多个 Future 的完成, 比如向数据库存入多个模型数据, 在这种情况下, 你可以使用 `flatten(on:)`:

```
static func save(_ users: [User], request: Request) -> Future<HTTPStatus> {
  // 1
  var userSaveResults: [Future<User>] = []
  // 2
  for user in users {
    userSaveResults.append(user.save())
  }
  // 3
  return userSaveResults.flatten(on: request)
    //4
    .transform(to: HTTPStatus.created)
}
```

代码分析:

1. 定义 `Future <User>` 数组
2. 轮询 users 数组中的每个 user, 并将 `user.save()` 方法的返回值存入第 1 步的数组.
3. 使用 `flatten(on:)`, 等待所有 future 的完成, 这使用了 **Worker** (执行操作的真实线程, 是 Vapor 中的一种 Request, 在后面会学到). 如果 `flatten(on:)` 有闭包, 则会将返回的集合作为参数.

`flatten(on:)` 会在同一个 Worker 下, 等待所有的 Future 异步执行完毕.
#### 多种 Future

有些时候, 我们需要等待多个不同类型, 并且互相没有关系的 Future 执行完毕. 比如在对请求数据解码的同时从数据库获取用户信息. Vapor 提供了一些全局的便利方法, 可以支持最多 5 种不同 Future 的处理, 这减少了代码嵌套, 降低代码冗杂程度.

如果你有两种 Future - 对请求数据解码的同时从数据库获取用户信息 - 你可以:

```
// 1
flatMap(
  to: HTTPStatus.self,
  database.getAllUsers(),
  // 2
  request.content.decode(UserData.self)) { allUsers, userData in
    // 3
    return allUsers[0]
      .addData(userData)
      .transform(to: HTTPStatus.noContent)
}
```

代码分析:

1. 使用全局方法 `flatMap(to: _: _:)` 等待这两种 Future 执行完毕. 
2. 闭包将执行完成的 Future 作为参数.
3. 执行 `addData(_:)`, 返回 Future, 并转变成需要的返回值 `.noContent`.

如果闭包返回了一个非 Future 结果, 可以使用 `map(to: _: _:)` 代替:

```
// 1
map(
  to: HTTPStatus.self,
  database.getAllUsers(),
  // 2
  request.content.decode(UserData.self)) { allUsers, userData in
    // 3
    allUsers[0].syncAddData(userData)
    // 4
    return HTTPStatus.noContent
}
```

代码分析:

1. 使用全局方法 `flatMap(to: _: _:)` 等待这两种 Future 执行完毕. 
2. 闭包将执行完成的 Future 作为参数.
3. 执行同步方法 `syncAddData(_:)`
4. 返回 `.noContent`.

#### 创建 Future

有些时候, 你需要创建自己的 Future, 如果 `if` 语句返回非 Future返回值, `else` 返回 Future 返回值, 编译器会提示需要返回同样类型. 未解决类似问题, 你需要使用 `request.future(_:)` 将非 Future 转换为 Future, 比如:

```
// 1
func createTrackingSession(for request: Request)
  -> Future<TrackingSession> {
  return request.makeNewSession()
}

// 2
func getTrackingSession(for request: Request)
  -> Future<TrackingSession> {
  // 3
  let session: TrackingSession? =
    TrackingSession(id: request.getKey())
  // 4
  guard let createdSession = session else {
    return createTrackingSession(for: request)
  }
  // 5
return request.future(createdSession)
}
```

代码分析:

1. 定义了一个通过 request 创建 `TrackingSession` 的方法, 返回值是 `Future<TrackingSession>`
2. 定义了一个通过 request 获取 tracking session 的方法.
3. 尝试通过 request 的 key 创建 tracking session, 如果创建失败结果为 nil.
4. 使用 `guard let` 确认创建成功, 或者创建一个新的对象.
5. 通过 `request.future(_:)` 创建 `Future<TrackingSession>`, 与 request 在同一 Worker 执行, 并返回 Future.

因为 `createTrackingSession(for:)` 方法返回值是 `Future<TrackingSession>`, 所以需要使用 `request.future(_:)` 通过编译.

#### 错误处理

Vapor 在框架中大量的使用了 Swift 的错误处理, 很多方法通过 `throw` 允许你自己按等级处理错误. 你可以在你的路由闭包内部或者利用中间件来捕获更高级的错误, 或者两者都有.

但是在异步环境中对错误进行处理是有一点不同的, 因为你不知道承诺什么时候会执行, 所以不能使用 `do/catch`. Vapor 提供了一些方法来应对这些情况. Vapor 有它自己的 `do/catch` 回调协同 Future 工作.

```
let futureResult = user.save()
futureResult.do { user in
  print("User was saved")
}.catch { error in
  print("There was an error saving the user: \(error)")
}
```

如果 `save()` 成功, `do` 闭包被执行, 并将 Future 解包作为参数; 如果 Future 失败, `.catch`闭包被执行, 传递 Error.

在 Vapor 中, 当你持有了 request, 就必须返回结果, 哪怕是一个 Future. 使用上面的 `do/catch` 不会防止错误发生, 但是可以让你知道错误是什么. 如果 `save()` 执行失败它不会停止, 在大多数场景下, 你会想要更正这个错误.

Vapor 提供 `catchMap(_:)` 和 `catchFlatMap(_:)` 处理这种类型的失败, 你可以持有错误然后处理或者将它抛出:

```
// 1
return user.save(on: req).catchMap { error -> User in
  // 2
  print("Error saving the user: \(error)")
  // 3
  return User(name: "Default User")
}
```

代码分析:

1. 尝试保存 user, 提供了 `catchMap(_:)` 来持有可能出现的错误, 闭包将错误作为参数, 而且必须返回 User 类型.
2. 打印错误日志.
3. 创建默认 User 并返回.

当闭包返回 Future 类型的时候, Vapor 也提供相似的 `catchFlatMap(_:)` 方法.

```
return user.save().catchFlatMap { error -> Future<User> in
  print("Error saving the user: \(error)")
  return User(name: "Default User").save()
}
```

因为 `save()` 返回 Future 类型, 所以需要使用 `catchFlatMap(_:)` 代替.

catchMap 和 catchFlatMap

**to be continued**