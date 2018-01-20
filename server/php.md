PHP 脚本可以放在文档中的任何位置。
PHP 脚本需要放在<?php ?>代码块中

```
<?php
// PHP 代码
?>
```

PHP 文件的默认文件扩展名是 ".php"。
PHP 文件通常包含 HTML 标签和一些 PHP 脚本代码。

PHP 中的每个代码行都必须以分号结束。

通过 PHP，有两种在浏览器输出文本的基础指令：echo 和 print。

### 变量
变量定义和使用的时候都必须在前面加$，变量名区分大小写

PHP 没有声明变量的命令。
变量在您第一次赋值给它的时候被创建

PHP变量作用范围有四种：
- local：函数内定义的变量，只能在函数内部使用
- global：函数外定义的变量是global变量，可以在函数外进行使用，如果要在函数内使用需要使用global关键字声明一下。使用global进行声明的同时不能赋值，会报错。因为实际上是独立的，所以在函数内可以声明同名变量并赋值。

这样子global挺麻烦的，另外一种在函数内访问全局变量的方式是通过$GLOBALS[variableName]，比如$GLOBALS['x']这样子。

- static：static用来修饰函数中的局部变量使其在函数执行完毕不被删除。根据理解是static赋值语句赋值后，之后都不会再执行这个赋值语句了，下次执行这个函数，使用之前的变量值。
- parameter：参数变量是在函数括号中声明的参数名，在函数范围内可以使用，类似java的形式变量。
- 
```
<?php
    function add($x) {
        echo $x;
    }
    add(5);
?>
```


echo 和 print 区别:
echo - 可以输出一个或多个字符串，多个值用逗号分隔
print - 只允许输出一个字符串，返回值总为 1
提示：echo 输出的速度比 print 快， echo 没有返回值，print有返回值1。

echo和print 是一个语言结构，使用的时候可以不用加括号，也可以加上括号： echo 或 echo()。因为是语言结构不是函数，所以不能用在表达式当中。

可以在字符串中直接使用变量，输出时输出对应的变量值，比如
```
<?php
    $x = '晴天';
    echo "今天天气是$x";
?>
```
会输出今天天气是晴天，注意此时要双引号才行，单引号会纯粹输出原内容不会替换。

var_dump() 函数打印变量的数据类型和值，如果是数组会把每个元素打印出来

print_r(obj) 打印human-readable的格式，数组可以显示内容

NULL是种数据类型，对应的值是null。

定义常量：
define ("常量名",常量值);
然后使用的时候只要直接使用常量名就好了，跟变量不一样，不需要$前缀。
常量是全局的，如果在函数外定义，可以直接在函数内使用，如果在函数外定义，可以在函数调用后在函数外使用。
常量默认区分大小写，如果要不区分大小写，则需要在定义的时候，添加一个参数为true。

### 操作符
字符串连接符.

strlen(字符串)：返回字符串长度
中文字符一个占3个字符长度，可以调用指定编码的方式进行长度衡量：
mb_strlen(字符串，"utf-8");
这种情况下中文字符一个占一个字符长度。

strpos(完整字符串，要查找的字符串)：返回所查找字符串的位置，如果没找到返回空

a**b：求a的b次方

php中/是求商，带小数的，跟java不一样。php中整除是调用intdiv(a,b)方法，这个是PHP7新增的方法。

++ -- 这个跟java一样。

== 等于，类型可以不同
=== 绝对等于，类型必须相同
!=不等于，两者不相等，可能类型不同但是相等。
<>不等于
!==不绝对等于，两者不相等，或者相等但类型不同

比较结果是bool类型，true或false，如果去echo的话，true打印为1，false不打印。需要用var_dump()才能显示为bool类型。

逻辑运算符：
与：
and
&&

或：
or
||

非：
!

异或：
xor


java中没有的数组运算符：
数组a，数组b

a + b：a和b的集合

a == b：a和b具有相同键值对时为true

a === b：a和b不仅具有相同键值对，而且顺序相同类型相同时，则返回true

a != b:不相等

a <> b:不相等

a !== b:不恒等

三元运算符：
expression1 ? expression2 : expression3
php5.3开始，可以省略中间的表达式，变成
expression1 ?: expression3
这种情况下，只要expressiont1非空或者不==0，就会返回expression1，否则返回expression3

php7新增一个NULL合并运算符
expression1 ?? expression2
只要expression1非空，就返回expression1的值，否则返回expression2


PHP7+ 支持组合比较符，实例如下：
```
<?php
// 整型
echo 1 <=> 1; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1
 
// 浮点型
echo 1.5 <=> 1.5; // 0
echo 1.5 <=> 2.5; // -1
echo 2.5 <=> 1.5; // 1
 
// 字符串
echo "a" <=> "a"; // 0
echo "a" <=> "b"; // -1
echo "b" <=> "a"; // 1
?>
```
组合运算符的意思是如果两个变量相等返回0，大于返回1，小于返回-1。这个运算符可以节省比较复杂的判断。

优先级 
&&和||大于赋值运算符，赋值运算符大于and和or

所以
```
$a = 3;
$b = false;
$c = $a || $b; $c结果是1
$c = $a or $b; $c结果是3
```

```
$c = &%a
```
这句代码意思是$c是$a的引用。

php中elseif是连在一起。php是弱类型语言，非0就是true。

switch语句与java一样,switch是使用==比较，即值相等类型不等也可以。大于小于的比较也是基于值，类型不同也可以。

### 数组
PHP中使用array()来创建数组。

PHP中有三种类型的数组

- 数值数组：key是数字id
- 关联数组：key是自定义的值
- 多维数组：包含一个多个数组的数组

使用` count(数组名) `来获取数组长度。

#### 数值数组
自动分配id的创建方式
```
$a = array('Toyota', "Nissan","BMW");
```

手动分配id的创建方式，不用事先声明$a
```
$a[0] = 'Toyota';
$a[1] = 'Nissan';
$a[2] = 'BMW';
```

```
$arrName[] = 'abc'; 数组名加空方括号表示数组的一个新元素
```
#### 关联数组
创建方式1
```
$a = array('Car1' => 'Toyota', 'Car2' => 'Benz', 'Car3' => 'Buick');
```
创建方式2
```
$a['Car1']='Toyota';
$a['Car2']='Benz';
$a['Car3']='Buick';
```

```
$arrName['key名'] = value;  新添加一个元素到数组中
```
#### 遍历数组的方式
通用方式foreach
```
foreach ($数组名 as $自定义键名 => $自定义值名) {
    # 可以使用自定义键名和自定义值名了
}
```
还有另外一种省略键名的语法
```
foreach ($数组名 as $自定义值名) {
    
}
```
```
count($arrName);    返回数组元素个数
implode('分隔符内容',$arrName); 将数组的内容组合成字符串，用指定的分隔符分隔
```
#### 数组排序函数
- sort():对于值进行升序排列，把排序后的结果放到0,1,2顺序的数值数组当中，即关联数组sort()之后会变数值数组，所以关联数组不能使用sort()进行排序。即使
- rsort()：sort()的降序版。排序后的key也是0,1，2这样升序的。只是value是降序。
- asort():根据数组的值对数组进行升序排列，是sort()的关联数组版。排列后键值对仍然保持原先对应关系，即如果是数值数组，对应的数值id会随着value的移动而移动。
- arsort()：asort()的降序版。
- ksort()：根据数组的key对数组进行升序排列。是asort()的键版。
- krsort()：是ksort()的降序版。

shuffle()：乱序排列

#### 数组函数

```
count($arr)：返回数组个数
current($arr)：返回数组当前元素的值，默认为第一个元素
end($arr)：返回数组最后一个元素的值
next($arr)：将数组指针指向下一个元素并输出
prev($arr)：将数组指针指向上一个元素并输出
reset($arr)：将数组指针指向第一个元素并输出
each($arr)：以数组形式返回数组当前元素的键名和键值,0和key是键名，1和value是键值，所以需要print_r才能打印完整。然后移动指针到下一个元素。如果当前元素已经超过数组边界了，那就返回false

in_array(搜索值,$arr[,boolean])  
搜索数组中是否包含指定值，最后一个参数设置是否要求类型相同。返回true或者false

list($var1...) = $arr_var; // 把数组的值依次赋值给参数中的多个变量，变量可以跳过，即var1,,var3会被赋予数组中下标为0和2的变量值。注意这个函数只能使用在数组索引的数组当中。

// 格式化打印数组
echo "<pre>";
print_r($arr);
echo "<pre>";
```


### PHP超级全局变量
PHP4.1.0开始预定义的全局变量，全局变量也是变量，所以前面都要加上$

$GLOBALS
$_SERVER
$_REQUEST
$_POST
$_GET
$_FILES
$_ENV
$_COOKIE
$_SESSION

#### $GLOBALS
$GLOBALS是一个数组，包含了所有的全局变量
```
<?php
	$x = 14;
	$y = 46;

	function add(){
		$GLOBALS['z'] = $GLOBALS['x'] + $GLOBALS['y'];
	}

	add();
	echo $GLOBALS['z'];
?>
```
也就是函数外定义的全局变量可以在函数内通过$GLOBALS数组的键名来访问。同样的，函数外可以访问在函数内通过$GLOBALS变量定义的全局变量

#### $_SERVER
一个包含了服务器信息的数组

#### $_REQUEST
用于收集HTML表单提交的数据

```
<html>
    <body>
        <form method='post' action="<?php echo $_SERVER['PHP_SELF']; ?>"
            Name:<input type='text' name='yourName'/>
            <input type='submit'/>
        </form>
        
        <?php 
            $name = $_REQUEST['yourName'];
            echo $name;
        ?>
    </body>
</html>
```
上述例子会报错，因为是把数据发送到本页面进行处理，刚进来是没有yourNam这个变量的。如果发送到其他页面就没事了。
#### $_POST
post请求的发送和接收方式和上述request的一样

#### $_GET
get的发送方式不是使用表单，而是使用url

```
<html>
    <body>
    
    <a href='test_get.php?key1=123&key2=哈哈'>test get</a>
    
    </body>
</html>
```
在test_get.php当中接收
```
<?php 
    echo $_GET['key1'].'<br>'.$_GET['key2']
?>
```

#### $_FILES
$_FILES数组包含了上传文件的信息，是一个二维数组。因为我们正常上传的类型是file，所以正常上传完文件，外层key是file，file这个key对应的值又是一个数组，包含name、type、tmp_name临时副本文件名、error错误提示、size等。
```
Array ( [file] => Array ( [name] => 未标题-1.png [type] => image/png [tmp_name] => /tmp/phpzefrQf [error] => 0 [size] => 23758 ) ) 
```

### 魔术变量
魔术变量是随着该变量位置改变而改变的变量。
```
__LINE__：文件的当前行号

__FILE__: 当前文件绝对路径

__DIR__：文件所在绝对目录名，除了根目录，否则目录名不包含末尾斜杠。

__FUNCTION__：函数名

__METHOD__：方法名，如果定义在类中返回格式是返回类名::方法名

__CLASS__：函数名

__NAMESPACE__：命名空间名

__TRAIT__：trait的名称
```

### 命名空间
类似Java的package一样来区分代码的。多层次使用\来分隔，比如MyProject\sub

```
<?php 
    namespace MyProject1;
    // MyProject1的代码
    
    namespace MyProject2;
    // MyProject2的代码
    
    // 声明方式2
    namespace MyProject3 {
        // MyProject3的代码
    }
?>
```
namespace语句必须是脚本中的第一句，或者在declare源文件编码语句的后面。

```
declare(encoding='UTF-8');
namespace mySpace;
```

如果在php脚本前面有例如html标签，就会报错：
```
<html>
    <?php 
        namespace mySpace; // 这样会报错
    ?>

```

### 面向对象

类的声明，成员变量必须使用var来声明：
```
<?php 
    class Site {
        var $url;
        var $title;
        
        function setUrl($par){
            $this->url = $par;
        }
        
        function getUrl(){
            echo $this->url.PHP_EOL;
        }
    }
    
    $s = new Site();
    $s.setUrl('www.baidu.com');
    $s.getUrl();
?>
```
$this代表对象本身。PHP_EOL是PHP的换行符。

使用new关键字来创建对象，无参时可以省略括号：

```
$a = new Site;
```

使用->来调用成员变量和方法。->调用成员变量就不用加$了。

#### 构造函数和析构函数
构造函数名固定为__construct(参数名)

```
function __construct($url, $title){
    $this->url = $url;
    $this->title = $title;
}
```
PHP中构造函数不会自动调用父类的构造函数，如果需要父类构造函数，则需要手动调用
```
    parent::__construct();
```

析构函数名固定为__destruct()
```
function __destruct(){
    echo 'destruct';
}
```

#### 继承和实现
PHP和Java一样是单继承extends，也有复写

同样使用interface来定义接口，同样使用implements来继承接口，接口这块跟Java几乎一样。

抽象类也是用abstract声明，抽象方法也是使用abstract来声明。

#### 权限修饰
PHP中有三种权限public、protected、private来修饰成员变量和函数。如果用var定义变量，则就是public的。如果函数没有定义权限，则就是public的。

public表示所修饰的变量或函数可在任意地方进行访问。

protected表示所修饰的变量或函数只能在本身或子类或父类中访问

private表示所修饰的变量或函数只能在其本身中进行访问。所以private的变量或函数不会重写父类的同名变量。即父类和子类中同时定义了一个同名函数时，而且父类提供一个方法访问这个变量或函数时，当这个函数是private的时候，调用这个方法访问到的是父类中的函数。而如果是public或者protected的，则调用这个方法访问到的是子类中的函数。

#### 类中的常量定义
```
const CONST_NUMBER = 3;
```
类中常量的调用与变量不一样，不能使用$this->，需要使用::常量名才可以。在函数内调用是
```
self::常量名
```
在函数外可以使用三种方式调用

```
类实例名称::常量名
类名::常量名
内容为类名的字符串变量::常量名
```

#### static
static声明一个变量为静态变量，或者一个方法为静态方法。

静态变量不能使用实例对象去调用，但是静态方法可以使用实例对象去调用。

静态方法中不能使用$this，因为不跟对象绑定。因为这个原因所以也不能用->操作符来访问。用的是::这个操作符。

#### final
和Java的差不多

## 数据库
### 数据库连接
```
<?php
    $servername = 'localhost';
    $username = 'username';
    $password = 'password';
    
    $conn = new mysqli($servername, $username, $password);
    
    if($conn->connect_error){
        die('connect database error: '.$conn->connect_error);
    }
    echo 'connect database successfully.';
    
?>
```
如果是阿里云虚拟主机，servername不能是localhost。
> https://help.aliyun.com/knowledge_detail/36302.html?spm=5176.7836145.2.12.BjI9nw

上面是创建mysqli对象，还有第二种连接方式是调用mysqli_connect()函数

```
$conn = mysqli_connect($servername,$username,$password);
if(!$conn){
	die('connect database error: '.mysqli_connect_error());
}
echo "连接数据库成功";
```

第三种是使用PDO（PHP Data Object）来连接，注意连接的时候还需要指定dbname，否则会抛出异常。

```
<?php
$servername = "localhost";
$username = "username";
$password = "password";
 
try {
    $conn = new PDO("mysql:host=$servername;dbname=myDB", $username, $password);
    echo "连接成功"; 
}
catch(PDOException $e)
{
    echo $e->getMessage();
}
?>
```

数据库连接会在脚本执行完毕后自动关闭，也可以手动关闭：
```
// 对应的三种关闭方式：
// mysqli调用close方法
$conn->close();
// 调用mysqli_close()
mysqli_close($conn);
// PDO置空
$conn = null
```
### 创建数据库
```
// 创建数据库
$sql = "CREATE DATABASE myDB";
if ($conn->query($sql) === TRUE) {
    echo "数据库创建成功";
} else {
    echo "Error creating database: " . $conn->error;
}

// 创建数据库
$sql = "CREATE DATABASE myDB";
if (mysqli_query($conn, $sql)) {
    echo "数据库创建成功";
} else {
    echo "Error creating database: " . mysqli_error($conn);
}

try { 
    $conn = new PDO("mysql:host=$servername;dbname=myDB", $username, $password); 

    // 设置 PDO 错误模式为异常 
    $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION); 
    $sql = "CREATE DATABASE myDBPDO"; 

    // 使用 exec() ，因为没有结果返回 
    $conn->exec($sql); 

    echo "数据库创建成功<br>"; 
} 
catch(PDOException $e) 
{ 
    echo $sql . "<br>" . $e->getMessage(); 
} 

```


### PHP跳转页面
```
<?php 
    // 重定向浏览器 
    header("Location: http://bbs.lampbrother.net"); 
    // 确保重定向后，后续代码不会被执行 
    exit;
?>  
```

### 上传文件

```
<form action="upload_file.php" method="post" enctype="multipart/form-data">
    <label for="file">文件名：</label>
    <input type="file" name="file" id="file"><br>
    <input type="submit" name="submit" value="提交">
</form>
```
method是HTTP请求方法。enctype是提交表单的内容类型，如果需要提交二进制内容表单，比如文件，则需要使用`multipart/form-data`。

input的type=file表示是把输入作为文件处理，浏览器会显示出一个选择按钮。
