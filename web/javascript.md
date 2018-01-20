字符串中使用反斜杠 \ 达到上下两行的字符串内容属于同一个字符串的作用，如果不用反斜杠则分行书写字符串会报错

对象声明方式：
```
var obj = new Object;
// 或者
var obj = new Object();
// 赋值
obj.firstName = "timshin";
obj.lastName = "Tim";
obj.age = 26;
obj.myFun = function(){
    
}

// 直接声明
var obj = {
    firstName : "timshin",
    lastName : "Tim",
    age : 27
}
```
以上方式是在函数中调用，另外一种方式是定义构造函数，构造函数的名称一般大写开头，以跟普通函数区分开来。
```
function Person(){
    this.name="Tim";
    this.age=14;
    this.myFun=function(){
        
    }
}

var person = new Person();
```
构造函数创建对象的方式与一般的面向对象语言的类class不太一样。并不是所有的属性都会复制到新对象上，而是采用原型链的方式，在新对象和类当中去共用属性。

采用普通函数的方式去创建一个person：
```
function createNewPerson(name) {
    var obj = {};
    obj.name = name;
    obj.greeting = function() {
        alert('Hi, I\'m ' + this.name + '.');
    }
    return obj;
}

var john = createNewPerson('John');
john.name;
john.greeting();
```
采用构造函数的方式去创建一个person，构造函数就是JS中的类：
```
function Person(name) {
    this.name = name;
    this.greeting = function() {
        alert('Hi, I\'m ' + this.name + '.');
    }
}

// 调用的时候采用new关键字
var bob = new Person('Bob');
bob.name;
bob.greeting();
```

### 创建对象的方式
1. 直接创建对象
```
var person = {
    name: 'Bob',
    age: 32,
    gender: 'male'
};
```
2. 构造函数
3. 使用Object()构造函数
```
var person1 = new Object();
person1.name = 'Chris';
person1['age'] = 23;
person1.greeting = function() {
  alert();  
};
// 或者直接传入属性
var person2 = new Object({
    name: 'Chris',
    age: 30,
    greeting: function(){}
});
```
4. 使用create()复制对象
```
var person3 = Object.create(person2);
```

### JavaScript Prototype
经典的面向对象中继承是复制父类所有属性的。而在JavaScript当中。

对象的原型是Object.getPrototypeOf(obj)（或者弃用的__proto__属性），每个对象都有对应的原型，这个原型是对象构造函数的prototype属性值，即Object.getPrototypeOf(new Foobar())与Foobar.prototype是指向同个对象。

对象属性的调用过程是：
- 首先在对象自身查找
- 没有的话，查找对象的prototype对象，也就是构造函数的prototype对象
- 再没有的话，查找构造函数的prototype对象的prototype对象。
- 直到找到Object类构造函数的prototype属性，Object类构造函数的prototype属性的prototype是null。

没有一个官方的方法来直接获取原型链中某个对象的原型，只能通过遍历原型链来达成。

至于对象的属性也不是全盘可以继承的，定义在prototype属性中的属性才能被子类继承。

JavaScript中Function也是一种对象。实际上 每个JavaScript函数都是一个Function对象。

注意：JavaScript中一个对象的原型是定义在__proto__属性当中的，而构造函数中prototype属性定义的是这个类中可以被子类继承的属性。

而创建对象的Object.create()方法，实际上是以参数对象为原型去创建新对象，所以
```
var obj2 = Object.create(obj1);
console.log(obj2.__proto__); // 输出的是obj1
```

每个构造函数的prototype属性中都有一个constructor属性，这个constructor属性指向所属的构造函数。所以通过构造函数创建的类实例，都可以使用constructor属性去获取到原本的构造函数：
```
person1.constructor; // 返回Person()构造函数
person2.constructor; // 返回Person()构造函数
```

因为__proto__.constructor指向构造函数，所以可以在constructor后面加上括号作为函数进行调用，此时我们只需要再使用new关键字把这个函数作为构造函数调用即可：
```
var person3 = new person1.constructor('abby',14);
```
这种方法可以用于没有办法调用到构造函数的时候。

可以对构造函数的prototype属性进行修改：
```
Person.prototype.farewell = function(){
    alert('Farewell.');
}
```
修改之后，对于已创建好的Person类实例，去调用farewell()是可行的。这也证明了JavaScript当中的继承并不是单纯从父类复制到子类。而是从子类到父类层层查询调用的属性。

在构造函数的prototype属性中，应该定义一些子类通用的属性或者方法，而属性一般不通用，所以一般会定义一些方法：
```
// prototype的成员属性使用this时会返回undefined（因为this引用的是全局范围，而不是函数范围）
Person.prototype.fullName = this.firstName + ' ' + this.lastName;

// 如果在成员函数中使用this，此时是函数范围，可以转换为对象实例范围，所以使用this可以达到效果
Person.prototype.fullName = function() {
    return this.firstName + ' ' + this.lastName;
}
```

**继承已有对象**
```
function Teacher(name, age, gender, interests, subject) {
    // 继承原有属性
    Person.call(this, name, age, gender, interests, subject);
    // 添加新属性
    this.subject = subject;
}

// 默认的prototype属性为空，所以我们也需要复制prototype属性
Teacher.prototype = Object.create(Person.prototype);
// 复制过来的prototype.constructor仍然指向Person构造函数，所以需要修改成为当前函数
Teacher.prototype.constructor = Teacher;

// 添加prototype属性
Teacher.prototype.greeting = function() {
    // do something
}
```
call()用来在当前上下文中调用一个已定义的函数，call()的第一个变量用来作为被调用函数的this的值。如果被调用函数没有参数，则后面就不需要参数，否则的话在第二第三等等参数去设置。

以上示例传入this表示把Teacher函数传给Person的this属性。

JavaScript中的继承可能叫做委托（Delegation）会更恰当，并不是复制而是特殊的对象把功能委托给通用的对象去完成。

### 变量

JavaScript中变量可以重复声明，如果原来已赋值，新声明未赋值的话，变量还是原来的值。

使用`var`关键字来声明变量，在函数外面声明的变量是全局变量。函数内声明的则为局部变量。
如果未使用`var`声明变量，就把值直接赋给未声明的变量的话，则该变量将自动声明为全局变量，不管在函数内函数外。

如果全局变量是函数内定义的，只有调用了这个函数，才会声明这个全局变量，没有调用这个函数的话全局变量就是undefined。

全局变量都是window这个对象里的变量

== 如果是数值和字符串，字符串所代表数字相等即为true

=== 则需要类型相同

对象属性的访问：
obj.propertyName或者obj["propertyName"]

### 变量提升Hoisting
变量提升：函数声明和变量声明总是会被解释器悄悄地被"提升"到方法体的最顶部。

所以变量可以先使用，再声明。

注意是单单声明的才会有变量提升，如果是声明+赋值的话，就没有变量提升了，先使用会变成undefined。

### 函数
#### 函数声明
正常语法：
```
function functionName(parameters){
    
}
```

---

把匿名函数储存在变量当中
```
var x = function(a,b){return  a* b};
```
这样变量可以当做函数来使用，传入参数
```
var z = x(1,2);
```

---

通过函数构造器来定义匿名函数
```
var myFunction = new Function("a","b","return a * b");
```

#### 函数提升
有变量提升也有函数提升，就是把函数作用域提前到前面，因此函数可以声明之前调用。

#### 自调用函数
格式是(匿名函数声明)();前面一个括号用来表示是函数声明，后面一个括号用来表示是自动调用的。

#### 函数是对象
虽然说使用typeof操作符判断函数类型返回function，但是JavaScript中函数描述为一个对象更为准确，因为JavaScript函数有属性和方法

属性有arguments.length，表示参数个数。

方法有toString()方法将函数作为一个字符串返回。

函数如果是用来作为对象的属性的话，就叫做对象方法。

函数如果是用来创建新的对象，就叫做对象的构造函数。

#### 函数参数
函数定义时指定的参数叫做显式参数，调用时传递的参数叫做隐式参数。

隐式参数的个数可以与显式参数不一致。分两种情况，一种是可以没有显式参数但是有多个隐式参数，这种情况下就没有参数名去引用传递进去的参数了，只能使用arguments对象去调用。第二种可以有显式参数，但是没有隐式参数，这时候参数名对应的值是undefined。

要为某个参数设置默认值有个简便的方式：
```
y = y || 默认值;
```
如果y是undefined就为false，就会等于默认值。

#### 参数传递
JavaScript中隐式参数通过值来传递，在函数中修改了隐式参数的值是对外界无影响的。

但是JavaScript中可以引用对象的值，在函数中修改对象值或者全局变量是影响到外界的。

#### 函数调用
JavaScript函数有四种调用方式，区别在于this的初始化。this关键字指向函数执行时的当前对象。this的值是不能修改的。

---

调用方式1：作为函数直接调用
```
function myFunction(a, b) {
    return a * b;
}
myFunction(10, 2);
```

本身直接调用是不属于任何对象的，但是JavaScript当中是默认的全局对象，在HTML当中默认的全局对象就是HTML页面，而在浏览器当中HTML页面是浏览器窗口（window对象），所以这种方式调用函数属于window对象函数，this指向window对象。也就是说myFunction()和window.myFunction()是一样的。

---
调用方式2：作为对象方法调用

```
var myObject = {
    firstName:"John",
    lastName: "Doe",
    fullName: function () {
        return this.firstName + " " + this.lastName;
    }
}
myObject.fullName();         // 返回 "John Doe"
```
这种情况下是使用匿名函数，这时候this表示该对象，如果引用对象属性需要使用this.去引用

---
调用方式3：使用构造函数调用函数

```
// 构造函数:
function myFunction(arg1, arg2) {
    this.firstName = arg1;
    this.lastName  = arg2;
}
 
// This    creates a new object
var x = new myFunction("John","Doe");
x.firstName;                             // 返回 "John"
```

定义的构造函数中使用this去指定属性，不需要返回值。调用的时候使用new关键字去调用，保存返回值到变量当中，就生成了一个新的对象。构造函数中this是没有任何值的，在new的时候才创建。

---
调用方式4：作为函数方法调用函数

在JavaScript当中函数是对象，有自己的属性和方法。方法中有call()和apply()两个方法用来调用函数，两个方法的第一个参数用来作为调用这个函数的对象，后面接着函数参数。两者区别在于call()逐个接收函数参数，而apply()是把参数放到数组里，使用数组去调用apply()。
```
function myFunction(a, b) {
    return a * b;
}
myObject = myFunction.call(myObject, 10, 2);     // 返回 20

myArray = [10, 2];
myObject = myFunction.apply(myObject,myArray);
```

在作为函数方法调用的时候，严格模式与非严格模式是不一样的。严格模式下，就算第一个参数不是一个对象，this的值仍然是这个参数。而非严格模式下，如果第一个参数是null或者undefined，则this将是全局对象。

#### JavaScript内嵌函数
JavaScript内嵌函数用来解决函数想要操作一个全局变量以便长久保存变量值，但是全局变量是不可控的，可能在函数外被修改的情况。

嵌套函数就是函数里面再定义函数，因为JavaScript当中所有函数都可以访问上一层的作用域，因此普通函数可以调用到全局变量，而嵌套函数可以访问到上层函数的变量。

```
function add() {
    var counter = 0;
    function plus() {counter += 1;}
    plus();    
    return counter; 
}
```

通过嵌套函数的方式，我们就可以达到变量私有的目的，然后要达到变量只初始化一次的效果，可以通过闭包的方式来达到。

闭包就是自调用函数本身定义需要的变量，然后返回另外一个函数，可以叫做子函数，子函数就可以调用到上层函数定义的值了。

```
var add = (function () {
    var counter = 0;
    return function () {return counter += 1;}
})();

add();
add();
add();
```

add变量所代表的函数是自调用函数，会自己先执行一次，初始化counter值，然后返回另一个函数，这样我们调用add()方法的时候就是调用子层函数，自调用函数用来初始化变量。

### JavaScript中的短路逻辑运算符&&和||
对于Java来说，&&是有真即真，无真为假，前真后不看；||是有假为假，全真为真，前假后不看。即能确定真假时，直接返回真假，剩下的就不看了。

对于JavaScript来说，返回的是确定真假的那一部分。

&&就是前真的话，真假由后面决定，所以返回后；前假的话直接返回前。

||就是前真的话，直接返回前；前假的话，真假由后面决定，返回后。

JavaScript中0和null和undefined为false，其他都是true。

### var、let和const
**var**的作用域是整个函数，函数范围内可以重复使用var去声明同名变量。作用域的意思就是整个函数内该名称变量只有一个值。

```
function varTest(){
	var x = 1;
	if (true){
		var x = 2;
		console.log('x inside = ' + x);
		// var x是函数级变量，此时x=2
	}
	console.log('x outside = ' + x)
	// var x是函数级变量，此时x=2
}
```


**let**是块级作用域的本地变量，子块的声明可以覆盖上级块的声明，例如：

```
function letTest(){
	let y = 1
	if (true){
		let y = 2
		// let y是块级变量，判断语句内的y会覆盖判断语句外的y，此时y=2
		console.log('y inside = ' + y)
	}
	console.log('y outside = ' + y)
	// 此处作用域超出了判断语句范围，因此y=1
}
```

以let声明后的变量在作用域内使用var重复声明会报错。

**const**也是块级作用域，与let区别在于不能重复声明，也不能重复赋值，而且声明的时候就必须赋值
初始化。

### 判断变量类型

```
var foo
console.log(foo.constructor == Array)
console.log(foo.constructor == String)
```

### 判断是否含有中文
```
if (escape(str).indexOf("%u") < 0) {
    // 不含中文
}
```
### void 0
void 0 等于undefined。
其实void本身就等于undefined，不过有时候void会被赋值就不是undefined了，而void 0一直等于undefined。

### 转义字符
```
var bigmouth = 'I\' got no right to take my place.';
```
\表示把后面的字符转义掉，不作为代码的关键字或者符号，而是作为普通文本。比如`\'`就是单纯的`'`，不是用来包含字符串的单引号，双引号同理。

### Number
JavaScript当中数字的类型都是number。数字字符串转数字使用Number(数字字符串)。

### String
字符串长度：strName.length

字符串中第n个字符：strName[n]

字符串中包含某个字符串的位置：str1.index(str2)

获取[i,j)的子字符串，j可省略：str1.slice(i[, j]);

更改大小写：strName.toLowerCase()  strName.toUpperCase()

替换指定字符串生成新字符串，原字符串不变：strName.replace(targetSubStr, substitudeStr)

按指定字符把字符串分割成数组：
```
var arr = strName.split(',');
```

按指定字符把数组组合成字符串：
```
var str = arrName.join(',');
```

直接使用逗号把数组组合成字符串：
```
var str = arrName.toString();
```

### Array
```
arrName.push(someItem); // 数组末尾添加元素
var removedItem = arrName.pop(); // 数组末尾移除元素

arrName.unshift(someItem); // 数组开头添加元素
var removedItem = arrName.shift(); // 数组开头移除元素
```

### 对象属性调用
person.age或者person['age']都可以

但是person.只能调用已有的成员变量，person[]可以调用变量名，比如
```
var newDataName = 'data_key';
var newDataValue = 'data_value';

person[newDataName] = newDataValue;
// 使用变量作为变量名，点表示法只能使用常量作为变量名。
```

### XMLHttpRequest(XHR)
XHR是JavaScript中一个用来进行网络请求的API
```
var requestURL = 'https://mdn.github.io/learning-area/javascript/oojs/json/superheroes.json';

var request = new XMLHttpRequest();

request.open('GET', requestURL);

request.responseType = 'json';

request.send();

request.onload = function() {
  var result = request.response;
  // 因为指定了responseType为json，所以result已经是解析过的对象了
}
```

### DOM(Document Object Model)
```
navigator.xxxx
window.innerWidth   window.innerHeight
document.xxxx

// 获取元素
var sect = document.querySelector('section');
// 创建元素
var para = document.createElement('p');
para.textContent = 'Hello world';
// 添加元素
sect.appendChild(para);
// 创建文本节点
var text = document.createTextNode('This is text content.');
para.appendChild(text);
// 已知父节点移除其子节点
para.removeChild(text);
// 节点无法移除自身，只能通过父节点去移除
text.parentNode.remove(text);

// 操作CSS HTMLNodeName.style，注意CSS属性名原生写法是以-连接，DOM中是驼峰写法
text.style.color = 'white';
text.style.textAligh = 'center';

// 动态添加属性
<style>
    .highlight {
        color: white;
    }
</style>
text.setAttribute('class', 'highlight');
```
