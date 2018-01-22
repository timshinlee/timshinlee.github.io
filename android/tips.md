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
