# WINP

WINP is not Pocsuite

## 综述

漏洞验证框架

### 简介

在项目根目录下，执行命令行指令

```>> python WINP.py -f test.py -t www.baidu.com ```

来运行Winp，其中-f执行POC代码文件，-t指定验证目标

### POC代码文件样例

``` python
#!/usr/bin/env python
# coding: gbk

from winp import POC # 导入POC标记

POC_NAME = 'POC 1' # POC代码名称

POC_LEVEL = 3 # 该漏洞的危险等级

@POC # 标记该函数为POC验证函数
def test1(target, extra): # 函数声明，target为传入的单个验证目标，extra为额外结果输出

    ... # 具体验证过程

    extra['digest'] = myarg # 额外信息摘要将作为主要结果输出

    extra['myextra'] = myextra # 将有/无漏洞以外的结果保存到extra中，仅保存到json文件中

    if ...: # 输出类似以下方式
        return True # 返回真表示该目标有此种漏洞
    else:
        return False # 返回假表示该目标没有此种漏洞
    
```
_注意，文件编码请使用GBK编码(或者GB2312、GB18030等中文Windows默认使用的编码)_

_注意：请使用函数传入的result参数来进行操作，若另外建立一个dict，Winp将无法获取结果_

_注意：所有字符串不要出现u前缀，例如：_

``` python
mystr = '这是一个中文字符串'
```

_POC验证函数的参数个数可以为1个或者2个，extra可以省略_

### 详细用法

Winp运行环境为Python2.7，请勿在Python3下运行

#### 命令行参数

+ -f POC代码文件或目录

指定用于执行验证函数的文件或者目录，按照以下规则解析

* 若指定.py文件或.pyc文件，则将该文件作为POC代码文件
* 若指定目录，则将该目录下(包括所有子目录下)所有.py文件和.pyc文件作为POC代码文件
* 若指定.txt文件(指定的txt中的每一行应为一个.py或.pyc文件的路径)，则将此txt中这些路径的.py或.pyc文件作为POC代码文件
* 若该参数缺省，默认为pocs目录

+ -t 验证目标

用于指定验证的目标，按照以下规则解析

* 若格式为"xxx.xxx.xxx.xxx-xxx.xxx.xxx.xxx"，则将两个ip段内所有的ip都加入目标（两个或以上参数不同时，各个数字位点互不影响）
* 若格式为"xxx.xxx.xxx.*"（其中4个数字位置多个位置为星号），则将该位置按照0~255遍历
* 若不符合以上任何格式，则按照当做单个目标加入队列
* 若指定的目标是一个存在的文件（无论格式），则读取该文件，每一行都按照以上规则添加一次目标
* 若该参数缺省，默认为targets.txt文件

+ -l 语言

用于指定项目运行期间语言，默认语言为中文

* 使用参数zh-CN表示中文
* 使用参数en-US表示英文
* 暂时不支持其他语言/地区
* 语言参数和您编写代码使用的语言无关
* 语言模块加载失败时可能使用英语表达

+ -o 输出文档

用于指定运行结果输出到指定目标，默认为console（控制台），按照以下规则解析

* 若指定的后缀名为.html或者.htm，则按照静态网页格式输出
* 若指定的后缀名为.txt，则按照文本格式输出
* 若指定的目标为console，则仅输出到控制台
* 不符合以上规则，将发出警告并输出到控制台

+ -c 配置文件

用于指定ini文件作为配置文件，以简化配置，默认配置文件为config.ini

_注意！命令行参数优先于配置文件，命令行参数将覆盖配置文件配置_

配置文件样例：

``` ini
[global]
debug=false
file=pocs
target=targets.txt
language=zh-CN
output=console
```

以上配置文件等价于命令行参数：

``` -f pocs -t targets.txt -l zh-CN -o console ```

_以上无论是命令行参数，还是配置文件，都可以用default来指定使用默认值，命令行参数也可以通过省略使用默认值_

_其中debug参数仅能使用配置文件配置，默认为false，读取失败则使用默认值。该参数将决定是否在错误时打印调用栈_

#### POC代码编写

##### POC装饰器

``` python
from winp import POC
```

POC是一个装饰器，用于标识某一个函数是该POC文件的验证函数

一个文件至少有一个POC函数，超过一个则会仅将最后一个POC函数作为验证函数

##### 验证函数

``` python
@POC
def verify(target, result):
```

使用POC装饰器装饰的函数为POC验证函数

该函数必须恰好有2个参数

框架将始终向第一个参数传递需要验证的_单个_目标，向第二个参数一个空字典

请将验证结果传入result结果集，请不要另外创建字典，否则框架将无法获取结果

(后续将考虑使用函数返回值)

##### 魔术变量

``` python
POC_NAME = 'POC 1'

POC_LEVEL = 3

print __winp__

print __winp__.currentTarget

print __winp__.currentExtra

print __winp__.globals

__winp__.setHandler(func)
```

+ POC_NAME为该POC代码的名称，默认使用文件名作为代码名称

+ POC_LEVEL为该漏洞的危险等级，默认为None

_注意！在函数外的任意位置规定名称均可，若定义两次或者以上，以最后一个为准_

+ 带有前后双下划线的winp变量为该验证函数的运行环境Verifier对象，目前用法有：

* 使用currentTarget属性，来_读取_当前的target变量
* 使用currentExtra属性，来_读取_当前的result变量，允许_写入值，但是禁止覆盖_
* 使用globals属性，来_读取_当前的全局变量
* 使用setHandler(func)方法，来_设置_POC验证函数，可以在函数内调用该方法来切换POC验证函数

_注意！setHandler仅在下一个验证目标时生效_
