也许你已经看过这两篇文章：
- [selenium怎么实现关键字驱动](https://blog.csdn.net/looker53/article/details/122830444) ，实现了基本的关键字驱动
- [selenium + pytest，实战关键字驱动](https://blog.csdn.net/looker53/article/details/122831500) ，使用 pytest 简化了关键字驱动用例的编写

但是这两篇文章都没有触及到关键字驱动优化的核心：让用例运行更简单！显然，他们还是有一点复杂。 首先我需要手工编写独立的 yaml 文件，再编写一个 python 的测试函数，虽然这个函数已经足够简单，但他并没有减少我们工作的次数，当有新用例需要添加时，我还是得加 yaml 文件，再加一个测试函数。

本篇文章，我希望避免测试函数的多次编写，自动发现指定目录下所有的 yaml 文件，并依次运行，最终我只会有一个用例函数：
```python
@pytest.mark.yaml_dir('realpython')
def test_keywords():
	pass
```
你会发现，他和我们上一版很像，但是效果有很大的不同哦，这里指定的是一个 yaml 目录，该目录下所有的 yaml 用例都会依次被执行，并不需要编写多个测试函数了。



# 设计过程

因为在实际的使用过程中，我们只需要编写一个测试函数，测试需要的数据传入只需要一个 yaml 目录 , 具体表现就是在测试函数上添加一个装饰器 `@pytest.mark.yaml_dir('realpython')`。



这个装饰器会自动遍历 realpython 目录下的所有文件，并验证是否是 yaml 文件。（还有可能需要满足其他的条件，比如：yaml 文件必须以 test_ 开头，但这里不做讨论）



![image-20220222205508121](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220222205508121.png)



如果满足条件，pytest 会根据文件的个数自动生成多个用例，这种自动生成用例的方式在 pytest 中叫做参数化（paramatrize）。如果有 3 个符合要求的 yaml 文件，pytest 会自动生成 3 个用例。



# Pytest 获取目录

于是，我们编写一个 pytest_generate_tests 的钩子函数自动生成用例。它是 pytest 提供的标准钩子函数，可以传入 metafunc 参数表示测试函数，也就是上文中的 `test_keywords` 函数。

```python
def pytest_generate_tests(metafunc):
    """用例生成"""
    pass
```



通过 metafunc，我们可以获取到函数中的相关参数和数据，你可以通过断点直接查看它的属性。

![image-20220222210739145](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220222210739145.png)



现在，我们可以通过其中的 definition 属性获取到测试函数对象，再通过`.get_closest_marker('yaml_dir')`方法获取测试函数中传过来的目录名`realpython`，然后就可以遍历目录下所有的文件了。

```python
yaml_dir = metafunc.definition.get_closest_marker('yaml_dir')
```



# 自动遍历

获取到目录参数后，我们通过路径遍历，获取目录下的所有子文件。这里的代码和测试无关，属于 Python 基础。

```python
if yaml_dir:
    dirname, *_ = yaml_dir.args
    files = Path(dirname).iterdir()
    yaml_cases = (file for file in files if file.suffix in ('.yaml', '.yml'))
```



最后，根据获取到的所有的 yaml 文件，使用 pytest 生成多个测试用例，因为后面还要创建固件处理测试数据，所以生成用例时使用参数 indirect=True 把生成的用例数据传给固件。在 yaml_case

```python
metafunc.parametrize('yaml_dir', yaml_cases, indirect=True)
```





这是生成用例的完整代码：

```python
def pytest_generate_tests(metafunc):
    """用例生成"""
    yaml_dir = metafunc.definition.get_closest_marker('yaml_dir')
    if yaml_dir:
        dirname, *_ = yaml_dir.args
        files = Path(dirname).iterdir()
        yaml_cases = (file for file in files if file.suffix in ('.yaml', '.yml'))
        metafunc.parametrize('yaml_dir', yaml_cases, indirect=True)
```





# 获取yaml文件的自动化步骤

从 request 固件的 param 参数中，能直接获取到文件路径，使用 pyyaml 库打开文件，解析其中的数据，就能得到每一个测试步骤和需要的参数，之后的操作和 [selenium + pytest，实战关键字驱动](https://blog.csdn.net/looker53/article/details/122831500)  完全一致。

```python
@pytest.fixture(autouse=True)
def yaml_dir(page, request):
    yaml_file = request.param
    with open(yaml_file, encoding='utf-8') as f:
        steps = yaml.safe_load(f)
        for step in steps:
            action = getattr(page, step['action'])
            action(**step['params'])
```





完整代码：

```python
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

def pytest_generate_tests(metafunc):
    """用例生成"""
    yaml_dir = metafunc.definition.get_closest_marker('yaml_dir')
    if yaml_dir:
        dirname, *_ = yaml_dir.args
        files = Path(dirname).iterdir()
        yaml_cases = (file for file in files if file.suffix in ('.yaml', '.yml'))
        metafunc.parametrize('yaml_dir', yaml_cases, indirect=True)

@pytest.fixture(autouse=True)
def yaml_dir(page, request):
    yaml_file = request.param
    with open(yaml_file, encoding='utf-8') as f:
        steps = yaml.safe_load(f)
        for step in steps—:
            action = getattr(page, step['action'])
            action(**step['params'])
```



# 总结

这篇文章，充分利用 pytest 这个框架的灵活性。

- 首先，通过钩子函数 pytest_generate_tests 获取标记的参数，并且参数化生成用例
- 然后，讲生成的用例参数传递给 yaml_dir 这个固件
- 紧接着，在 yaml_dir 固件中，读取yaml 文件的测试步骤，并且通过动态调用 Page 类中的方法，进行浏览器操作。
- 最后，在测试时，我们只需要在用例函数上添加 `@pytest.mark.yaml_dir('realpython')` 这个标记，把需要测试的文件目录 realpython 传递给程序，调用非常方便。