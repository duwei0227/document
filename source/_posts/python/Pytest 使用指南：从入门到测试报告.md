---
layout: post
title: Pytest 使用指南：从入门到测试报告
categories: [Python]
permalink: python/pytest/tutorial.html
tags: 自动化
date: {{date}}
---

## 一、入门

### 1、安装`pytest`

```shell
pip install -U pytest
```



### 2、简单使用

编写测试用例：

```python
def func(x):
    return x + 1

def test_answer():
    assert func(3) == 5
```

执行用例：

```shell
pytest  -- 执行当前目录及其子目录下所有格式为 test_*.py或*_test.py 的文件
pytest 文件名称  -- 可以指定执行具体的测试模块
```



## 二、`fixture`的使用

### 1、`fixture`的定义

`fixture` 是 `pytest` 中用于提供测试数据、初始化环境、清理资源等功能的可重用代码块

```python
@pytest.fixture
def function_name():
    paas
```

常用参数：

* `scope`: 作用域，支持的数据集为`function(默认)`,`class`,`module`,`session`
  * **`function`**：每个测试函数执行时创建和销毁 `fixture`，适用于需要独立测试环境的场景。
  * **`class`**：每个测试类执行时创建和销毁 `fixture`，适用于测试类内的测试方法共享资源的场景。
  * **`module`**：每个测试模块执行时创建和销毁 `fixture`，适用于测试模块内的测试共享资源的场景。
  * **`session`**：整个测试会话只创建和销毁一次 `fixture`，适用于全局资源的初始化和清理，如数据库连接、服务器启动等
* `autouse`:当 `fixture` 的 `autouse` 参数设置为 `True` 时，`pytest` 会自动在每个符合作用域（`scope`）的测试函数执行前后调用该 `fixture`，而无需在测试函数的参数列表中指定该 `fixture` 的名称，不同的作用域下`autouse=True`的效果也不同
  * **`function` **：每个测试函数执行前后都会调用该 `fixture`。
  * **`class` **：每个测试类中的所有测试方法执行前后调用一次该 `fixture`。
  * **`module` **：每个测试模块中的所有测试函数和测试方法执行前后调用一次该 `fixture`。
  * **`session **：整个测试会话（即所有测试模块）执行前后调用一次该 `fixture`。
* `name`:`fixture`的名称，默认不指定时，`fixture`的名称和函数名称相同



### 2、`fixture`的示例

#### 2.1 显示引用

```python
@pytest.fixture
def get_one():
    return 1

def test_simple_fixture(get_one):
    assert 1 == get_one
```



#### 2.2 使用`autouse`

```python
@pytest.fixture(autouse=True)
def setup_and_teardown():
    logging.info("Setting up before test")
    yield
    logging.info("Tearing down after test")


def test_function_1():
    logging.info("Executing test function 1")
    assert True


--执行日志如下：
--------------------------------------------------------------- live log setup ------------------------------------INFO     root:test_use_fixture.py:8 Setting up before test
--------------------------------------------------------------- live log call -------------------------------------INFO     root:test_use_fixture.py:14 Executing test function 1
PASSED                [100%]
------------------------------------------------------------- live log teardown -----------------------------------INFO     root:test_use_fixture.py:10 Tearing down after test
```



## 三、`pytest.mark.parametrize`的使用

### 1、定义

`pytest.mark.parametrize` 是 `pytest` 框架里用于参数化测试的一个装饰器

```python
@pytest.mark.parametrize()
def function_name():
    paas
```

参数：

* `argnames`:字符串格式的参数名称
* `argvalues`:可迭代对象（像列表、元组等），其中包含了要传递给测试函数的参数值。如果 `argnames` 有多个参数名，`argvalues` 中的每个元素就应该是一个与参数名数量对应的元组。



### 2、示例

```python
def double(num):
    return num * 2

@pytest.mark.parametrize("input_num, expected", [(1, 2), (3, 6), (5, 10)])
def test_double(input_num, expected):
    result = double(input_num)
    assert result == expected
```



## 三、记录用例日志

### 1、输出日志到控制台

在`pytest.ini`文件中添加以下内容开启控制台日志输出并指定输出日志格式，完整的日志配置可以参考[logging文档](https://docs.python.org/3/library/logging.html)


```ini
[pytest]
log_cli=true
log_cli_level=info
log_cli_format=%(asctime)s [%(levelname)s] %(filename)s:%(levelno)s ==> %(message)s
log_cli_date_format=%Y-%m-%d %H:%M:%S
```



### 2、简单日志文件输出

通过`pytest.ini`中配置`log_file`相关属性，指定文件日志输出

```ini
log_file=app.log
log_file_mode=a
log_file_level=info
log_file_format=%(asctime)s [%(levelname)s] %(filename)s:%(levelno)s ==> %(message)s
log_file_date_format=%Y-%m-%d %H:%M:%S
```



## 四、结合`Allure Report`输出测试报告

### 1、安装`allure report`

```shell
pip install allure-pytest
```



### 2、生成`Allure`测试结果

```shell
pytest --alluredir=./allure-result 
```



### 3、生成`Allure`测试报告

```shell
allure generate -c -o allure-report --name allure-report.html --lang zh --single-file allure-result
```

选项说明：

* `-c`:生成测试报告前清理已有报告
* `-o`:测试报告输出目录，默认为 `allure-report`
* `--name`:生成的测试报告名称
* `--lang`:生成的报告初始显示语言，`zh`:中文
* `--single-file`:是否生成单文件，默认为`false`



### 4、`Allure Report`常用属性

`feature`和`story`按照功能需求维度划分用例，一个`feature`可以有多个`story`，一个`story`下可以有多个`testcase`,`testcase`为具体的测试用例，`issure`用于标记当前用例和一个问题关联，`suite`可以将相关的测试用例组织在一起，可以按照需求维度，也可以按照`testcase`和`issure`维度进行分组。

| 属性 | 定义 | 用途 |
| --- | --- | --- |
| `@allure.feature` | 对测试用例按软件主要功能模块进行高级分类 | 从宏观层面组织测试用例，便于查看不同功能模块的测试情况 |
| `@allure.story` | 在 `@allure.feature` 基础上，按具体业务场景或用户故事细分测试用例 | 明确测试用例针对的具体业务场景，细化测试覆盖范围 |
| `@allure.testcase` | 将测试用例与测试用例管理系统中的条目关联 | 便于查看测试用例的详细信息和历史记录 |
| `@allure.suite` | 指定测试用例所属的测试套件 | 对测试用例进行分组，方便查看和管理 |
| `allure.issue` | 在测试用例中添加问题单链接，关联缺陷管理系统中的问题 | 便于追踪和管理测试中发现的问题 |
| `@allure.tag` | 为测试用例添加自定义标签，可基于测试类型、优先级等维度 | 方便筛选和搜索具有特定属性的测试用例 |
| `@allure.severity` | 标记测试用例的严重程度，`BLOCKER`（阻塞级）、`CRITICAL`（关键级）、`NORMAL`（正常级）、`MINOR`（次要级）、`TRIVIAL`（轻微级） | 确定测试问题的处理优先级，优先关注高严重程度问题 |
| `@allure.description` | 为测试用例添加详细文字描述，说明目的、步骤、预期结果等 | 帮助他人理解测试用例的意图和预期行为 |
| `@allure.title` | 为测试用例指定自定义标题，替代默认函数名 | 使测试用例在报告中显示更具可读性和明确性 |
| `allure.step` | 在测试用例中定义具体测试步骤，每个步骤有描述和结果 | 详细展示测试执行过程，便于定位问题所在步骤 |
| `allure.attach` | 在测试用例中添加附件，如日志、截图、视频等 | 为测试结果分析提供更多上下文信息，辅助问题排查 |
| `allure.link` | 在测试用例中添加外部链接，如文档、问题单等 | 方便查看相关信息，增强测试用例的可追溯性 |
| `allure.epic` | 用于标记测试用例的史诗级别，通常用于大型项目或产品 | 便于从宏观层面组织和管理测试用例 |
| `allure.label` | 用于标记测试用例的自定义标签，如版本、模块等 | 便于对测试用例进行分类和筛选 |

