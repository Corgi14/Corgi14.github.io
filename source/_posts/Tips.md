---
title: Tips
---

### 图片改色
 
```
var imageView = UIImageView()
if var imageToChange = imageView.image?.withRenderingMode(.alwaysTemplate) {
    imageView.image = imageToChange
    imageView.tintColor = .orange
}
```
### 子线程开启/关闭timer
 
 ```
DispatchQueue.global().async { [weak self] in
    self?.timer = Timer(timeInterval: 0.1, target: self!, selector: #selector(self?.timerTest), userInfo: nil, repeats: true)
    RunLoop.current.add((self?.timer!)!, forMode: .commonModes)
    RunLoop.current.run()
}
//关闭timer(官方建议同一线程)
DispatchQueue.global().sync { [weak self] in
    self?.stopTimer()
}
 ```

#### 系统右滑返回手势

```
navigationController?.interactivePopGestureRecognizer?.isEnabled = true
```