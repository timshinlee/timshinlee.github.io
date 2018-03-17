# 状态栏透明

API 19(4.4 KitKat)通过设置FLAG_TRANSLUCENT_STATUS标志设置透明状态栏：

```xml
<item name="android:windowTranslucentStatus">true</item>
```
或者

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    getWindow().setFlags(LayoutParams.FLAG_TRANSLUCENT_STATUS, LayoutParams.FLAG_TRANSLUCENT_STATUS);
}
```

设置FLAG_TRANSLUCENT_STATUS之后，布局会扩充到状态栏显示，为了让布局为状态栏留出padding，可以设置布局的fitsSystemWindow属性为true。如果设置最外层布局为true，则padding是添加在最外层布局上的，如果设置内部控件的fitsSystemWindow为true，则padding是设置在内部控件上的。

设置透明状态栏后，状态栏图标仍然保持原色，例如白色，这时候在浅色背景上就无法清晰识别到状态栏图标。如果要将状态栏图标设置为灰色，以适应浅色背景，则可以在API 23(Android M)及以上设置：

```xml
<item name="android:windowLightStatusBar">true</item>
```

或者

```java
final View decorView = getWindow().getDecorView();
int uiVisibility = decorView.getWindowSystemUiVisibility();
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    uiVisibility |= View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR;
}
decorView.setSystemUiVisibility(uiVisibility);
```
