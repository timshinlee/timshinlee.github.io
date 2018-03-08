
- Enter transition是一个Activity中的view进入场景的方式
- Exit transition是一个activity中的view退出场景的方式
- shared elements transition是一个Activity中共享元素的变换方式

# Activity变换

Android支持以下的进入退出变换：
- explode：从场景中间进出
- slide：从场景的一边进出
- fade：淡入淡出
- 继承自Visibility的类

实现步骤：
1. 开启window content transitions并设置进出动画。可以在XML中声明或者代码中设置

```xml
<!-- values-v21 -->
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <!-- Customize your theme here. -->
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorAccent</item>

    <!-- 开启window content transitions -->
    <item name="android:windowActivityTransitions">true</item>

    <item name="android:windowEnterTransition">@transition/slide</item>
    <item name="android:windowExitTransition">@transition/explode</item>
</style>

<!-- res/transition/explode.xml -->
<?xml version="1.0" encoding="utf-8"?>
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android">
    <explode/>
</transitionSet>
```

```java
// 如果从代码开启content transition，注意调用Activity和被调用Activity都要开启
getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);

getWindow().setExitTransition(new Explode());
getWindow().setEnterTransition(new Slide());

getWindow().setSharedElementEnterTransition();
getWindow().setSharedElementExitTransition();
```

2. 使用变换开启Activity

```java
// 调用当前Activity的退出动画，如果有设置下一Activity的进场动画也会调用其进场动画
startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this).toBundle());

// 如果想要关闭当前Activity，并且触发退场动画的话，需要调用这个方法
finishAfterTransition();
```

# 共享元素变换
Android支持以下的shared elements变换：

- changeBounds：改变布局边界
- changeClipBounds：改变裁剪边界
- changeTransform：缩放旋转
- changeImageTransform：图片大小缩放

步骤：

1. 启用window content transition，同时指定transition类型，同样也是可以XML和代码
2. 使用`android:transitionName`标识共享元素，或者使用`view.setTransitionName()`
3. 使用`ActivityOptions.makeSceneTransitionAnimation()`进行跳转


1.

```xml
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>


        <item name="android:windowActivityTransitions">true</item>
        <item name="android:windowSharedElementEnterTransition">@transition/image_transform</item>
        <item name="android:windowSharedElementExitTransition">@transition/image_transform</item>
    </style>
```

2.

```xml
<!-- activity_main.xml -->
    <ImageView
        android:id="@+id/icon"
        app:layout_constraintRight_toRightOf="parent"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:transitionName="launcher"
        android:src="@mipmap/ic_launcher_round"
        app:layout_constraintTop_toBottomOf="@id/button"/>

<!-- activity_second.xml -->
    <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        app:layout_constraintBottom_toBottomOf="parent"
        android:transitionName="launcher"
        android:src="@mipmap/ic_launcher_round"/>
```

3.

```java
    final View icon = findViewById(R.id.icon);
    final Intent intent = new Intent(this, SecondActivity.class);
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this,icon,"launcher").toBundle());

	// 多个共享元素时
	startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this, new Pair<>(icon, "launcher"), new Pair<>(icon2, "launcher2")).toBundle());
    }
```

单独设置共享元素动画的时候，activity可能自动设置了淡入淡出动画，可以设置一个空的transitionSet作为activity切换动画来关闭淡入淡出动画

