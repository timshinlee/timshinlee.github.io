CONNECTIVITY_ACTION这个广播对于target API >= 24(7.0)的应用需要代码注册，在manifest中注册无效。

如果应用target API >= 26(8.0)，则大部分的隐式广播都需要在代码中注册，不能在manifest中注册。大多数情况下应该使用JobScheduler。

# 接收广播
## 清单声明广播接收器
在清单中声明了广播接收器时，在安装应用的时候，系统会注册这个接收器，如果接到符合条件的广播，应用会被调起。对于每一条接收到的广播，系统都会创建一个新的BroadcastReceiver实例，该实例只在onReceive()期间有效，当onReceive()方法返回的时候，进程就可能会结束掉。

```xml
<receiver android:name=".MyBroadcastReceiver" android:exported="false">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
	<action android:name="android.intent.action.INPUT_METHOD_CHANGED" />
    </intent-filter>
</receiver>
```
`android:exported`属性决定了是否能接收外部广播
```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    private static final String TAG = "MyBroadcastReceiver";
    @Override
    public void onReceive(Context context, Intent intent) {
        // to do
    }
}

```

## 代码注册
```java
BroadcastReceiver br = new BroadcastReceiver();

IntentFilter filter = new IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION);
filter.addAction(Intent.ACTION_AIRPLANE_MODE_CANGED);
// this.registerReceiver(br, filter);
LocalBroadcastManager.registerReceiver(br, filter);
```
代码注册的广播接收器在注册上下文的范围内有效，注意要在对应的生命周期中注销。

---

BroadcastReceiver的运行状态影响了所在进程存活的优先级，例如onReceive()方法正在运行的时候，该进程作为前台进程保留，除非内存极度不足，否则不会被终止掉。但是onReceive()方法运行在主线程中，不能进行耗时操作，如果要进行耗时操作需要在子线程中进行。

但是当onReceive()返回后，BroadcastReceiver就不再活跃了，所在进程的优先级就由其他组件决定了。

所以不能从BroadcastReceiver中开启长时间运行的子线程，当onReceive()返回后，这个进程如果没有其他组件存在，就是低优先级进程，随时会被终止掉，开启的子线程也会被终止掉。

这时候可以调用`goAsync()`方法来延长时间（注意要低于10s），在子线程处理。或者使用JobScheduler开启一个JobService，来保持进程优先级。

使用goAsync()：
```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    private static final String TAG = "MyBroadcastReceiver";

    @Override
    public void onReceive(final Context context, final Intent intent) {
        // 调用goAsync生成一个PendingResult对象，可以把PendingResult发送到子线程中进行处理，处理完毕必须调用pendingResult.finish()结束广播。
        final PendingResult pendingResult = goAsync();
	AsyncTask<String, Integer, String> asyncTask = new AsyncTask<String, Integer, String>() {
            @Override
	    protected String doInBackground(String... params) {
                // operations
		...
		pendingResult.finish();
		return data;
	    }
        };
	asyncTask.execute();
    }
}
```

# 发送广播
- `sendOrderedBroadcast()`：有序广播，按优先级逐个发送广播，上个接收者可以给下个接收者发送信息，或者拦截广播。
- `sendBroadcast()`：正常广播，无序发送广播给所有接收器。
- `LocalBroadcastManager.sendBroadcast()`：本地广播，不需要跨进程通信所以更高效，无需担心安全问题。

```java
Intent intent = new Intent();
intent.setAction("com.example.broadcast.MY_NOTIFICATION");
intent.putExtra("data", "Notice me senpai!");
sendBroadcast(intent);

// 可以通过调用intent.setPackage()来限制可以接收广播的包名
```
# 权限控制
## 发送设置权限
可以在发送广播时添加权限，则接收方需要具备指定权限才能接收到广播
```java
sendBroadcast(new Intent("com.example.NOTIFY"), Manifest.permission.SEND_SMS);
```
接收方需要具备相应权限：
```xml
<uses-permission android:name="android.permission.SEND_SMS" />
```


## 接收设置权限
或者当接收方指定了权限时，发送方需要获取指定权限才能发送给接收方

接收方：
```xml
<receiver android:name=".MyBroadcastReceiver"
          android:permission="android.permission.SEND_SMS">
    <intent-filter>
        <action android:name="android.intent.action.AIRPLANE_MODE"/>
    </intent-filter>
</receiver>
```
或者
```java
IntentFilter filter = new IntentFilter(Intent.ACTION_AIRPLANE_MODE_CHANGED);
registerReceiver(receiver, filter, Manifest.permission.SEND_SMS, null);
```

则发送方需要获取相应权限：
```xml
<uses-permission android:name="android.permission.SEND_SMS" />
```

---

注意不要从BroadcastReceiver中启动Activity，可以考虑展示一个Notification。
