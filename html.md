HTML(HyperText Markup Language)

不要用*通配符，会降低加载效率
不要用background，包含了11种属性。
兼容性问题。

href：Hypertext Reference

为什么script要放在后面？

我们将 <script> 元素放在 HTML 文件底部的原因是，浏览器按照代码在文件中的顺序解析 HTML。如果 JavaScript在最前面被加载，HTML还未加载，JavaScript将无法作用于HTML，所以JavaScript无效，如果 JavaScript 代码出现问题则 HTML 不会被加载。所以将 JavaScript 代码放在底部是最好的选择。

条件语句当中所有不是false, undefined, null, 0, NaN的被当做true。空字符串''也是true。

isNaN()：判断一个值是不是非数字。

### 函数
方法是定义在对象内部的函数。

```
// 函数
function myFunction() {
    alert('Hello');
}

// 匿名函数
function() {
    
}
// 匿名函数的使用
var myButton = document.querySelector('button');
myButton.onclick = function() {
    alert('Hello');
}

// 可以把匿名函数赋值给变量
var myGreeting = function() {
    alert('Hello');
}
// 赋值后函数的使用
myGreeting();

function displayMessage() {
    // ...
}
btn.onclick = displayMessage; // 赋值点击事件，函数不会马上触发。

btn.onclick = displayMessage(); // 函数会马上触发，函数名后面的括号叫做函数调用运算符。


```
### 事件
```
// 添加事件监听，同个控件同个事件可以多次添加
elementName.addEventListener('eventName', functionNameOrAnonymousFunction);
// 移除事件监听
elementName.removeEventListener('eventName',functionNameOrAnonymousFunction);

// onsubmit属性在提交前触发。
// e.preventDefault()阻止表单提交
form.onsubmit = function(e) {
  if (fname.value === '' || lname.value === '') {
    e.preventDefault();
    para.textContent = 'You need to fill in both names!';
  }
}

// 
function bgChange(e) {
  var rndCol = 'rgb(' + random(255) + ',' + random(255) + ',' + random(255) + ')';
  e.target.style.backgroundColor = rndCol;
  console.log(e);
}
```

### DOM
Document Object Model。根据W3C的HTML DOM标准，HTML中的所有内容都是节点：

- 整个文档是一个文档节点
- 每个HTML元素是元素节点
- HTML元素内的文本也是一个节点，叫做文本节点。例如<title>This is title</titile>这里面的文本就是一个单独的节点，不同于title节点。
- 每个HTML属性也是一个属性节点


```
// 根据id获取元素
var element = document.getElementById('intro');
// 根据标签名
var element = document.getElementByTagName('p');
// 根据类名
var element = document.getElementByClassName('custom');
// 插入子节点
appendChild(node);
// 删除子节点
removeChild(node);
// 替换子节点
replaceChild(node);
// 节点元素文本值
node.innerHTML
// 节点元素父节点
node.parentNode
// 节点元素子节点
node.childNodes
// 节点元素的属性节点
node.attributes
```


