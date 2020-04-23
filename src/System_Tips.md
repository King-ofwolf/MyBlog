# 总结系统开发的Tips

## 代码规范

* 使用Camel_Case命名py文件
* 使用Camel_Case_ui命名ui界面文件对应的ui.py文件
* 逗号、运算符使用空格相隔
* 一行代码的长度不要过长，使用\切分
* for、if语句冒号后边不要跟随语句
* 变量命名规则
	+ 使用UPPER_WORD写全局常量
	+ 使用CamelCase convention写全局变量
	+ 使用_lower_word写局部临时变量
	+ 使用CamelCase convention写类名
	+ 使用CamelCase convention写类静态函数名
	+ 使用lower_word写自有函数名
	+ 使用CamelCase convention写类全局变量
	+ 使用lower_word写类全局对外变量
	+ 使用_lower_word写类全局私有变量
* 合理使用继承、重载，可以极大的提高代码重用率，减少代码冗余度
* 编写类时，尽量减少外部环境依赖，提高类的独立性，首先需要满足独立测试需求，其次需要考虑无外部环境的执行情况。

## 版本控制

* 使用git bare仓库（可以使用本机地址作为远程仓库）,详情见[git仓库的bare方式]
	* 创建空的bare仓库
	
	```shell
	git init --bare
	```
	
	* 根据已有的仓库创建bare仓库
	
	```shell
	git clone --bare path
	```

* 精简化的log版本日志

```shell
git log --graph --pretty=oneline --abbrev-commit
```

## 结构代码

### 全局变量:Global.py

* 使用类的静态全局常量作为全局变量
* 使用UPPER_WORD写全局常量，使用CamelCase convention写全局变量
* 将所有使用到的常量绝对地址放到全局变量或者ini配置文件中，包括数据文件地址等
* 使用绝对路径替换./相对路径

```python
import sys,os
os.path.join(os.path.dirname(os.path.abspath(sys.argv[0])), './')
```

### 图形界面接口文件:LayoutInterface.py

* 使用接口文件import所有常用的PyQt的module
* 使用接口文件重载常用的Dialog函数，如```QtWidgets.QFileDialog.OpenFileNames()```
* 使用接口文件继承QMainWindow、QWidget、QDialog等常用的窗口框架，声明常用函数，方便后期进行窗口界面的布局改换。
	* 菜单栏的设定按照字典常量的方式设定，包含名称、图标等参数

```python
class Window(QtWidgets.QMainWindow):
	WorkDoneSignal = QtCore.pyqtSignal(bool)
	NextPartSignal = Qtcore.pyqtSignal(int, str)
	InformationAppendSignal = QtCore.pyqtSignal(str)
	def __init__(self):
		super(Window, self).__init__()
		self.var_init()
		self.pain_UI()
		self.activty_bind()
		self.set_default()
		self.show()
```

* 表格自动改变大小的设定（实测将会无法拖动每列改变宽度，不知道有什么更好的办法）

```python
table_viewer.horizontalHeader().setSectionResizeMode(QtWidgets.QHeaderView.ResizeToContents)
table_viewer.verticalHeader().setSectionResizeMode(QtWidgets.QHeaderView.Fixed)
```

### 系统运行日志

* 添加控制台和文件双重log

```python
import logging
import os
RUN_LOG_DIR = './'
LOG_FILE = os.path.join(RUN_LOG_DIR, 'System.log')
logging.basicConfig(level=logging.INFO,
					format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
					datefmt='%a, %d %b %Y %H:%M:%S',
					filename=LOG_FILE,
					filemode='a')
# 添加控制台日志
ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)
ch.setFormatter(logging.Formatter('%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s'))

SysLogger = logging.getLogger('System')
SysLogger.addHandler(ch)
```

* 使用LogReset类重载logger的各项方法，方便将日志重定向到任意位置

```python
class LogReset:
	def __init__(self):
		self.widget = None
		self.sys_logger = logger
	# info, error, warning, debug, percent etc.
	def error(self, info):
		self.sys_logger.error(info)
		if self.widget is not None:
			self.widget.InformationAppendSignal.emit(info)
	def reset(self, widget):
		self.widget = widget
	# 其他方法都由sys_logger提供
	def __getattr__(self, item):
		return getattr(self.sys_logger, item)
```

### 全局异常控制器:Exception_Handler.py

主要是因为PyQt的运行机制是多线程，导致异常无法通过主函数直接进行捕获，因此需要对每一个可能出现异常的函数都加上try except来对异常进行重定向，然而这样会导致代码很复杂，重复工作量太多，因此采用装饰器对需要捕获异常的函数进行修饰。

* 使用装饰器@ExceptionTry对未知和已知的异常进行捕获，装饰器获取相应参数（是否退出，显示状态等）

```python
import traceback
# 构建装饰器
def ExceptionTry(func):
	def wrapper(*args, **kwargs):
		try:
			return func(*args, **kwargs)
		except Exception as e:
			Logger.exception(e)
			if Window is not None:
				Window.ErrorInfoExceptSignal.emit(traceback.format_exc())
				return False
			else:
				raise e
	return wrapper
```

* 使用常驻的全局异常显示窗口显示异常信息，针对不同参数，设置返回值、处理方式等（exit、continue）

### 自动编译文件:makefile

* 使用自动化方式判断Python.exe执行路径、位数

```shell
PYTHON_EXEPATH := $(shell where python)
PYTHON_PATH := $(patsubst %\python.exe,%,$(PYTHON_EXEPATH))
ifeq ($(findstring Python35-32,%(PYTHON_PATH)), Python35-32)
	PYTHON_DIGIT := x86
else
	PYTHON_DIGIT := x64
endif
```

* 使用dir获取当前路径，并使用-C进行makefile的调用

```shell
mkfile_path := $(abspath $(lastword $(MAKEFILE_LIST)))
srcdir := $(dir $(mkfile_path))
$(MAKE) -C $(srcdir)Layout/
```

* 使用函数的方式切分各项编译任务

```shell
define build_support_p1
	echo $1
endef
```

* 针对同一类别的文件，使用同一语句进行编译

```shell
SRC := $(wildcard *_ui.ui)
OBJ := $(patsubst %.ui, %.py, $(SRC))
all : $(OBJ)
%.py:%.ui
	$(pyuic) $< -o $@
```

* makefile行尾不能出现‘\'符号，否则需要在下一行空行（常出现在copy语句最后的路径地址，其实可以省略'\'不写）

## 系统测试

### 主测试文件:Test_Main.py

* 使用addTest添加测试用例

```python
import unittest
suite = unittest.TestSuit()
loader = unittest.TestLoader()
suite.addTest(loader.loadTestFromModule(PackTest))
runner = unittest.TextTestRunner(verbosity=2)
runner.run(suite)
```

 ### 模块测试文件:Test_*.py
 
 * 一个类对应一个测试用例，各个测试方法之间不能有逻辑的先后关系，也不能有任何的数据交互
 * 测试用例所生成的临时文件和内存需要在测试完成后进行释放（通过tearDownClass）
 
 ```python
 import unittest
 class Test(unittest.TestCase):
	@classmethod
	def setUpClass(cls) -> None:
		pass
	@classmethod
	def tearDownClass(cls) -> None:
		pass
	def setUp(self) -> None:
		pass
	def tearDown(self) -> None:
		pass
	def test_case1(self):
		pass
```

* 使用TDD方法进行系统开发，使用测试程序驱动开发的进行，先按照需求写测试程序
* 测试方法先编写判断代码功能的断言语句，然后再编写必要的辅助语句

## 其他Tips

### sqlite数据库

* 不能使用多线程进行写入
* 当一个读取事件未结束时，无法执行DML语句，否则将出现lock异常。主要体现在使用QTableModel时，当其未显示出所有数据时（初始最大只能显示250条），此时进行数据导入会出现lock锁。（sqlite的事件机制，具体可参考[sqlite数据库的锁和事件]）
* 使用order by进行结果排序时，需要考虑到null元素的影响，不同的执行器对null元素的排序方式不同，最后体现的结果就是同样的语句，同样的数据库，在不同的执行器下，执行出来的结果不一样（巨坑，结合group使用时，自己的代码完全没问题，最后在数据库中能直接找到数据，而在自己的系统中却显示为null）
* update语句部分执行器不支持```set (n1, n2, n3)=(v1, v2, v3)```,需要分别使用n1=v1的形式进行更新
* 数据导入时，尽量不要每插入一条数据都对数据库进行检索（体现在插入前检索是否存在的问题），最好是一次性先导入，最后再进行匹配后生成错误报告

### PyQt5图形界面

* 输入框lineEdit若使用复制粘贴的形式，会导致获取的字符串带\n等特殊字符（单纯如我，天真的以为键盘敲不进去回车符就不会出现回车符，最后程序crash了）

### Python相关技巧

* 合理使用yield作为函数返回值，构建构造器，可以提高函数执行效率
* 可以使用协程快速构建多线程任务

```python
def gen()
	count = 0
	_result = 0
	_msg = 0
	while True:
		v_input = yield count
		if v_input is None:
			break
		count += 1
	return _result, _msg
def proxy():
	while True:
		_result, _msg = yield from gen()
```

* tuple和list等集合的空值判断可以通过```bool()```进行，同时兼顾了None值的判断，或者可以直接使用其本身作为bool值也可以，如```if []```
* ```a == None```的形式最好替换成```a is None```的形式
* Python支持同时比较大小:```0 < a < 10```

### 加密/编码

* Python的字符串编码方式为变动的（Latin1, UCS-2, UCS-4），按照当前字符串的字符范围决定具体每个字符的长度
	+ 可以使用```sys.getsizeof()```获取绝对字符串内存大小（需要使用减法获取单个字符的绝对大小）
* bytes类型的数据每个位大小为1byte，str类型的每个位长度由编码方式决定(Latin1 为1byte）
* 二进制数据数据编码为字符串（长度将加倍）

```python
import binascii
def encode_b2s(b_input):
	return b_input.hex()
def decode_s2b(s_input):
	return binascii.a2b_hex(s_input)
```

* 获取hash码

```python
import hashlib
hl = hashlib.md5(sqlt.encode('utf-8'))
hl.update(passwd.encode('utf-8'))
hl.hexdigest() # 32bytes length hashcode type of str
hl.digest() # 16bytes length hashcode type of bytes
```

### 开发日志

* 使用markdown文件记录系统开发过程中的问题和解决方式

[sqlite数据库的锁和事件]: sqlite3/Database_Lock_and_Transactions.md
[git仓库的bare方式]: https://blog.csdn.net/chenzhengfeng/article/details/81743626