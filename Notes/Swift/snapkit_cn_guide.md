# SnapKit 中文文档翻译

![SnapKit](http://upload-images.jianshu.io/upload_images/571495-8ad534ec6d5cad5c.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[SnapKit](https://github.com/SnapKit/SnapKit) 


- [要求](#要求)
- [迁移指南](#迁移指南)
- [交流](#交流)
- [安装](#安装)
- [使用](#使用)

# 要求
*  iOS 8.0+ / Mac OS X 10.11+ / tvOS 9.0+
*  Xcode 9.0+
*  Swift 4.0+

# 迁移指南
* [SnapKit 3.0 Migration Guide](https://github.com/SnapKit/SnapKit/blob/master/Documentation/SnapKit%203.0%20Migration%20Guide.md)

# 交流
* 如果你需要帮助，使用  [Stack Overflow](http://stackoverflow.com/questions/tagged/snapkit) 。（标签 `snapkit`）
* 如果你想问一些简单的问题，使用 [Stack Overflow](http://stackoverflow.com/questions/tagged/snapkit) 。
* 如果你发现一个 bug  ，请使用 issue 。
* 如果你有一个特别的需求， 请使用 issue 。
* 如果你想贡献一份自己的力量，你可以提交一个 pull 请求。

# 安装
### CocoaPods
[CocoaPods](http://cocoapods.org/) 是第三方的管理库。你可以使用下面的命令安装：
```
$ gem install cocoapods
```
> SnapKit 4.0.0+ 需要 CocoaPods 版本是 1.1.0 以上

为了使用 CocoaPods 能把 SnapKit 完整的安装到你的项目中，需要把下面内容加入到你的 `Podfile` 中：
```
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '10.0'
use_frameworks!

target '<Your Target Name>' do
   pod 'SnapKit', '~> 4.0'
end
```
然后，运行下面的命令：
```
$ pod install
```
### Carthage
[Carthage](https://github.com/Carthage/Carthage) 是一个会编译每个依赖框架，然后提供二进制文件的去中心化依赖管理器。

你可以使用  [Homebrew](http://brew.sh/) 用以下命令安装 Carthage ：
```
$ brew update
$ brew install carthage
```
为了使用 Carthage 能把 SnapKit 完整的安装到你的项目中，需要把下面内容加入到你的 `Cartfile` 中：
```
github "SnapKit/SnapKit" ~> 4.0
```
运行 `carthage update` 编译你的 framework ，然后把 `SnapKit.framework` 拖进你的项目中。

### Manually
如果你不喜欢上述的依赖管理，你可以手动地把 SnapKit 集成到你的项目中。

# 使用
SnapKit 设计的就是为了更简单的使用。假设我们想布局一个 box ，让它到父视图边缘距离是 20pts

```
let box = UIView()
superview.addSubview(box)

box.snp.makeConstraints { (make) -> Void in 
    make.top.equalTo(superview).offset(20)
    make.left.equalTo(superview).offset(20)
    make.bottom.equalTo(superview).offset(-20) 
    make.right.equalTo(superview).offset(-20)
}

```

或者更短：

```
let box = UIView()
superview.addSubview(box)

box.snp.makeConstraints { (make) -> Void in
    make.edges.equalTo(superview).inset(UIEdgeInsetsMake(20, 20, 20, 20))
}
```
在这个过程中， SnapKit 不仅做了大大的缩短和增加了约束的可读性，还处理了以下这几个关键步骤：
* 确定了最佳的父视图来添加约束。
* 追踪约束，以便以后能简单的移除。
* 在适当的视图上确保 `setTranslatesAutoresizingMaskIntoConstraints(false) ` 的调用。

### 并不是所有的都只能用 equal

> `.equalTo` 等同于 **NSLayoutRelation.Equal** 

>  `.lessThanOrEqualTo` 等同于 **NSLayoutRelation.LessThanOrEqual**

>  `.greaterThanOrEqualTo` 等同于 **NSLayoutRelation.GreaterThanOrEqual**
   
以上是接受一个参数的约束，下面的这些中任意一个也都可以：
##### 1. ViewAttribute
```
make.centerX.lessThanOrEqualTo(view2.snp.left)
```

 ViewAttribute              | NSLayoutAttribute | 
 ---------------------------|-------------------------------| 
 view.snp.left              | NSLayoutAttribute.left 
 view.snp.right             | NSLayoutAttribute.right 
 view.snp.top               | NSLayoutAttribute.top 
 view.snp.bottom            | NSLayoutAttribute.bottom 
 view.snp.leading           | NSLayoutAttribute.leading 
 view.snp.trailing          | NSLayoutAttribute.trailing 
 view.snp.width             | NSLayoutAttribute.width 
 view.snp.height            | NSLayoutAttribute.height 
 view.snp.centerX           | NSLayoutAttribute.centerX 
 view.snp.centerY           | NSLayoutAttribute.centerY 
 view.snp.lastBaseline      | NSLayoutAttribute.lastBaseline

##### 2. UIView/NSView
如果你想让 view.left 大于或等于 label.left ：
```
// these two constraints are exactly the same 这两个约束是完全一样的
make.left.greaterThanOrEqualTo(label)
make.left.greaterThanOrEqualTo(label.snp.left)
```
##### 3. Strict Checks
Auto Layout 允许你把宽高设置成一个常量。如果你想设置一个 view 宽度的最大值和最小值，你可以像下面这样：
```
// width >= 200 && width <= 400
make.width.greaterThanOrEqualTo(200)
make.width.lessThanOrEqualTo(400)
```
但是，像 left，right，centerY 等等这样的对齐属性， Auto Layout 是不允许你把它们设置成常量的。但是如果你把这些属性设置成了常量，SnapKit 会把这些转换成相对于父视图的约束：
```
// creates view.left <= view.superview.left + 10
make.left.lessThanOrEqualTo(10)
```
你也可以使用其他的方法构建你的约束，如下：
```
make.top.equalTo(42)
make.height.equalTo(20)
make.size.equalTo(CGSize(width: 50, height: 100))
make.edges.equalTo(UIEdgeInsetsMake(10, 0, 10, 0))
make.left.equalTo(view).offset(UIEdgeInsetsMake(10, 0, 10, 0))
```
### 优先级
> `.priority`允许你来指定明确的优先级

优先级可以写在约束链的末尾，如下：
```
make.top.equalTo(label.snp.top).priority(600)
```
### 一些组合

SnapKit 还提供了一些便利的方法来同时创建多个约束。

##### edges
```
// make top, left, bottom, right equal view2
make.edges.equalTo(view2)

// make top = superview.top + 5, left = superview.left + 10,
// bottom = superview.bottom - 15, right = superview.right - 20
make.edges.equalTo(superview).inset(UIEdgeInsetsMake(5, 10, 15, 20))
```
##### size
```
// make width and height greater than or equal to titleLabel
make.size.greaterThanOrEqualTo(titleLabel)

// make width = superview.width + 100, height = superview.height - 50
make.size.equalTo(superview).offset(CGSize(width: 100, height: -50))
```
##### center
```
// make centerX and centerY = button1
make.center.equalTo(button1)

// make centerX = superview.centerX - 5, centerY = superview.centerY + 10
make.center.equalTo(superview).offset(CGPoint(x: -5, y: 10))
```
你也可以创建视图的属性链以增加可读性：
```
// All edges but the top should equal those of the superview
make.left.right.bottom.equalTo(superview)
make.top.equalTo(otherView)
```
### 更多的选择
有时候为了动画或者删除、替换约束，你需要修改现有的约束。SnapKit 提供了一些不同的方法来更新约束。

##### 1. References
你可以通过约束的 make 表达式将局部变量或者类属性的约束结果分配给一个指定的约束。你也可以通过将它们存在数组中来引用多个约束。
```
var topConstraint: Constraint? = nil

...

// when making constraints
view1.snp.makeConstraints { (make) -> Void in 
  self.topConstraint = make.top.equalTo(superview).offset(padding.top).constraint 
  make.left.equalTo(superview).offset(padding.left)
}

...
// then later you can call
self.topConstraint.uninstall()

// or if you want to update the constraint
self.topConstraint.updateOffset(5)
```
##### 2. snp.updateConstraints
如果你只是想更新约束的值，你可以使用 `snp.updateConstraints` 方法来替代 `snp.makeConstraints`
```
// this is Apple's recommended place for adding/updating constraints
// this method can get called multiple times in response to setNeedsUpdateConstraints
// which can be called by UIKit internally or in your code if you need to trigger an update to your constraints

override func updateConstraints() { 
    self.growingButton.snp.updateConstraints { (make) -> Void in 
        make.center.equalTo(self)
        make.width.equalTo(self.buttonSize.width).priority(250)  
        make.height.equalTo(self.buttonSize.height).priority(250)
        make.width.lessThanOrEqualTo(self) make.height.lessThanOrEqualTo(self) 
     } 

// according to Apple super should be called at end of method 
     super.updateConstraints()
}
```
##### 3. snp.remakeConstraints
`snp.remakeConstraints` 和 ` snp.makeConstraints` 类似。不同的是，使用`snp.remakeConstraints` 需要先删除 SnapKit 安装的所有约束。
```
func changeButtonPosition() { 
  self.button.snp.remakeConstraints { (make) -> Void in 
    make.size.equalTo(self.buttonSize) 

    if topLeft { 
      make.top.left.equalTo(10) 
     } else {
      make.bottom.equalTo(self.view).offset(-10) 
      make.right.equalTo(self.view).offset(-10) 
     }
  }
}
```

##### 4. Snap view to topLayoutGuide and bottomLayoutGuide
