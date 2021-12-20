最近，移动端自动化测试框架 Appium 开启了一个全新的项目 Appium 2.0 ，基本功能的转移正在紧锣密鼓的进行，并且已经发布了好几个 beta 版本。 GitHub 上可以看到这个项目近期所有的计划，大部分原有的功能已经完成，还有一些不太紧要的配置和文档工作正在进行。

![image-20211220154932663](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211220154932663.png)



## Appium 2.0 的新能力

我个人对 Appium 2.0 非常看好，虽然现在像 airtest 这样的框架也很很好用，但是 Appium 2.0 试图打造的全新的跨平台测试生态还是非常吸引人的。 Appium 希望在 2.0 做的几件大事有：

- 解绑所有的 drivers。 Appium 1.0 是和系统有绑定关系的，比如安卓绑定了 Uiautomator2、iOS 绑定了 xcuitest，其他的测试驱动安装起来比较麻烦。解绑 drivers 后，appium 2.0 能非常轻松的通过命令行添加需要的 driver。
- 创建完善的 drivers 生态。Appium 1.0 其实已经具备了跨平台测试的能力，iOS、android、window、Mac、flutter 都已经有对应的driver 可以使用，但是部分 driver 还是没什么人去使用。 Appium 2.0 支持每个人基于已经有的驱动修改和定制，甚至自己做一个完全新的驱动进行测试，一行命令就可以集成进 Appium，非常方便。
- 创建完善的插件系统。现在的移动端测试场景多变而复杂，对于特殊场景的测试手段，往往需要一些具备特殊功能的插件，比较典型的是基于图形识别的定位和 diff 机制，Appium 2.0 支持简单易用的插件管理系统，能非常方便的扩展它原有的能力， 越来越多的开发者融入进来，开发各种功能的插件，一定会极快的推动 Appium 的发展。



## 安装 Appium 2.0 注意事项

花了 10 几分钟实验了一下简单的运行过程，除了少数几个坑，升级过程还是很流畅的。



**第一步：安装 node.js**

Appium 2.x 版本还没有正式发布，没有正式的界面版下载，要尝鲜 2.0 的新功能，只能通过 node 安装。 首先访问 node.org 官网下载 node 环境，点击安装，安装时注意设置环境变量。

![image-20211220142638123](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211220142638123.png)



**第二步：升级 Appium 服务**

升级 Appium 服务有两种选择，第一种是全局安装，这种方式可以在系统任意目录下便捷的操作 appium 指令，但是我现在主要用的还是 appium 1.x 的版本，不想把 appium 2.0 作为主力使用，所以先放弃这种方式。

全局安装：

```
npm install -g appium@next
```

第二种方式是本地安装，侵入性更小，Appium 会被安装到指定目录，进入目录才能运行 appium 指令，比较适合多个版本的管理。先创建一个本地目录 appium_server，进入目录， 运行：

```
npm install appium@next
```

安装后直接运行本地的 appium 服务， 本地服务可以直接进入 node_modules 目录的appium 命令启动，也可以用 npx 运行：

```
node_modules\.bin\appium
# 或者
npx appium 
```

运行完成之后会提示默认端口 4723 启动服务，由于 2.0 已经对驱动和插件解绑，作为独立的系统，所以服务器启动后会提示 No drivers 和 No Plugins：

```
C:\Desktop\appium_server>node_modules\.bin\appium
[Appium] Welcome to Appium v2.0.0-beta.23 (REV HEAD)
[Appium] Appium REST http interface listener started on 0.0.0.0:4723
[Appium] No drivers have been installed. Use the "appium driver" command to install the one(s) you want to use.
[Appium] No plugins have been installed. Use the "appium plugin" command to install the one(s) you want to use.

```



**第三部：安装测试 driver**

Appium 2 的测试驱动是分开管理的，默认不会安装任何的测试驱动，通过 appium driver 子命令管理所有的驱动。官方可用的驱动目前也很多。

![image-20211220144502297](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211220144502297.png)



现在我只装常用的安卓驱动 uiautomator2 和 iOS 驱动 xcuitest：

```
appium driver install uiautomator2
appium driver install xcuitest
```

![image-20211220144906581](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211220144906581.png)



**第四步：启动测试脚本**

```python
from appium.webdriver import Remote


caps = {
    "platformName": "Android",
    "deviceName": "Android Emulator",
    "automationName": "UiAutomator2",
    "app": file
}

driver = Remote(desired_capabilities=caps, 
                command_executor = 'http://127.0.0.1:4723')
```

使用 Python 程序启动一个 app，用的 Python 库是 Appium-Python-Client，建议安装最新的版本。 需要注意的是 2.0 的接口前缀不再是 http://127.0.0.1:4723/wd/hub， `/wd/hub` 已经被去掉了，直接访问 ip:端口号就行，如果还用之前的默认地址，会提示找不到路由。

```
[debug] [HTTP] No route found for /wd/hub/session
[HTTP] <-- POST /wd/hub/session 404 10 ms - 211
```



这里只列举了升级 2.0 的安装步骤，像连接手机、准备 java 环境、准备安卓环境都已经忽略，仅供参考。如果出现没提到的问题，欢迎回复 ”交流“ 一起探讨。
