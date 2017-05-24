---
title: Python之subprocess模块
date: 2017-04-22 10:44:12
tags: Python
categories: 编程
---
### subprocess模块
subprocess模块,即子进程模块,允许您生成新进程，并连接到输入/输出/错误管道，并获取其返回代码。

### subprocess模块使用
调用subprocess模块的推荐方法是:推荐使用run()函数来处理几乎所有用例或情景。对于更高级的用例，可以直接使用基础的Popen接口。
```
subprocess.run(args, *, stdin=None, input=None, stdout=None, stderr=None, shell=False, timeout=None, check=False, encoding=None, errors=None)
```
运行由args描述的命令。等待命令完成，然后返回一个CompletedProcess实例。
完整的run函数签名与Popen构造函数大致相同(除了超时，输入和检查外)，该函数的所有参数都传递给该接口。
默认情况下，这不捕获stdout或stderr。要做到这一点，通过PIPE的stdout和/或stderr参数。
超时参数传递给Popen.communicate（）。如果超时过期，则子进程将被杀死并等待。在子进程终止后，将重新提出TimeoutExpired异常。
输入的参数传递给Popen.communicate（），从而传递给子进程的stdin。如果使用，它必须是一个字节序列，如果指定了编码或错误，或者是universal_newlines为真，则为字符串。使用时，内部Popen对象将自动使用stdin = PIPE创建，并且stdin参数也可能不被使用。
如果检查为真，并且进程以非零退出代码退出，则将引发CalledProcessError异常。该异常的属性保存参数，退出代码以及stdout和stderr（如果被捕获）。
如果指定了编码或错误，或者universal_newlines为true，则使用指定的编码和错误或io.TextIOWrapper默认文件在文本模式下打开stdin，stdout和stderr的文件对象。否则，文件对象将以二进制模式打开。

args是所有调用所必需的，应该是一个字符串或一系列程序参数。通常优选提供参数序列，因为它允许模块处理任何所需的转义和引用参数（例如，允许文件名中的空格）。如果传递单个字符串，则shell必须为True（见下文），否则字符串必须简单地命名要执行的程序，而不指定任何参数。

### Popen构造函数
```
class subprocess.Popen(args, bufsize=-1, executable=None, stdin=None, stdout=None, stderr=None, preexec_fn=None, close_fds=True, shell=False, cwd=None, env=None, universal_newlines=False, startupinfo=None, creationflags=0, restore_signals=True, start_new_session=False, pass_fds=(), *, encoding=None, errors=None)
```
在新进程中执行子程序。在POSIX上，该类使用os.execvp（）类似的行为来执行子程序。在Windows上，该类使用Windows CreateProcess（）函数.
args应该是程序参数的序列，或者是一个单个的字符串。默认情况下，如果args是序列，则要执行的程序是args中的第一个项目。如果args是字符串，则解释是平台依赖的，并在下面描述。请参阅shell和可执行参数，以获得与默认行为的更多差异。除非另有说明，否则建议将args作为序列。

在Windows上，如果args是一个序列，它将以转换参数序列到Windows上的字符串中所述的方式转换为字符串。这是因为底层的CreateProcess（）对字符串运行.
shell参数（默认为False）指定是否使用shell作为程序执行。如果shell为True，则建议将args作为字符串而不是序列传递。

### Popen Objects(Popen对象)
Popen类的实例有以下几种方法：
Popen.poll():检查子进程是否终止。设置并返回returncode属性。
Popen.wait(timeout=None):等待子进程终止。设置并返回returncode属性.如果进程在超时秒后没有终止，请引发超时突发异常。捕获这个异常是安全的，并重试等待。
当使用stdout = PIPE或stderr = PIPE时，这将会死锁，并且子进程向管道生成足够的输出，从而阻止等待OS管道缓冲区接受更多数据。使用Popen.communicate（）使用管道避免这种情况。
Popen.communicate(input=None, timeout=None):与进程交互：将数据发送到stdin。从stdout和stderr读取数据，直到文件到达。等待进程终止。可选的输入参数应该是要发送到子进程的数据，否则，如果没有数据发送给子进程，则为None。如果流以文本模式打开，则输入必须是字符串。否则，它必须是字节。
communication（）返回一个元组（stdout\_data，stderr\_data）。如果流以文本模式打开，数据将为字符串;否则，字节。
请注意，如果要将数据发送到进程的stdin，则需要使用stdin = PIPE创建Popen对象。类似地，要在结果元组中获得除None之外的任何内容，您还需要给出stdout = PIPE和/或stderr = PIPE.
如果进程在超时秒后没有终止，则会引发一个TimeoutExpired异常。捕捉此异常并重试通信不会丢失任何输出。
如果超时过期，子进程不会被终止，所以为了正确清理，运行良好的应用程序应该杀死子进程并完成通信：
```
proc = subprocess.Popen(...)
try:
    outs, errs = proc.communicate(timeout=15)
except TimeoutExpired:
    proc.kill()
    outs, errs = proc.communicate()
```
读取的数据被缓冲在内存中，因此如果数据很大或无限制，则不要使用此方法。

Popen.terminate():

### 旧的高级API
在Python 3.5之前，这subprocess中函数中的check\_output, check\_call, call这三个函数包含了高级API到子进程。现在可以在很多情况下使用run（），但很多现有的代码调用这些函数。
