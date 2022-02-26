测试和爬虫对selenium并不会陌生，现有的教程已经非常多。但是因为 selenium 封装的方法比较底层，所以灵活性非常高，我们可以基于这种灵活性来实现非常丰富的定制功能。

本系列文章：
- [selenium还能这么玩1：连接手工打开的浏览器](https://blog.csdn.net/looker53/article/details/122951590)
- [Selenium还能这么玩3：重复使用登录状态实战](https://blog.csdn.net/looker53/article/details/122967340)


这篇文章介绍一个操作，可以让selenium 控制浏览器共用同一个 session。他的应用场景是：共用登录状态、中断后持续运行。 

在传统的自动化脚本中，一旦程序运行完成，我们就再也获取不到该浏览器的会话对象了。当你下一次想重新运行脚本时，可能会有比较多的重复步骤，通过这种方式，我们可以绕过这些重复步骤，直接使用之前的会话对象。

一个典型的场景就是记住用户登录状态。我们可以先用第一个脚本实现登录，然后把这次的浏览器会话存下来， 然后我们可以实现第二个脚本，第三个脚本，直接使用这个登录后的浏览器操作后面的步骤。  

只要第一个浏览器的会话没有失效，则后面每次我需要运行时，都可以使用这个会话。
![在这里插入图片描述](https://img-blog.csdnimg.cn/f98f4a150fd447d395a238033a7a66b7.png)

# 半永久的 chrome
首先，我们通过 selenium 创建一个半永久的 chrome 会话，代码是每一个会 selenium 的小伙伴闭着眼睛都能写出来的。
```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get('http://www.baidu.com')
```
需要注意的是，我们在后面没有执行 driver.quit() 函数。这有什么区别呢？ quit 后 chromedriver.exe 进程就直接退出了，但是没有 quit 的 chromedriver 进程是会保留在后台运行的， 所以当你运行上面的代码后，你可以发现 chromedriver 进程还停留在后台，而加了 quit 函数后，进程就会消失。

你可以通过任务管理器查看，或者在 cmd 命令行输入 `tasklist | findStr chromedriver`
![在这里插入图片描述](https://img-blog.csdnimg.cn/da1237516b784042ba67168c8146f926.png)
通过 `taskkill /im chromedriver.exe -f` 命令强制终止进程。

要 chromedriver.exe 成为半永久，而不是运行脚本就结束，只需要保持进程在运行，不添加 driver.quit() 就可以啦。只要你的电脑不重启，那么就可以一直利用这个 chromedriver.exe 提供服务。

# 获取 session id 和 webdriver 服务地址
每个浏览器会话，在 selenium 中都会有一个唯一的 session id, 他记录了浏览器的所有状态，包括用户目录，用户登录的状态， 只要 selenium 使用的 session id 是一致的，那么浏览器的状态就是一致的。

其次，chromedriver 服务有一个远程的连接地址，只要连接该地址，就可以使用同一个 chromedriver 服务。

在 selenium 中，可以这样获取到 session id 和远程地址：
```python
session_id = driver.session_id
remote_executor = driver.command_executor._url
```


# 连接半永久的 selenium session
有了 session id 和服务地址，接下来就可以通过 selenium 初始化客户端建立链接了。 selenium 中的 Chrome 对象默认每次都会初始化一个新的 chromedriver 服务，虽然在最新的 selenium4 中，这种情况有所改善，但是我们在这还是采用更加通用的方式，使用 Remote 对象来建立连接。
```python
caps = {
  "capabilities": {
    "firstMatch": [
      {"browserName": "chrome"},
    ]
  }
}
driver2 = webdriver.Remote(remote_executor, desired_capabilities=caps)
driver2.close()
driver2.session_id = session_id

driver2.find_element('id', 'kw').send_keys('hello')
```
这里用了比较标准的 webdriver capbilities 参数，如果觉得麻烦， caps 可以直接用 {} 代替。 Remote 会访问半永久的chromedriver 服务地址，然后我们把这个 driver2 的 session_id 设置成同一个 session,  因为 driver2 会重新创建一个新的 tab，因此我们通过 driver.close() 先把这个多余的 tab 关掉。

# 总结
可能你会觉得，这种方式和直接用一次浏览器操作并没有什么区别。实际上，效果其实和只使用一次 selenium driver 对象是差不多的，但是，在实际应用中，你的自动化程序不可能永远都在正常运行的状态。

如果程序中断，你将无法再获取之前的 driver 对象了。 而通过这种方式，管理 driver 对象和运行是分开的，就算程序中断，浏览器对象还在后台。  你可以像下载大文件的时候使用断点续传那样，无论什么时候中断，都能重新获取之前的浏览器。


完整的代码：
```python
from selenium import webdriver

# 半永久的 chrome session
driver = webdriver.Chrome()
driver.get('http://www.baidu.com')

# 获取 service url 和 session_id
remote_executor = driver.command_executor._url
session_id = driver.session_id

# 连接之前的 session
caps = {
  "capabilities": {
    "firstMatch": [
      {"browserName": "chrome"},
    ]
  }
}
driver2 = webdriver.Remote(remote_executor, desired_capabilities=caps)
driver2.close()
driver2.session_id = session_id

driver2.find_element('id', 'kw').send_keys('hello')
```

效果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/28ef3412dad4495c8c4f815f43807cb5.gif)