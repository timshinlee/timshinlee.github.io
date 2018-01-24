#Reactive Programming
传统的主动式编程
```
// 开关打开电灯，此时开关是proactive，电灯是passive
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
