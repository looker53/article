要做 ui 自动化测试，使用关键字驱动可以说是必须会的一种测试方式，它既可以在纯代码的自动化程序中运行，也可以在测试平台中使用。

使用纯代码方式时，自动化工程师先写好一个通用的程序，其他手工测试人员只需要把执行的页面操作关键字填入到一个表格当中，就可以执行自动化测试了。这个表格可以是 Excel 表格，也可是 yaml 文件。

使用测试平台时，测试开发工程会编写一个界面，在假面中，手工测试人员也可以选择需要操作的页面关键字，选择完以后就可以执行自动化测试了。

关键字驱动实现成本低，操作起来也方便，真是个做 ui 自动化测试的好方式。本文介绍一种简单的关键字驱动实现方式，不需要很高深的编程知识也能掌握。

关键字驱动实现主要分为三个部分：
- 关键字方法的定义，其实就是页面的通用操作，比如点击、输入等常见操作，由自动化测试工程师编写。
- 关键字表格配置。谁编写自动化用例，谁配置这个表格，它主要定义用例需要执行的页面操作和测试数据，**最好通过脚本自动生成固定格式的文件，并且把可以选择的操作名称说明，方便使用**， 在测试平台中，可以通过下拉框搜索和选择的方式选择，会更加方便。
- 调用关键字，执行页面操作。这个也是由自动化测试工程师编写，读取 yaml 文件中的页面操作，并且执行。


## 定义关键字方法
如果要对 web 网页进行自动化测试，我们可以定义 Page 类，封装点击、输入等页面操作。
```python
# keywords.py
from selenium.webdriver.remote.webdriver import WebDriver
from selenium.webdriver.remote.webelement import WebElement
from selenium.webdriver.support import expected_conditions
from selenium.webdriver.support.wait import WebDriverWait

DEFAULT_TIMEOUT = 8

class Page:
    def __init__(self, driver: WebDriver):
        self.driver = driver

    def goto(self, url):
    	"""页面访问"""
        self.driver.get(url)

    def wait_clickable(self, locator, timeout=None) -> WebElement:
        timeout = DEFAULT_TIMEOUT if timeout is None else timeout
        wait = WebDriverWait(self.driver, timeout=timeout)
        return wait.until(expected_conditions.element_to_be_clickable(locator))

    def wait_visible(self, locator, timeout=None) -> WebElement:
        timeout = DEFAULT_TIMEOUT if timeout is None else timeout
        wait = WebDriverWait(self.driver, timeout=timeout)
        return wait.until(expected_conditions.visibility_of_element_located(locator))

    def click(self, locator):
    	"""页面点击"""
        el = self.wait_clickable(locator)
        el.location_once_scrolled_into_view
        el.click()

    def fill(self, locator, words):
        """页面输入"""
        el = self.wait_visible(locator)
        el.send_keys(words)
```

## 关键字表格
关键字表格是一个自动化用例的测试步骤和数据，这里我们使用 yaml 文件，action 表示要使用的页面操作，它对应前面 Page 类中的同名方法， params 表示方法需要传入的测试参数。

每个不同的用例，编写一个 yaml 文件，当需要运行自动化测试时，把 yaml 文件作为参数传入自动化程序中就可以了。
```yaml
# login.yaml
-
  action: goto
  params:
    url: "https://petstore.octoperf.com/actions/Account.action?signonForm="
-
  action: fill
  params:
    locator: ['name', 'username']
    words: 'yuze'
-
  action: fill
  params:
    locator: ['name', 'password']
    words: '1234'
-
  action: click
  params:
    locator: ['name', 'signon']
```

## 使用关键字操作浏览器
最后是通用的测试用例编写。定义一个  test_keyword 函数， 首先读取 login.yaml 文件中的测试步骤和数据，使用 getattr  方法， 把需要调用的页面操作名称 goto, fill, click 传入，获取 Page 类中的同名方法，调用这些方法就能实现对应的自动化操作。

这些代码大多数不需要进行额外修改，当需要运行不同的测试用例，只需要修改 yaml 文件的名称，调用不同的 yaml 测试步骤。
```python
# test_login.py

import yaml
import pytest
from selenium import webdriver
from keyworks import Page


@pytest.fixture
def driver():
    d = webdriver.Chrome()
    d.implicitly_wait(8)
    d.maximize_window()
    yield d
    d.quit()

def test_keyword(driver):
    """获取 yaml 文件"""
	with open('signin.yaml', encoding='utf-8') as f:
        steps = yaml.safe_load(f)
    page = Page(driver)
    for step in steps:
        action_name = step.get('action')
        params = step.get('params')
        action = getattr(page, action_name)
        action(**params)
```

## 运行
现在，我们只需要定义不同的 yaml 文件，然后使用 pytest 这样的测试框架来运行自动化用例就可以。 但是，作为一个简单版关键字驱动实现方式，这个程序还有很多的优化空间。

比如，现在编写多个 yaml 文件后，还需要创建多个 python 的 test 文件。而更加常见的方式是，可以通过命令行配置一个 yaml 文件，我就可以运行该 yaml 文件的测试步骤。

配置多个 yaml 文件，就可以运行多个 yaml 文件的测试步骤。甚至可以配置文件夹名称，就运行文件夹下所有的 yaml 文件的测试步骤。

yaml 文件的生成也可以通过命令行生成。

这个我们在后面在实现，目前来说，我们至少已经知道关键字驱动的实现方式了。