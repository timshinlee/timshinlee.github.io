# Autosizing TextView
使用Support library来自动缩放TextView。

- 代码方式：

```java
TextViewCompat.setAutoSizeTextTypeWithDefaults(textView, TextViewCompat.AUTO_SIZE_TEXT_TYPE_UNIFORM);
```

- XML方式：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

  <TextView
      android:layout_width="match_parent"
      android:layout_height="200dp"
      app:autoSizeTextType="uniform" />

</LinearLayout>
```

可以进一步自定义字体自动缩放范围，以及每一步缩放的粒度：

- 代码方式：

```java
// 定义自动缩放范围，unit是TypedValue中的值
TextViewCompat.setAutoSizeTextTypeUniformWithConfiguration(int autoSizeMinTextSize, int autoSizeMaxTextSize, int autoSizeStepGranularity, int unit）；
```
- XML方式：

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView
    android:layout_width="match_parent"
    android:layout_height="200dp"
    android:autoSizeTextType="uniform"
    android:autoSizeMinTextSize="12sp"
    android:autoSizeMaxTextSize="100sp"
    android:autoSizeStepGranularity="2sp" />
```

设置预定义大小，缩放的时候会自动从预定义大小中选择：

- 代码方式：

```java
// 定义预定义大小
TextViewCmpat.setAutoSizeTextTypeUniformWithPresetSizes(TextView textView, int[] presetSizes, int unit)
```

- XML方式：

```xml
<!-- 预定义大小 -->
<?xml version="1.0" encoding="utf-8"?>
<TextView
    android:layout_width="match_parent"
    android:layout_height="200dp"
    android:autoSizeTextType="uniform"
    android:autoSizePresetSizes="@array/autosize_text_sizes" />

<!-- res/value/arrays.xml -->
<resources>
  <array name="autosize_text_sizes">
    <item>10sp</item>
    <item>12sp</item>
    <item>20sp</item>
    <item>40sp</item>
    <item>100sp</item>
  </array>
</resources>
```
