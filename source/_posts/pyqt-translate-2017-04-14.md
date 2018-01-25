---
title: PyQt5的翻译机制分析
date: 2017-04-14 14:17:42
tags: [PyQt5]
categories: python
---
### 1. PyQt中的翻译流程
1. 遍历要翻译的文件,如\*.py和\*.ui,以及其他需要翻译的文件等,并创建\*.pro文件,(如translations.pro),将扫描到的要翻译的文件按照指定格式写入到*.pro文件中,其格式如下所示:
```
// 如translations.pro中的内容格式
// SOURCES指定要翻译的文本文件的路径,最好用相对路径
// FORMS指定要翻译的ui文件的路径
// TRANSLATIONS指定翻译文件的路径 
SOURCES += \
    E:/working/srd/plugins/tool/tool1.py \
    E:/working/srd/plugins/tool/tooldialog1.py \
    E:/working/srd/tool/tooldialog2.py

FORMS += \
    E:/working/srd/plugins/tool/res/tool.ui \
    E:/working/srd/plugins/tool/res/tooldialog1.ui \
    E:/working/srd/plugins/tool/res/tooldialog2.ui

TRANSLATIONS += \
    E:/working/srd/plugins/tool/res/translations/zh_CN.ts
```
2. 在*.py或者其他文本文件中创建翻译对象,并标记要翻译的文本源,这是用来提取要翻译的文本源,也是标明哪些文件需要被翻译,对于\*ui文件一般都是默认可翻译的.
同时在应用程序运行时,根据翻译对象去查找翻译后的文本,
**注意:**创建的翻译对象要统一,不要不同的文件创建不同的翻译对象,这样很容易错乱掉,且不要和系统函数或者内建函数或变量重名.
```
_translate = QtCore.QCoreApplication.translate
pushButton = QtWidgets.QPushButton(_translate("centering", "Record {}").format(item))
_logger.error(_translate("centering", "y2 is equal to y1, please pick fit coordination"))
```
3. 使用Linguist翻译工具中的lupdate工具运行加载translations.pro文件,lupdate工具就会扫描SOURCES和FORMS中的文件(如\*.py和*.ui文件),并创建ts文件,并将扫描到的要翻译的源文本提取出来按照指定的格式写入到ts文件中去,然后在Linguist中打开ts文件并开始翻译.ts的格式如下:
```
<?xml version="1.0" encoding="utf-8"?>
<TS version="2.0" language="zh_CN">
    <context>
        <name>MACRO</name>
        <message>
            <location filename="../user/PROBE_SUB/9336" line="231"></location>
            <source>74</source>
            <translation type="unfinished"></translation>
        </message>
        <message>
            <location filename="../user/ARRAYRECT_PATH_CYCLE" line="34"></location>
            <source>Please select 2 plane in 2 plane machining</source>
            <translation>两板加工时请选择两板</translation>
        </message>
        <message>
            <location filename="../../centering.py" line="180"/>
            <source>Y1 and Y2 centering has done??</source>
            <translation type="obsolete">Y1与Y2分中完成!</translation>
        </message>
    </context>
</TS>
```
4. 翻译好并保存ts文件后,使用lrelease工具生成qm文件,lrelease工具会加载ts文件,并做一些去重的工作,以节省qm文件的大小.
5. 最后在应用程序中加载qm翻译文件,qm文件也是一种资源文件,在应用程序中需要注意加载的时机,比如应当在界面显示前或者日志打印前加载好.
```
translator = QtCore.QTranslator()
translator.load(QtCore.QLocale(), "", "", ":/{}/translations".format(__name__.replace(".", "/")))
QtWidgets.qApp.installTranslator(translator)

# 指定名称的翻译文件 
if os.path.exists(os.path.join(basic.sysDir, 'macro/translations/common_zh_CN.qm')):
    _common_macro_translator = QtCore.QTranslator()
    _common_macro_translator.load(QtCore.QLocale(), "common", "_", os.path.join(
        basic.sysDir, "macro/translations"))
    QtWidgets.qApp.installTranslator(_common_macro_translator)
```

### 2. 关于PyQt的Linguist
在PyQt5的安装包有一个Linguist翻译工具,为应用程序的国际化提供很好的支持.
Linguist提供了两款工具lupdate和lrelease.它们可以处理qmake项目文件(\*.pro)，可直接在文件系统上运行.

1. lupdate会逐个扫描应用程序中的.ui和.py文件,然后产生翻译源文件(\*.ts),里面包含所有用户可见的文本,但未经过翻译(unfinished).
生成的.ts文件通过Linguist翻译工具打开即可看到所扫描到的要被翻译的源文本.
当\*.py或\*.ui的源文本被修改了,再次运行lupdate,就可以从应用程序中同步可见的文本,且不会破坏已经翻译好的文本.
lupdate工具从应用程序中提取的用户界面字符串。它读取应用程序的.pro文件，以确定哪些源文件包含的文本需要被翻译。这意味着源文件都必须被列在.pro中。如果文件没有列出，其中的文本则不会被发现。
2. 运行lrelease会读取对应的.ts文件,并生成用于应用程序运行是用的.qm文件.
运行lrelese时生成.qm文件时会删除相同的源文本,只保留一个,这样.qm文件会比较小,且生成的.qm文件是二进制的.

### 3. 使用lupdate和lrelease
```
# Qt
lupdate myproject.pro 
# PyQt
pylupdate translations.pro 
lrelease myproject.pro
```
### 4. ts文件规格分析
ts文件是可读的XML文件,包含源短语及其翻译，ts文件通常由lupdate创建与更新.
1. `<?xml version="1.0" encoding="utf-8"?>`ts文件的开头说明,其实是xml文件的开头说明.指明xml版本和编码
2. `<!DOCTYPE TS>`指明文档类型是一个ts文件,可有可无
3. TS标签
TS标签格式如下,指定Linguist版本和翻译后的语言,TS标签里的内容是由context标签组成
```
<TS version="2.1" language="zh_CN">
    <context>
        ...
    </context>
    <context>
        ...
    </context>
    <context>
        ...
    </context>
    ...
</TS>
```
4. context标签
context标签是一个上下文标签,不同的context是根据子标签的<name>来区分,指明这个context的对象名.
通常一个文件就是一个context,如果是以ui文件,那么name就是这个ui的对象名,注意不是ui文件名,如果是一个py文件,那么name就是py文件名.
如果想把多个文件合并到一个context中去,那么需要指定一个name,那么这多个文件的翻译都会放在这一个context中.这个name最终传给翻译对象的第一个参数.
```
<context>
    <name>...</name>
    <message>
        <location filename="..." line="..."/>
        <source>...</source>
        <translation>...</translation>
    </message>
    <message>
        <location filename="..." line="..."/>
        <source>...</source>
        <translation>...</translation>
    </message>
    ...
</context>
```
**注意:**这里的name其实是对应这翻译对象里的name的参数.如:
```
macro_text = \_translate("MACRO", dst\_text)
pushButton = QPushButton(_translate("centering", "Record {}").format(item))
```

5. message标签
message标签由location,source和translation标签组成,其中:
location:指定被翻译文件的路径和行号,注意:被翻译文件路径是相对与这个ts文件的相对路径
source:翻译的源文本
translation:翻译文本
translation的翻译状态有type指定,type有三种情况:unfinished(未翻译),空(已翻译),obsolete(废弃的)
```
<message>
        <location filename="..." line="..."/>
        <source>...</source>
        <translation>...</translation>
</message>
```
### 5. qm文件
qm(Qt message)文件和ts文件的文件名称一样,一一对应,qm文件由lrelease创建.

### 6. 创建ts文件
ts文件其实就是xml文件,创建ts文件可以应用python的第三方包:xmltodict,可将字典数据结构直接转化成xml了.
```python
TS_XML = OrderedDict()
TS = OrderedDict()
context = OrderedDict()
TS["@version"] = "2.1"
TS["@language"] = "zh_CN"
context["name"] = "MACRO"
message = OrderedDict()
location = OrderedDict()
translation = OrderedDict()
# message/location 标签
location["@filename"] = os.path.relpath(file_path, trans_path).replace("\\", "/")
location["@line"] = line_index + 1
message["location"] = location
if line_content.strip()[0:3] == 'ASK':
    source = ((match.group()[4:-1]).split(','))[2]
# message/source 标签
message["source"] = source
# message/translation 标签
translation["@type"] = "unfinished"
message["translation"] = translation
TS["context"] = context
TS_XML["TS"] = TS
# 创建ts文件
with open(os.path.join(ts_path, trans_file_name), "w", encoding="utf-8") as f:
    xmltodict.unparse(TS_XML, f, pretty=True, newl=new_line, indent=indent)
```

有了ts文件,就可以应用lupdate和lrelease工具生成qm文件
当翻译源文本有了修改,就需要重新生成ts文件了,这里需要注意一个问题,没有修改的翻译会被丢失.
这就需要先解析修改前的ts文件,然后对比修改后的ts文件的源文本,只更新修改了的.
```python
def parseTS(ts_path):
    if 'base' in ts_path and 'macro' in ts_path:
        trans_file_name = "common_zh_CN.ts"
    elif 'sys' in ts_path and 'macro' in ts_path:
        trans_file_name = "user_zh_CN.ts"
    ts_file = os.path.join(ts_path, trans_file_name)
    with open(ts_file, "r", -1, "utf-8") as f:
        document = f.read()
    ts_xml = xmltodict.parse(document)
    macro_context_list = ts_xml['TS']['context']['message']
    return macro_context_list

def updateTS(ts_path, new_message_list):
    old_message_list = parseTS(ts_path)
    for new_message in new_message_list:
        for old_message in old_message_list:
            if new_message["source"] == old_message["source"]:
                new_message["translation"] = old_message["translation"]
    return new_message_list
```
