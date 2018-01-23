1. `build.gradle`中的signingConfigs用来定义正式版签名，因为build.gradle会包含在版本控制当中，所以不能把签名信息保存到build.gradle，比如下面是**错误**的：
```
signingConfigs {
	release {
		// 错误示范
		storeFile file("myapp.keystore")
		storePassword "password123"
		keyAlias "thekey"
		keyPassword "password789"
	}
}
```
**正确**的做法应该是把签名信息保存到gradle.properties当中，并且gradle.properties不能包含在版本控制当中：
```
// gradle.properties
KEYSTORE_PASSWORD=password123
KEY_PASSWORD=password789

// Gradle会自动在build.gradle中导入gradle.properties的内容，可以直接使用定义的变量
signingConfigs {
	release {
		try {
			storeFile file("myapp.keystore")
			storePassword KEYSTORE_PASSWORD
			keyAlias "thekey"
			keyPassword KEY_PASSWORD
		} catch (ex) {
			throw new InvalidUserDataException("You should define KEYSTORE_PASSWORD and KEY_PASSWORD in gradle.properties.")
		}
	}
}
```
2. 非正式构建使用不同的包名，这样的话debug和release才能同时安装到同个设备上，例如：
```
android {
	buildTypes {
		debug {
			applicationIdSuffix '.debug'
			versionNameSuffix '-DEBUG' // 也可以修改版本名
		}

		release {
			// ...
		}
	}
}
```
如果要区分不同版本使用的资源的话，可以通过`app/src/debug/res`和`app/src/release/res`来实现。
3. 尽量使用style，以用途命名
```
<style name="ContentText">
	<item name="android:textSize">@dimen/font_normal</item>
	<item name="android:textColor">@color/basic_black</item>
</style>
```
style的文件名可以任意命名，并不一定要是`styles.xml`，关键是文件里的`<style>`标签，所以可以通过拆分style文件提高可维护性，比如分出`<styles_home.xml>`等文件。
4. 以特性作为包的区分标准？
5. `colors.xml`仅仅用来作为颜色名到颜色值的映射，不应该用来作为颜色用途的映射，比如下面这样是不建议的：
```
// 不建议
<resources>
	<color name="button_foreground">#FFFFFF</color>
	<color name="button_background">#2A91BD</color>
</resources>
```
因为这样会导致颜色值重复，而且很难去修改基础颜色值。应该把颜色用途定义在对应控件的style当中
```
<resources>
	<!-- grayscale -->
	<color name="white">#FFFFFF</color>
	<color name="gray_light">#DBDBDB</color>
	<color name="gray">#939393</color>
	<color name="gray_dark">#5F5F5F</color>
	<color name="black">#323232</color>

	<!-- basic colors -->
	<color name="green">#27D34D</color>
	<color name="blue">#2A91BD</color>
	<color name="orange">#FF9D2F</color>
	<color name="red">#FF432F</color>
</resources>
```
这样的话可以保持app的颜色统一，不会用到太多种颜色。`colors.xml`不仅仅可以使用红蓝黄等颜色名，还可以使用"brand_primary"、"brand_secondary"、"brand_negative"等来命名。
6. `dimen.xml`也同样定义基础变量，不要根据某个控件具体用途来定义，而是定义该控件的普遍大小。
```
<resources>
	<!-- font sizes -->
	<dimen name="font_larger">22sp</dimen>
	<dimen name="font_large">18sp</dimen>
	<dimen name="font_normal">15sp</dimen>
	<dimen name="font_small">12sp</dimen>

	<!-- typical spacing between two views -->
	<dimen name="spacing_huge">40dp</dimen>
	<dimen name="spacing_large">24dp</dimen>
	<dimen name="spacing_normal">14dp</dimen>
	<dimen name="spacing_small">10dp</dimen>
	<dimen name="spacing_tiny">4dp</dimen>

	<!-- typical sizes of views -->
	<dimen name="button_height_tall">60dp</dimen>
	<dimen name="button_height_normal">40dp</dimen>
	<dimen name="button_height_short">32dp</dimen>
</resources>
```
7. `strings.xml`中字符串的命名应该使用前缀区分同类字符串，比如：
```
// 不建议
<string name="network_error">Network error</string>
<string name="call_failed">Call failed</string>
<string name="map_failed">Map loading failed</string>

// 推荐做法
<string name="error_message_network">Network error</string>
<string name="error_message_call">Call failed</string>
<string name="error_message_map">Map loading failed</string>

// 不建议
<string name="error_message_call">CALL FAILED</string>

// 建议
<string name="error_message_call">Call failed</string>
// 同时在TextView的属性中设置textAllCaps
```
8. Test Frameworks
使用JUnit进行单元测试，使用Espresso进行UI测试，使用AssertJ-Android进行assertion测试。尽量不要使用Robolectric进行测试。

> [futurice android best practices](https://github.com/futurice/android-best-practices)
