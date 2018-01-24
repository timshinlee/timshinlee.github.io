# Reactive Programming
传统的主动式编程
```java
// 开关打开电灯，此时开关是proactive，电灯是passive，两者耦合度高
public class Switch {
	LightBulb lightBulb;

	void onFlip(boolean enabled) {
		lightbulb.power(enabled);
	}
}
```
响应式编程
```java
// 开关打开电灯，此时开关是observable（可观测的），电灯是reactive
public static LightBulb create(Switch theSwitch) {
	LightBulb lightBulb = new LightBulb();
	theSwitch.addOnFlipListener(enabled -> lightBulb.power(enabled));
	return lightBulb;
}
```
在实际编程当中，例如数据库操作主动去刷新UI就属于proactive model，但是这样耦合度较高，如果让UI监听数据库修改，就属于reactive model。

不过上述的模型还不够完美，不足之处有两点。第一点就是Switch类需要创建自己的监听器，这样每个reactive model的observable都需要创建自己特有的监听器，这就无法达到复用了。第二点就是LightBulb直接引用到了Switch对象，两者耦合度也比较高。

要解决这两个问题可以考虑一下函数的返回值。从个数方面来看，函数可以返回一个或者多个对象；从时序方面来看，一个函数可以同步返回或者异步返回。如果是同步的，单返回值可以是`T`，多返回值可以是`Iterable<T>`。如果是异步返回，单个可以是`Future<T>`，多返回值可以是`Observable<T>`。如下表格所示：
      |   One   |    Many
------|---------|-------------
 Sync |    T    | Iterable<T>
 Async|Future<T>|Observable<T>

这时候就可以把之前的代码优化成如下：
```java
public class Switch {
	Observable<Boolean> flips {
		// ...
	}
}

public static LightBulb create(Observable<Boolean> observable) {
	LightBulb lightBulb = new LightBulb();
	observable.subscribe(enabled -> lightBulb.power(enabled);
	return lightBulb;
}
```
首先第一点，响应式编程因为是监听，都是异步返回的，所以多个参数异步情况下返回`Observable<T>`，这样就可以复用了。第二点，监听的函数中传入的是`Observable<T>`，不是具体的Switch，也降低了耦合度。

所以Observable是一段时间内一系列事件的组合，可以用箭头表示时间，marble表示事件。事件处理结果成功使用竖线表示，处理失败使用叉号表示
![marble and timeline](http://blog.danlew.net/content/images/2017/07/slide17-thumb.png)
![event success](http://blog.danlew.net/content/images/2017/07/slide18-thumb.png)
![event failed](http://blog.danlew.net/content/images/2017/07/slide19-thumb.png)

# Functional Programming
首先了解一下pure functions。pure functions表示该函数既不改变外部状态（打印输出或者退出程序），也不会对传入的参数进行修改。同时对于同样的输入值，pure functions的返回值必须永远保持一致，也就是说并不会依赖于外界环境而输出。所以：
- pure functions是一定要有返回值的，如果返回值为void，则这个函数既不能改变外部状态，也不能修改传入参数，也没有返回值，那这个函数就毫无意义了。
- pure functions的传入参数必须是不可变的，否则并发情况下会产生错误
- pure functions的返回值也必须是不可变的，否则就无法被其他pure functions使用。
而函数式编程就是基于pure functions的。

以下就是一个简单的pure function：
```java
public static List<Integer> doubleValues(List<Integer> input) {
	List<Integer> output = new ArrayList<>();
	for (Integer value: input) {
		output.add(value * 2);
	}
	return output;
}
```
但是上面这个函数不够通用，我们可以把算术操作抽离出来，形成函数式编程，此时这个map就是一个高阶函数，即把函数作为参数：（Android Studio需要开启JDK8）
```
    public interface Function{
        Integer apply(Integer integer);
    }

    public static List<Integer> map(List<Integer> input, Function fun){
        List<Integer> output = new ArrayList<>();
        for(Integer value: input){
            output.add(fun.apply(value));
        }
        return output;
    }

    List<Integer> numbers = Arrays.asList(1, 2, 3);
    List<Integer> doubled = map(numbers, i -> i * 2);
```
在上例的基础上更进一步，使用泛型来设置返回值：
```
    public interface Function<T,R>{
        R apply(T t);
    }

    public static List<String> map(List<Integer> input, Function<Integer,String> fun) {
        List<String> output = new ArrayList<>();
        for (Integer value : input) {
            output.add(fun.apply(value));
        }
        return output;
    }

    List<Integer> numbers = Arrays.asList(1, 2, 3);
    List<String> doubled = map(numbers, integer -> integer + "--Str");
```

因为高阶函数是基于pure function的，而pure function具有不受外界因素影响，也不影响外界的特点，所以如果`A -> B`，以及`B -> C`，那么就可以推出`A -> C`。

# Functional Reactive Programming


# References
> [Introction to functional reactive programming](http://blog.danlew.net/2017/07/27/an-introduction-to-functional-reactive-programming/)

