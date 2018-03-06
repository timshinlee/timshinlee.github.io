Android的transition框架可以方便地把初始布局中的所有控件通过动画转变到目标布局当中的状态。

transition框架包含以下特点：

- 整组生效：所应用的动画效果对于布局内所有控件都生效。
- 内置动画：框架自带多个动画效果。
- 支持资源文件：可以在资源文件中定义动画效果。
- 动画回调：提供动画和视图层级变化过程回调。

在两个布局间进行动画的步骤如下：

1. 创建初始布局和结束布局的Scene，初始布局可以默认从当前布局生成。
2. 创建一个Transition，指定要使用的动画类型。
3. 调用`TransitionManager.go()`进行动画。

# 创建Scene
## 从布局文件创建
从布局文件中生成Scene对象，需要使用完整的布局文件，可以是`<include>`所指向的布局文件。这种方法适合视图层级相对固定的情况，如果视图层级改变了，Scene对象也需要重新创建。

```xml
// res/layout/activity_main.xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/master_layout">
    <TextView
        android:id="@+id/title"
        ...
        android:text="Title"/>
    <FrameLayout
        android:id="@+id/scene_root">
        <include layout="@layout/a_scene" />
    </FrameLayout>
</LinearLayout>

// res/layout/a_scene.xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/scene_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <TextView
        android:id="@+id/text_view1
        android:text="Text Line 1" />
    <TextView
        android:id="@+id/text_view2
        android:text="Text Line 2" />
</RelativeLayout>

// res/layout/another_scene.xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/scene_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <TextView
        android:id="@+id/text_view2
        android:text="Text Line 2" />
    <TextView
        android:id="@+id/text_view1
        android:text="Text Line 1" />
</RelativeLayout>

// MainActivity.java
Scene mAScene;
Scene mAnotherScene;

ViewGroup sceneRoot = (ViewGroup) findViewById(R.id.scene_root); // SceneRoot中的View才会进行transition。

Scene startScene = Scene.getSceneForLayout(sceneRoot, R.layout.a_scene, this); // 好像无法指定初始Scene
Scene endScene = Scene.getSceneForLayout(sceneRoot, R.layout.another_scene, this); // 结束Scene，框架自动生成动画，从初始Scene转变到到结束Scene，进行必要的移动、隐藏、变形等动画。
```

## 从代码中创建
代码中可以从一个ViewGroup对象创建出一个Scene对象，适用于代码中修改了视图层级，或者动态生成视图层级的情况。

```java
Scene scene = new Scene(sceneRoot); // 只指定了SceneRoot，未指定应用此场景时的变化方式，可以通过setEnterAction(Runnable)和setExitAction(Runnable)进行指定。
Scene scene = new Scene(sceneRoot, layoutView); // API 21支持，进入该场景时，会先移除掉原本sceneRoot所自带的内部布局，使用layoutView作为sceneRoot的内部布局。
```
# 应用变换
```java
// 开始Scene是上次变换的结束Scene，如果还未变换过，则使用当前界面作为开始Scene。
TransitionManager.go(mEndingScene); // 使用默认的AutoTransition
TransitionManager.go(mEndingScene, mFadeTransition);
```

# 自定义scene action
可以通过自定义action来达到对非相同视图层级的View，或者transition框架无法自动动画的View，例如ListView，这两种情况下进行自定义动画。

自定义action也很简单，就是在Runnable中定义具体操作，然后传入`Scene.setExitAction()`和`Scene.setEnterAction()`，退出action会在start scene运行动画之前调用，进入动画会在ending scene进行动画之后调用。

# 设置变换种类
可以使用框架内置变换，例如AutoTransition、Fade，或者使用自定义的变换，然后传递给TransitionManager的go()方法。

## 使用内置变换
代码创建：
```java
// 可以使用预定义的Fade、ChangeBounds、AutoTransition等类
Transition transition = new Fade();
```
XML创建：

- <autoTransition/>：默认变化，以淡出、移动、调整大小、淡入的顺序进行动画
- <fade android:fadingMode="[fade_ind | fade_ out | fade_in_out]"/>：默认是fade_out再fade_in
- <changeBounds/>：移动调整大小
```xml
<!-- 创建res/transition/fade_transition.xml -->
<fade xmlns:android="http://schemas.android.com/apk/res/android" />

<!-- 也可以创建变换组合，下面变换组合和autoTransition效果一样 -->
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android"
    android:transitionOrdering="sequential">
    <fade android:fadingMode="fade_out" />
    <changeBounds />
    <fade android:fadingMode="fade_in" />
</transitionSet>
```

```java
Transition mFadeTransition = TransitionInflater.from(this)
        .inflateTransition(R.transition.fade_transition);
```

# 编辑变换目标列表
transition框架自动对开始场景和结束场景的所有view进行动画，如果需要对场景中需要动画的view进行增加或删除，可以调用`removeTarget()`和`addTarget()`方法。

# 不使用Scene进行变换
以上使用Scene的方式是在改变view hierarchies的情况下进行动画，也可以在当前视图层级下对控件进行增删改，`ViewGroup.addView()`或者`ViewGroup.removeView()`。或者说前后两个视图层级基本一样的时候，就不需要创建两份布局文件，只要在代码中对当前视图层级进行修改就可以了，也就不需要Scene对象了，而是使用delayed transition。延时变化是从当前视图状态开始，记录下view的改变，然后在系统重绘的时候应用变换动画。

具体步骤如下：

1. 当变换事件触发的时候，调用`TransitionManager.beginDelayedTransition()`，传入要变换view的父view，以及想要应用的变换。框架会记录下当前初始状态。
2. 改变子view，框架也记录下改变后的属性。
3. 当系统下一帧重绘的时候，就会使用动画完成变换。

```java
        final TextView textView = new TextView(this);
        textView.setText("Label");

        final ViewGroup root = findViewById(R.id.root);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            TransitionManager.beginDelayedTransition(root); // 下一帧自动生成动画进行变换
        }
        root.addView(textView); // 进行改变，系统会自动生成动画
```

# 监听变换生命周期
Activity可以实现TransitionListener接口的多个生命周期回调，来监听变换的状态。可以用于从开始场景传递参数到结束场景，因为结束场景的视图层级在变换结束才生成，无法在一开始就传过去，可以在变换开始时保存，然后在变换结束时传递过去。

```java
public class MyActivity implements TransitionListener {
    @Override
    public void onTransitionStart(Transition transition) {
        Log.e(TAG, "onTransitionStart: " );
    }

    @Override
    public void onTransitionEnd(Transition transition) {}
    
    @Override
    public void onTransitionCancel(Transition transition) {}

    @Override
    public void onTransitionPause(Transition transition) {}

    @Override
    public void onTransitionResume(Transition transition) {}
}
```

# Transition的限制
- 无法对SurfaceView正常进行变换，因为SurfaceView是从非UI线程进行更新的
- 对TextureView应用的变换可能不正常
- 继承AdapterView的类，例如ListView无法正常使用Transition框架进行变换
- 如果对TextView进行大小调整的动画，文本内容会直接跳到新大小位置上。
