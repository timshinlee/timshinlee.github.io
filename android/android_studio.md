# Android Studio配置
AS提供两个设置文件：
- studio.vmoptions:定制AS的JVM属性，比如堆大小和缓存大小。
- idea.properties:配置AS的属性，例如插件文件夹路径或者最大支持的文件大小。
这两个文件的储存在AS的配置文件夹当中，配置文件夹是`%USERPROFILE%\.<AS版本号>`。默认的`%USERPROFILE%`是`C:\Users\Administrator`。

下面给出一些常用的配置属性，注意修改后需要重启AS才能生效：
```
-Xmx2g	// Maximum heap size 2G
```
要查看默认参数及当前生效参数，可以使用`jps -lvm`命令查看。

< [Configure the IDE](https://developer.android.google.cn/studio/intro/studio-config.html#customize_ide)

# 开发配置
### 使用Java 8
build.gradle of module:
```groovy
android {
	...
	compileOptions {
		sourceCompatibility JavaVersion.VERSION_1_8
		targetCompatibility JavaVersion.VERSION_1_8
	}
}
```
# XML tools命名空间
`tools`这个命名空间包含了一些设计时特性和编译时特性，在构建app的时候，构建工具会移除掉这些属性，就不会对APK大小或者运行时行为有什么影响。

`tools`命名空间的声明
```xml
<RootTag xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" >
```
## 错误处理属性

- tools:ignore——使用在Lint上面的，忽略Lint警告，使用逗号分隔要忽略的属性

```xml
<string name="show_all_apps" tools:ignore="MissingTranslation">All</string>
```

- tools:targetApi——使用在Lint上面，用来当minSdkVersion没有达到该控件或其子控件api要求时忽略警告。

- tools:locale——用于`<resources>`标签，用来指明说所包含资源的语言环境，避免拼写检查器的报错
```
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:locale="es">
```
## 设计时属性
- tools:任意android命名空间属性
```xml
<Button
    android:id="@+id/button2"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    tools:text="sample text"
    tools:visibility="invisible" />
```
- tools:context
```

```
