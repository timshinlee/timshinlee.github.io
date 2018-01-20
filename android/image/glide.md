教学系列：https://futurestud.io/tutorials/glide-placeholders-fade-animations

Target使用注意事项
1. The first is the field declaration of the SimpleTarget object. Technically, Java/Android would allow you to declare the target anonymously in the .into() method. However, this significantly increases the chance that the Android garbage collector removes the anonymous target object before Glide was done with the image request. Eventually, this would lead to a situation, where the image is loaded, but the callback can never be called. So please make sure you declare your callback objects as field objects, so you're protecting it from the evil Android garbage collector ;-)

java允许匿名声明target在.into()方法里，但是这样做会显著增加在Glide完成image请求之前，Android垃圾回收就已经移除这个匿名target的可能性，就会导致图片加载了，但是却无法回调，所以要把target声明为成员变量

2. The second critical part is the Glide builder line .with( context ). The issue here is actually a feature of Glide: when you pass a context, for example the current app activity, Glide will automatically stop the request when the requesting activity is stopped. This integration into the app's lifecycle is usually very helpful, but can be difficult to work with, if your target is independent of the app's activity lifecycle. The solution here is to use the application context: .with( context.getApplicationContext() ). Then Glide will only kill the image request, when the app is completely stopped itself. Please keep that in mind.

Glide传入context为activity或fragment时，会跟随其生命周期停止请求，大部分情况下十分有用，但是如果需要独立于界面单独下载图片时，就需要传递applicationContext为参数才行

SimpleTarget可以用于单独下载图片，创建simpleTarget同时实现onResourceReady等方法对图片进行处理
ViewTarget可以用于加载图片到不继承于ImageView的自定义控件中，创建viewTarget对象，以该自定义控件为参数，同时实现onResourceReady方法对获取到的图片设置到自定义控件中的ImageView

Glide的transformation会覆盖掉xml的scaleType属性
