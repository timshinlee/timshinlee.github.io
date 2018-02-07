# Espresso配置
首先要到开发者设置中关闭window animation scale、transition animation scale、animator duration scale三个动画。

然后添加依赖：
```groovy
// module's build.gradle
android {
    defaultConfig {
        ...
	testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
}

dependencies {

    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.1'
    androidTestImplementation 'com.android.support.test:runner:1.0.1'
}
```
# API组成
- Espresso：与view进行交互的入口（通过`onView()`和`onData()`），以及没有与view绑定的api，例如`pressBack()`。
- ViewMatchers：一系列实现了`Matcher<? super View>`接口的对象，可以传入一个或者多个这种对象到`onView()`方法上，来定位当前视图层级中的一个view。
- ViewActions：一系列的ViewAction对象，可以用来传递给`ViewInteraction.perform()`方法，例如`click()`。
- ViewAssertions：一系列的ViewAssertion对象，可以用来传递给`ViewInteraction.check()`方法。大部分情况下会使用matches断言，判断当前选中view的状态。

示例：
```
onView(withId(R.id.my_view))        // withId(R.id.my_view)是一个ViewMatcher
    .perform(click())               // click()是一个ViewAction
    .check(matches(isDisplayed())); // matches(isDisplayed())是一个ViewAssertion
```

# 确定view
当view的id唯一时，直接使用下列方式获取这个view：
```java
onView(withId(R.id.my_view)) // 一次只能确定一个view
```
当view的id不唯一时，可以通过指定条件确定view：
```java
onView(allOf(withId(R.id.my_view), withText("Hello!")))
onView(allOf(withId(R.id.my_view), withContentDescription())
// 或者相反
onView(allOf(withId(R.id.my_view), not(withText("Unwanted"))))
```

有几个注意点需要考虑：
- 一般使用`withText()`和`withContentDescription()`就可以正常定位，否则可能需要添加这些属性
- 不要过度指定描述符，用最少最精确的方式去查找需要的view
- 如果是ListView、GridView或者Spinner这些AdapterView，需要使用`onData()`去查找

# 操作view
```java
onView(...).perform(click()); // 正常操作
onView(...).perform(typeText("Hello"), click()); // 多个操作
onView(...).perform(scrollTo(), click()); // 在ScrollView中先滑到显示view出来，再操作，如果当前view已经显示，则调用scrollTo()没有影响
```

# 检查view
```java
onView(...).check(matches(withText("Hello"))); // 检查一个view是不是含有Hello文本

onView(...).check(matches(isDisplayed())); // 检查一个view是不是可见
```

## 检查AdapterViews的数据
下面测试选择spinner中的一项，并检查结果
```java
onView(withId(R.id.spinner_simple)).perform(click()); // 选择spinner
onData(allOf(is(instanceOf(String.class)), is("Americano"))).perform(click()); // 选择某一项
onView(withId(R.id.spinnnertext_simple))
    .check(matches(withText(containsStrig("Americano")))); // 检查是否正确
```

# 常用测试场景
