[toc]

测试和爬虫对selenium并不会陌生，现有的教程已经非常多。但是因为 selenium 封装的方法比较底层，所以灵活性非常高，我们可以基于这种灵活性来实现非常丰富的定制功能。

这篇文章介绍一个操作，可以让 selenium 连接我们手动打开的浏览器。他的应用场景主要是：你可以通过手工先实现一些自动化难处理的操作，让 selenium 绕过这些复杂操作，然后再执行自动化。

**自动化的时候经常头痛的扫码登录，可以通过这种方式绕过，短期无法解决的验证码的识别，也可以通过这种方式绕过。**

# 手工打开浏览器
可能有的小伙伴不知道，打开浏览器有两种方式：图标点击和命令行运行。

图标点击不用多说，我们经常使用这种方式打开浏览器。命令行方式允许我啰嗦一句，找到浏览器的安装目录，在安装目录中输入 chrome.exe 就可以了。  比如我的 chrome 浏览器安装在 C:\Program Files (x86)\Google\Chrome\Application 这个路径，则在这个目录下打开 cmd 命令行，输入 chrome.exe，就可以打开一个浏览器。
![在这里插入图片描述](https://img-blog.csdnimg.cn/6e12fc85d6f84375a2516a5a87b75f62.png)


# 浏览器配置参数
我们还可以在浏览器打开的时候设置额外的参数，为它提供不同的功能。 通过 selenium 连接浏览器，需要用到两个参数 --remote-debugging-port 和 --user-data-dir ，其他 chrome 参数如果感兴趣可以查看 [peter.sh 的博客](https://peter.sh/experiments/chromium-command-line-switches/)， 英语不好的可以看 [中文版](https://www.cnblogs.com/gurenyumao/p/14721035.html)。

- `--remote-debugging-port` 这个参数允许我们通过远程的方式连接，selenium 当然也可以。
- `--user-data-dir` 这个参数指定一个独立的目录存放产生的用户数据，在连接时也要设置，否则会失效。

现在我们可以通过以下命令行打开浏览器：
```
chrome.exe --remote-debugging-port=9222 --user-data-dir="C:\selenium\ChromeProfile"
```

打开后做两件事情：
1、在浏览器地址栏输入 http://localhost:9222/json/list ，可以查看到浏览器的一些信息。
2、打开本地的 C:\selenium\ChromeProfile 目录，会出现很多子文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d02b3fc4ca0448c5a025732decdab6f8.png)
# 快捷方式设置参数
手工打开用命令行比较用得少，界面照样可以配置参数。在 chrome 的快捷方式上右击，选择属性，快捷方式的目标栏后面加空格加上：
```
--remote-debugging-port=9222 --user-data-dir="C:\selenium\ChromeProfile"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ec7b9bfeba3e4c699c57ae0b13f507ec.png)
现在，单击快捷方式打开的浏览器也可以通过远程调试了。




# selenium 连接浏览器
有了远程调试地址，selenium 连接浏览器就变的很简单，只需要加 2 行代码。 创建一个选项，绑定debuggerAddress 远程调试地址。 此时就可以用 selenium 控制之前手工打开的浏览器了。
```
from selenium.webdriver.chrome.options import Options

chrome_options = Options()
chrome_options.add_experimental_option("debuggerAddress", "127.0.0.1:9222")
driver = webdriver.Chrome(options=chrome_options)
print(driver.title)
```

效果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/a3bd58d5396f488283de5d76ff2ce434.gif)
对本文有任何疑惑，欢迎在评论区 [ 留言讨论 ]，也可以点击下方 [ 关注 ] ，一起交流技术。