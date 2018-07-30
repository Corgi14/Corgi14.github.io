---
title: Runtime in Swift
---

为了验证Swift中使用runtime机制, 我们先创建一个类

```
class SwiftRuntimeClass {
    var propertyA: Bool = true
    var propertyB: Int = 0
}
```

然后试着去获取**SwiftRuntimeClass**的参数列表

```
var count: UInt32 = 0
let `class` = SwiftRuntimeClass()
guard let propertyList = class_copyPropertyList(object_getClass(`class`), &count) else {
    print("propertyList is nil")
    return
}
for i in 0..<numericCast(count) {
    let property = property_getName(propertyList[i])
    print(String(utf8String: property) ?? "no property")
}
```

打印结果为:

```
propertyList is nil
```

这是因为**Swift**与**OC**不同, 对象在编译时就是固定类型, 为了能够使用runtime, 就要靠 *dynamic* 修饰符来实现.

```
class SwiftRuntimeClass {
    @objc dynamic var propertyA: Bool = true
    var propertyB: Int = 0
}
```

为了对照, 我们在 *propertyA* 前面加 *dynamic* 修饰符, 系统自动在前面加上了 *@objc* 修饰符.

再次编译运行, 打印结果为:

```
propertyA
```

##其他Runtime方法

1.获取函数

```
let `class` = SwiftRuntimeClass()
        guard let methodList = class_copyMethodList(object_getClass(`class`), &count) else {
            print("methodList is nil")
            return
        }
        for i in 0..<numericCast(count) {
            let method = method_getName(methodList[i])
            print(String(NSStringFromSelector(method)) ?? "no method")
        }
```

2.交换函数

```
guard let methodA = class_getInstanceMethod(object_getClass(`class`), #selector(`class`.functionA)) else { return }
guard let methodB = class_getInstanceMethod(object_getClass(`class`), #selector(`class`.functionB)) else { return }
method_exchangeImplementations(methodA, methodB)
`class`.functionA()
```

打印结果

```
FunctionB
```