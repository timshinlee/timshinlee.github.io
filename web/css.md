CSS(Cascading Style Sheet):
### CSS的格式：
selector { property1: value1; property2: value2; }
设置颜色的三种格式
p { color: #ff0000; }
p { color: rgb(255,0,0); }
p { color: rgb(100%,0%,0%); } 此处百分比号不可省略，如果是0也不能省略。

1. 可以多个元素公用一个选择器
```
h1,h2,h3,h4,h5,h6 {
  color: green;
  }
```

子元素的属性继承自父属性，比如说设置了body的选择器，则包含在body里面的都会默认应用这个选择器。如果要复写掉父类的选择器，可以再定义某个子类的选择器。


2. 派生选择器：就是复合多种条件的选择器，比如说
```
style {
p strong {
color:red;
}
}
<body>
<p>text</p>
<strong>text2</strong>
<p><strong>text3</p></strong> 只有这个满足复合条件
</body>
```
3. id选择器：使用#id名来定义选择器，对应使用元素中的id属性，id属性在同一文档中是不能重复的。
比如定义
#red {color:red;}
#green {color:green;}

使用的时候
```
<p id="red">这个段落是红色。</p>
<p id="green">这个段落是绿色。</p>
```
id可以复合元素种类形成派生选择器，比如说下面的示例就是应用在id名对应元素中的段落。
```
#id名 p {
	font-style: italic;
	text-align: right;
	margin-top: 0.5em;
	}
```
4. 类选择器：使用.类名来定义。例如
.center {text-align: center}	就是定义类名为center的元素的样式，可以应用在下面的示例
注意类名的开头不能是数字，否则Mozilla和Firefox无法识别。
```
<h1 class="center">
This heading will be center-aligned
</h1>

<p class="center">
This paragraph will also be center-aligned.
</p>
```
类选择器也可以综合元素种类指定某个类中的某个元素的样式；
```
.fancy td {
	color: #f60;
	background: #666;
	}
```
这种定义对于类名的fancy的某种元素中包含的td单元格生效，隐含的意思是使用fancy类来选择td元素。
```
td.fancy {
	color: #f60;
	background: #666;
}
```
这种定义对于类名是fancy的td单元格生效，隐含的意思是使用td元素来选择fancy类

5. 属性选择器
规定了！DOCTYPE时，IE7和8才支持属性选择器。

属性选择器的定义：
[title]
{
color:red;
}
这样子只有定义了title属性的元素才会应用这个样式
```
[title=W3School]
{
border:5px solid blue;
}
```
这种是只有定义了titile=W3School属性的元素才会应用这个样式	

不仅可以精准匹配，还可以范围匹配
[title~=hello] { color:red; }
这种是包含hello的就符合要求

具体运算符
[attribute] 用于选取带有指定属性的元素。
[attribute=value] 用于选取带有指定属性和值的元素。
[attribute~=value] 用于选取属性值中包含指定词汇的元素。
[attribute|=value] 用于选取带有以指定值开头的属性值的元素，该值必须是整个单词。
[attribute^=value] 匹配属性值以指定值开头的每个元素。
[attribute$=value] 匹配属性值以指定值结尾的每个元素。
[attribute*=value] 匹配属性值中包含指定值的每个元素。

css可以定义在外部、内部、内联三种
1. 外部：（mystyle.css文件中定义）
hr {color: sienna;}
p {margin-left: 20px;}
body {background-image: url("images/back40.gif");}

在HTML中使用：
```
<head>
<link rel="stylesheet" type="text/css" href="mystyle.css" />
</head>
```
2. 内部，在head标签中定义style标签
```
<head>
<style type="text/css">
  hr {color: sienna;}
  p {margin-left: 20px;}
  body {background-image: url("images/back40.gif");}
</style>
</head>
```
3. 内联，在目标元素中定义style属性

```
<p style="color: sienna; margin-left: 20px">
This is a paragraph
</p>
```
具体到元素上会综合所有符合条件的样式，优先级从内到外降低，相同样式高的会覆盖低的。

### 弹性盒子模型
display规定元素盒子类型。flex表示弹性盒子
```
display:flex;
```
弹性盒子默认是单行显示。如果要变成列的话要flex-direction:column;如果要变成自动换行的话要flex-wrap:wrap;

弹性盒子的各项属性：

---

flex属性是flex-grow、flex-shrink、flex-basis的综合属性
```
flex:1;
```
flex属性用在盒子内的元素，用来指定盒子内的子元素如何分配空间，如果是1的话就是平分。

---
```
flex-grow:1;
```
指定一个盒子子元素相对于其他子元素的比例。实际显示会把该盒子所有空间按flex-grow的比例分配给子元素，如果子元素没有指定flex-grow属性，那就不会显示。


---

```
flex-shrink:3;
```
指定一个盒子元素相对于其他子元素的收缩比例，如果是3就是三分之一。

---

```
flex-basis:40px;
```
说是伸缩基准，但是测试下去好像是最少的长度

---

flex-direction用在盒子父元素上，设置盒子子元素方向，可用属性值包括row（水平显示、默认）、row-reverse、column（竖直显示）、column-reverse
```
flex-direction:row;
```

---

flex-wrap用在盒子父元素上，规定是否自动换行或者自动换列，可以有nowrap（不自动换）、wrap（自动换）、wrap-reverse（自动反向换）

---

flex-flow是flex-direction和flex-wrap的综合属性
```
display:flex;
flex-flow:row-reverse wrap;
```

---

align-content多行未占满空间情况下如何排列每行。可选值包括stretch（拉伸占满）

align-items垂直方向上，容器比子项高时，排列子项的方式，stretch（垂直拉伸填满容器，默认）、center（垂直居中）、flex-start（头部）、flex-end（尾部）、baseline（基线）

align-self用在子项上，设置自身在垂直方向上排列的方式，值和align-items一样，优先级比其高。

justify-content水平方向上，容器比所有子项宽时，排列子项的方式，flex-start（头部）、flex-end（尾部）、center（水平居中）、space-between（空白部分平均分配）、space-around（子项前后空白区域一致，相邻两个就有两个空白区域）、space-evenly(间距一样)

---

弹性盒子中元素居中很方便，只要容器为flex，元素添加margin:auto;即可实现水平竖直两个方向的居中。

---
子元素设置margin:auto;可以把margin自动设置为剩余空间，其余子元素就挤到另一边。

---

order可以设置子元素的显示顺序，小的在前面，可以为负数。

### 组合选择符
- 后代选择符：空格。
例如div p表示选择div中的p
- 子元素选择符：>。例如div>p表示选择div中为直接子元素的p
- 相邻兄弟选择符：+。意思是相邻，而且是兄弟，就是说父元素一样。比如div+p表示选择div后第一个相同父元素中的p。
- 后续兄弟选择符：~。与相邻兄弟选择符差别在于，相邻兄弟只选之后一个，后续兄弟选的是之后所有的。

### position(定位)
定位有四种：static、fixed、relative、absolute

- static：默认的定位方式，元素在正常文档流当中，不受top、right、bottom、left和z-index影响。
- fixed：元素相对于浏览器窗口固定住，即使窗口滚动也不会移动，是脱离了文档流的且钉住窗口的。此时的top、right、bottom、left的值指的是距离上右下左的距离，如果未指该维度定则保持原位置，如果指定了，就是按照该维度相对于浏览器窗口的位置来定位。比如单独指定left:50px或者right:50px，则垂直方向位置不变，水平方向距浏览器左边或者右边50px。同时指定left和right的话，该元素的宽度就是left到right这个范围间。注意此时元素会变成内联元素？因为脱离了文档流，所以其他元素会排列到这个元素原先位置上。如果该元素原本位置在浏览器窗口外，则因为钉住，所以即使滚动浏览器窗口也无法显示出该元素。
- relative：定位方式相对于原先位置进行偏移。元素本身还是处于文档流当中的，所以还占着原先位置，只是进行偏移了。此时的top、right、bottom、left指的是相对于原先位置进行偏移的量。
- absolute：和fixed一样，fixed总结出的规律目前来看都适用于absolute。两者区别在于absolute元素会随着浏览器滚动而移动。还有定位位置的决定，fixed是相对于浏览器窗口，而absolute是根据父容器的position来决定，如果父容器是position:static的，则absolute定位相对于浏览器窗口，如果父容器是非static，则absolute定位相对于该父容器进行定位。

#### 应用：一个层显示在另一个层上
只要上面的层使用absolute定位即可，如果要定义上下顺序，可以下层使用relative定位，然后两个一起定义z-index。

https://zhidao.baidu.com/question/536890674.html

### box-sizing
定义宽高的具体含义，默认值是content-box。

- content-box：width和height包括padding和border。
- border-box：width和heigth不包含padding和border。
- 

### display
display:inline
inline情况下指定宽高无效、上下margin无效、float也无效。

### 内容不超出div
设置overflow:hidden即可。
