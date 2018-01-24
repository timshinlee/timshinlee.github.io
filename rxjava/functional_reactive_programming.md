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

不过上述的模型还不够完美

# References
> [Introction to functional reactive programming](http://blog.danlew.net/2017/07/27/an-introduction-to-functional-reactive-programming/)

