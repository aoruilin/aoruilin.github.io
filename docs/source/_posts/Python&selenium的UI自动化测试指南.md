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

*至此，我们的环境就搭建好了，总结一下的话基本只需要把浏览器和对应的驱动弄好就完成了。下面让我们开始写代码吧~*

---


## selenium


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
 
 
 *我们已经完成了所有的准备工作，接下来我们就开始写代码吧~*
 
 ```python
import time

from selenium import webdriver

driver = webdriver.Chrome()  # 实例化一个driver对象，打开Chrome浏览器
driver.get('https://www.baidu.com')  # 访问百度
driver.find_element_by_id('kw').send_keys('python')
time.sleep(1)
driver.find_element_by_id('su').click()
time.sleep(3)
driver.close()  # 关闭浏览器，释放资源

```


