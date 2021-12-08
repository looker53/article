升级selenium4，坑了。



最近 selenium 进行了一个更新，官网宣传有比较大的更新， 于是我毫不犹豫的着手升级，没想到直接掉坑里了...现在我们来还原一下整个操作过程。



我之前的 selenium 版本是 3.14.0，在Python 升级 selenium 很简单，只需要一行命令就可以搞定。

```
pip install -U selenium
```



升级完成之后，可以通过 pip show selenium 查看最新版本是 4.1.0。到目前为止，一切都很好，尽在掌握中。

![image-20211129191257818](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211129191257818.png)



接下来，我把之前的自动化测试代码拿出来，重新用新的 selenium 版本运行一下。代码很简单，就是在百度主页搜索了"永别了武器"几个字。

```python
from selenium import webdriver

def test_01():
    with webdriver.Chrome() as browser:
        browser.get("http://www.baidu.com")
        input_el = browser.find_element(By.XPATH, '//input[@id="kw"]')
        input_el.send_keys('永别了武器')
```



坑来了，直接报错。同一份代码，在 selenium3 中完美运行。升级到 selenium4之后，虽然元素能正常找到，但是调用 send_keys 方法，会报错，报错信息为 dict 没有 send_keys 方法。也就是说，find_element 方法返回的是一个字典对象，不是官方定义的 WebElement 对象。

![image-20211129191711127](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211129191711127.png)



直接打印 input_el，得到的真的是一个字典：

```python
{'ELEMENT': '0.5678200594548724-1'}
```

查看 find_element 方法的源码注释，确实是 WebElement 对象，官方说明和实际运行结果不匹配。

![image-20211129193846504](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211129193846504.png)



盲猜是驱动不兼容导致的，下载最新的 chromedriver 驱动，保存到本地，在初始化浏览器对象时指定驱动路径：

```python
webdriver.Chrome(executable_path='chromedriver_95.exe')
```



程序果然正常运行了，但是报了一个警告信息，大家盲猜一下啥意思。

![image-20211129192610447](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211129192610447.png)



翻译一下：Chrome 类中的 executable_path 参数在最新的 selenium4 中已经被移除了，需要传入 Service 对象，阅读官方文档可以看到相关说明。



修改后的代码就是正确的打开方式了：

```python
def test_01():
    service = Service(executable_path='chromedriver_95.exe')
    with webdriver.Chrome(service=service) as browser:
        browser.get("http://www.baidu.com")
        input_el = browser.find_element(By.XPATH, '//input[@id="kw"]')
        input_el.send_keys('永别了武器')
```



之前的 selenium, 在 Chrome 类中同时管理 Service 类和  Connection 客户端类。 更新之后，service 提取出去，Chrome 类的职责单一了，结构更加清晰，以前老的教程估计又要淘汰一批。



发布：http://testingpai.com/article/1638186487500
