---
layout: post
title: "Python logging"
date: 2015-05-15
comments: true
category: Python
tag: Python
---

### Python日志记录logigng模块

简单使用,代码如下：

```python
# 导入日志模块
import logging

# 初始化一个logger
logger = logging.getLogger(__name__)
# 设置日志级别
logger.setLevel(logging.DEBUG)
# 初始化一个Handler
fh = logging.StreamHandler()
# 准备格式串
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
# 把格式化串加入设置到Handle中
fh.setFormatter(formatter)
# 设置日志Handle
logger.addHandler(fh)
# 打印
logger.info('the loggint print')

```

### 给日志设置LoggerAdapter

在LoggerAdapter代码如下

```python
class LoggerAdapter(object):

    def __init__(self, logger, extra):
        self.logger = logger
        self.extra = extra

    def process(self, msg, kwargs):
        kwargs["extra"] = self.extra
        return msg, kwargs

    def debug(self, msg, *args, **kwargs):
        """
        Delegate a debug call to the underlying logger, after adding
        contextual information from this adapter instance.
        """
        msg, kwargs = self.process(msg, kwargs)
        self.logger.debug(msg, *args, **kwargs)
```

所以LoggerAdapter可以这样使用

```python
class CustomAdapter(logging.LoggerAdapter):
    """
    This example adapter expects the passed in dict-like object to have a
    'connid' key, whose value in brackets is prepended to the log message.
    """
    def process(self, msg, kwargs):
        return '[%s] %s' % (self.extra['connid'], msg), kwargs

some_conn_id = 'test-adapter'
logger = logging.getLogger(__name__)
adapter = CustomAdapter(logger, {'connid': some_conn_id})
adapter.info('this is print by adapter')
```

打印结果为：
    
    2015-05-15 22:40:37,755 - __main__ - INFO - [test-adapter] this is print by adapter
