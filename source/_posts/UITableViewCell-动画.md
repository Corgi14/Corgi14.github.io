# UITableViewCell 动画

最近发现了一篇文章: [一行代码让你的TableView动起来-iOS动画](https://juejin.im/post/59b11f606fb9a024a27c2a69), 自己写了一个 Utilities 工具类, 让 UIView 动起来. 

## Utilities 

### Utilities 类, 以及常量的声明

```
class Utilities: NSObject {
    private static let animationDuration = 0.5
    private static let animationDelay = 0.0
    private static let animationSpring: CGFloat = 0.2
    private static let screenWidth = UIScreen.main.bounds.width
    private static let screenHeight = UIScreen.main.bounds.height
}
```

### 动画效果

 - 随机动画效果(暂时只有 rotate 和 spring 动画)

```
class func animateRandomly(view: UIView, completion: (() -> Void)?) {
    let random = Int(arc4random_uniform(2))
    switch random {
    case 1:
        self.springOutAnimation(view: view, completion: completion)
    default:
        self.rotateAnimation(view: view, completion: completion)
    }
}
```

 - rotate 动画: 

```
private class func springOutAnimation(view: UIView, completion: (() -> Void)?) {
    view.layer.transform = CATransform3DMakeTranslation(screenWidth / 3, 0, 0)
    UIView.animate(withDuration: animationDuration, delay: animationDelay, usingSpringWithDamping: animationSpring, initialSpringVelocity: 1 / animationSpring, options: .curveEaseIn, animations: {
        view.layer.transform = CATransform3DIdentity
    }) { (result) in
        if result {
            if let tempCompletion = completion {
                tempCompletion()
            }
        }
    }
}
```

 - spring 动画:

```
private class func springOutAnimation(view: UIView, completion: (() -> Void)?) {
    view.layer.transform = CATransform3DMakeTranslation(screenWidth / 3, 0, 0)
    UIView.animate(withDuration: animationDuration, delay: animationDelay, usingSpringWithDamping: animationSpring, initialSpringVelocity: 1 / animationSpring, options: .curveEaseIn, animations: {
        view.layer.transform = CATransform3DIdentity
    }) { (result) in
        if result {
            if let tempCompletion = completion {
                tempCompletion()
            }
        }
    }
}
```

 - 一组 UIView 依次动画

```
class func dropDownAnimation(views: [UIView]) {
    for i in 0..<views.count {
        let view = views[i]
        view.layer.transform = CATransform3DMakeTranslation(0, -screenHeight, 0)
        UIView.animate(withDuration: animationDuration, delay: TimeInterval(CGFloat(views.count - i) * (0.8 / CGFloat(views.count))), options: .layoutSubviews, animations: {
            view.layer.transform = CATransform3DIdentity
        }, completion: nil)
    }
}
```

## How to use

 - Requirement 1: UITableViewCell 依次动画展示

```
override func viewDidLoad() {
    super.viewDidLoad()
    perform(#selector(animateList), with: nil, afterDelay: 0.1)
}

@objc private func animateList() {
    let cells = goodList.visibleCells
    Utilities.dropDownAnimation(views: cells)
}
```

 - Requirement 2: UITableViewCell 点击动画

```
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    guard let cell = tableView.cellForRow(at: indexPath) else { return }
    Utilities.animateRandomly(view: cell) {
        //Do something after the animation
    }
}
```