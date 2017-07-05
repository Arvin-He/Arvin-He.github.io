---
title: cookbook之脚本编程与系统管理
date: 2017-07-05 14:33:20
tags: Python
categories: 编程
---
### 通过重定向/管道/文件接受输入
命令行的输出通过管道传递给该脚本、重定向文件到该脚本，在命令行中传递一个文件名或文件名列表给该脚本。
Python 内置的 fileinput 模块让这个变得简单

### 终止程序并给出错误信息
```python
import sys
sys.stderr.write('It failed!\n')
raise SystemExit(1)
```

### 解析命令行选项
argparse 模块可被用来解析命令行选项, argparse 模块是标准库中最大的模块之一，拥有大量的配置选项。

### 运行时弹出密码输入提示
你写了个脚本，运行时需要一个密码。此脚本是交互式的，因此不能将密码在脚本中硬编码，而是需要弹出一个密码输入提示，让用户自己输入.
Python 的 getpass 模块正是你所需要的.
```python
import getpass

user = getpass.getuser()
passwd = getpass.getpass()

if svc_login(user, passwd): # You must write svc_login()
    print('Yay!')
else:
    print('Boo!')
```

注意在前面代码中 getpass.getuser() 不会弹出用户名的输入提示。它会根据该用户的 shell 环境或者会依据本地系统的密码库（支持 pwd 模块的平台）来使用当前用户的登录名，如果你想显示的弹出用户名输入提示，使用内置的 input 函数：`user = input('Enter your username: ')`
还有一点很重要，有些系统可能不支持 getpass() 方法隐藏输入密码。这种情况下，Python 会提前警告你这些问题（例如它会警告你说密码会以明文形式显示）

###  获取终端的大小
使用 os.get terminal size() 函数来做到这一点

### 执行外部命令并获取它的输出
使用 subprocess.check output() 函数,
```python
import subprocess
out_bytes = subprocess.check_output(['netstat','-a'])
```

这段代码执行一个指定的命令并将执行结果以一个字节字符串的形式返回。如果你
需要文本形式返回，加一个解码步骤即可。
```python
out_text = out_bytes.decode('utf-8')
# 或者直接将输出解码
out_text = subprocess.check_output(['netstat','-a']).decode('utf-8')
```

默认情况下，check output() 仅仅返回输入到标准输出的值。如果你需要同时收集标准输出和错误输出，使用 stderr 参数：
`out_bytes = subprocess.check_output(['cmd','arg1','arg2'], stderr=subprocess.STDOUT)`
如果你需要用一个超时机制来执行命令，使用 timeout 参数：
```python
try:
    out_bytes = subprocess.check_output(['cmd','arg1','arg2'], timeout=5)
except subprocess.TimeoutExpired as e:
    ...
```

你想让命令被一个shell 执行，传递一个字符串参数，并设置参数 shell=True . 有时候你想要 Python 去执行一个复杂的 shell 命令的时候这个就很有用了，比如管道流、I/O 重定向和其他特
性。例如：`out_bytes = subprocess.check_output('grep python | wc > out', shell=True)`
使用 check output() 函数是执行外部命令并获取其返回值的最简单方式,如果你需要对子进程做更复杂的交互，比如给它发送输入，你得采用另外一种方法。这时候可直接使用 subprocess.Popen 类。

### 复制或者移动文件和目录
你想要复制或移动文件和目录，但是又不想调用 shell 命令,shutil 模块有很多便捷的函数可以复制文件和目录.
```python
import shutil
# Copy src to dst. (cp src dst)
shutil.copy(src, dst)
# Copy files, but preserve metadata (cp -p src dst)
shutil.copy2(src, dst)
# Copy directory tree (cp -R src dst)
shutil.copytree(src, dst)
# Move src to dst (mv src dst)
shutil.move(src, dst)
```

如果源文件是一个符号链接，那么目标文件将会是符号链接指向的文件。如果你只想复制符号链接本身，那么需要指定关键字参数 follow symlinks,如果你想保留被复制目录中的符号链接
`shutil.copytree(src, dst, symlinks=True)`.
copytree() 可以让你在复制过程中选择性的忽略某些文件或目录,
```python
def ignore_pyc_files(dirname, filenames):
    return [name in filenames if name.endswith('.pyc')]

shutil.copytree(src, dst, ignore=ignore_pyc_files)
```

对于文件元数据信息，copy2() 这样的函数只能尽自己最大能力来保留它。访问时间、创建时间和权限这些基本信息会被保留，但是对于所有者、ACLs、资源 fork 和其他更深层次的文件元信息就说不准了，这个还得依赖于底层操作系统类型和用户所拥有的访问权限。你通常不会去使用 shutil.copytree() 函数来执行系统备份。当处理文件名的时候，最好使用 os.path 中的函数来确保最大的可移植性（特别是同时要适用于 Unix 和Windows）。

使用 copytree() 复制文件夹的一个棘手的问题是对于错误的处理。例如，在复制过程中，函数可能会碰到损坏的符号链接，因为权限无法访问文件的问题等等。为了解决这个问题，所有碰到的问题会被收集到一个列表中并打包为一个单独的异常，到了最后再抛出。
```python
try:
    shutil.copytree(src, dst)
except shutil.Error as e:
    for src, dst, msg in e.args[0]:
        # src is source name
        # dst is destination name
        # msg is error message from exception
        print(dst, src, msg)
```

如果你提供关键字参数 ignore dangling symlinks=True ，这时候 copytree() 会忽略掉无效符号链接。

### 创建和解压归档文件
shutil 模块拥有两个函数—— make archive() 和 unpack archive() 可派上用场.
```python
>>> import shutil
>>> shutil.unpack_archive('Python-3.3.0.tgz')
>>> shutil.make_archive('py33','zip','Python-3.3.0')
'/Users/beazley/Downloads/py33.zip'
```

`make archive()` 的第二个参数是期望的输出格式。可以使用`get_archive_formats()` 获取所有支持的归档格式列表。
```python
>>> shutil.get_archive_formats()
[('bztar', "bzip2'ed tar-file"), ('gztar', "gzip'ed tar-file"),
('tar', 'uncompressed tar file'), ('zip', 'ZIP file')]
>>>
```

Python 还有其他的模块可用来处理多种归档格式（比如 tarfile, zipfile, gzip, bz2）的底层细节。

### 通过文件名查找文件
查找文件，可使用 `os.walk()` 函数，传一个顶级目录名给它.查找特定的文件名并答应所有符合条件的文件全路径
```python
#!/usr/bin/env python3.3
import os

def findfile(start, name):
    for relpath, dirs, files in os.walk(start):
        if name in files:
            full_path = os.path.join(start, relpath, name)
            print(os.path.normpath(os.path.abspath(full_path)))

if __name__ == '__main__':
    findfile(sys.argv[1], sys.argv[2])
```

`os.walk()` 方法为我们遍历目录树，每次进入一个目录，它会返回一个三元组，包含相对于查找目录的相对路径，一个该目录下的目录名列表，以及那个目录下面的文件名列表。
对于每个元组，只需检测一下目标文件名是否在文件列表中。如果是就使用`os.path.join()` 合并路径。为了避免奇怪的路径名比如 `././foo//bar` ，使用了另外两个函数来修正结果。第一个是 `os.path.abspath()` , 它接受一个路径，可能是相对路径，最后返回绝对路径。第二个是 `os.path.normpath()` ，用来返回正常路径，可以解决双斜杆、对目录的多重引用的问题等。

### 读取配置文件
读取普通.ini 格式的配置文件, configparser 模块能被用来读取配置文件, 使用 `cfg.write()` 方法将其写回到文件
**注意:**配置文件中的名字是不区分大小写的. 但可以指定是否对大小写敏感(通过指定configparser示例对象的optionxform属性`cf.optionxform = str`).
配置文件并不是从上而下的顺序执行,是一个整体被读取.如果碰到了变量替换，它实际上已经被替换完成了,
在下面这个配置中，prefix 变量在使用它的变量之前后之后定义都是可以的
```
[installation]
library=%(prefix)s/lib
include=%(prefix)s/include
bin=%(prefix)s/bin
prefix=/usr/local
```

ConfigParser 有个容易被忽视的特性是它能一次读取多个配置文件然后合并成一个配置。

### 给简单脚本增加日志功能
```python
import logging

def main():
    # Configure the logging system
    logging.basicConfig(filename='app.log', level=logging.ERROR)
    # Variables (to make the calls that follow work)
    hostname = 'www.python.org'
    item = 'spam'
    filename = 'data.csv'
    mode = 'r'
    # Example logging calls (insert into your program)
    logging.critical('Host %s unknown', hostname)
    logging.error("Couldn't find %r", item)
    logging.warning('Feature is deprecated')
    logging.info('Opening file %r, mode=%r', filename, mode)
    logging.debug('Got here')

if __name__ == '__main__':
    main()
```

上面的日志配置都是硬编码到程序中的。如果你想使用配置文件，可以像下面这样修改 basicConfig() 调用：
```python
import logging
import logging.config

def main():
    # Configure the logging system
    logging.config.fileConfig('logconfig.ini')
    ...
```

创建一个下面这样的文件，名字叫 logconfig.ini ,如果你想修改配置，可以直接编辑文件 logconfig.ini 即可。
```
[loggers]
keys=root

[handlers]
keys=defaultHandler

[formatters]
keys=defaultFormatter

[logger_root]
level=INFO
handlers=defaultHandler
qualname=root

[handler_defaultHandler]
class=FileHandler
formatter=defaultFormatter
args=('app.log', 'a')

[formatter_defaultFormatter]
format=%(levelname)s:%(name)s:%(message)s
```

### 实现一个计时器记录程序执行多个任务所花费的时间
```python
import time

class Timer:
    def __init__(self, func=time.perf_counter):
        self.elapsed = 0.0
        self._func = func
        self._start = None

    def start(self):
        if self._start is not None:
            raise RuntimeError('Already started')
        self._start = self._func()
    
    def stop(self):
        if self._start is None:
            raise RuntimeError('Not started')
        end = self._func()
        self.elapsed += end - self._start
        self._start = None
    
    def reset(self):
        self.elapsed = 0.0
    
    @property
    def running(self):
        return self._start is not None
    
    def __enter__(self):
        self.start()
        return self
    
    def __exit__(self, *args):
        self.stop()

def countdown(n):
    while n > 0:
        n -= 1

# Use 1: Explicit start/stop
t = Timer()
t.start()
countdown(1000000)
t.stop()
print(t.elapsed)

# Use 2: As a context manager
with t:
    countdown(1000000)
    print(t.elapsed)

with Timer() as t2:
    countdown(1000000)
    print(t2.elapsed)
```

### 限制内存和 CPU 的使用量
对在 Unix 系统上面运行的程序设置内存或 CPU 的使用限制。resource 模块能同时执行这两个任务

### 启动一个 WEB 浏览器
通过脚本启动浏览器并打开指定的 URL 网页,webbrowser 模块能被用来启动一个浏览器，并且与平台无关.
```python
>>> import webbrowser
>>> webbrowser.open('http://www.python.org')
True
```

它会使用默认浏览器打开指定网页。如果你还想对网页打开方式做更多控制，还可以使用下面这些函数：
打开一个新的浏览器窗口或者标签，只要浏览器支持就行。
```python
>>> # Open the page in a new browser window
>>> webbrowser.open_new('http://www.python.org')
True
>>> # Open the page in a new browser tab
>>> webbrowser.open_new_tab('http://www.python.org')
True
```

如果你想指定浏览器类型，可以使用 webbrowser.get() 函数来指定某个特定浏览器。
```python
>>> c = webbrowser.get('firefox')
>>> c.open('http://www.python.org')
True
>>> c.open_new_tab('http://docs.python.org')
True
```