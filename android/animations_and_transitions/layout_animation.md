Android框架提供了一个布局改变自动进行动画的功能

步骤：

1. 开启animateLayoutChanges

```xml
<LinearLayout android:id="@+id/container"
    android:animateLayoutChanges="true"
    ...
/>
```

2. 代码中动态添加、更新或删除布局内的控件

```java
private ViewGroup mContainerView;

private void addItem() {
    View newView;
    ...
    mContainerView.addView(newView, 0);
}
```

3. 如果要自定义布局动画，可以创建一个LayoutTransition对象，传入`viewGroup.setLayoutTransition()`当中

ViewGroup默认自带一个LayoutTransition对象，默认开启的是内部控件增删的效果，如果要开启内部控件大小改变的效果，可以启用LayoutTransiton.CHANGING

```java
final LayoutTransition layoutTransition = mLinearLayout.getLayoutTransition();
layoutTransition.enableTransitionType(LayoutTransition.CHANGING);
```

