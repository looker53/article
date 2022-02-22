# 5 个 APP 自动化测试辅助定位工具，你用过几个？

UI 自动化测试时，先要定位到需要操作的元素，然后才能执行指令。 在网页端可以非常方便的通过 devtools 工具（也就是经常说的 F12) 获取和编写元素定位表达式。



但是在移动端自动化时，这种辅助工具都还有比较大的优化空间。下面列举 5 种我经常使用的元素定位辅助工具，以及他们的优劣。



## 1、 Appium Inspector

Appium Inspector 是 Appium 实现的页面元素查看器，老版本中直接集成进了 appium 的桌面应用中，新版本将它直接独立出来，做成了一个小工具，就算不使用 appium 做自动化测试，也可以使用这个工具定位元素，也是我默认使用的辅助定位工具。

他的优点有：

1、独立工具，可以单独下载

2、验证元素表达式是否正确

3、既可以定位原生应用，也可以定位混合应用和web页面

缺点是启动比较麻烦，需要配置参数，响应速度也比较慢，web页面定位功能比较鸡肋，精度不够。



安装直接在 https://github.com/appium/appium-inspector 下载，像普通软件安装完成就可以，启动之后需要配置参数表示你连接的手机、平台信息，也是这个工具麻烦的地方。

![image-20211221144724708](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211221144724708.png)



这个是示例，完成后点击 “start session" 就可以连接手机开始定位了。如果觉得每次都要填参数比较麻烦，可以直接保存下来，下次直接用。

```
{
  "platformName": "Android",
  "appium:automationName": "Uiautomator2",
  "appium:packageName": "com.package.name",
  "appium:activity": ".lauchable.activity",
  "appium:chromedriverExecutable": "path/to/chromedriver.exe",
  "appium:deviceName": "emulator-5554"
}
```



在页面中获取元素的表达式之后，可以通过搜索按钮验证是否能在页面中找到指定元素，这个功能还是比较好用的。

![image-20211221145121564](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211221145121564.png)



## Uiautomatorviewer 

这是安卓 sdk 中自带的页面查看工具，安装好安卓 sdk 后，直接在 `\tools\bin` 目录下找到 uiautomatorviewer.bat 文件点击运行就可以了。 它的优势在于安卓自带，开箱即用，可以把文件设置快捷方式，直接点击运行；劣势在于功能比较简陋，只能获取基本元素信息。

![image-20211221145941361](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211221145941361.png)



## Weditor

weditor 是在 atx 生态下的一个小工具，目前由阿里巴巴维护，它支持自从生成 python uiautomator2 的代码，但是如果你不用 u2，那这个功能没什么用。它也支持直接在界面上操作 app，但是这个工具不是很稳定，连接非常容易断开，而且和 appium 不兼容，要定位先要把 appium 服务断开，appium 运行时也需要把 weditor 断开，操作起来比较繁琐。

![image-20211221150324014](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211221150324014.png)



## chrome inspect

这是定位 app 中嵌入网页时需要用到的工具，常见于混合应用和 web 应用。当 app 进入一个网页时，直接在 chrome 浏览器的 url 栏输入 chrome://inspect 就可以进入网页调试工具，页面中会显示有哪些手机设备连接上了。它不需要安装任何的扩展，浏览器内置功能。点击页面就可以直接打开 devtools 工具，和在浏览中按 F12 没有什么区别。

![image-20211221151600535](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211221151600535.png)



## UC Devtools

这是 UC 浏览器开发的网页调试工具，作用和 chrome inspect 一样，主要定位 app 中的网页元素。 打开官网 https://plus.ucweb.com/download/ 完成下载。

![image-20211221151938003](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211221151938003.png)



加我好友，一起讨论：

![image-20211224204259254](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211224204259254.png)