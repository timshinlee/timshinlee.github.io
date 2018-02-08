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
