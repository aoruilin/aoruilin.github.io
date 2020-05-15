---
title: 基于Python+selenium的UI自动化测试指南
abbrlink: 973aa772
date: 2020-04-16 22:47:29
tags: 
    - Python
    - selenium
category:
  - Python
  - 自动化测试
comments: true
---

# Python&selenium自动化测试

## 简介

*本篇将介绍如何使用`Python`和`selenium`，从编写你的第一个测试脚本到创建一个web端的UI自动化测试项目。主要针对刚开始或准备接触自动化测试的人，我们假设你已经拥有了基本的Python技能并且有一些面向对象编程的经验，那么这篇文章应该会对你有所帮助。*

---

## 环境搭建

 > 在开始前，我们先来了解一下一条自动化脚本是如何运作的，如果预先了解过或者见过自动化脚本运行效果就会知道，其实就是通过我们编写的代码运行后使浏览器去执行我们制定的各种操作步骤。这个场景就和坐出租车一样，乘客告诉司机要去哪，司机按照乘客的要求开车把我们送到目的地，在这里我们就是乘客，现在我们需要的就是**司机**和**车**。

### 1.浏览器
 > 浏览器就是我们需要的**车**，一个web端项目在开发、调试、测试的过程中主要使用的浏览器就是chrome和Firefox了，我们主要介绍一下这两个浏览器。
 
 - **chrome**
 在写这篇文章时chrome稳定版已经更新到81的版本了，最新版本前两天刚踩了坑，所以推荐使用60-79范围中的版本，因为我们需要一个稳定的运行环境，而且我们做的是自动化测试，不需要考虑兼容性测试。
 这里肯定有人会问，chrome有自动更新啊！解决方法有几种，但我不保证对你就有用（手动狗头）。
  - a.打开C:\Program Files (x86)\Google，强行删除Update目录
  - b.安装的时候把网断掉
  
 以上的方法都能用一段时间，如果它还是自己更新了，emmmmm...删掉重新再装一次那个版本吧，所以最好把浏览器安装包统一管理起来，一是好找，二是如果要测兼容也可以派上用场。
 
 - **Firefox**
 推荐使用55以后的版本，Firefox关闭自动更新就很简单了，只需要几步，在设置中找到高级，点到更新后选择不检查更新，搞定~

### 2.浏览器驱动

 > 现在我们有了车，需要的就剩**司机**了，但是就像出租车和司机都有自己对应的出租车公司一样，浏览器和浏览器驱动也需要对号入座，而且每一个浏览器驱动的版本也需要和特定的几个浏览器版本对应。


 - **chromedriver**
 chromedriver是chrome浏览器指定的驱动，按照之前推荐的浏览器版本，我们需要下载一个对应的驱动版本，版本对照如下图：
 ![llqdz (2).png](https://i.loli.net/2020/04/21/TtESH41G8Pdyi7K.png)
 
 最近较新的chrome版本也会有单独对应的驱动版本，我们可以根据安装的chrome版本[下载](https://npm.taobao.org/mirrors/chromedriver/)对应的驱动


 - **geckodriver**
 geckodriver是Firefox浏览器指定的驱动，最新版本的Firefox我暂时还没踩到坑，所以对应的浏览器驱动版本我们也可以[下载](https://npm.taobao.org/mirrors/geckodriver/v0.26.0/)最新的。


 > 至于其他的浏览器驱动比如IE的IEDriverServer，IE呢......emmmm......你们应该懂的。

 **驱动下载好后是一个`.exe`文件，我们可以把所有的驱动文件放到一个`driver`文件夹中，再将这个文件夹的路径添加到系统变量`Path`中。**
 > 如果觉得太麻烦也有偷懒的方法，直接把文件放到一个文件夹中，然后丢到你的Python安装目录下就行了。。

*至此，我们的环境就搭建好了，总结一下的话基本只需要把浏览器和对应的驱动弄好就完成了。*

---


## selenium

> *在开始写代码之前不要忘了还需要在**Python环境**中安装`selenium`，我们之后在网页上的所有操作都会通过它来实现。*


selenium简介
 <details>
   <summary>selenium是...</summary>

 > selenium是一个涵盖了一系列工具和库的总体项目，这些工具和库支持Web浏览器的自动化。它提供了扩展，以模拟用户与浏览器的交互，用于扩展浏览器分配的分发服务器，以及用于实现W3C WebDriver规范的基础结构 ，使你可以为所有主要的Web浏览器编写可互换的代码。Selenium的核心是WebDriver，它是编写可以在许多浏览器中互换运行的指令集的接口。
 </details>
   
#### 安装`selenium`
  ```bash
  pip install selenium
  ```

 > selenium支持多个浏览器，支持分布式运行且支持多种语言开发，能完成网页上的大部分操作，但是一些非网页操作比如文件上传操作，打开一个Windows资源管理器窗口选择上传的文件，这种操作是selenium无法完成的。

#### selenium的`webdriver`
 
  webdriver是selenium的核心，我们主要通过调用它的API来实现在浏览器的各种操作，打开selenium目录就会看到如下：
```
 ├─common
 ├─webdriver
```
 common中主要包含了各个`Exception`类，主要用法我们之后介绍。我们再看一看webdriver的目录：
```
├─android
├─blackberry
├─chrome
├─common
├─edge
├─firefox
├─ie
├─opera
├─phantomjs
├─remote
├─safari
├─support
├─webkitgtk
```
 从这里我们就可以看出我们可以使用哪些浏览器来做自动化测试了，在本文中会主要使用`chrome`来完成。
 
 
 ## My First Script
 
 
> *我们已经完成了所有的准备工作，接下来我们就开始写代码吧~*

**先来看一个demo**
 ```python
from selenium import webdriver

driver = webdriver.Chrome()  # 实例化一个driver对象，打开Chrome浏览器
driver.get('https://www.baidu.com')  # 访问百度
driver.find_element_by_id('kw').send_keys('python')  # 在搜索输入框输入
driver.find_element_by_id('su').click()  # 点击搜索
driver.close()  # 关闭浏览器，释放资源

```
我们先不看代码的部分，直接运行这个demo我们可以看到整个的操作和代码中的**注释**中描述的一样，如果你觉得操作的太快看不清楚的话可以在各个操作中间加上**等待**，就像这样：

> 等待的操作在自动化测试中是很必要的，selenium的等待方式有3种，这里我们先介绍强制等待

```python
import time

from selenium import webdriver

driver = webdriver.Chrome()
driver.get('https://www.baidu.com')
time.sleep(2)  # 强制等待2秒
driver.find_element_by_id('kw').send_keys('python')
```
在你想要停顿的地方插入等待的代码后再次运行demo就会看到你想要的效果了。

**那么，这些操作到底是怎么做到的呢？** 
- 同样是出租车的例子，我们作为乘客下达指令，司机开车到达目的地。这里我们需要思考一下我们要通过什么告诉司机我们要去的具体位置呢？

**下面我们需要了解的就是selenium识别网页页面元素的方法了！**


### 页面元素定位的方法

 > selenium的识别原理是通过元素的一个唯一的属性和属性值去HTML页面中定位和识别元素，我们可以通过标签的id、name、class属性或者标签的名称等方法定位到我们想定位的元素位置，然后传入webdriver的方法中取做各种操作。

**selenium的元素定位方法有以下几种：**

    find_element_by_id() 通过id属性
    find_element_by_name() 通过name属性
    find_element_by_tag_name() 通过标签名称
    find_element_by_class_name() 通过属性的值
    find_element_by_link_text() 通过超链接文本
    find_element_by_partial_link_text() 通过超链接文本
    find_element_by_xpath() 通过xpath表达式
    find_element_by_css_selector() 通过css表达式
    
   这些方法都会返回一个元素`element`对象，我们可以使用`send_keys()`或`click()`等方法对这个对象进行想要的操作。
    
 这些方法需要的**参数**怎么确定呢？
 

 - **第一步**
  在浏览器中打开**开发者工具**（以下操作都使用chrome实现，就是F12），在左上角有鼠标样式的图标，点它之后再去页面中点击你想定位的元素，如下图：
  
  ![baidu_dingwei.png](https://i.loli.net/2020/04/25/3R6PfYQZp5bKBi9.png)
 
 - **第二步**
  这里能看到输入框的input标签中，id、name、class属性都有，这里我们需要判断这些属性是否是**唯一**的，非常简单，将标签中的属性双击后复制出来用`Ctrl + F`搜索一下就能看到了，就像下图一样：
  
  ![baidu_ysdingwei.png](https://i.loli.net/2020/04/25/pckYugHG7n1LN35.png)
  
  > 这里搜索结果需要注意的是如果搜索结果在全局样式的代码中的话，我们可以忽略这个结果，只查看页面标签中的搜索结果是不是唯一的。**tip：**form标签下的子标签的id都是唯一的，可以直接用哦~


这里的`kw`是这个input标签的`id`，我们已经知道了我们想要传入的参数是`id`，所以我们需要使用对应的方法来完成定位，我们再回顾一下之前demo中的代码：
  ```python
driver.find_element_by_id('kw').send_keys('python')
```
  
  我们在实例化webdriver的对象后，通过调用它的`find_element_by_id()`方法并传入input标签的id属性`kw`，然后使用`send_keys()`方法输入我们想输入的内容，这样就完成了一次输入操作了。
  

 > *其他的定位方法比如`name`和`class_name`的查找也是一样，找到定位参数传入相应的定位方法中即可，其他定位方法要详细介绍的话都可以单独写一篇了（后面应该会写一篇xpath的），如果之前没有了解的话可以自行百度&Google一下**xpath**和**css_selector**，这里不过多介绍。*



如果需要一次定位到**多个元素**，也有对应的方法：

    find_elements_by_id() 通过id属性
    find_elements_by_name() 通过name属性
    find_elements_by_tag_name() 通过标签名称
    find_elements_by_class_name() 通过属性的值
    find_elements_by_link_text() 通过超链接文本
    find_elements_by_partial_link_text() 通过超链接文本
    find_elements_by_xpath() 通过xpath表达式
    find_elements_by_css_selector() 通过css表达式

这些方法在定位到这些元素后，会将这些元素装入列表并返回，我们也可以对这个列表中的元素进行操作：

```python
import time

from selenium import webdriver


driver = webdriver.Chrome()
driver.get('https://www.baidu.com')
driver.find_element_by_id('kw').send_keys('日历')  # 搜索日历
driver.find_element_by_id('su').click()
time.sleep(3)  # 根据你的网页加载时间可以增加或减少等待的时间
date_elements_list = driver.find_elements_by_xpath('//table[@class="op-calendar-pc-table"]'
                                                   '/descendant::tr[4]/descendant::a')  # 定位日历的第三排中的所有日期元素
for date_element in date_elements_list:
    date_element.click()  # 遍历日期元素列表，对每个元素进行点击
    time.sleep(0.2)  # 加一个短暂等待以便看出运行效果
driver.close()
```

运行这段代码看看效果吧~


### 设置等待和等待时间

 > 在demo运行时，我们可以看到点击、输入这些操作的速度非常快，既然这么快，为什么要设置等待时间呢？
 > 可以设定一个场景，如果性能不稳定或者运行环境的网络速度不稳定使页面加载变得很慢，会遇到一种情况就是我们的代码已经运行了，但是页面还没加载出来，最终的结果就是报错后退出运行，这样使得我们的脚本非常不稳定，怎么解决呢？
 > 这时候设置**等待**就派上用场了~

**我们常用的等待方式有三种，一种是之前介绍的强制等待，另外两种是selenium的等待方法：**


 - **隐式等待**
 
   > 设置一个粘性超时，以隐式地等待找到一个元素或完成一个命令。
 
```python
driver.implicitly_wait(time_to_wait)
```

   加入这行代码后，在运行时会按照传入的`time_to_wait`参数的值对应的秒数等待页面所有的元素加载完成，我们将它带入之前的demo中看看效果吧

   ```python
from selenium import webdriver
    
driver = webdriver.Chrome()  # 实例化一个driver对象，打开Chrome浏览器
driver.implicitly_wait(5)  # 隐式等待5秒
driver.get('https://www.baidu.com')  # 访问百度
driver.find_element_by_id('kws').send_keys('python')  # 这里将id改成了错误的以便看出效果
   ```

 > *注意隐式等待只需要设置一次，它会在整个周期中都起作用*

   运行这段代码后可以看到，在打开网站后等待了5秒，最后抛出一个异常后停止运行，如果没有这行代码，在打开网站后就会马上抛出异常，假设我们的定位是正确的，但我们要找的元素不会再页面加载完后马上出现，这种情况就需要隐式等待了。
   
   *这里如果换成另外一种情况，我们想要的元素已经加载完成了，但是有个别元素还没有完成加载，设置了隐式等待后我们仍需要等待页面全部加载完成才执行下一步，这样就不是很灵活，所以接下来我们来介绍一下显示等待。*

 - **显示等待**
 
 > 传入driver对象，设置等待时长和检查间隔，可以灵活地对特定的元素进行等待。

```python
from selenium.webdriver.support.wait import WebDriverWait

WebDriverWait(driver=driver, timeout=12, poll_frequency=0.6).until(method, message='')
WebDriverWait(driver=driver, timeout=12, poll_frequency=0.6).until_not(method, message='')
```

`driver`和`timeout`是必要参数，driver即我们实例化的webdriver对象，timeout即设置的等待时间，超过设置的等待时间后抛出TimeoutException。这里用到的两种方法`until`和`until_not`，需要传入一个**method**方法作为参数，我们传入后会在设置的等待时间中循环调用这个method，until会在传入的method返回值不为False时停止循环并返回，until_not会在返回值为False时停止循环并返回。
`poll_frequency`是调用的频率，这里设置为每0.6秒调用一次，不写该参数时默认0.5秒。
另外还有一个参数为`ignored_exceptions`，即在调用时需要忽略的异常，默认为NoSuchElementException，通常我们不用写这个参数，默认即可。

我们可以使用`expected_conditions`模块中的各个类传入until和until_not的method参数，就像这样：

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as ec
from selenium.webdriver.support.wait import WebDriverWait

driver = webdriver.Chrome()
driver.implicitly_wait(5)
driver.get('https://www.baidu.com')

input_loc = By.ID, 'kw'  # 定位器
input_element = WebDriverWait(driver, 15).until(ec.presence_of_element_located(input_loc))  # 等待搜索输入框加载完成，完成后返回输入框元素对象
input_element.send_keys('python')

driver.find_element_by_id('su').click()
driver.close()
```

下面列举几个常用的expected_conditions模块提供的条件判断：

 - `presence_of_element_located(locator)` -> 参数传入定位器，判断定位的元素是否在DOM中加载完成（可以使不可见的元素），定位到元素后返回该元素
 
 - `visibility_of_element_located(locator)` -> 参数传入定位器，用于检查某个元素是否存在于页面的DOM上并且是可见的。可见性意味着不仅显示元素，而且元素的高度和宽度也大于0
 
 - `visibility_of(element)` -> 传入已知的存在于DOM中的元素，检查这个元素是否可见，一旦是可见的就返回这个元素
 
 - `visibility_of_any_elements_located(locator)` -> 参数传入定位器，检查web页面上至少有一个期望的元素是可见，定位到元素后返回这些元素的列表
 
 - `invisibility_of_element(invisibility_of_element_located)` -> 参数传入希望不可见的定位器，用于检查某个元素是否在DOM上不可见或不存在的期望
 
 - `title_is(title)` -> 参数传入预期的title字符串，检查页面标题是否符合预期，使用精确匹配，匹配返回True，若不匹配返回False
 
 - `title_contains(title)` -> 参数传入预期的title字符串，检查页面标题是否包含预期，使用模糊匹配，匹配返回True，若不匹配返回False
 
 - `url_contains(url)` -> 参数传入预期的url，检查当前url是否包含预期的url，使用模糊匹配，匹配返回True，若不匹配返回False
 
 - `url_matches(url)` -> 参数传入预期的url，检查当前url是否符合预期的url，使用精确匹配，匹配返回True，若不匹配返回False


需进一步了解所有预期条件判断可以查看`expected_conditions`的源码。


## 测试框架

 > 现在我们已经能通过调用selenium的webdriver中的API实现网页操作了，但我们的目的是通过这些操作来实现自动化测试，其中必不可少的就是测试框架了，常用的测试框架包括`unittest`、`pytest`、`doctest`等等，我们这里主要使用**unittest**来完成。

### unittest创建用例

 unittest是Python内置的标准库，通过继承`unittest.TestCase`来创建测试用例，使用时需要注意的是你的方法名称必须以**test**开头，没有以此开头的都不会被执行。更加具体的介绍可以参考[文档](https://docs.python.org/2/library/unittest.html)，这里我们直接开始撸代码吧~
 
 创建一个项目文件夹，在文件夹中创建名为`test_case.py`的文件，写入以下代码：
 
 ```python
import unittest  # 导入unittest

from selenium import webdriver


class MyTest(unittest.TestCase):  # 继承unittest.TestCase创建测试用例

    def setUp(self) -> None:  # 测试用例运行前的设置
        self.driver = webdriver.Chrome()
        self.driver.implicitly_wait(8)
        self.base_url = 'https://www.baidu.com'

    def test_baidu(self):
        self.driver.get(self.base_url)

        title = self.driver.title
        tc = unittest.TestCase()  # 实例化TestCase对象
        tc.assertEqual('百度一下，你就知道', title, '标签页标题错误')  # 使用assertEquil方法断言标签名称

    def tearDown(self) -> None:  # 释放资源
        self.driver.close()


if __name__ == '__main__':
    unittest.main()
```

这段代码中的`setUp`和`tearDown`方法是unittest框架中的测试固件，setUp是定义一些有固定用途的方法，tearDown用于用例运行完后释放资源，中间以`test_`开头命名的方法就是测试用例，在运行时，它们会按照特定的顺序依次运行，先运行setUp，然后是test开头命名的测试用例*（测试用例可以有多个，默认以方法名首字母排序的顺序运行，当然你也可以在命名时在"test_"后面加上阿拉伯数字来规定你的用例运行顺序，只是这样的命名不是很优雅）*，最后运行tearDown，可以参考下图：
![yunxing (2).png](https://i.loli.net/2020/05/05/DhMLxjQyYVafoqF.png)

测试用例都会检查实际和期望结果是否一致，要实现这个操作就要用到代码中的`tc.assertEqual()`了，这个方法接收3个参数，前两个必要参数`first`和`second`写入预期和实际结果，如果`first != second`，则抛出`AssertionError`并中断运行，`msg`参数是抛出异常的描述，可不传。

> 除了assertEqual，框架中还有很多其他的断言方法（具体参考**unittest.TestCase**源码），比如`assertNotEqual`，参数与assertEqual方法相同，区别是如果`first == second`，则抛出`AssertionError`并中断运行。

现在我们已经能使用unittest创建测试用例了，那么创建的用例我们每次要一个个点运行来执行吗？
当然不需要，我们要实现的是自动执行所有测试用例*PS:这样也只能算半自动化测试*，接下来介绍unittest中的用例集成。

### unittest用例集成

 要将所有创建的用例集成起来一起运行，需要用到`unittest.suite`的`TestSuite`类，使用方法为创建一个**TestSuite实例**，然后添加测试用例实例。
 将所有测试用例添加到**TestSuite实例**中后，再实例一个**runner**对象，这个runner可以使用TextTestRunner(`unittest.TextTestRunner()`)创建，也可以使用`HTMLTestRunner`等第三方工具来创建并输出报告
 
 创建一个test_suits.py文件，写入以下代码：
 
 ```python
from unittest.suite import TestSuite
from unittest import TextTestRunner

from test_case import MyTest  # 导入测试用例

suite = TestSuite()  # 实例化测试集suite对象

# 可以创建多个用例并添加到测试集中
# test_用例号 = 类名('方法名')
test_01 = MyTest('test_baidu')  # 实例化测试用例对象

suite.addTests([test_01])  # 将测试用例加载到测试集中

runner = TextTestRunner()  # 实例化运行runner对象
runner.run(suite)  # 使用runner运行测试集

```
运行时会按照你定义的**用例编号**的顺序运行，在运行时会输出测试用例的名称，在发生错误时输出报错信息，并在测试运行结束时打印出结果。
如果觉得这样的输出不够全面，可以使用第三方工具来输出报告，但这里不做介绍了，`HTMLTestRunner`和[`allure`](http://allure.qatools.ru/)都是不错的选择，用法也很简单，可以自行了解。



## POM设计思想

 > 之前我们介绍了测试框架unittest，并且使用这个框架创建并集成运行测试用例，以后所有的测试用例都会按照这个方法来开发和调试，但是如果每写一个用例都要从打开浏览器写到关闭浏览器的话，很多步骤就重复了，这也代表我们的代码也有**很多重复**的，这会导致代码维护成本越来越大，如果**前端UI变化**非常频繁的话，这样的代码维护起来简直就是噩梦。。。
 怎么解决呢？
 我们来看接下来介绍的一种框架设计思想-POM
 
 
 **POM**,即**Page Object Model**，用中文就是页面对象模型，这是一种非常普遍的框架设计思想，POM是解决问题的一种思想，并不是框架，在这里我们主要用来解决前端UI变化频繁导致我们代码维护成本过大的问题。
 我们现在的测试用例把页面元素和页面操作代码和测试用例都写在了一起，使用POM后，区别就是把页面元素和页面操作代码从测试用例分离出来单独写成一个**PageObject类**文件，如果前端UI有变化，我们就只需要改**PageObject类**中的代码，不需要改测试用例。
 经过文章之前的操作，我们创建了一个test_case文件和一个test_suits文件，现在我们创建一个`page_object.py`文件，将页面元素和操作代码分离到这个文件中，代码如下：**(代码有点长，建议点代码区域右上角复制到你自己的编辑器中查看)**
 
 ```python
import time
from unittest import TestCase
from selenium.common.exceptions import NoSuchElementException, ElementNotVisibleException
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC


class ElementLocator:
    input_loc = By.ID, 'kw'  # 输入框
    search_btn_loc = By.ID, 'su'  # 搜索按钮
    tab_inner_loc = By.CLASS_NAME, 's_tab_inner'  # 顶部tab
    tab_inner_web_loc = By.XPATH, '//div[@class="s_tab_inner"]/b'  # 顶部网页tab


class PageObject:
    def __init__(self, driver):
        self.driver = driver

    def click_button(self, btn_loc, wait=True):
        """
        单独点击一个按钮
        :param btn_loc: 按钮定位器
        :param wait: 是否等待
        :return: None
        """
        if wait:
            time.sleep(0.5)
            self.driver.find_element(*btn_loc).click()
        else:
            self.driver.find_element(*btn_loc).click()

    def send_text(self, input_loc, text):
        """
        输入文本
        :param input_loc: 输入框定位器
        :param text: 输入文本内容
        :return: None
        """
        self.driver.find_element(*input_loc).send_keys(text)

    def webdriver_wait(self, timeout):
        return WebDriverWait(self.driver, timeout)

    @staticmethod
    def element_presence(element_loc):
        """
        判断页面中是否存在这个元素
        :param element_loc: 元素定位，传入元组(by, locator)
        :return:
        """
        return EC.presence_of_element_located(element_loc)

    def assert_equal(self, text, text_loc):
        """
        断言文本相等
        :param text: 期望文本
        :param text_loc: 实际文本定位器
        :return: None
        """
        try:
            wait = self.webdriver_wait(15)
            show_up = self.element_presence(text_loc)
            wait.until(show_up)
            actual_text = self.driver.find_element(*text_loc).text
            print(f'期望： "{text}", 实际： "{actual_text}"')
        except NoSuchElementException:
            print(f'没有{text}')
        except ElementNotVisibleException:
            print(f'未找到{text}元素')
        except Exception as e:
            print(f'{e}， {text}断言异常，与期望不符')
        else:
            tc = TestCase()
            tc.assertEqual(text, actual_text)

    def assert_in(self, text, text_loc, reverse=False):
        """
        断言文本在页面中
        :param text: 期望文本
        :param text_loc: 实际文本定位器
        :param reverse: True->取出文本all_text在text中，False->text在取出文本all_text中
        :return: None
        """
        try:
            wait = self.webdriver_wait(15)
            show_up = self.element_presence(text_loc)
            wait.until(show_up)
            actual_text = self.driver.find_element(*text_loc).text
            print(f'所有文本：{actual_text}')
        except NoSuchElementException:
            print(f'没有{text}')
        except ElementNotVisibleException:
            print(f'未找到{text}元素')
        except Exception as e:
            print(f'"{e}, {text}断言异常，不在页面中')
        else:
            tc = TestCase()
            if reverse:
                tc.assertIn(actual_text, text)
            else:
                tc.assertIn(text, actual_text)

    def assert_text_in_page(self, text, text_loc):
        """
        断言文本在指定区域中
        :param text: 期望文本
        :param text_loc: 实际文本定位器
        :return:
        """
        try:
            element_list = self.driver.find_elements(*text_loc)
            text_list = []
            for s in element_list:
                actual_text = s.text
                text_list.append(actual_text)
            all_text = ''.join(text_list)
        except NoSuchElementException:
            print(f'没有{text}')
        except ElementNotVisibleException:
            print(f'未找到{text}元素')
        except Exception as e:
            print(f'{e}, {text}断言异常，不在指定区域中')
        else:
            tc = TestCase()
            tc.assertIn(text, all_text)

    def baidu_search(self, search_content):
        """
        搜索操作
        :param search_content:搜索内容
        :return:
        """
        self.send_text(ElementLocator.input_loc, search_content)
        self.click_button(ElementLocator.search_btn_loc, wait=True)

```

在这个文件中，一共有两个类，`ElementLocator`类中将所有的元素定位器写进去统一管理，前端UI变化后我们就可以直接在这个类中找到对应的元素修改就行了。`PageObject`类中封装了一些常用的方法，比如点击，输入，等待和断言，还有很多其他的方法比如刷新页面，切换窗口等等都可以封装在这里，这个类最后一个方法`baidu_search`就是页面操作了，这里每个方法中的操作颗粒度可以根据你的用例情况来决定要多细，在用例的类中，我们就通过调用这些方法来完成不同的操作步骤。

让我们再来看看用例的类文件，在之前创建的`test_case.py`文件中，我们修改我们的代码如下：
```python
import unittest
from selenium import webdriver

from page_object import ElementLocator, PageObject  # 导入之前创建的两个类


class MyTest(unittest.TestCase):
    def setUp(self) -> None:
        self.driver = webdriver.Chrome()
        self.driver.implicitly_wait(8)
        self.base_url = 'https://www.baidu.com'

        self.page = PageObject(self.driver)  # 实例化页面操作对象
        self.element = ElementLocator()  # 实例化元素定位器对象

    def test_baidu(self):
        self.driver.get(self.base_url)
        self.page.baidu_search('selenium')  # 搜索selenium的操作
        self.page.assert_in('网页', self.element.tab_inner_loc)  # 断言操作
        self.page.assert_equal('网页', self.element.tab_inner_web_loc)  # 断言操作

    def tearDown(self) -> None:
        self.driver.close()


if __name__ == '__main__':
    unittest.main()

```
现在如果前端UI更改甚至需求更改的话，我们就只需要更改`page_object.py`中的两个类中的代码就行了，特别是将页面元素定位器统一管理后，可以有效的减少维护成本，再创建新的用例也能方便许多。


## 代码结构与配置文件

 > 时间不够了，后面继续写。。。。
