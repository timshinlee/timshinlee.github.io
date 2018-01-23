# Constraints
ConstraintLayout的核心就是constraint。

视图编辑器当中四周空心圆表示可设置约束，点击空心圆拖动到其他控件空心圆即可设置约束。点心圆表示已设置约束，普通的约束线使用直线箭头表示。点击圆心可删除该约束。点击左下角x号可删除该控件所有约束。继承于TextView的控件还有个ab按钮可以显示baseline以供设置约束。
![anchor_points](https://constraintlayout.com/assets/images/basics/anchor_points.png)

在XML当中，ConstraintLayout的属性以`app:layout_contraint`开头。
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
可以通过`app:layout_constraintDimensionRatio`属性来设置某个子控件的宽高比，注意此时可变边要设为`match_constraint`，表示满足约束条件的情况下充满布局，这样才能保持宽高比进行变动。
```
android:layout_constraintDimensionRatio="2:1" // 表示宽高比为2:1，默认朝向为水平朝向
android:layout_constraintDimensionRatio="h,2:1" // 表示朝向为竖直朝向，此时宽高比是1:2
```
# Barriers(v1.1 added)
barrier与guideline同样都是用来作为参照物约束控件的，不同的是barrier的位置是基于多个view来决定的。

Barrier的添加同样是通过右键-Helpers-add barrier来添加，可以通过设置`app:barrierDirection`来决定这个barrier是放到控件组的哪个位置，然后通过`app:constraint_referenced_ids`属性来设置控件组包含的控件。在视图编辑器当中可以通过拖动的形式快速添加控件：
![barrier_references](https://constraintlayout.com/assets/images/basics/barrier_references.gif)


### References
[constraintlayout.com](https://constraintlayout.com)
