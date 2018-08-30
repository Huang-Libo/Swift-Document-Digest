
# 简介

**版本**: `Swift 4.2` (`Xcode 10`)  

**创建仓库的起因**: 我去年把 `Swift` 官方文档看了一遍, 今年再回想, 很多细节都不记得了(公司项目用的是 `Objective-C`, 所以用 `Swift` 的机会也不多), 如果忘了就再读一遍, 或者需要时再去茫茫文档中查询, 感觉是在消耗时间做重复的事, 倒不如写摘要以记录官方文档的要点和关键的示例代码, 日后的查询起来效率就高多了(也有一种把书读薄的功效).
 
**阅读提示**: 由于摘要比较简洁, 所以需要读者对 `Swift` 已有一定的了解, 或者大致看过官方文档([英文](https://docs.swift.org/swift-book)|[中文翻译版](https://www.cnswift.org/)). 个人推荐阅读英文版文档, 因为中文版的翻译会滞后一段时间, 并且翻译总有出错或失真部分. 可以在遇到看不懂的内容时再看中文翻译, 这样可以避免很多因翻译不准确而导致的误解.

# 目录

- [初始化](https://github.com/Huang-Libo/Swift-Digest#初始化-initialization)
    - [为存储属性设置初始化值](https://github.com/Huang-Libo/Swift-Digest#为存储属性设置初始化值)
    - [自定义初始化](https://github.com/Huang-Libo/Swift-Digest#自定义初始化)
    - [默认初始化器](https://github.com/Huang-Libo/Swift-Digest#默认初始化器)
    - [值类型的初始化器委托](https://github.com/Huang-Libo/Swift-Digest#值类型的初始化器委托)
    - [类的继承和初始化](https://github.com/Huang-Libo/Swift-Digest#类的继承和初始化)
    - [两段式初始化](https://github.com/Huang-Libo/Swift-Digest#两段式初始化)
- [参考资料](https://github.com/Huang-Libo/Swift-Digest#参考资料)

# 初始化 ([Initialization](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html))

指初始化`类`, `结构体`, `枚举`的实例, 并给`存储属性`赋值等操作.

和 `Objective-C` 的初始化器不同的是, `Swift` 的初始化器不 `return` 值.

类的实例还可以实现一个`反初始化器`.

## 为存储属性设置初始化值

> `类`和`结构体`在创建实例时, 必须给他们的`存储属性`设置一个合适的初始值. (通过初始化器设置, 或者在定义时给一个默认值, 这两种途径都不会触发`属性观察`)  

如果一个存储属性**总是使用固定值**, 最好是在定义时就给定初始化值, 而不是在初始化器中设置. 这样更简洁易读, 并且有利于 Swift 根据默认值去推断属性的类型, 还可以使我们更容易地利用`默认初始化器`和`初始化器继承`.(后面会讲到...待补充...) 例如, 如果 temperature 属性是一个固定值:


推荐:

```swift
struct Fahrenheit {
    var temperature = 32.0
}
```

不推荐:

```swift
struct Fahrenheit {
    var temperature: Double
    init() {
        temperature = 32.0
    }
}
```

## 自定义初始化

#### 参数名和内容提要标签

`参数名(parameter name)`: 在内部使用.  
`内容提要标签(argument label)`: 在调用时使用.  

这和`函数`和`方法`是一样的.

#### 可选属性类型

可选属性会被自动初始化为 nil.  

#### 初始化时给常量属性赋值

常量被赋值后, 就不能再更改了.   
注意: 对`类`而言, 常量属性只能在本类初始化时被修改, 而不能被其子类修改.  

## 默认初始化器

如果一个`结构体`或`类`给它所有的属性都设置了默认值, 并且自身一个初始化器也没有提供(比如: 没有父类, 自身也没实现初始化器), 那么 Swift 就会为它提供一个`默认初始化器`. 例如:

```swift
// 属性都有默认值, 类没有实现初始化器(也没有父类), 
// 所以 Swift 会提供一个默认初始化器.
class ShoppingListItem {
    var name: String?
    var quantity = 1
    var purchased = false
}
var item = ShoppingListItem()
```

#### 结构体类型的成员初始化器

和`默认初始化器`不同的是, 结构体的`成员初始化器(memberwise initializer)`不需要其存储属性有默认值. 

```swift
struct Size {
    var width, height: Double
}
let twoByTwo = Size(width: 2.0, height: 2.0)
```

## 值类型的初始化器委托

`初始化器委托(Initializer Delegation)`: 初始化器可以调用其他初始化器来完成实例的部分初始化, 这样可以减少冗余代码.  

注意: 
1. `值类型`和`类类型`的初始化器规则不同, 值类型(结构体和枚举)不支持继承, 所以他们的初始化器委托过程相对简单. 类类型支持继承, 所以它要保证其所继承的所有存储属性都要在初始化时赋一个合适的值.
2. 如果你给值类型定义了一个自定义的初始化器, 那么你将无法访问`默认初始化器`(或者`成员初始化器`, 对结构体而言). 这个限制防止别人意外地使用自动初始化器而把复杂初始化器里提供的额外必要配置给绕开(中文文档里面把这里翻译成了覆盖, 是不对的)的情况发生。
3. 如果你想让值类型使用默认初始化器或成员初始化器, 同时也使用自定义初始化器, 那么请在`拓展(Extension)`中实现自定义初始化器.

对值类型而言, 可以通过 `self.init`来调用其他初始化器, `self.init`只能在初始化器中调用.

实例: 

```swift
struct Size {
    var width = 0.0, height = 0.0
}
struct Point {
    var x = 0.0, y = 0.0
}
```

```swift
struct Rect {
    var origin = Point()
    var size = Size()
    // 初始化器一
    init() {}
    // 初始化器二
    init(origin: Point, size: Size) {
        self.origin = origin
        self.size = size
    }
    // 初始化器三
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}
```

解读:  

第一个初始化器, 和`默认初始化器`功能相同.
```swift
let basicRect = Rect()
// basicRect's origin is (0.0, 0.0) and its size is (0.0, 0.0)
```

第二个初始化器, 和`成员初始化器`功能相同.
```swift
let originRect = Rect(origin: Point(x: 2.0, y: 2.0),
                      size: Size(width: 5.0, height: 5.0))
// originRect's origin is (2.0, 2.0) and its size is (5.0, 5.0)
```

第三个初始化器中调用了第二个初始化器(初始化器委托).
```swift
let centerRect = Rect(center: Point(x: 4.0, y: 4.0),
                      size: Size(width: 3.0, height: 3.0))
// centerRect's origin is (2.5, 2.5) and its size is (3.0, 3.0)
```

如果不想自己实现 `init()` and `init(origin:size:)`, 则需要在`扩展(Extension)`中实现 `init(center:size:)` 初始化器.  

## 类的继承和初始化

Swift 定义了两种初始化器来确保所有的存储属性(包括自身的和继承的)都接收到一个初始值: `指定初始化器(designated initializer)`和`便捷初始化器(convenience initializer)`.

类的指定初始化器的语法(和值类型的语法相同):
```swift
init(parameters) {
    statements
}
```

类的便捷初始化器的语法:
```swift
convenience init(parameters) {
    statements
}
```

#### 类类型的初始化器委托

类类型的初始化器委托遵循三个规则:
1. 指定初始化器必须调用其直接父类的指定初始化器.
2. 便捷初始化器必须调用*同一个类*中的初始化器.
3. 便捷初始化器必须最终调用到指定初始化器.

示例1:  
<img src="./media/initializerDelegation01_2x.png" width="50%" height="50%">

示例2:  
<img src="./media/initializerDelegation02_2x.png" width="50%" height="50%">

## 两段式初始化

# 参考资料

- 官方文档(英文): https://docs.swift.org/swift-book  
- 官方文档的中文翻译: https://www.cnswift.org/  
- http://swiftguide.cn/  


