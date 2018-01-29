### 修改字符串编辑时语言环境
修改内容的语言环境，只会对lint检测起作用，不会影响正式运行
```xml
<resources xmlns:tools="http://schemas.android.com/tools" tools:locale="pl">
```
### 嵌套引用
达到资源复用，方便修改
```xml
<string name="tab_foo">Foo</string>
<string name="bar_foo">@tab_foo</string>
```
### 自定义XML实体
达到资源复用，方便修改
```xml
<!DOCTYPE resources [
    <!ENTITY foo "Foo">
    ]>
<resources>
    <string name="app_name">&foo; good</string>
</resources
```

# References
> https://android.jlelse.eu/android-strings-xml-tips-tricks-52b0c820cf7a
