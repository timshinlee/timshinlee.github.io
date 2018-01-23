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
该图右上角正方形代表当前view，里面的标线有三种类型`>>>`表示`wrap_content`，`|--|`表示fixed固定dp，`|zz|`弹簧型表示`match_constraint`，注意在XML中使用`0dp`表示match_constraint。

如果当前view是`match_constraint`时，可以通过设置`horizontal_weight`来设置权重。在一个连接组当中，所有设置为`match_constraint`的控件会根据权重分配除了`wrap_content`控件之外的所有空间。如果所有控件权重均未设置，则所有控件默认权重为1。如果有控件设置了权重，则其他控件就不会设置默认权重了。
### References
[constraintlayout.com](https://constraintlayout.com)