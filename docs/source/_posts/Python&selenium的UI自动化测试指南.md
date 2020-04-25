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

#### selenium的元素定位方法有以下几种：

    find_element_by_id() 通过id属性
    find_element_by_name() 通过name属性
    find_element_by_tag_name() 通过标签名称
    find_element_by_class_name() 通过属性的值
    find_element_by_link_text() 通过超链接文本
    find_element_by_partial_link_text() 通过超链接文本
    find_element_by_xpath() 通过xpath表达式
    find_element_by_css_selector() 通过css表达式
    
 这些方法传入的


 - **第一步**
  在浏览器中打开**开发者工具**（以下操作都使用chrome实现，就是F12），在左上角有鼠标样式的图标，点它之后再去页面中点击你想定位的元素，如下图：
  
  ![baidu_dingwei.png](https://i.loli.net/2020/04/25/3R6PfYQZp5bKBi9.png)
  
  这里能看到输入框的input标签中，id、name、class属性都有，这里我们需要判断这些属性是否是**唯一**的，非常简单，将标签中的属性双击后复制出来用`Ctrl + F`搜索一下就能看到了，就像下图一样：
  
  ![baidu_ysdingwei.png](https://i.loli.net/2020/04/25/pckYugHG7n1LN35.png)
  
  > 这里搜索结果需要注意的是如果搜索结果在全局样式的代码中的话，我们可以忽略这个结果，只查看页面标签中的搜索结果是不是唯一的。

  这里的`kw`是这个input标签的`id`，我们再回顾一下之前demo中的代码：
  ```python
driver.find_element_by_id('kw').send_keys('python')
```
  
  我们在实例化webdriver的对象后，通过调用它的`find_element_by_id()`方法并传入input标签的id属性`kw`，然后使用`send_keys()`方法输入我们想输入的内容，这样就完成了一次输入操作了。

除了`find_element_by_id()`，还有其他的

