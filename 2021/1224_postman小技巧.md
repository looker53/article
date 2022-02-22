[toc]



## Postman如何做接口测试1：如何导入 swagger 接口文档



在使用 postman 做接口测试过程中，测试工程师会往界面中填入非常多的参数，包括 url 地址，请求方法，消息头和消息体等一系列数据，在请求参数比较多的情况下非常花时间。



我们可以使用 postman 的文档导入功能，直接导入 swagger 这样的开放式文档，postman 会根据文档内容以及参数限制自动生成请求相关数据，这样就可以节省大量手工填写参数的时间了。



而且导入 swagger 文档的步骤只有 2 步，非常方便。



首先，第一步，打开 swagger 文档的地址，点击 export ，导出在线文档，可以选择 json 格式或者 yaml 格式。 导出的文档会保存到本地。

![image-20220106142432405](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106142432405.png)



第二步，打开 postman, 选择 "file" 菜单， 点击 “import" ，选择文件导入，把刚刚下载好的 json 文件导入就可以啦。

![image-20220106142707644](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106142707644.png)





导入后的文档会自动生成 request 请求信息，接下来只需要对这些参数进行简单的修改，就可以发送请求了，不需要一个个手动填写。

![image-20220106142858259](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106142858259.png)





swagger 还支持在线链接导入的方式，直接把文档的 json 格式在线地址填到 link 中的 url 栏就可以。

![image-20211224213852249](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211224213852249.png)

**关注我，GET 下一个软件测试小技巧。**



## Postman如何做接口测试2：如何切换测试环境



用postman进行接口测试的时候，我们经常会把接口地址的全路径填在url地址栏当中。这种做法不太好的地方在于，当你需要从一个测试环境切换到另一测试环境时， 需要把所有的url全部修改一遍，当你有 500个用例都需要修改，直接累瘫。



那在postman当中，如何更方便的修改测试环境呢？其实只需要两步。



第一步，点击 postman 左侧工具栏的 environment， 点击 + 号添加新的测试环境。 一个测试环境当中可以创建很多环境变量，有了环境变量，在请求数据中就可以引用这些变量。



在这里，我创建一个了一个开发环境，一个线上环境，里面都有一个叫 baseUrl 的变量。

![image-20220106151825958](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106151825958.png)



第二步，在请求发送界面，点击右上角 “选择环境” 的按钮，切换想使用的环境。 我们先把它设成 dev 环境。然后在url当中，使用双花括号引用环境中的 baseUrl 变量。  当需要切换环境时，点击切换成 prod 环境就可以啦。

![image-20220106152305949](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106152305949.png)

**关注我，GET 下一个软件测试小技巧。**



## Postman如何做接口测试3：什么？postman 还可以做压力测试？



我们都知道， postman 是一款很好用的接口测试工具。不过 postman 还可以做简单的压力测试，而且步骤只需要 2 步。



首先，打开 postman, 编写接口的请求参数。

![image-20220106155212966](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106155212966.png)



然后，点击右下方的 runner 运行器，把需要测试的接口拖动到左侧展示框，在右侧设置压力测试的参数。

iterations 表示请求发起次数，Delay 表示请求间隔时间，点击运行。

![image-20220106155402057](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106155402057.png)

运行后，可以查看实时的响应结果，我们可以根据响应时间判断服务器的处理性能。

![image-20220106160107013](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106160107013.png)



当然，postman 毕竟不是专业的性能测试工具，这些性能数据和指标需要我们自己计算，只适合用来顺手检测一下性能，对服务器的处理能力有个大概印象。如果要专门做性能测试，还是使用 jmeter 这样的工具更方便。

**关注我，GET 下一个软件测试小技巧。**

## Postman如何做接口测试4：如何自动添加请求头

在发送请求之前，测试人员往往要对请求数据进行预处理，从而达到请求发送要求，归纳起来主要有：

- 从外部读取数据
- 数据格式转化
- 增加、修改、删除请求字段
- 添加前置请求等等。

这些操作基本都是在 postman 的 pre-request 页面进行处理，使用 javascript 编程。不够经常使用的操作只需要基本的javascript 语法，不需要太深入学习就能使用。



我们以请求头的添加为例。



第一步，添加一个简单的请求，然后复制几次，模拟有大量请求需要处理的情况。

![image-20220106171128015](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106171128015.png)



第二步，打开请求所属的目录。点击 pre-request， 编写以下代码添加 appId 的请求头。

```js
pm.request.addHeader({key: "appId", value: "7722"})
```



第三步，点击 run 运行。点开运行过程中的 request headers 能够发现，每个请求中都自动添加了 appId 请求头。

![image-20220106171516082](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106171516082.png)



除了添加请求头以外，pre-request 还有大量的应用场景，后面我们再一一讲解，不想错过的可以点一波关注哦。



## Postman如何做接口测试5：如何自动校验接口响应

postman 是一款纯正的自动化测试工具，但是实用它的人却被称为手工测试， 原因是因为大量测试人员还是以手工方式在使用它。



比如说当使用 postman 发送一个测试的用例以后，我们还是以人工方式来检验测试结果，这在大量测试用例需要运行时非常耗费精力。 其实我们只需要多加一步，就可以实现自动校验。



首先，编写测试用例的请求数据，然后点击 tests 标签。

![image-20220106172742783](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106172742783.png)

我们需要校验响应结果中的 args 字段中是不是有 name 字段

![image-20220106172951956](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106172951956.png)

现在编写以下代码断言，实现自动校验

```js
pm.test("name存在", function() {
    let response = pm.response.json()
    let name = response.args.name
    pm.expect(name).to.exist
})
```

运行以后，可以查看到自动校验的结果

![image-20220106173717348](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106173717348.png)



如果有大量的请求都需要同样的自动校验，可以在文件目录中统一编写， 然后点击 run 运行

![image-20220106173812828](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106173812828.png)



运行完成后， 能直观的查看到哪些用例通过，哪些用例没有通过。

![image-20220106173935472](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106173935472.png)

**关注我，GET 下一个软件测试小技巧。**

## Postman如何做接口测试6：如何使用外部 json 文件数据

在软件测试过程中，你可能已经提前配置了一些环境数据放在通用的配置文件中，你想直接导入这些文件，在 postman 中使用，这样的话，你就不需要重复在 postman 软件中设置了。

比如你可以直接从开发那里得到一个配置文件，里面有 host 地址，数据库地址，每页的列表限制信息，我们能不能把这些数据直接导入到 postman 中使用呢？ 按以下步骤操作。



首先，准备好外部的数据文件，postman 目前支持 csv 格式和 json 格式数据，如果是其他类型的文件，可以先转化成这两种格式。我们以 json 格式为例，文件中存放了 host,  dbname, perpage 三个数据

```json
[
    {
        "host": "http://httpbin.org",
        "dbname": "demodb",
        "perpage": 10
    }
]
```





接下来，打开请求所属的目录，点击 run， 选择提前准备好的 json 文件， 确保 postman 可以从文件中读取数据。

![image-20220106165230395](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106165230395.png)



点击 Pre-req ， 编写以下代码。从文件中读取数据，设置成 postman 可以识别的变量。

```js
pm.variables.set('host', pm.iterationData.get('host'))
pm.variables.set('dbname', pm.iterationData.get('dbname'))
pm.variables.set('perpage', pm.iterationData.get('perpage'))
```



最后，打开 postman， 编写请求。请求当中需要的 host， dbname， perpage 参数直接用花括号表示。点击 run 运行，就完成啦。

![image-20220106165020870](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220106165020870.png)



**关注我，GET 下一个软件测试小技巧。**



## Postman如何做接口测试7：如何通过命令行运行

1、导出 postman 中的数据， 导出的 postman 数据存储为 json 格式文件。 这些文件既可以重新导入到 postman 工具中来，也可以导入到其他的测试软件，比如 yapi 等用例管理平台。

![image-20220120150817256](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220120150817256.png)

2、安装命令行工具 newman

newman 就是命令行版的 postman, 它由 nodejs 开发，如果需要使用这个命令行工具，先安装 nodejs， 然后在命令行输入下面指令就可以安装好 newman

```
npm install newman
```

![image-20220120162459072](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220120162459072.png)

3、使用 newman 运行 json 数据,  运行时如果是本地安装，用 npx 运行，如果是全局安装，可以直接用 newman 命令。

```
npx run newman 'postman.json'
```



需要注意的是，postman 导入的 json 文件是不包含环境变量的，如果需要在 JSON 文件中使用环境变量，需要先导出 环境变量的 json 文件，再运行：

```
npx run newman -e "env.json" "postman.json"
```

**关注我，GET 下一个软件测试小技巧。**



## Postman如何做接口测试8：如何生成 HTML 测试报告

1、安装 html 报告插件

在命令行中输入以下命令完成 html 报告模板的安装：

```
npm install newman-reporter-htmlextra
```

报告的具体使用说明可以在官方文档查看： https://www.npmjs.com/package/newman-reporter-htmlextra





2、通过参数运行用例，运行后自动生成 html 报告，测试报告会默认在本地的 newman 目录中保存。

```
npx run newman "postman.json" -r htmlextra
```



3、查看测试报告

![image-20220120164728756](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220120164728756.png)



**关注我，GET 下一个软件测试小技巧。**

## Postman如何做接口测试9：如何生成 allure 测试报告

1、安装 allure 报告插件

在命令行中输入以下命令完成 allure 报告模板的安装：

```
npm install newman-reporter-allure
```

报告的具体使用说明可以在官方文档查看：https://www.npmjs.com/package/newman-reporter-allure



2、通过参数运行用例，运行后自动生成 allure 报告，测试报告会默认在本地的 newman 目录中保存。

```
npx run newman "postman.json" -r allure
```



3、启动 allure 服务, 启动之前要事先安装 allure 命令行工具，在 jenkins 中安装插件后可以不用手动安装。

```
allure serve allure-results
```



4、查看测试报告

![image-20220120165952196](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20220120165952196.png)



**关注我，GET 下一个软件测试小技巧。**





## 如何使用外部 csv 文件数据



**关注我，GET 下一个软件测试小技巧。**



## 如何结合文件做接口自动化测试



## 如何删除请求头



## 如何修改请求头



## 如何修改请求体



## 如何发送前置请求



## 如何校验参数结构



## 如何校验参数的值







## 如何跟 jenkins 集成





## 如何校验





## 设置变量

1、在界面中可以设置 collection 和 environment

![image-20211224214207005](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211224214207005.png)



2、其他的都可以在脚本中编写。

```
pm.global
```



获取有两种方式，在界面中通过 `{{url}}` 获取，在脚本中用 pm.global.get()



## 变量作用域

![image-20211224214943891](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211224214943891.png)



## 获取内置变量

- `{{$guid}}` : A v4 style guid
- `{{$timestamp}}`: The current timestamp (Unix timestamp in seconds)
- `{{$randomInt}}`: A random integer between 0 and 1000



在脚本中：

```
let uuid = pm.variables.replaceIn('{{$guid}}')
```



## file data 做数据驱动





## pre-request script

```js
pm.request.addHeader({"xxx": "yyy"})
```



## 查看访问日志

![image-20211224214421301](https://yuztuchuang.oss-cn-beijing.aliyuncs.com/img/image-20211224214421301.png)



## Test



## 参考地址

- newman 官方文档： https://github.com/postmanlabs/newman
- postman 官方文档： 



