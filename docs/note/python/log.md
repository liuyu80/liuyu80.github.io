包括我在内的大多数人，当编写小型脚本时，习惯使用`print`来debug，肥肠方便，这没问题，但随着代码不断完善，日志功能一定是不可或缺的，极大程度方便问题溯源以及甩锅，也是每个工程师必备技能

Python自带的`logging`我个人不推介使用，不太Pythonic，而开源的`Loguru`库成为众多工程师及项目中首选，本期将同时对`logging`及`Loguru`进行使用对比，希望有所帮助。

### 快速示例

在`logging`中，默认的日志功能输出的信息较为有限

```python
import logging

logger = logging.getLogger(__name__) 

def main(): 
	logger.debug("This is a debug message") 
	logger.info("This is an info message") 
	logger.warning("This is a warning message") 
	logger.error("This is an error message") 

if __name__ == "__main__": 
	main()
```

输出(**logging默认日志等级为`warning`**，故此处未输出info与debug等级的信息)

```shell
WARNING:root:This is a warning message 
ERROR:root:This is an error message
```

再来看看`loguru`，默认生成的信息就较为丰富了

```python
from loguru import logger 
def main(): 
	logger.debug("This is a debug message") 
	logger.info("This is an info message") 
	logger.warning("This is a warning message") 
	logger.error("This is an error message") 
	
if __name__ == "__main__": 
	main()
```

![](https://pic3.zhimg.com/v2-22b584b36d065f57edec05fe282e958e_b.jpg)

  
提供了执行时间、等级、在哪个函数调用、具体哪一行等信息

### 格式化日志

格式化日志允许我们向日志添加有用的信息，例如时间戳、日志级别、模块名称、函数名称和行号

在`logging`中使用`%`达到格式化目的

```python
import logging 
# Create a logger and set the logging level 
logging.basicConfig( level=logging.INFO, format="%(asctime)s | %(levelname)s | %(module)s:%(funcName)s:%(lineno)d - %(message)s", datefmt="%Y-%m-%d %H:%M:%S", ) 
logger = logging.getLogger(__name__) 
def main(): 
	logger.debug("This is a debug message") 
	logger.info("This is an info message") 
	logger.warning("This is a warning message") 
	logger.error("This is an error message")
```

输出

```shell
2023-10-18 15:47:30 | INFO | tmp:<module>:186 - This is an info message 
2023-10-18 15:47:30 | WARNING | tmp:<module>:187 - This is a warning message 
2023-10-18 15:47:30 | ERROR | tmp:<module>:188 - This is an error message
```

而`loguru`使用和f-string相同的`{}`格式，更方便

```python
from loguru import logger 
logger.add( sys.stdout, level="INFO", format="{time:YYYY-MM-DD HH:mm:ss} | {level} | {module}:{function}:{line} - {message}", )
```

### 日志保存

```python
import logging 

logging.basicConfig( level=logging.DEBUG, format="%(asctime)s | %(levelname)s | %(module)s:%(funcName)s:%(lineno)d - %(message)s", datefmt="%Y-%m-%d %H:%M:%S", handlers=[logging.FileHandler(filename="/your/save/path/info.log", level=logging.INFO), logging.StreamHandler(level=logging.DEBUG), ], ) 

logger = logging.getLogger(__name__) def main(): logging.debug("This is a debug message") logging.info("This is an info message") 

logging.warning("This is a warning message") logging.error("This is an error message") if __name__ == "__main__": main()
```

但是在`loguru`中，只需要使用`add`方法即可达到目的

```python
from loguru import logger 

logger.add( 'info.log', format="{time:YYYY-MM-DD HH:mm:ss} | {level} | {module}:{function}:{line} - {message}", level="INFO", ) 

def main(): 
	logger.debug("This is a debug message") 
	logger.info("This is an info message") 
	logger.warning("This is a warning message") 
	logger.error("This is an error message") 

if __name__ == "__main__": 
	main()
```

### 日志轮换

日志轮换指通过**定期创建新的日志文件并归档或删除旧的日志来防止日志变得过大**

在`logging`中，需要一个名为 `TimedRotatingFileHandler` 的附加类，以下代码示例代表每周切换到一个新的日志文件 ( when=“WO”, interval=1 )，并保留最多 4 周的日志文件 ( backupCount=4 )

```python
import logging 
from logging.handlers import TimedRotatingFileHandler 

logger = logging.getLogger(__name__) 
logger.setLevel(logging.DEBUG) 
# Create a formatter with the desired log format 

formatter = logging.Formatter( "%(asctime)s | %(levelname)-8s | %(module)s:%(funcName)s:%(lineno)d - %(message)s", datefmt="%Y-%m-%d %H:%M:%S", ) 
file_handler = TimedRotatingFileHandler( filename="debug2.log", when="WO", interval=1, backupCount=4 ) 
file_handler.setLevel(logging.INFO) 
file_handler.setFormatter(formatter) 
logger.addHandler(file_handler) 

def main(): 
	logger.debug("This is a debug message") 
	logger.info("This is an info message") 
	logger.warning("This is a warning message") 
	logger.error("This is an error message") 
	
if __name__ == "__main__": 
	main()
```

在`loguru`中，可以通过将 `rotation` 和 `retention` 参数添加到 `add` 方法来达到目的，如下示例，同样肥肠方便

```python
from loguru import logger

logger.add("debug.log", level="INFO", rotation="1 week", retention="4 weeks") 

def main(): 
	logger.debug("This is a debug message") 
	logger.info("This is an info message") 
	logger.warning("This is a warning message") 
	logger.error("This is an error message") 

if __name__ == "__main__": 
	main()
```

### 日志筛选

日志筛选指**根据特定条件有选择的控制应输出与保存哪些日志信息**

在`logging`中，实现该功能需要创建自定义日志过滤器类

```python
import logging

logging.basicConfig( filename="test.log", format="%(asctime)s | %(levelname)-8s | %(module)s:%(funcName)s:%(lineno)d - %(message)s", level=logging.INFO, ) 

class CustomFilter(logging.Filter): 

def filter(self, record):
	return "Cai Xukong" in record.msg 
# Create a custom logging filter 
custom_filter = CustomFilter() 
# Get the root logger and add the custom filter to it 
logger = logging.getLogger()
logger.addFilter(custom_filter)

def main(): 
	logger.info("Hello Cai Xukong") 
	logger.info("Bye Cai Xukong") 
if __name__ == "__main__": 
	main()
```

在`loguru`中，可以简单地使用`lambda`函数来过滤日志

```python
from loguru import logger

logger.add("test.log", filter=lambda x: "Cai Xukong" in x["message"], level="INFO")
def main(): 
	logger.info("Hello Cai Xukong") 
	logger.info("Bye Cai Xukong") 
if __name__ == "__main__": 
	main()
```

### 捕获异常

在`logging`中捕获异常较为不便且难以调试，如

```python
import logging 

logging.basicConfig( level=logging.DEBUG, format="%(asctime)s | %(levelname)s | %(module)s:%(funcName)s:%(lineno)d - %(message)s", datefmt="%Y-%m-%d %H:%M:%S", ) 

def division(a, b): 
	return a / b 
	
def nested(c): 
	try: 
		division(1, c) 
	except ZeroDivisionError: 
		logging.exception("ZeroDivisionError") 

if __name__ == "__main__": 
	nested(0)
```

```python
Traceback (most recent call last): File "logging_example.py", line 16, in nested division(1, c) File "logging_example.py", line 11, in division return a / b ZeroDivisionError: division by zero
```

上面输出的信息未提供触发异常的`c`值信息，而在`loguru`中，通过显示包含变量值的完整堆栈跟踪来方便用户识别

```python
from loguru import logger

def division(a, b): 
	return a / b 

def nested(c): 
	try: 
		division(1, c)
	except ZeroDivisionError:
		logger.exception("ZeroDivisionError") 
		
if __name__ == "__main__": 
	nested(0)
```

![](https://pic3.zhimg.com/v2-4bc64d022e2f990f8319b719d981326e_b.jpg)

  
值得一提的是，`loguru`中的`catch`装饰器允许用户捕获函数内任何错误，且还会标识发生错误的线程

```python
from loguru import logger 

def division(a, b): 
	return a / b 

@logger.catch 
def nested(c): 
	division(1, c) 
	
if __name__ == "__main__": 
	nested(0)
```

![](https://pic3.zhimg.com/v2-4b96b73c19ea055716e73e751b8a7cb6_b.jpg)

OK，作为普通玩家以上功能足以满足日常日志需求，通过对比`logging`与`loguru`应该让大家有了直观感受，哦对了，`loguru`如何安装？

```bash
pip install loguru
```
