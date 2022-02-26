#『移动端测试』uiautomator2 如何支持混合应用测试

移动端 app 自动化框架很多，但是有一些框架因为不支持混合应用测试，一直没有完全流行。比较典型的是经典的 Python 框架 uiautomator2, 这个框架简单好用，没有 appium 那样复杂的 api 调用，受到不少 python 自动化工程师的青睐。

但是不管是官方文档，还是民间教程，基本都没有涉及到用它做混合应用测试，本文提供一种非常简单的方法，只需要多加 4 行代码，就能让 uiautomator2 支持混合应用测试。


## 什么是混合应用
移动端应用有两种典型的开发方式，一种是原生的 native app，一种是基于网页开发技术的 web app。 原生应用的体验感更好，但是如果想同时开发安卓应用和 ios 应用，需要不同的原生开发技术。 web 应用可以非常轻松的做到安卓和 ios 的跨平台开发，它的体验感要稍微差一些，不像原生应用那么流畅。

Hybrid App（混合模式移动应用）是介于 web app和native app之间的开发方式，可以在原生界面中嵌套网页，因而可以同时具备体验感和跨平台能力。

目前主流的移动端 app 测试框架 appium 具备混合应用测试的能力，但是这个框架搭建和使用都比较麻烦，封装的方法也没有那么 pythonic，因此有很多公司不想使用，他们更加喜欢简洁优雅的 python uiautomator2 框架。 非常遗憾，这个框架目前没有支持混合应用测试。


## 混合应用测试步骤
- 第一步，通过原生操作进入 webview 网页；
- 第二步，使用 selenium 等网页测试工具进入网页；
- 第三步，使用 selenium 等网页测试工具测试。

这中间的关键步骤在于如何使用 selenium, 如果直接打开一个新的 selenium 会话，那么会打开一个新的页面，和 app 中的 webview 是分开的，因此无法测到嵌套网页。 selenium 必须要和 app 建立某种关系，使他们绑定在一起，操作 selenium 时就是直接操作 app 当中的网页。
![在这里插入图片描述](https://img-blog.csdnimg.cn/885364bc37aa40e490890755bf19a3f1.png)


## 通过 uiautomator2 进入 webview
这里就是最基本的 uiautomator2 操作，详细操作可以查看 [官方文档](https://github.com/openatx/uiautomator2)， 这里使用的 app 是 android bootstrap，可以直接 [点击下载](https://npmcdn.com/android-app-bootstrap@latest/android_app_bootstrap/build/outputs/apk/android_app_bootstrap-debug.apk) 。
```python
d = u2.connect()
d.app_start('com.github.android_app_bootstrap')
d(text='Login').click()
d(text='Baidu').click()
```


## selenium 连接 webview
APP 的第三个菜单可以直接打开 baidu 网页，从而到 webview 网页。 如果不使用 selenium 连接，直接通过 uiautomator2 定位网页中的元素，是会报错的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/df517487e4d448e7a00e323fd70a8195.png)
**那如何通过 selenium 连接 webview 呢？**

- 首先，通过设备 d 获取当前操作的 app， 从而得到包名
- 然后，初始化一个 Chrome 浏览器对象
- 需要注意的是，Chrome 对象需要添加额外的参数，指定连接的设备，webview 绑定的报名，以及配置在 app 中使用 android 浏览器。
![在这里插入图片描述](https://img-blog.csdnimg.cn/c295d8b96270430c904d34df0678a61a.png)
之后的操作就和 selenium 做 web 自动化测试没有任何区别了，无非就是点击网页，做自动化操作：
```python
driver = webdriver.Chrome('chromedriver_68.exe', options=options)
driver.implicitly_wait(8)

driver.find_element('id', 'index-kw').send_keys('你好')
```

## 这段代码我在哪里学到的
是不是很好奇，我怎么知道要这么写代码，才能连接 webview ，达到测试效果？ 其实世间没有那么多神秘的学习方法，只要你肯用心，一定是可以找到学习的路径的。 这段代码清清楚楚的写在 [chrome webdriver](https://sites.google.com/a/chromium.org/chromedriver/getting-started/getting-started---android) 的官方文档中，只是，你可能打不开。 截屏看一下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/1aa381418fb747f0a93d028fea64e039.png)
其他 webdriver 的配置也可以查看 [MDN web docs](https://developer.mozilla.org/en-US/docs/Web/WebDriver/Capabilities)。

**展示效果：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/af3be9006efd42f5899d14fa88880548.gif)

**`如果你需要完整代码和工具包，欢迎在评论区回复 [ u2 webview ] 获取。`**