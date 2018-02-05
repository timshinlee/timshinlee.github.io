# Android Studio配置
AS提供两个设置文件：
- studio.vmoptions:定制AS的JVM属性，比如堆大小和缓存大小。
- idea.properties:配置AS的属性，例如插件文件夹路径或者最大支持的文件大小。
这两个文件储存在AS的配置文件夹当中，配置文件夹是`%USERPROFILE%\.<AS版本号>`。WINDOWS中默认的`%USERPROFILE%`是`C:\Users\Administrator`。

下面给出一些常用的配置属性，注意修改后需要重启AS才能生效：
```
-Xmx2g	// Maximum heap size 2G
```
要查看默认参数及当前生效参数，可以使用`jps -lvm`命令查看。

> [Configure the IDE](https://developer.android.google.cn/studio/intro/studio-config.html#customize_ide)

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

```xml
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
用来设置XML布局文件对应的Activity或Fragment，方便使用对应界面的主题进行预览，或者检测点击事件。但是AS3.0以上好像不需要设置这个属性也可以了。

```xml
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context=".MainActivity" >
```

- tools:itemCount
使用在RecyclerView，设置预览的个数

- tools:layout
使用在`<fragment>`中，指定该Fragment控件要预览的布局

```xml
<fragment android:name="com.example.master.ItemListFragment"
    tools:layout="@layout/list_content" />
```

- tools:listitem / tools:listheader / tools:listfooter
使用在`<AdapterView>`或者其子类，例如`<ListView>`，指定预览布局

- tools:showIn
使用在被include的布局上，这样可以在使用include的布局中预览到被包含的布局

```xml
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:text="@string/hello_world"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    tools:showIn="@layout/activity_main" />
```

- tools:menu
用在根布局当中，用来指定当前app bar所要显示的菜单，可以指定多个menu id，使用逗号分隔，不需要`@menu`以及`.xml`后缀

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:menu="menu1,menu2" />
```
- tools:minValue / tools:maxValue
使用在`<NumberPicker>`上，设置最大最小值预览
- tools:openDrawer
使用在`<DrawerLayout>`上，设置预览中打开抽屉及其方向

```xml
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:openDrawer="start" />
```

- `@tools:sample/*` 资源
引用AS内置的sample资源，例如

```xml
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    tools:text="@tools:sample/cities" />
```

## 资源缩减属性
- tools:shrinkMode
- tools:keep
- tools:discard

> [Tools Attributes Reference](https://developer.android.google.cn/studio/write/tool-attributes.html)

