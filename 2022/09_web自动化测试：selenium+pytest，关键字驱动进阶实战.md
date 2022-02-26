 在 [selenium怎么实现关键字驱动](https://blog.csdn.net/looker53/article/details/122830444) 中，我编写了一个非常简单的关键字驱动程序， 不过这个程序只是跑通了功能，还有很多可以优化的地方，这篇文章我想通过 pytest 来简化自动化测试用例的编写，使用的是比较基础的 pytest 功能。 下篇文章我再写个复杂版的，从底层直接执行 yaml 文件作为用例。

## 优化前的用例
在没有优化之前，如果想添加用例，首先需要编写一个 yaml 文件， 然后再写一个 python 的自动化测试用例，用例的代码如下：
```python
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

虽然这个程序用起来比较简单，如果要新建其他的用例，只需要把这个函数复制一遍，修改 signin.yaml 文件名，其他代码不需要动，但是还达不到不需要动脑筋的程度，每次复制这么多重复代码也不好看。

## 优化后的用例
```python
import pytest

@pytest.mark.yaml_case('signin.yaml')
def test_keyword():
	pass
```
优化后的用例明显更简单，函数体中甚至一行代码也没有。 yaml 文件的配置采用装饰器形式配置在测试函数上方，更加清晰，修改的时候很容易找到地方。

## 具体实现
实现方式其实只用到了 pytest 的两个知识点：mark 和 fixture， 先看一下代码：
```python
# conftest.py
import pytest
import yaml
from selenium import webdriver
from keyworks import Page

@pytest.fixture
def driver():
    d = webdriver.Chrome()
    d.implicitly_wait(8)
    d.maximize_window()
    yield d
    d.quit()

@pytest.fixture
def page(driver):
    """获取page"""
    return Page(driver)

@pytest.fixture(autouse=True)
def yaml_case(page, request):
    """yaml 测试步骤"""
    yaml_marker = request.node.get_closest_marker('yaml_case')
    yaml_file, *_ = yaml_marker.args
    with open(yaml_file, encoding='utf-8') as f:
        steps = yaml.safe_load(f)
        for step in steps:
            action = getattr(page, step['action'])
            action(**step['params'])
```

重点在最后一个 fixture。 首先我把 yaml_case 这个 fixture 设成了自动使用，这样在测试函数中我就不需要手工调用了，因此在用例函数中我不需要传入任何参数。

在 yaml_case 这个 fixture 中，第一行代码`request.node.get_closest_marker('yaml_case')`获取到 yaml_case 这个标记 mark， 第二行代码 `yaml_marker.args` 得到标记中的参数，也就是 signin.yaml 这个文件的路径。接下来，读取这个文件中的测试步骤，并且调用具体的执行操作就可以了，调用代码在前一篇文章中已经讲过，如果有疑问可以再翻回去看看。

```python
@pytest.mark.yaml_case('signin.yaml')
def test_keyword():
	pass
```

## 总结
这个代码的实现主要使用了 pytest 灵活的 mark 机制和 fixture 管理， 只要 pytest 用得熟练，实现起来并不难。如果有任何疑问和建议，欢迎私信我一起讨论。