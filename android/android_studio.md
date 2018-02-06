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
<resources xmlns:android="http://schemas.android.com/apk/res/android"
    tools:locale="es">
</resources>
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

# 构建过程

1. 编译器把源码转为DEX文件（Dalvik Executable），然后把资源转为编译资源
2. APK管理器把DEX文件和编译资源整合成单个APK
3. 使用debug或者release keystore进行签名
4. zipalign

# 构建参数
1. Build Types：定义了构建和打包过程中Gradle使用的参数，例如debug和release。
2. Product Flavors：app不同的版本，例如免费版和收费版，不同的版本可以使用不同的代码和资源，也可以共用这些部分。
3. Build Variants：Build Variants是build type和product flavor的组合，也是Gradle实际上用来构建app的配置。
4. Manifest Entries：可以在build variant配置当中指定一些属性，这些属性用来覆盖manifest文件中的属性。
5. Dependencies
6. Signing：可以通过配置签名属性，来自动签名。debug版本会自动签名debug的key，release版本需要手动指定配置参数。
7. ProGuard：可以为每一种build variant指定一种ProGuard规则。
8. Multiple APK Support：可以为不同的屏幕密度或者ABI（Application Binary Interface）构建不同的apk。

# 配置Gradle
## build.gradle全局变量配置
Project的build.gradle中
```groovy
buildscript {...} // 配置gradle
allprojects {...} // 配置所有项目
ext {             // 配置公共属性，虽说可以配置在module级别，但是会导致无法单独编译
    compileSdkVersion = 26
    supportLibVersion = "27.0.2"
    otherProperty = "123"
}
```
module中的build.gradle进行引用
```groovy
android {
    compileSdkVersion rootProject.ext.compileSdkVersion // 使用rootProject.ext.property_name进行引用
    ...
}

denpendencies {
    compile "com.android.support:appcompat-v7:${rootProject.ext.supportLibVersion}"
    compile "com.android.support:appcompat-v7:$rootProject.ext.supportLibVersion" // 两种都可以，注意使用双引号
}
```
## variant-aware dependency management
Gradle的Android插件3.0提供一个新的依赖库管理机制，也就是app的build variant会对应使用依赖库中相同variant。例如app的debug版将会使用库的debug版，在flavor上也是一样的，app的freeDebug版将会使用库的freeDebug版。

为了让插件能自动匹配variant成功，首先要对所有的flavor进行声明，声明到一个dimension当中。其次要对variant不匹配的情况进行处理。

### 声明dimension
```groovy
// 声明dimension，优先级由高到低，不同的dimension就可以组合生成不同的variant，例如freeminApi23。
flavorDimensions "tier", "minApi"

// 每种flavor都需要声明所属的dimension
productFlavors {
    free {
        dimension "tier"
	...
    }
    paid {
        dimension "tier"
	...
    }
    minApi23 {
        dimension "minApi"
	...
    }

    minApi18 {
        dimension "minApi"
    }
}
```
### 处理variant不匹配情况
当app的variant和依赖库的variant无法相互匹配的时候，就会报错，此时需要根据实际情况进行处理：

- app包含依赖库所没有的build type时，使用matchingFallbacks
```
// app's build.gradle
android {
    buildTypes {
        debug {}
	release {}
	staging {
	    // 列出未匹配到时，要从依赖库中使用的build type，使用匹配到的第一个
            matchingFallbacks = ['debug', 'qa', 'release']
	}
    }
}
```
- app和依赖库都包含同名dimension，但是app的dimension当中包含依赖库不具有的flavor，同样使用matchingFallbacks

```
// app's build.gradle
android {
    defaultConfig {}
    flavorDimensions "tier"
    productFlavors {
        paid {
            dimension "tier"
	    // 已有匹配，不必列出fallback
	}
	free {
            dimension "tier"
	    // 列出未匹配时的备选
	    matchingFallbacks = ['demo', 'trial']
	}
    }
}
```

- 依赖库包含app没有的dimension，则在defaultConfig处使用missionDimensionStrategy，或在具体flavor指定strategy

```
android {
    defaultConfig {
        missingDimensionStrategy 'minApi', 'minApi18', 'minApi23'
	missingDimensionStrategy 'abi', 'x86', 'arm64'
    }
    flavorDimensions "tier"
    productFlavors {
        free {
            dimension 'tier'
	    missingDimensionStrategy 'minApi', 'minApi23', 'minApi18'
	}
	paid {}
    }
}
```

### 本地库的依赖
```
dependencies {
    implementation project(':library') // 本地库依赖不能在前面加build variant，variant-aware会自动识别

    debugImplementation 'com.example.android:app-magic:12.3' // 外部依赖仍然可以指定variant，但是这种方式不推荐
}
```
### 新的依赖关键字

- implementation：该依赖在编译时对其他模块不可见，只在运行时对其他模块可见。一般使用该关键字，因为所依赖的库改变时，只会重新编译该库及直接依赖该库的模块，大大减少构建时间。
- api：这个依赖对于其他模块在编译时也可见，如果该依赖改变的话，所有编译时可以使用到依赖库的模块都会进行编译，所以一般是用在把依赖库分享给分开的测试模块上的情况中。
- compileOnly：Gradle把这个依赖添加到编译路径，但是不会添加到构建输出当中。所以要注意代码中需要检测这个依赖是否可用。可以减少apk大小。
- runtimeOnly：Gradle只把这个依赖添加到构建输出apk当中，不会添加到编译路径。

### 发布依赖
- 变体名称ApiElements
- 变体名称RuntimeElements

< [Migrate to Plugin 3.0.0](https://developer.android.google.cn/studio/build/gradle-plugin-3-0-0-migration.html)

## 应用id
applicationId虽然和包名一样命名风格，默认完全一样，但是应用id和包名其实是独立的，可以单独修改包名，对于应用id没有影响。

相对于包名来说，应用id的命名规则更严格：
- 至少有两部分组成，由句号分隔
- 每个部分必须使用字母开头
- 所有字符必须是[a-zA-Z0-9_]

因为应用id一般与包名一致，所以一些Android API中指的packageName其实是应用id，例如`Context.getPackageName()`。

### 根据build variant修改应用id
可以覆盖声明applicationId，或者使用suffix加在默认id的后面。
```
android {
    defaultConfig {
        applicationId "com.example.myapp"
    }

    productFlavors {
        free {
	    applicationIdSuffix ".free"
	}
	pro {
	    applicationIdSuffix ".pro"
        }
    }

    buildTypes {
        debug {
            applicationIdSuffix ".debug"
	}
    }
}
```

不同的应用id就代表不同的应用，可以安装在同一个设备上。如果是想要对同一个应用进行不同设备配置的区分，则应用id必须一致，只是要区分versionCode。

如果build.gradle当中没有声明applicationId，则会使用AndroidManifest当中的报名座位applicationId，此时修改包名也会修改应用id。

如果要在Manifest文件中引用gradle中定义的applicationId，可以使用`${applicationId}`进行引用。

### 包名修改注意事项
实际项目目录结构必须和Manifest文件中定义的包名一致

```xml
<manifest xmlns:android="http://schemas.android.com/apk/android"
    package="com.example.myapp"
    android:versionCode="1"
    android:versionName="1.0">
```

Manifest中的包名用来给R.java作为命名空间，例如`com.example.myapp.R'，还有给Manifest文件中的类作为命名空间，例如`<activity android:name=".MainActivity">`。

当gradle中定义的applicationId和Manifest中的包名不一致时，首先会使用包名去作为R.java和Manifest的命名空间，然后就使用applicationId去替换包名，最后应用商店和手机识别到的是applicationId。

> [Set the Application ID](https://developer.android.google.cn/studio/build/application-id.html)

## 依赖语法
```
apply plugin: 'com.android.application'

androd {...}
dependencies {
    implementation project(":mylibrary") // 依赖本地library
    implementation fileTree(dir: 'libs', include: ['*.jar']) // 依赖本地目录所有jar包
    implementation files('libs/foo.jar', 'libs/bar.jar') // 依赖本地某些jar包
    implementation 'com.example.android:app-magic:12.3" // 依赖远程库
    implementation group: 'com.example.android', name: 'app-magic', version: '12.3' // 依赖远程库完整写法
}
```

## 优化构建速度
- 为开发时设置不同的配置提高编译速度
```
android {
    ...
    defaultConfig {...}
    productFlavors {
        dev {
            minSdkVersion 21 // 设置为21是为了从命令行构建应用的时候，不会使用到legacy multidex，AS2.3以上不用设置会自动避免使用
	    versionNameSuffix "-dev"
	    applicationIdSuffix '.dev'
	    resConfigs "en", "xxhdpi" // 限制要使用的资源类型，提高编译速度
	}

	prod {
            // 如果在defaultConfig中配置了发布时的配置，质变可以留空，但是仍然要创建这个flavor否则所有的variant都会使用dev flavor。
	}
    }
}
```
