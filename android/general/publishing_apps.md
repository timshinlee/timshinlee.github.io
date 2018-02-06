# 在应用中安装应用
- 8.0（API 26）开始，用户必须到系统设置中的Install unknown apps中允许某个app的安装权限，才能从这个app中安装应用。此时需要调用`packageManager.canRequestPackageInstalls()`查询这个应用是否有安装权限，要调用这个API必须是API26，并且声明了REQUEST_INSTALL_PACKAGES权限。

- 7.1.1（API 25）及以下的需要在设置>安全里面开启允许安装未知来源应用选项。 

# 签名
最好对所有APK使用同样的证书进行签名，原因如下：

- 应用升级：同样证书的新版本app才能够正常升级。
- 应用模块化：Android允许使用相同证书签名的应用运行在同个进程当中，这样系统会把这些app当成单个app看待，可以实现应用的模块化。
- 代码/数据分享：同样证书签名的应用可以安全地共享数据，会进行签名检查。

## 隐藏签名关键信息
默认的签名配置：
```groovy
android {
    signingConfigs {
        release {
            keyAlias 'myKey'
	    keyPassword 'myPassword'
	    storeFile file('C:/Users/Administrator/Desktop/test.jks')
	    storePassword '123456'
	}
    }
}
```
抽离关键信息的配置：
```groovy
// 自定义的keystore.properties文件
storeFile=C:/Users/Administrator/Desktop/test.jks
storePassword=123456
keyAlias=123456
keyPassword=123456

// module的build.gradle

def keystorePropertiesFile = rootProject.file("keystore.properties") // 根据真实路径去读取配置文件

def keystoreProperties = new Properties()

keystoreProperties.load(new FileInputStream(keystorePropertiesFile))

android {
    ...
    signingConfigs {
        config {
            keyAlias keystoreProperties.keyAlias
	    keyPassword keystoreProperties.keyPassword
	    storeFile file(keystoreProperties.storeFile
	    storePassword keystoreProperties.storePassword
	}
    }
}
```

## 命令行构建应用并签名
### 创建签名文件
```
// 使用jdk/bin目录下的keytool命令生成key，下列命令创建了一个my-release-key.jks，有效期是10000天。
keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-alias
```

### 构建未签名应用，手动签名
1. 在项目根目录运行命令，会在build/outputs/apk生成my-app-unsigned.apk
```
gradlew assembleRelease
```
2. 使用zipalign进行未压缩资源对齐，生成my-app-unsigned-aligned.apk
```
zipalign -v -p 4 my-app-unsigned.apk my-app-unsigned-aligned.apk
```
3. 使用apksigner进行签名，生成my-app-release.apk：
```
apksigner sign --ks my-release-key.jks --out my-app-release.apk my-app-unsigned-aligned.apk
```
4. 证明APK已经签名：
```
apksigner verify my-app-release.apk
```
