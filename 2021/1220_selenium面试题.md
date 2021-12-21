**selenium中如何判断元素是否存在？**

方法一：使用 find_element ，如果元素存在，不会触发异常；如果元素不存在，会触发异常。

方法二：使用 find_elements， 该方法返回一个列表。 判断长度，如果列表长度等于0，则没有找到元素；如果列表长度大于0，则表示元素存在。





**selenium中隐藏的元素能定位吗？**

隐藏的元素通常会使用 hidden 属性或者通过 css 控制 display ＝ none 等实现。这些元素在 selenium 中通常是不能直接定位的，只有当他们能在界面中可见时，才能被定位。





**selenium 如何提高点击事件的成功率**

- 使用显性等待，等待某个元素被点击
- 使用 ActionChains 中的 click 方法或者原生 JS 的 click 语句代替 el.click 
- 元素没有被其他元素遮挡
- 确定是否点击元素中心位置
- 点击只能在 viewport 中，元素不可见时需要先将元素滚动到可视范围之内。



**如何提高selenium 测试用例的执行效率？**

- 尽量避免使用 sleep，使用隐性等待或者显性等待

- 对于firefox，考虑使用测试专用的profile，因为每次启动浏览器的时候firefox会创建1个新的profile，对于这个新的profile，所有的静态资源都是从服务器直接下载，而不是从缓存里加载，这就导致网络不好的时候用例运行速度特别慢的问题

- 使用更高配置的电脑和选择更快的网络环境

- 使用并发执行方式。

- 使用 selenium grid 调度多台执行节点

  

**如何提高selenium 用例运行的稳定性？**

- 尽量使用隐性和显性等待机制

- 注意元素定位表达式编写，避免使用绝对路径和多节点关系定位，减少索引使用，避免动态属性的使用

- 尽量避免用例之间相互关联

  



**你的自动化用例的执行策略是什么？**

- 每日执行：比如每天晚上在主干执行一次
- 周期执行：每隔2小时在开发分之执行一次
- 动态执行：每次代码有提交就执行



**UI 自动化测试需要做数据库校验吗？**

取决于测试策略。如果产品不方便做接口测试，主要是通过大量的端对端测试运行，可以插入适量数据库断言。 但是如果有进行分层测试，更建议在服务端接口测试中进行数据库校验。



**id,name,class,xpath, css selector 这些属性，你最偏爱哪一种，为什么？**

优先使用 css selector 定位，他本身功能足够强大，而且执行速度快，编写的表达式简单易懂， 在 selenium 的实现中，其实 id, name，class 这些方式都使用的 CSS Selector 的定位方式。 对于部分很复杂的定位，可以用 xpath 补足。





**selenium的原理是什么？**

selenium 涉及到三个组件的通讯，分别是

- 浏览器
- webdriver 
- client 

client 负责通过对应的编程语言函数发送请求给 webdriver

client其实并不知道浏览器是怎么工作的，但是driver知道，在selenium启动以后，driver其实充当了服务器的角色，跟client和浏览器通信，client根据webdriver协议发送请求给driver，driver解析请求，并在浏览器上执行相应的操作，并把执行结果返回给client。这就是selenium工作的大致原理。





**webdriver的协议是什么？**

client与driver之间的约定，无论client是使用java实现还是c#实现，只要通过这个约定，client就可以准确的告诉drier它要做什么以及怎么做。



webdriver协议本身是http协议，数据传输使用json。

这里有webdriver协议的所有endpoint，稍微看一眼就知道这些endpoints涵盖了selenium的所有功能。





**启动浏览器的时候用到的是哪个webdriver协议？**

New Session，如果创建成功，返回sessionId和capabilities。





**什么是page object设计模式？**

简单来说就是用class去表示被测页面。在class中定义页面上的元素和一些该页面上专属的方法。