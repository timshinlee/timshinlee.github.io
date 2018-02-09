# Android 4.4 KitKat (API 19)
## 外存权限修改
    读取非本应用外存目录需要权限，也就是说`getExternalStoragePublicDirectory()`需要READ_EXTERAL_STORAGE权限。但是读取本应用外存目录无需权限，也就是`getExternalFilesDir()`
## AlarmManager
    如果targetVersion在19及以上，AlarmManager的`set()`或者`setRepeating()`将会不准确。因为Android为了省电，把临近时间的闹钟都挪到一起了。这个操作只会对targetSdkVersion在19或以上的app生效。

    假如闹钟不是设置在某个时间，而是某个时间段的话，可以使用`setWindow()`，这个方法会接收一个最早闹铃时间，还有一个窗口时间，闹铃会在这个窗口时间中响。

    假如闹钟必须要准确定时的话，需要使用`setExact()`方法。

## Scenes和transitions
场景变换需要以下操作：
1. 指定包含想要改变的UI组件的ViewGroup
2. 指定下一场景的布局
3. 指定变换类型
4. 执行变换

步骤1和2可以使用Scene对象来完成，`Scene.getSceneForLayout()`，Scene对象包含了当前场景的父视图及当前布局。

步骤3和4使用TransitionManager完成，可以通过`TransitionManager.go(yourScene)`。

或者也可以完全不用创建Scene对象，直接调用`beginDelayedTransition()`，指定包含想要改变的ViewGroup。

可以在XML中指定变换详情，每个transition指定场景及变换方式，未指定的话默认使用AutoTransition，可以继承默认的变换方式类创建自己的变换方式：
```xml
// res/transition/
<transitionManager>
    <transition android:fromScene="@layout/transition_scene1"
                android:toScene="@layout/transition_scene2"
                android:transition="@transition/changebounds"/>
    <transition android:fromScene="@layout/transition_scene2"
                android:toScene="@layout/transition_scene1"
                android:transition="@transition/changebounds"/>
    <transition android:toScene="@layout/transition_scene3"
                android:transition="@transition/changebounds_fadein_together"/>
    <transition android:fromScene="@layout/transition_scene3"
                android:toScene="@layout/transition_scene1"
                android:transition="@transition/changebounds_fadeout_sequential"/>
    <transition android:fromScene="@layout/transition_scene3"
                android:toScene="@layout/transition_scene2"
                android:transition="@transition/changebounds_fadeout_sequential"/>

</transitionManager>
```
然后使用`inflateTransitionManger()`生成TransitionManager，用来调用`transitionTo()`，传入XML中包含的一个Scene来执行变换。

## Animator暂停
Animator新增`pause()`和`resume()`及相应监听。

## 可复用bitmap
可以复用BitmapFactory中任何mutable bitmap来解码其他bitmap，即使尺寸不一也没关系，只要新bitmap解码出来的字节数(`getByteCount()`)不大于原先bitmap分配到的字节数(`getAllocationByteCount()`)大小就好了。

可以在BitmapFactory之外复用相同的reconfiguration，用来手动生成bitmap或者自定义解码逻辑。可以使用`setHeight()`和`setWidth()`来设置一个bitmap的宽高，以及通过`setConfig()`来指定一个新的Bitmap.Config，而不会影响到底下的bitmap分配。也可以使用`reconfigure()`来一次性改变这些属性。

但是不能设置当前视图系统正在使用的bitmap，因为底下的像素缓存将无法正常重新分配。

## Storage access framework
之前版本的Android中使用ACTION_GET_CONTENT意图来从其他app获取特定种类文件。这种操作会把文件import到app当中。

4.4引入了一种新的ACTION_OPEN_DOCUMENT意图action，通过这种action可以让用户选择一个特定种类文件，并且授予app长期读取（甚至写入）该文件的权限，而不用把这个文件import到app当中来。

如果是开发一个提供文件储存服务的app，可以通过继承DocumentsProvider来让用户可以从系统统一的UI界面中选择文件。继承DocumentsProvider的类必须包含一个intent filter，接收PROVIDER_INTERFACE action，并实现四个抽象方法：
- queryRoots()：必须返回一个描述了自定义文件存储的所有根目录的Cursor，使用DocumentsContract.Root的列。
- queryChildDocuments()：必须返回一个描述了特定目录下所有文件的Cursor，使用DocumentsContract.Document定义的列。
- queryDocument()：必须返回一个描述特定文件的Cursor，使用DocumentsContract.Document定义的列。
- openDocuments()：必须返回一个代表特定文件的ParcelFileDescriptor。当用户选择一个文件，然后app通过调用`openFileDescriptor()`来访问文件时，系统会调用这个方法。

## External storage access
当存在多个外存的时候，新的`getExternalFilesDirs()`方法返回了多个外存目录，默认第一个是设备基本外存，也是`getExternalFilesDir()`方法返回的目录。需要使用`getStorageState()`方法传入File对象来检测所使用的外存目录是否可用。同样的多了`getExternalCacheDirs()`和`getObbDirs()`方法。

## 沉浸式全屏模式
新增一个SYSTEM_UI_FLAG_IMMERSIVE标志，可以在`getWindow().getDecorView().setSystemUiVisibility()`传入SYSTEM_UI_FLAGE_IMMERSIVE和SYSTEM_UI_FLAG_HIDE_NAVIGATION进入沉浸式全屏模式。用户可以顶部下滑显示系统栏，这样会清空SYSTEM_UI_FLAG_HIDE_NAVIGATION和SYSTEM_UI_FLAG_FULLSCREEN标志（若有）而退出沉浸式。如果要系统栏重新隐藏，则可以使用SYSTEM_UI_FLAG_IMMERSIVE_STICKY标志。

实测小米7.0沉浸式需要IMMERSIVE、HIDE_NAVIGATION、FULL_SCREEN三种标志。下滑显示是系统栏突兀出现的形式，系统栏占用空间。如果是STICKY的话，系统栏显示并不会占用空间。

## 透明系统栏
可以使用新的主题Theme.Holo.NoActionBar.TranslucentDecor和Theme.Holo.Light.NoActionBar.TranslucentDecor。透明系统栏启动的话，布局就会自动充满系统栏部分，所以需要在不想被系统栏遮住的外部ViewGroup启用fitsSystemWindows来创建padding。

如果要在自定义主题中启用透明系统栏，可以继承这两个主题，或者在自定义主题中设置windowTranslucentNavigation和windowTranslucentStatus属性。

## notification listener加强
在4.3中添加了NotificationListenerService，app可以在系统发送新通知的时候接收到相关信息，4.4开始listener可以接收到更多的metadata。新的Notification.extras字段包含一个Bundle，可以传递例如EXTRA_TITLE和EXTRA_PICTURE等内容。还有Notification.Action类可以定义绑定到通知的特性，通过actions去获取。

> [Android 4.4 APIs](https://developer.android.google.cn/about/versions/android-4.4.html)

# Android Lollipop (5.0 Api 21)
新变化：

- Material design：3D视图（实时阴影）、activity transitions、shared elements、ripple animations、 vector drawables in XML、RenderThread
- 性能：ART虚拟机取代Dalvik、支持64位架构
- 通知：通知可以显示在锁屏界面。通知可以添加新的metadata来收集相关的联系人、category和priority。
- Recents界面：重新设计。新的API可以把app内的activity在最近任务界面显示为单独的文件。
- Bluetooth Low Energy(BLE)
- 多网络特性让app可以请求可用网络类型。
- NFC API加强
- 支持OpenGL ES 3.1
- 全新camera2 API
- app支持屏幕截图
- 弃用org.apache.http和android.net.http.ANdroidHttpClient，应该使用URLConnection类。

## 通知
- 使用`setColor()`设置app通知icon背后圆圈的重色（accent color）
- 不要使用Ringtone、MediaPlayer、Vibrator类来作为通知提醒的铃声震动，应该使用Notification.Builder来添加铃声和震动。

RINGER_MODE_SILENT使设备进入新的priority模式，如果修改为RINGER_MODE_NORMAL或者RINGER_MODE_VIBRATE模式则会离开priority模式。

- 5.0上通知直接显示在锁屏界面，为了安全性会删减通知内容。如果要完全显示通知内容或者需要媒体播放控制，使用VISIBILITY_PUBLIC作为`setVisibility()`的参数。如果要自定义内容，可以先设置级别为VISIBILITY_PRIVATE，然后通过Notification.Builder创建替代的通知，再使用`setPublicVersion()`来设置这个替代通知。

- 非锁屏界面下，接收到通知可以以小悬浮窗的形式（heads-up notification）显示在屏幕上方。可以触发heads-up通知的条件包括当前界面全屏、该通知优先级较高且使用铃声或震动。

Notification.Builder可以设置这些新功能：

- setCategory()：当设备进入priority模式，告诉系统如何处理本app的通知
- setPriority()：设置优先级，PRIORITY_MAX和PRIORITY_HIGH的通知如果带有声音或者震动，会以小个悬浮窗形式展现
- addPerson()：添加与通知有关的人，可以让系统按人进行分类，也可以把与特定人相关的通知设为重要。

## 绑定服务
`Context.bindService()`需要传入显式意图，如果传入隐式意图会抛出异常。为了保证app的安全，start或bind Service的时候需要使用显式意图，且不要声明任何过滤条件。

## WebView
对于targetApi在21及以上的，mixed content和第三方cookies默认禁止，可以调用api开启。系统也会自动智能部分加载HTML界面减少内存，可以调用API开启一次性加载。

## 自定义权限
5.0开始如果不同应用使用了相同名称的自定义权限，而且签名不同的话，就无法正常安装第二个应用。尽量不使用自定义权限。

## 最近任务栏
启动Activity的时候传入FLAG_ACTIVITY_NEW_DOCUMENT，可以把这个activity在Recents界面显示为一个单独的task。或者可以通过设置`<activity>`的documentLaunchMode属性为intoExisting或者always。

可以在`<application>`处设置android:maxRecents属性来限制app最大task个数，目前最大可设为50。

一般Recents界面的task在重启后还有，要改变这个特性可以设置`<activity>`的android:persistableMode属性。如果要修改recents界面中activity的显示效果，可以调用`setTaskDescription()`。

## Camera2
使用`getCameraIdList()`来获取可用摄像头，使用`openCamera()`开启摄像机。创建CameraCaptureSession并指定Surface对象来发送照片。

实现CameraCaptureSession.CaptureCallback接收拍照结果。

CameraCharacteristics类可以检测当前摄像头支持的特性，其中INFO_SUPPORTED_HARDWARE_LEVEL代表其功能级别：

- INFO_SUPPORTED_HARDWARE_LEVEL_LEGACY是所有设备都支持的级别，大概功能与弃用的Camera差不多。
- INFO_SUPPORTED_HARDWARE_LEVEL_FULL级别支持控制拍照过程及后续处理，还有拍摄高解析度高帧数图片。

## 存储
5.0扩展了Storage Access Framework让用户可以选择一整个目录的子数，允许应用读写所有包含的文件，不用每个都需要用户确认。

要选择一个目录的子树，构建并发送一个OPEN_DOCUMENT_TREE意图，系统会显示所有支持子树选择的DocumentsProvider，让用户浏览并选中一个目录，返回的UIR就可以访问选中的子树。然后可以通过`buildChildDocumentsUriUsingTree()`和`buildDocumentUriUsingTree()`以及`query()`来访问这个子树。

新的`createDocument()`方法可以用来在子树下创建新的文件或目录，要修改现有文件的话，使用`renameDocument()`和`deleteDocument()`，使用前需要检查一下COLUMN_FLAGS看是否支持。

如果自定义的DocumentsProvider需要支持子树选择，实现`isChildDocument()`并在COLUMN_FLAGS中包含FLAG_SUPPORTS_IS_CHILD。

5.0也在共享存储上引入了一个新的与包绑定的目录，app可以把媒体文件放这个目录中，就会自动包含到MediaStore当中。使用`getExternalMediaDirs()`来获取这个目录，跟`getExternalFilesDir()`获取应用外部私有文件一样，这个操作不需要权限。平台会自动扫描external media dirs的目录来更新媒体库。当然我们也可以手动调用MediaScannerConnection来扫描文件。

## 多网络连接
5.0允许应用动态搜索可用网络并连接，可以用在app需要特定网络的场景下。

## 低功耗蓝牙 Bluetooth Low Energy
5.0支持把Android设备作为BLE的外部设备，就是其他蓝牙设备可以搜索到这个设备。允许Android设备可以广播或者搜索回复并与其他蓝牙设备建立连接。

## NFC提升

## JobScheduler
JobScheduler可以用来安排任务，例如在设备充电、连接到特定网络、设备闲置、在限期前完成等特殊场景。

## 屏幕固定
屏幕固定模式：

- 状态栏为空，通知和状态信息都被隐藏起来
- Home键和Recent Apps键被隐藏
- 其他app无法启动新activity
- 当前app可以打开新activity，只要没有创建新task
- 当一个device owner app开启屏幕固定时，只能由app调用`stopLockTask()`来解除固定
- 当其他非owner app开启屏幕固定时，用户可以按按照提示退出固定模式。

进入固定模式：调用`startLockTask()`。

## IME（Input Method Editor）
5.0开始可以调用`shouldOfferSwitchingToNextInputMethod()`来设置是否支持通过点击输入法的地球图标切换到下一个输入法。

同时还会检测下一个输入法是否支持切换，不会切换到不支持切换的输入法。

# Android 6.0 Marshmallow (API 23)

## Adoptable Storage Devices
用户可以采用外部存储设备，例如SD卡。采用外存设备会加密并格式化这个设备，作为内部存储使用。这样子用户就可以在存储设备间移动app和私人数据。移动app的时候，系统会优先考虑Manifest中的`android:installLocation`设置。

因为app可以在存储设备间移动，所以文件路径可能会动态改变，因此建议实时调用路径API，而不是保存固有路径。

路径API包括：

- Context：
  - getFilesDir()
  - getCacheDir()
  - getCodeCacheDir()
  - getDatabasePath()
  - getDir()
  - getNoBackupFilesDir()
  - getFileStreamPath()
  - getPackageCodePath()
  - getPackageResourcePath()
- ApplicationInfo:
  - dataDir 
  - sourceDir
  - nativeLibraryDir
  - publicSourceDir
  - splitSourceDirs
  - splitPublicSourceDirs

## 通知
- 新的INTERRUPTION_FILTER_ALARMS级别，勿扰模式仅允许闹钟
- 新的CATEGORY_REMINDER用来作为用户安排的提醒事项标准，区别于其他事件CATEGORY_EVENT和闹钟CATEGORY_ALARM
- 新的Icon类用来绑定到通知，通过`setSmallIcon()`和`setLargeIcon()`来调用。`addAction()`改为接收一个Icon对象。
- 新的`getActiveNotifications()`方法判断当前是否有本应用中还存活的通知。

## Camera
`setTorchMode()`在不开启摄像头的情况下开启闪光灯。

访问Camera资源，例如开启和设置摄像头，优先给高优先级应用使用，例如用户可见或者前台程序。低优先级的应用可能会被让出摄像头给高优先级应用，在弃用的Camera上是`onError()`，在Camera2上是`onDisconnected()`。

## 运行时权限管理
## Doze模式和应用待机
- Doze：设备静止未充电且屏幕关闭一段时间后，设备进入Doze模式，设备定时继续一些正常操作。
- App Standby：用户一段时间没有接触某个app后，这个app就进入待机模式。当设备未充电时，系统会关闭待机应用的网络和同步、工作等。

## 移除Apache HTTP库
应该使用HttpURLConnection类。要继续使用Apache HTTP API，需要声明依赖
```
android {
    useLibrary 'org.apache.http.legacy'
}
```
## 通知
要重复更新一个通知的话，复用Notification.Builder实例，调用build()方法来获取更新的Notification实例。

## 文本选择
1. 首先创建ActionMode.Callback2对象
```java
    private ActionMode.Callback2 mActionModeCallback = new ActionMode.Callback2() {
        @Override
        public boolean onCreateActionMode(ActionMode mode, Menu menu) {
            final MenuInflater menuInflater = mode.getMenuInflater();
            menuInflater.inflate(R.menu.main,menu);
            return true;
        }

        @Override
        public boolean onPrepareActionMode(ActionMode mode, Menu menu) {
            return false;
        }

        @Override
        public boolean onActionItemClicked(ActionMode mode, MenuItem item) {
            switch (item.getItemId()) {
                case R.id.menu_action1:
                    Toast.makeText(ChronoActivity1.this, "action1", Toast.LENGTH_SHORT).show();
                    mode.finish();
                    return true;
                default:
                    break;
            }
            return false;
        }

        @Override
        public void onDestroyActionMode(ActionMode mode) {
            mActionMode = null;
        }

        @Override
        public void onGetContentRect(ActionMode mode, View view, Rect outRect) {
            super.onGetContentRect(mode, view, outRect);
        }
    };
```
2. 然后在长按点击事件里调用`startActionMode()`获取漂浮类型的ActionMode（API23）
```java
        helloText.setOnLongClickListener(new View.OnLongClickListener() {
            @RequiresApi(api = Build.VERSION_CODES.M)
            @Override
            public boolean onLongClick(View v) {
                if (mActionMode != null) {
                    return false;
                }
                mActionMode = startActionMode(mActionModeCallback,ActionMode.TYPE_FLOATING);
                v.setSelected(true);
                return true;
            }
        });
```
3. 重写`onGetContentRect()`方法设置内容矩形的坐标
4. 如果矩形定位不合法，而且这是需要invalidate的唯一view，则调用`invalidateContentRect()`。


# Android 7.0 Nougat （API 25）
## 多窗口支持
## 通知升级

- 模板升级：强调新的主图片和头像。
- 消息格式定制更多样化
- 通知分组：可以把通知按照标题等进行分组展示
- 直接回复：在通知界面直接回复短信和信息
- 在通知中使用自定义视图
## Doze模式升级
7.0中，即使设备在移动，只要屏幕关闭一段时间而且未充电，就会进入Doze模式。

## SurfaceView
7.0中SurfaceView在一些情况下耗电表现比TextureView更好，因此从7.0开始建议使用SurfaceView。

## 节省数据模式
设置中可以开启数据节省模式Data saver，ConnectivityManager现在可以获取用户的数据节省模式设置并检测变化。

## Vulkan
7.0整合了Vulkan，这是一个3D渲染API。

## 通知栏快速设置API
新API可以让应用在通知栏快速启动栏中定义按钮。

## 号码拦截
## 同时支持多语言
## 新的emoji
## WebView
从7.0上的51版本Chrome开始，直接设备上的Chrome应用来渲染Android的WebView。可以在开发者选项中修改。
## 直接启动
Direct boot加快了设备启动时间，并且让一些注册的应用在重启后仍然发挥有限的功能，例如闹钟、信息、来电等。

启动后，系统处于限制模式，只能访问设备加密数据，无法访问应用或其他数据。如果有想要在这个模式下运行的应用，则需要在清单中声明一个标志，如此一来这个应用会在重启后被激活，通过发送广播LOCKED_BOOT_COMPLETED，这样这个应用的数据在用户解锁前是可以访问的。其他未注册的数据直到用户解锁才能访问到。

## APK Signature Scheme v2
7.0引入了APK Signature Scheme v2，安装应用速度更快，安全性也更高。如果要关闭，可以在module的build.gradle当中
```groovy
android {
    ...
    defaultCOnfig {...}
    signingConfigs {
        release {
            storeFile file("myreleasekey.keystore")
	    storePassword "password)
	    keyAlias "MyReleaseKey"
	    keyPassword "password"
	    v2SigningEnabled false
	}
    }
}
```
注意v2签名必须在zipalign之后，否则签名会失效。

## 作用域目录访问（Scoped Directory Access）
7.0开始，可以单独请求访问某个特定的外存目录。

## 虚拟文件
7.0在存储访问框架的基础上加上了虚拟文件的概念，让DocumentsProvider可以返回document URI，用于ACTION_VIEW的intent上，即使这个URI并没有直接的字节码表示。

## 背景任务优化

- 目标api为24及以上的，如果CONNECTIVITY_ACTION的广播接收器是注册在Manifest当中的话，将不会接收到这个广播。在代码中注册CONNECTIVITY_ACTION的在Context有效的情况下仍然可以收到。

- 对于所有应用来说，系统都不会发送ACTION_NEW_PICTURE和ACTION_NEW_VIDEO广播了。

## 文件系统权限修改


