---
title: Android Permission(runtime)
---

## 需求
一个简单的拨打电话的demo, 通过界面上的按钮来进行拨号. 初次进入App时需要检查&&开通权限, 如果用户没有开通, 则弹出提示框提示用户到设置界面自行开通.

## 实现
 - 初始化

```
 override fun onCreate(savedInstanceState: Bundle?) {
 	super.onCreate(savedInstanceState)
 	setContentView(R.layout.activity_main)
 	val callBtn = findViewById<Button>(R.id.button)
 	callBtn.setOnClickListener {
 		val permission = android.Manifest.permission.CALL_PHONE
 		val hasPermission = checkSelfPermission(permission)
		if (hasPermission != PackageManager.PERMISSION_GRANTED) {
			requestPermissions(arrayOf(permission), requestCode)
		} else {
			val callIntent = Intent(Intent.ACTION_CALL, Uri.parse("tel:18516172427"))
			startActivity(callIntent)
		}
	}
	//检察权限
	checkPermission()
}
```
- 检查权限
 
```
private fun checkPermission() {
	val permission = android.Manifest.permission.CALL_PHONE
	val hasPermission = checkSelfPermission(permission)
	if (hasPermission != PackageManager.PERMISSION_GRANTED) {
		requestPermissions(arrayOf(permission), requestCode)
	}
}
```
- 重写权限注册的回调

```
 override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<out String>, grantResults: IntArray) {
	super.onRequestPermissionsResult(requestCode, permissions, grantResults)
	if (requestCode == this.requestCode) {
		for (i in grantResults.indices ) {
			if (grantResults[i] != PackageManager.PERMISSION_GRANTED) {
				if (ActivityCompat.shouldShowRequestPermissionRationale(this@MainActivity, permissions[i])) {
					checkPermission()
				} else {
					Toast.makeText(this@MainActivity, "请去设置权限", Toast.LENGTH_LONG).show()
				}
				return
			}
		}
	}
}
```
 
## 关于 *ActivityCompat.shouldShowRequestPermissionRationale* 方法
 - 这个方法的返回值分以下几种情况
 
| 权限对话框选择情况 | 方法返回值 |
| :-----: | :-----: |
| 第一次打开App时 | false |
| 上次弹出权限点击了禁止 | true |
| 上次选择禁止并勾选"下次不再询问" | false |
