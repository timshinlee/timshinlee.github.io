#Reactive Programming
传统的主动式编程
```
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

这时候可以考虑到一个函数的返回值。从个数方面来看，一个函数可以返回一个或者多个对象；从时序方面来看，一个函数可以同步返回或者异步返回。如果是同步的，单返回值可以是`T`，多返回值可以是`Iterable<T>`。如果是异步返回，单个可以是`Future<T>`，多返回值可以是`Observable<T>`。如下表格所示：

      |   One   |    Many
------|---------|-------------
 Sync |    T    | Iterable<T>
 Async|Future<T>|Observable<T>

# References
> [Introction to functional reactive programming](http://blog.danlew.net/2017/07/27/an-introduction-to-functional-reactive-programming/)

