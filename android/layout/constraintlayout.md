# Constraints
ConstraintLayout的核心就是constraint。

视图编辑器当中四周空心圆表示可设置约束，点击空心圆拖动到其他控件空心圆即可设置约束。点心圆表示已设置约束，普通的约束线使用直线箭头表示。点击圆心可删除该约束。点击左下角x号可删除该控件所有约束。继承于TextView的控件还有个ab按钮可以显示baseline以供设置约束。
![anchor_points](https://constraintlayout.com/assets/images/basics/anchor_points.png)

在XML当中，ConstraintLayout的属性以`app:layout_contraint`开头。

# Positioning
## Centering
设置相反方向上的相反约束，即可实现在ConstraintLayout或者**其中某个控件**中居中
```
    <TextView
        android:id="@+id/textView5"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="TextView"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"/>
```
居中后可以通过设置`app:layout_constraintHorizontal_bias="0.4"`属性来修改偏移量，0.4表示距离左边40%。

## Circular positioning(added in 1.1)
圆形定位，可以把某个View设置为位于另外一个View的圆周上某个位置
```
<Button android:id="@+id/buttonA" ... />
<Button android:id="@+id/buttonB" ...
    app:layout_constraintCircle="@id/buttonA"
    app:layout_constraintCircleRadius="100dp"
    app:layout_constraintCircleAngle="45" />
```
## Gone margins
在ConstraintLayout当中，如果一个View是gone的，则他本身的margin就会为0，但是约束关系不会消失。

如果A View约束到的B View是gone的，则可以设置该情况下A View的margin：
```
app:layout_goneMarginStart // 等其他方位
```

# Chains
chain是一种特殊的约束，可以设置一组view间的空间分布。

创建chain只需要选中多个view，右键设置水平连接或者垂直连接
![chains_create](https://constraintlayout.com/assets/images/basics/chains_create.gif)

创建完成的连接组，组内控件间使用锁链表示连接，头尾控件与父布局使用折线表示空间可调整，组内控件右下角多出一个锁链图标，点击可在`spread`,`spread_inside`和`packed`三个模式间切换
![chains_create](https://constraintlayout.com/assets/images/basics/chains_create.png)
- spread: 多余空间平均分配。
- spread inside: 首尾控件顶格到父布局边缘，内部控件多余空间平均分配。
- packed: 所有控件集中到一起（内部可设置margin），然后根据bias设置在父布局的位置，默认0.5为居中。点击滑块可调整位置，该滑块只在packed模式下生效。
![chains_packed_bias](https://constraintlayout.com/assets/images/basics/chains_packed_bias.gif)

该图右上角正方形代表当前view，里面的标线可点击切换，共有三种类型：
- `>>>`表示`wrap_content`
- `|--|`表示fixed固定dp
- `|zz|`弹簧型表示`match_constraint`，注意在XML中使用`0dp`表示match_constraint。这点与LinearLayout类似。

如果当前view是`match_constraint`时，可以通过设置`horizontal_weight`来设置权重。在一个连接组当中，所有设置为`match_constraint`的控件会根据权重分配除了`wrap_content`控件之外的所有空间。如果所有控件权重均未设置，则所有控件默认权重为1。如果有控件设置了权重，则其他控件就不会设置默认权重了。

注意权重只能使用在`spread`或者`spread_inside`模式当中才会生效，在`packed`模式当中会变成宽度为0。

实际XML中chain的形成是通过相互约束达成的：
```
    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="TextView"
        app:layout_constraintEnd_toStartOf="@id/textView2"
	app:layout_constraintHorizontal_chainStyle="spread_inside"
        app:layout_constraintStart_toStartOf="parent"/>

    <TextView
        android:id="@+id/textView2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="TextView"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@+id/textView"/>
```
即view1右边约束在view2左边，view2左边约束在view1右边，形成了连接，然后可以在连接开头元素定义连接类型。
# Guidelines
Guideline就是实际界面中不可见的，用来对齐控件的控件。在视图编辑器中添加：
![guideline_create](https://constraintlayout.com/assets/images/basics/guideline_create.gif)

点击蓝色圆形按钮可以切换guideline的类型，包括
- begin: 距离起始边固定dp距离
- end: 距离终止边固定dp距离
- percenter: 父容器固定百分比处

使用guideline可以用来给其他view作为约束的参照物，即把约束设置到guideline上，这样子只要移动guideline的位置，其他所有绑定的view都会随之移动了。

Guideline主要用于相对于父容器固定位置的情况下，比如相对左边多远，相对右边多远，或者处于容器中哪个固定位置。对于相对于某一组控件的位置的话，用Barrier比较合适。

# Dimensions
## ConstraintLayout最小宽高、最大宽高
当ConstraintLayout本身是WRAP_CONTENT时，可以通过设置`android:minWidth``android:maxWidth`等确定最小最大宽高
## 子控件宽高比
可以通过`app:layout_constraintDimensionRatio`属性来设置某个子控件的宽高比，注意此时可变边要设为`match_constraint`，表示满足约束条件的情况下充满布局，这样才能保持宽高比进行变动。
```
android:layout_constraintDimensionRatio="2:1" // 表示宽高比为2:1

android:layout_constraintDimensionRatio="2" // 表示w:h为2

android:layout_constraintDimensionRatio="h,2:1" // 宽高比是1:2，此时宽不变高变。如果高已经是wrap_content或者固定了，而且宽是0dp的话，就会以宽变来满足宽高比。
```
也可以把两边都设为MATCH_CONSTRAINT，此时会在满足约束以及宽高比的条件下，把控件设置为最大宽高。
## 子控件宽高
当子控件设置为WARP_CONTENT时，表示约束并不会限制该控件的宽高。此时如果要让约束仍然可以限制该控件的宽高，可以通过`app:layout_constrainedWidth="true"`来约束宽度。

当子控件设置为MATCH_CONSTRAINT时，表示该控件会占据所有可用空间，此时可以设置以下几个属性设置最大最小或者所占空间百分比
```
layout_constraintWidth_min和layout_constraintHeight_min：可以指定固定dp值，或者指定为wrap，即与wrap_content一致。
layout_constraintWidth_max和layout_constraintHeight_max：同上
layout_constraintWidth_percent和layout_constraintHeight_percent设置百分比，注意需要先设置宽或者高为0dp（MATCH_CONSTRAINT)
```
# Barriers(added in 1.1)
barrier与guideline同样都是用来作为参照物约束控件的，不同的是barrier的位置是基于多个view来决定的。

Barrier的添加同样是通过右键-Helpers-add barrier来添加，可以通过设置`app:barrierDirection`来决定这个barrier是放到控件组的哪个位置，然后通过`app:constraint_referenced_ids`属性来设置控件组包含的控件。在视图编辑器当中可以通过拖动的形式快速添加控件：
![barrier_references](https://constraintlayout.com/assets/images/basics/barrier_references.gif)
# Group(added in 1.1)
Group是用来批量控制visibility的控件，添加方式和操作方式类似barrier。组内的控件会随着Group控件的visibility改变而改变。同一个控件被包含在多个组的时候，其visibility由最后声明的Group来决定。
```
    <android.support.constraint.Group
        android:id="@+id/group"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:constraint_referenced_ids="imageView,textView2"/>
```
# Placeholder
Placeholder是用来给某个View进行定位的，当调用`holder.setContentId()`时，传入id对一个的View就会移动到Placeholder的位置来，移动后Placeholder就gone掉了，剩下content view。

Placeholder默认是不可见的，可以通过`setEmptyVisibility()`来设置无内容时的可见性。
```
    <android.support.constraint.Placeholder
        android:id="@+id/holder"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_width="100dp"
        android:background="@color/colorPrimary"
        android:layout_height="100dp"/>
```
# Tips
## Setting background without adding ViewGroup
给一组View添加背景的普遍做法是把这组View放到一个ViewGroup当中，但是这样会增加布局层级，降低渲染效率。使用ConstraintLayout的话就可以通过把背景View的左右设置到这些View中的最左最右，上下设置到最上最下，包裹住这些View达到添加背景的效果：
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout 
  xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:app="http://schemas.android.com/apk/res-auto"
  xmlns:tools="http://schemas.android.com/tools"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:background="#AAA">

  <View
    android:id="@+id/background"
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:background="#FFF"
    app:layout_constraintBottom_toBottomOf="@+id/textView3"
    app:layout_constraintEnd_toEndOf="@+id/textView1"
    app:layout_constraintStart_toStartOf="@+id/textView1"
    app:layout_constraintTop_toTopOf="@+id/textView1" />

  <TextView
    android:id="@+id/textView1"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    android:padding="8dp"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent"
    tools:text="TextView" />

  <TextView
    android:id="@+id/textView2"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:padding="8dp"
    app:layout_constraintEnd_toEndOf="@+id/textView1"
    app:layout_constraintStart_toStartOf="@+id/textView1"
    app:layout_constraintTop_toBottomOf="@+id/textView1"
    tools:text="TextView" />

  <TextView
    android:id="@+id/textView3"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:padding="8dp"
    app:layout_constraintEnd_toEndOf="@+id/textView1"
    app:layout_constraintStart_toStartOf="@+id/textView1"
    app:layout_constraintTop_toBottomOf="@+id/textView2"
    tools:text="TextView" />
</android.support.constraint.ConstraintLayout>
```

### References
> [constraintlayout.com](https://constraintlayout.com)
