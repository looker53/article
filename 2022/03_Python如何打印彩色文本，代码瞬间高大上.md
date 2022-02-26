

# Python如打印彩色文本，代码瞬间高大上



想在打印信息中重点突出内容

在 Python 中打印彩色文本，不是必须掌握的编程技巧。对，不是比较掌握的，平时显示的黑色字体也不是不能用。 
但是，在一些特定的场合，你能通过彩色标注出非常关键的信息，**让用户一眼能把握重点，别人自然更愿意使用你编写的程序。**

这是一个花里胡哨的功能，只是这些表面功夫确实能显得技术更强，代码更好用。 就像现在去买商品，我们都更愿意去挑选颜值高的，那些看起来比较丑的，在第一时间已经被我们过滤掉了。

不知道你有没有发现，当一个程序运行过程中如果出现了某些异常，那些能帮助我们快速定位问题的提示信息通常会用不同的颜色标注，让我们一眼能看到重点。
![在这里插入图片描述](https://img-blog.csdnimg.cn/9039ca8289db4369b5b530fc2247eabe.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rWL6K-V55qE6IO96YeP,size_20,color_FFFFFF,t_70,g_se,x_16)




## 一行代码突出重点内容
现在我们通过一个例子，说明彩色文本怎么打印。先看下黑白文本打印：
```
string = "这是一段非常重要的内容，如果错过了，损失一个亿，希望对你有帮助。"
print(string)
```
这里面【一个亿】是程序想向用户传达的重点，因此我们想通过不同的颜色让他凸显出来，用户第一时间能抓住重点。 于是，我把代码修改成这样：
```
string = "这是一段非常重要的内容，如果错过了，损失\033[43m一个亿\033[0m，希望对你有帮助。"
print(string)
```
修改后，【一个亿】的背景色加上了黄色，在很多文字中非常亮眼，达到了突出重点的效果。是不是有点看不明白 `\033[43m` 是什么意思？往后看，会有公式。
![在这里插入图片描述](https://img-blog.csdnimg.cn/06455deb8d82478eb3f1689f0f7b2fda.png)

同样的，我们可以把这几个字改成红色加粗，并且配上蓝色背景。
```
string = "这是一段非常重要的内容，如果错过了，损失\033[1;31;44m一个亿\033[0m，希望对你有帮助。"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/488539514a1c44efbef2df41efaf3e54.png)
## 彩色打印的公式
这部分比较枯燥，但是公式很重要。彩色打印的公式这这样的，**认真看三遍**：
```
\033[效果;前景色;背景色m要打印的文本\033[0m
```
举个例子：
```
\033[0;31;44m一个亿\033[0m
```
- 0 表示用普通效果
- 31 表示字体用红色
- 44 表示背景色用蓝色
- `\033[0m` 表示重置成默认

显示效果分为三部分：效果，前景色，背景色 都是可以省略的，因此以下的用法也是合法的。
- `\033[31m` 红色字体
- `\033[44m` 蓝色背景
- `\033[1m`  加粗

列举一下主要的打印效果：
```
0		终端默认设置
1		高亮显示
4 		使用下划线
5		闪烁
7		反白显示
-----------------------
30		黑色前景 
31		红色前景						
32		绿色前景
33		黄色前景
34		蓝色前景
35		紫色前景
36		青色前景
37		灰色前景
-----------------------
40		黑色背景
41		红色背景					
42		绿色背景
43		黄色背景
44		蓝色背景
45		紫色背景
46		青色背景
47		灰色背景
```

收藏这个表格，需要彩色打印的时候翻出来就好了。现在我们通过一个示例进一步巩固一下用法。
我们打印一个班级学生的考试成绩，如果低于 60 分，用红色标记。
```python
students = ['张三', '李四', '海明', '艾米']
scores = [30, 90, 100, 88]

info = '姓名\t\t分数'
for idx, name in enumerate(students):
    info += '\n'
    info += name
    info += '\t\t'
    score = scores[idx]
    if score < 60:
        info += f'\033[31m{score}\033[0m'
    else:
        info += str(score)

print(info)
```
可以看到打印信息中低于60分的被标记成了红色，一眼就能抓住重点。
![在这里插入图片描述](https://img-blog.csdnimg.cn/b98c36d800c5425eaa19a557164446b6.png)



## 彩色效果汇总
彩色打印效果有很多，我们不可能一个个记住，最好的方式是在用的时候直接查询，把效果代码复制过来就好。

**一个效果查询表非常有用，下面这个查询表你可以保存下来。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/12730138a78f4e82bc4bbe1c67d01910.png)
需要哪个彩色效果，直接复制一个效果组合就可以。比如想要黑色背景，绿色字体，则复制 `0;32;40` 到前面总结的公式中：
```
\033[0;32;40m要打印的文本\033[0m
```
你也可以通过自己运行下面的代码，直接得到查询表。
```python
def print_color_range():
    """
    prints table of formatted text format options
    """
    for fg in range(30,38):
        s1 = ''
        for bg in range(40,48):
            format = ';'.join(['0', str(fg), str(bg)])
            s1 += f'\033[{format}m {format} \033[0m'
        print(s1)

print_color_range()
```


当然这里只是普通效果的，如果需要加粗，或者闪烁，把0改成1或者5 就可以了，你也可以在评论区留言 【python彩色文本图】获取完整的效果。
![在这里插入图片描述](https://img-blog.csdnimg.cn/467fb1e13f594024a4530458e5d0872e.png)


## 封装，让彩色打印更好用
**你以为到这里结束了吗？ 接下来才是重点。** 如果需要经常使用彩色打印，最好是有一个现成的代码库，需要用到的时候直接调用，而且最好用法非常简单，这样我们才会去频繁使用。

但是上面的例子其实都是比较难用的，现在我们把这些效果代码封装到类里面。  Style 表示显示效果，Font 表示字体颜色，Background 表示背景颜色。 在使用时可读性就变得很强了，以前 `\33[0m` 不知道什么意思，下载用 Style.END 表示是不是好很多呢？

调用的时候，直接在要特效表示的字符串前加上 Font.RED 这样的属性，就能非常方便的打印彩色了。
```python
class Style:
    END = '\33[0m'
    BOLD = '\33[1m'
    ITALIC = '\33[3m'
    URL = '\33[4m'
    BLINK = '\33[5m'
    BLINK2 = '\33[6m'
    SELECTED = '\33[7m'

class Font:
    BLACK = '\33[30m'
    RED = '\33[31m'
    GREEN = '\33[32m'
    YELLOW = '\33[33m'
    BLUE = '\33[34m'
    VIOLET = '\33[35m'
    BEIGE = '\33[36m'
    WHITE = '\33[37m'

class Background:
    BLACK = '\33[40m'
    RED = '\33[41m'
    GREEN = '\33[42m'
    YELLOW = '\33[43m'
    BLUE = '\33[44m'
    VIOLET = '\33[45m'
    BEIGE = '\33[46m'
    WHITE = '\33[47m'

string = f"这是一段非常重要的内容，如果错过了，损失{Style.BOLD}{Font.RED}{Background.BLUE}一个亿{Style.END}，希望对你有帮助。"
print(string)
```
## 函数封装
也可以通过函数进一步封装，让代码更加简洁。
```python
def highlight(string, fcolor='', bgcolor='', style=''):
    """彩色打印的函数"""
    fcolor_code = getattr(Font, fcolor.upper(), '')
    bgcolor_code = getattr(Background, bgcolor.upper(), '')
    style_code = getattr(Style, style.upper(), '')
    return f"{style_code}{fcolor_code}{bgcolor_code}{string}{Style.END}"
```

使用：
```python
h = highlight('一个亿', fcolor='red', bgcolor='yellow')
print(f'这是一段非常重要的内容，如果错过了，损失{h}，希望对你有帮助。')
```

##  logging 中的使用
彩色打印不仅可以在 print 的时候用，而且可以在 logging 日志记录的时候用。
```python
import logging
h = highlight('一个亿', fcolor='red', bgcolor='yellow')
logging.warning(f'这是一段非常重要的内容，如果错过了，损失{h}，希望对你有帮助。')
```
## 第三方库
如果你觉得自己封装还是比较麻烦的，也可以直接用第三方库，不过我觉得自己封装的和第三方库的库也差不多，以下是比较流行的 colorama 库的用法，实际上和我们上面封装的类属性的方式非常类似。
```python
# pip install colorama
from colorama import Fore, Back, Style

string = f'这是一段非常重要的内容，如果错过了，损失{Fore.RED}{Back.BLUE}一个亿{Style.RESET_ALL}，希望对你有帮助。'
print(string)
```

另一个更强大的库是 rich, 功能太强，后面再慢慢研究。


参考：
- https://stackoverflow.com/questions/287871/how-to-print-colored-text-to-the-terminal