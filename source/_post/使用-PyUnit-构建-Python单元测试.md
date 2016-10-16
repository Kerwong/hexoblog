---
title: 使用 PyUnit 构建 Python单元测试
date: 2016-10-16 22:25:16
tags:
- python
- test
---
## 概况
Python 单元测试框架（The Python unit testing framework） ，简称为PyUnit ， 是 Kent Beck 和 Erich Gamma 这两位聪明的家伙所设计的 JUnit  的Python 版本。
此文档仅阐述针对Python 的单元测试PyUnit 的设计与使用。
自从 Python 2.1  版本后，PyUnit 成为 Python 标准库的一部分。

## 系统要求
PyUnit 可以在Python 1.5.2 及更高版本上运行。

## 使用PyUnit构建自己的测试
###  安装
编写测试所需的类可以在“unittest” 模块中找到。此模块是Python 2.1 和更高版本的标准库的一部分。
为使此模块能在你的代码中正常工作,你只需确保包含 `unittest.py` 文件的目录在你的Python 搜索路径中。
> 注意，只有完成此项工作才能运行PyUnit 所自带的例子，除非将 `unittest.py` 复制到例子目录。

### 测试用例介绍
单元测试是由一些测试用例（Test Cases） 构建组成的。测试用例是被设置用来检测正确性的单独的场景。在PyUnit 中，unittest 模块中的TestCase 类 代表测试用例。
TestCase类 的实例是可以完全运行测试方法和可选的设置（set-up） 以及清除（tidy-up） 代码的对象。
TestCase 实例的测试代码必须是自包含的，换言之，它可以单独运行或与其它任意数量的测试用例共同运行。

** 以下测试皆为对此类测试 **
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
#
# -- class & functions --
class Widget:
    '''need to test'''
    def __init__(self, size = (40, 40)):
        self._size = size
    def getSize(self):
        return self._size
    def resize(self, width, height):
        if width < 0 or height < 0:
            raise VauleError, "illegal size"
        self._size = (width, height+1)
    def dispose(size):
        pass
```
### 创建一个简单测试用例
通过覆盖runTest方法即可得到最简单的测试用例子类以运行 一些测试代码：
```python
# 静态方法
# 采用静态方法，要为每个要测试的方法编写一个测试类
# （该类通过覆盖runTest()方法来执行测试），
# 并在每一个测试类中生成一个待测试的对象。
class WidgetSizeTestCase(unittest.TestCase):
    def runTest(self):
        widget = Widget()
        self.assertEqual(widget.getSize(), (40, 40))

class WidgetResizeTestCase(unittest.TestCase):
    def runTest(self):
        widget = Widget()
        widget.resize(100, 100)
        self.assertEqual(widget.getSize(), (100, 100))
```
> 注意：为进行测试，我们只是使用了Python 内建的“assert ”语句。如果在测试用例运行时断言（assertion ）为假，AssertionError 异常会被抛出，并且测试框架会认为测试用例失败。其它非“assert ”检查所抛出的异常会被测试框架认为是“errors ”。

运行测试用例的方法会在后面介绍。现在我们只是通过调用无参数的构造器（constructor）  来创建一个测试用例的实例：
```python
testCase = WidgetSizeTestCase()
```
### 复用设置代码：创建固件
这样的测试用例数量巨大且它们的设置需要很多重复性工作。在上面的测试用例中， 如若在100个Widget 测试用例的每一个子类中都创建一个“Widget ”，那会导致难看的重复。
幸运的是，我们可以将这些设置代码提取出来并放置在一个叫做setUp 的 钩子方法（hook method） 中。测试框架会在运行测试时自动调用此方法：
```python
import unittest
class SimpleWidgetTestCase(unittest.TestCase):
    def setUp(self):
        self.widget = Widget("The widget")
    def tearDown(self):
        self.widget.dispose()
        self.widget = None

class WidgetSizeTestCase(SimpleWidgetTestCase):
    def runTest(self):
        widget = Widget()
        self.assertEqual(widget.getSize(), (40, 40))

class WidgetResizeTestCase(SimpleWidgetTestCase):
    def runTest(self):
        widget = Widget()
        widget.resize(100, 100)
        self.assertEqual(widget.getSize(), (100, 100))
```
如果setUp 方法在测试运行时抛出异常，框架会认为测试遇到了错误并且 runTest 不会被执行。
类似的，我们也可以提供一个tearDown 方法来完成在runTest 运行之后的清理工作。
如果setUp 执行成功， 那么无论runTest 是否成功，tearDown 方法都将被执行。
> Such a working environment for the testing code is termed a fixture. 这个测试代码的运行环境被称为固件 (fixture，译者注：此为暂定译法，意为固定的构件或方法)。

### 包含多个测试方法的测试用例类
很多小型测试用例经常会使用相同的固件。在这个用例中，我们最终从SimpleWidgetTestCase 继承产生很多仅包含一个方法的类，如 DefaultWidgetSizeTestCase 。这是很耗时且不被鼓励的，因此，沿用JUnit 的风格，PyUnit 提供了一个更简便的方法：
```python
# 动态方法
class WidgetTestCase(unittest.TestCase):
    # 执行测试类
    # dynamic 测试方法
    # 覆盖unittest中 setUp, 在其中完成初始化
    # 覆盖unittest中tearDown, 释放资源
    # dynamic 测试不需要覆盖 runTest 方法
    def setUp(self):
        self.widget = Widget();
    def tearDown(self):
        self.widget.dispose()
        self.widget = None
    def testSize(self):
        self.assertEqual(self.widget.getSize(), (40, 40))
    def testResize(self):
        self.widget.resize(100, 100)
        self.assertEqual(self.widget.getSize(), (100, 100))
```
在这个用例中，我们没有提供runTest 方法，而是两个不同的测试方法。类实例将创建和销毁各自的self.widget 并运行某一个test 方法。 当创建类实例时，我们必须通过向构造器传递方法的名称来指明哪个测试方法将被运行：
```python
defaultSizeTestCase = WidgetTestCase("testSize")
resizeTestCase = WidgetTestCase("testResize")
```
### 将测试用例聚合成测试套件
测试用例实例可以根据它们所测试的特性组合到一起。PyUnit 为此提供了一个机制叫做”测试套件“（test suite) 。它由unittest模块 中的TestSuite类 表示,在每个测试模块中提供一个返回已创建测试套件的可调用对象，会是一个使测试更加便捷的好方法：
```python
# 测试用例集, 方法一
# 全局函数
def suite():
    suite = unittest.TestSuite()
    suite.addTest(WidgetTestCase("testSize"))
    suite.addTest(WidgetTestCase("testResize"))
    return suite
```
甚至可写成:
```python
# 测试用例集, 方法二
# 定义 TestSuite 子类
class WidgetTestSuite(unittest.TestSuite):
    def __init__(self):
        unittest.TestSuite.__init__(self, map(WidgetTestCase,("testSize","testResize")))

    def suite():
        return WidgetTestSuite()
```
因为创建一个包含很多相似名称的测试方法的TestCase 子类是一种很常见的模式，所以unittest模块 提供一个便捷方法，makeSuite ，来 创建一个由测试用例类内所有测试用例组成的测试套件：
```python
# 测试用例集, 方法三
# 如果用于测试的类中所有的测试方法都以test开头，
# Python程序员可以用PyUnit模块提供的makeSuite()方法来构造一个TestSuite
def suite():
    return unittest.makeSuite(WidgetTestCase, "test")
```
需要注意的是，当使用makeSuite 方法时，测试套件运行每个测试用例的顺序是由测试方法名根据Python 内建函数cmp 所排序的顺序而决定的。

### 嵌套测试套件
我们经常希望将一些测试套件组合在一起来一次性的测试整个系统。这很简单，因为多个TestSuite 可以被加入进另一个TestSuite ，就如同 多个TestCase 被加进一个TestSuite 中一样：
```python
suite1 = module1.TheTestSuite()
suite2 = module2.TheTestSuite()
alltests = unittest.TestSuite((suite1, suite2))
```
### 测试代码放置位置
可以将测试用例定义与被测试代码置于同一个模块中（例如“widget.py ”），但是将测试代码放置在单独的模块中（如“widgettests.py ”）会有一些优势：
* 测试模块可以从命令行单独执行
* 测试代码可以方便地从发布代码中分离
* 少了在缺乏充足理由的情况下为适应被测试代码而更改测试代码的诱惑
* 相对于被测试代码，测试代码不应该被频繁的修改
* 被测试代码可以更方法的进行重构
* 既然C语言代码的测试应该置于单独的模块，那何不保持这个一致性呢？
* 如果测试策略改变，也无需修改被测试源代码
* 交互式运行测试

我们编写测试的主要目的是运行它们并检查我们的软件是否工作正常。测试框架使用“TestRunner”类 来为运行测试提供环境。最常用的TestRunner 是TextTestRunner ， 它可以以文字方式运行测试并报告结果：
```python
# 实施测试
# PyUnit使用TestRunner类作为测试用例的基本执行环境，
# 来驱动整个单元测试过程。
# Python开发人员在进行单元测试时一般不直接使用TestRunner类，
# 而是使用其子类TextTestRunner来完成测试，并将测试结果以文本方式显示出来

# -- start --
if __name__ == '__main__':
    # 构造测试集
    suite = suite()

    # 执行测试
    runner = unittest.TextTestRunner()
    runner.run(suite)
```
TextTestRunner 默认将输出发送到sys.stderr ，但是你可以通过向它的构造器传递一个不同的类似文件（file-object ）对象来改变默认方式。
如需在Python 解释器会话中运行测试，这样使用TextTestRunner 是一个理想的方法。

### 更多关于测试条件
建议过应使用Python 内建断言机制来检查测试用例中的条件，而不应使用自己编写的替代品，因为assert 更简单，简明且为大家所熟悉。
但是值得注意的是，如果在运行测试的同时Python 优化选项被打开（生成“.pyo ”字节码文件），那么assert 语句将会被跳过，使得测试用例变得无用。
我为那些需要使用Python 优化选项的用户编写了一个 assert 方法并添加进TestCase类 内。它的功能和内建的assert 相同且 不会被优化删除，但是使用较麻烦且所输出错误信息帮助较小：
```python
def runTest(self):
    self.assert_(self.widget.size() == (100,100), "size is wrong")
```
我还在TestCase类 中提供了failIf 和failUnless 两个方法：
```python
def runTest(self):
    self.failIf(self.widget.size() <> (100,100))
```
测试方法还可以通过调用fail 方法使得测试立即失败：
```python
def runTest(self):
    ...
    if not hasattr(something, "blah"):
    self.fail("blah missing")
    # or just 'self.fail()'
```
### 测试相等性
最常用的断言是测试相等性。如果断言失败，开发者通常希望看到实际错误值。
TestCase 包含一对方法assertEqual 和assertNotEqual 用于此目的(如果你喜欢，你还可以使用别名：failUnlessEqual  和 failIfEqual ):
```python
def testSomething(self):
    self.widget.resize(100,100)
    self.assertEqual(self.widget.size, (100,100))
```
### 测试异常
测试经常希望检查在某个环境中是否出现异常。如果期待的异常没有抛出，测试将失败。这很容易做到：
```python
def runTest(self):
    try:
        self.widget.resize(-1,-1)
    except ValueError:
        pass
    else:
        fail("expected a ValueError")
```
通常，预期异常源（译者注：将抛出异常的代码）是一个可调用对象；为此，TestCase 有一个assertRaises 方法。此方法的前两个参数是应该出现在“except ”语句中的异常和可调用对象。剩余的参数是应该传递给可调用对象的参数。
```python
def runTest(self):
    self.assertRaises(ValueError, self.widget.resize, -1, -1)
```

## 附录
完整测试代码
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
#
# Copyright © 2013 Al™, All Rights Reserved.
#

'''
Created on 2013/03/21 14:48:49
@module: unitest_test.py
@version:
@license: Copyright © 2013 Al™, All Rights Reserved.
@author: Wang Wenchao
@contact: U{B{Wang Wenchao}}
@see:
@note:
'''

# -- modules --
from testmodule import Widget
import unittest

# -- global --

# -- class & functions --

# ============================================================================
# 静态方法
# 采用静态方法，要为每个要测试的方法编写一个测试类
# （该类通过覆盖runTest()方法来执行测试），
# 并在每一个测试类中生成一个待测试的对象。
#class WidgetSizeTestCase(unittest.TestCase):
#   def runTest(self):
#       widget = Widget()
#       self.assertEqual(widget.getSize(), (40, 40))

#class WidgetResizeTestCase(unittest.TestCase):
#   def runTest(self):
#       widget = Widget()
#       widget.resize(100, 100)
#       self.assertEqual(widget.getSize(), (100, 100))

# 动态方法
class WidgetTestCase(unittest.TestCase):
    # 执行测试类
    # dynamic 测试方法
    # 覆盖unittest中 setUp, 在其中完成初始化
    # 覆盖unittest中tearDown, 释放资源
    # dynamic 测试不需要覆盖 runTest 方法
    def setUp(self):
        self.widget = Widget();
    def tearDown(self):
        self.widget.dispose()
        self.widget = None
    def testSize(self):
        self.assertEqual(self.widget.getSize(), (40, 40))
    def testResize(self):
        self.widget.resize(100, 100)
        self.assertEqual(self.widget.getSize(), (100, 100))

# ============================================================================
# 测试用例集, 方法一
# 全局函数
#def suite():
#   suite = unittest.TestSuite()
#   suite.addTest(WidgetTestCase("testSize"))
#   suite.addTest(WidgetTestCase("testResize"))
#   return suite

# ----------------------------------------------------------------------------
# 测试用例集, 方法二
# 定义 TestSuite 子类
#class WidgetTestSuite(unittest.TestSuite):
#   def __init__(self):
#       unittest.TestSuite.__init__(self, map(WidgetTestCase,("testSize","testResize")))

#   def suite():
#       return WidgetTestSuite()

# ----------------------------------------------------------------------------
# 测试用例集, 方法三
# 如果用于测试的类中所有的测试方法都以test开头，
# Python程序员可以用PyUnit模块提供的makeSuite()方法来构造一个TestSuite
def suite():
    return unittest.makeSuite(WidgetTestCase, "test")

# ============================================================================

# 实施测试
# PyUnit使用TestRunner类作为测试用例的基本执行环境，
# 来驱动整个单元测试过程。
# Python开发人员在进行单元测试时一般不直接使用TestRunner类，
# 而是使用其子类TextTestRunner来完成测试，并将测试结果以文本方式显示出来

# -- start --
if __name__ == '__main__':
    # 构造测试集
    suite = suite()

    # 执行测试
    runner = unittest.TextTestRunner()
    runner.run(suite)
```
运行输出
```
wang@Wang-Satellite-M300:~/Workspace/Python/Template/unittest$ ./unittest_test.py
F.
======================================================================
FAIL: testResize (__main__.WidgetTestCase)
———————————————————————-
Traceback (most recent call last):
File "./unittest_test.py", line 58, in testResize
self.assertEqual(self.widget.getSize(), (100, 100))
AssertionError: Tuples differ: (100, 101) != (100, 100)

First differing element 1:
101
100

- (100, 101)
? ^

+ (100, 100)
? ^

———————————————————————-
Ran 2 tests in 0.001s

FAILED (failures=1)
```
