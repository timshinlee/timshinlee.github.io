# 简介
Monkey是一个命令行工具，可以发送模拟的任意操作流到设备上，进行压力测试。

# 使用方法
可以从命令行或从脚本启动Monkey，因为Monkey运行在模拟器或者设备环境，必须从shell启动才行，可以通过每个命令前添加`adb shell`，或者进入shell模式输入Monkey命令。

基本语法：
```
$ adb shell monkey [options] <event-count> // 默认对设备上所有包进行测试

$ adb shell monkey -p your.package.name -v 500 // 对设备上指定包名的app发送500次操作
```

命令行参数：
- --help：使用帮助
- -v：每个-v都会提高日志级别，日志最多0/1/2个级别
- -s：随机种子，同样随机种子发送的操作是一样的
- --throttle <millseconds>：每个操作间的延迟，默认无延迟
- --pct-touch <percent>：点击事件比例
- --pct-motion <percent>：点击-移动-收起事件比例
- --pct-majornav <percent>：UI界面内的返回键或菜单键之类的导航键
- --pct-syskeys <percent>：Home键、Back键、音量键等系统键
- --pct=appswitch <percent>：activity启动的比例，使用startActivity去启动
- --pct-anyevent <percent>：所有其他类型的事件，例如按键
- -p <allowed-package-name>：要进行测试的包，多个则指定多个-p
- -c <main-category>：要测试的activity的category，如果未指定则会测试带有Intent.CATEGORY_LAUNCHER或者Intent.CATEGORY_MONKEY的activity。多个category则指定多个-c。

> [UI/App Exerciser Monkey](https://developer.android.google.cn/studio/test/monkey.html)
