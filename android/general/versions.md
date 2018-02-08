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
新增一个SYSTEM_UI_FLAG_IMMERSIVE标志，可以在`setSystemUiVisibility()`传入SYSTEM_UI_FLAGE_IMMERSIVE和SYSTEM_UI_FLAG_HIDE_NAVIGATION进入沉浸式全屏模式。用户可以顶部下滑显示系统栏，这样会清空SYSTEM_UI_FLAG_HIDE_NAVIGATION和SYSTEM_UI_FLAG_FULLSCREEN标志（若有）而退出沉浸式。如果要系统栏重新隐藏，则可以使用SYSTEM_UI_FLAG_IMMERSIVE_STICKY标志。

## 透明系统栏
可以使用新的主题Theme.Holo.NoActionBar.TranslucentDecor和Theme.Holo.Light.NoActionBar.TranslucentDecor。透明系统栏启动的话，布局就会自动充满系统栏部分，所以需要在不想被系统栏遮住的地方启用fitsSystemWindows。

如果要在自定义主题中启用透明系统栏，可以继承这两个主题，或者在自定义主题中设置windowTranslucentNavigation和windowTranslucentStatus属性。

## notification listener加强
在4.3中添加了NotificationListenerService，app可以在系统发送新通知的时候接收到相关信息，4.4开始listener可以接收到更多的metadata。新的Notification.extras字段包含一个Bundle，可以传递例如EXTRA_TITLE和EXTRA_PICTURE等内容。还有Notification.Action类可以定义绑定到通知的特性，通过actions去获取。

> [Android 4.4 APIs](https://developer.android.google.cn/about/versions/android-4.4.html)
