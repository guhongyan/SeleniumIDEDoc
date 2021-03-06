类似定位器参数，文本模式是另一种常用的 Selenium 命令参数。需要使用文本模式的命令，例如：verifyTextPresent, verifyTitle, verifyAlert, assertConfirmation, verifyText, verifyPrompt。上面已经提到，LinkText 定位器可使用文本模式。文本模式使用特殊字符来模糊匹配预期的文本，而不必准确的描述该文本。

有三种类型的模式：Globbing 模式，正则表达式和 exact。

# Globbing 模式

---
很人都熟悉 globbing ，因为在 DOS 或 Unix / Linux 命令行下经常用 globbing 模式来匹配文件名，如： ls *.c。这个例子中，globbing 模式用于匹配当前目录下所有的 .c 扩展名的文件。Globbing 的功能很有限。Selenium 只支持和实现两个特殊字符：

- \* 匹配任何事情，例如：什么也没有，一个字符，或许多字符。 
- \[ \]（字符类）匹配方括号中的任何单个字符。一个连字符指定一定范围的字符（通常是连续的 ASCII 字符集）。下面几个例子将演示字符类的用法：  

   - [aeiou] 匹配任何一个小写的元音字母   
   - [0 - 9]  匹配任何数字  
   - [a-zA-Z0-9]  匹配任何字母和数字符号  

在很多其他系统中，globbing 还包括第三个特殊字符问号 “?”。然而，Selenium globbing 模式只支持星号和字符类。

在一个 Selenese 命令中，你可以用 glob: 标记来声明，指定使用 globbing 文本模式参数。然而，由于 globbing 文本模式是默认的，所以你也可以省略 glob: 标签而只保留文本模式本身。

下面例子中，有两个命令使用 globbing 文本模式。在页面上实际的链接文本是 “Film/Television Department”，使用了文本模式而不是准确的文本，click 命令将继续工作，即使链接文本改为“Film & Television Department” 或 “Film and Television Department”。glob 文本模式的星号会匹配 “Film” 单词和 “Television”单词中间的任何符号或者没有符号。

|  命令  |     目标      |   值      |    
| ------------- | ------------------------------------------- | ------------ |
|   click       |    link=glob:Film\*Television Department    |              |     
|  verifyTitle  |    glob:\*Film\*Television\*                |              |     

点击链接所打开页面的实际标题是 “De Anza Film And Television Department - Menu”。通过使用文本模式而不是准确的文本，只要页面标题中的任何地方前后出现 “Film” 和 “Television” 这两个单词，则 verifyTitle 将验证通过。例如，如果网页作者缩短标题为 “Film & Television Department”，测试会通过。在链接文本或者普通文本中使用文本模式将会大大减小测试案例的维护成本。

# 正则表达式文本模式

---
正则表达式文本模式在 Selenese 支持的三种文本模式中是功能最强大的。很多高级编程语言、很多文本编辑器以及很多工具中都支持正则表达式。包括 Linux / Unix 命令行实用工具 grep、sed 和 awk。在 Selenese 中，正则表达式文本模式允许用户执行许多非常困难的任务。例如，假设测试需要确保一个特定表单元中只包含一个数字。regexp: [0 - 9]+ 这个简单的文本模式，它将匹配任何长度的十进制数。

而 Selenese globbing 文本模式只支持通配符\*和[ ]（字符类）功能，Selenese 正则表达式文本模式提供存在于 JavaScript 中的相同广泛的特殊字符。下面是一个特殊字符的子集：

|  模式              |   匹配                                                            |
| -------- | ------------------------  |
|  .       | 任何单个字符                                                 |
| [ ]      | 字符类：出现在方括号中的任意单个字符   |
|  \*      | 数量：0 个或多个前面的字符（或者组） |
|  + 	   | 数量：1 个或多个前面的字符（或者组）   |
|  ?       | 数量：0 个或 1 个前面的字符（或者组） |
|  {1,5}   | 数量：1 至 5 个 前面的字符（或者组）    |
| &#124;   | 交替：竖道左边的字符/组或者右边的字符/组 |
| ( )      | 组：经常用在 and/or 替换的量                    |

正则表达式文本模式在 Selenese 命令参数中需要用 regexp: 或 regexpi: 前缀。前者是大小写敏感的，后者是不区分大小写的。

下面几个例子将帮助阐明如何使用正则表达式模式与 Selenese 命令。第一个例子是正则表达式最常用的方式 .\* （点星）。这两个字符序列可以表示 0 或出现任何字符或者更简单，任何字符或没有字符。它相当于 globbing 模式中的一个 * （星号）。

|  命令                       |                 目标                                                            |   值  |    
| ----------- | ---------------------------------------- | --- |
| click       | link=regexp:Film.\*Television Department |     |     
| verifyTitle | regexp:.\*Film.\*Television.\*           |     |     

上面的例子与前面使用 globbing 字符模式例子的功能相同。唯一的区别是前缀（regexp: 代替了 glob:）任何字符或没有字符模式（.* 代替了 *）。

下面例子稍微复杂一些，测试 Anchorage Alaska 地区的 Yahoo! 天气页面中的日出时间：

|  命令                                       |                              目标                                               |   值  |    
| ----------------- | ------------------------------------------------ | --- |
|   open            |  http://weather.yahoo.com/forecast/USAK0012.html |     |     
| verifyTextPresent |   regexp:Sunrise: *[0-9]{1,2}:[0-9]{2} [ap]m     |     |     

下面将逐一解释上面的正则表达式：

|  Sunrise: \*| Sunrise: 字符串跟着 0 个或多个空格    | 
| ----------- | --------------------------- |
| [0-9]{1,2}  |  1 或 2 位数字（表示小时）                         |
|  :          |  字符 ： 小时和分钟中间的分割字符                |
| [0-9]{1,2}  |  2 位数字（表示分钟）后面跟一个空格         |
|  [ap]m      |  a 或者 p 后面跟着 m 字符，表示 ap 或者 pm  |

# Exact 文本模式

---
Selenium 的 exact 文本模式只有非常少的使用场合。这种文本模式没有什么特殊字符，所以顾名思义称为准确模式。所以，当你需要匹配一个包含星号的文本时，globbing 和正则表达式都是用星号作为特殊字符，这时 exact 文本模式就派上用场了。例如，你想在下拉列表中选择 “Real *” 选项，下面的代码可能有效或者无效。glob:Real * 文本模式将匹配 Real 开头的任何字符串。这样，如果这个选项前面有一个选项文本为 “Real Numbers”，这个选项将匹配成功，而不是 “Real *”。

|  select |  //select   |  glob:Real *  |
| ------- | ---------   | ------------- |

为了确保 "Real *" 选项被选择，exact: 前缀的 exact 文本模式如下所示：

|  select |  //select   |  exact:Real *  |
| ------- | ---------   | -------------- |

实际上正则表达式文本模式通过转义星号可以实现同样的效果：

|  select |  //select   |  regexp:Real \\*  |
| ------- | ----------- | ---------------- |

测试人员平时很少需要匹配带星号或方括号（globbing 模式的字符类）的文本。因此，globbing 文本模式和正则表达式文本模式能解决绝大多数的字符匹配的问题。

---
[定位元素](Locating.md) | [目录](README.md) | [AndWait 命令](AndWait.md)
