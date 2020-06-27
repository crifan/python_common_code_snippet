# 文件

## 提取文件名后缀

```python
def extractSuffix(fileNameOrUrl):
    """
    extract file suffix from name or url
    eg:
https://cdn2.qupeiyin.cn/2018-09-10/15365514898246.mp4 -> mp4
        15365514894833.srt -> srt
    """
    return fileNameOrUrl.split('.')[-1]
```

## 创建空文件

```python
import os

def createEmptyFile(fullFilename):
    """Create a empty file like touch"""
    filePath = os.path.dirname(fullFilename)
    # create folder if not exist
    if not os.path.exists(filePath):
        os.makedirs(filePath)

    with open(fullFilename, 'a'):
        # Note: not use 'w' for maybe conflict for others constantly writing to it
        os.utime(fullFilename, None)
```

## 保存（二进制）数据到文件

```python
def saveDataToFile(fullFilename, binaryData):
    """save binary data info file"""
    with open(fullFilename, 'wb') as fp:
        fp.write(binaryData)
        fp.close()
        # print("Complete save file %s" % fullFilename)
```

## 保存json到文件

```python
import json
import codecs

def saveJsonToFile(fullFilename, jsonValue):
    """save json dict into file"""
    with codecs.open(fullFilename, 'w', encoding="utf-8") as jsonFp:
        json.dump(jsonValue, jsonFp, indent=2, ensure_ascii=False)
        # print("Complete save json %s" % fullFilename)
```

## 从文件中加载出json

```python
import json
import codecs

def loadJsonFromFile(fullFilename):
    """load and parse json dict from file"""
    with codecs.open(fullFilename, 'r', encoding="utf-8") as jsonFp:
        jsonDict = json.load(jsonFp)
        # print("Complete load json from %s" % fullFilename)
        return jsonDict
```

## 通过二进制生成文件类型对象

（1）Python 3

```python
import io

audioBinaryData = audioObj.read()
audioFileLikeObj = io.BytesIO(audioBinaryData)
```

得到对应的文件类型的对象的：`<_io.BytesIO object at 0x115964468>`，即可去像操作文件一样去操作这个io。

（2）Python 2

```python
import StringIO

audioFileLikeObj = StringIO.StringIO()
audioFileLikeObj.write(audioBinaryData)
```

## 一行代码把字符串写入保存到文件

```python
open(fullFilePath, "w").write(fileContentStr).close()
```

举例：

```python
open("0408_1600.xml", "w").write(page).close()
```

## 给文件增加可执行权限：实现`chmod +x`的效果

代码：

```python
import os
import stat

curState = os.stat(someFile)
newState = curState.st_mode | stat.S_IEXEC
os.chmod(someFile, newState)
```

再去优化为函数：

```python
import os
import stat

def chmodAddX(someFile):
    """add file executable mode, like chmod +x

    Args:
        someFile (str): file full path
    Returns:
        soup
    Raises:
    """
    if os.path.exists(someFile):
        if os.path.isfile(someFile):
            # add executable
            curState = os.stat(someFile)
            newState = curState.st_mode | stat.S_IEXEC
            os.chmod(someFile, newState)
```

调用：

```python
chmodAddX(shellFullPath)
```

继续优化：

参考 [How do you do a simple "chmod +x" from within python? - Stack Overflow](https://stackoverflow.com/questions/12791997/how-do-you-do-a-simple-chmod-x-from-within-python)

如果想要加上，给任何人都有可执行权限，则可以用：

```python
def chmodAddX(someFile):
    """add file executable mode, like chmod +x

    Args:
        someFile (str): file full path
    Returns:
        soup
    Raises:
    """
    if os.path.exists(someFile):
        if os.path.isfile(someFile):
            # add executable
            curState = os.stat(someFile)
            # STAT_OWNER_EXECUTABLE = stat.S_IEXEC
            # executableMode = STAT_OWNER_EXECUTABLE
            STAT_EVERYONE_EXECUTABLE = stat.S_IXUSR | stat.S_IXGRP | stat.S_IXOTH
            executableMode = STAT_EVERYONE_EXECUTABLE
            newState = curState.st_mode | executableMode
            os.chmod(someFile, newState)
```

效果：

* 之前：`-rw-r--r--`
* 之后：`-rwxr-xr-x`
  * 给 user group other 都加上`x`的**可执行权限**

详见：

【已解决】Python中给Mac中文件加上可执行权限

## 判断是否是文件对象

```python
import sys

def isFileObject(fileObj):
    """"check is file like object or not"""
    if sys.version_info[0] == 2:
        return isinstance(fileObj, file)
    else:
        # for python 3:
        # has read() method for:
        # io.IOBase
        # io.BytesIO
        # io.StringIO
        # io.RawIOBase
        return hasattr(fileObj, 'read')
```

## 计算当前文件名，如果重名，则位数加1

```python
import os
import re

def findNextNumberFilename(curFilename):
    """Find the next available filename from current name

    Args:
        curFilename (str): current filename
    Returns:
        next available (not existed) filename
    Raises:
    Examples:
        (1) 'crifanLib/demo/input/image/20201219_172616_drawRect_40x40.jpg'
            not exist -> 'crifanLib/demo/input/image/20201219_172616_drawRect_40x40.jpg'
        (2) 'crifanLib/demo/input/image/20191219_172616_drawRect_40x40.jpg'
            exsit -> next until not exist 'crifanLib/demo/input/image/20191219_172616_drawRect_40x40_3.jpg'
    """
    newFilename = curFilename

    newPathRootPart, pointSuffix = os.path.splitext(newFilename)
    # 'crifanLib/demo/input/image/20191219_172616_drawRect_40x40_1'
    filenamePrefix = newPathRootPart
    while os.path.exists(newFilename):
        newTailNumberInt = 1
        foundTailNumber = re.search("^(?P<filenamePrefix>.+)_(?P<tailNumber>\d+)$", newPathRootPart)
        if foundTailNumber:
            tailNumberStr = foundTailNumber.group("tailNumber") # '1'
            tailNumberInt = int(tailNumberStr)
            newTailNumberInt = tailNumberInt + 1 # 2
            filenamePrefix = foundTailNumber.group("filenamePrefix") # 'crifanLib/demo/input/image/20191219_172616_drawRect_40x40'
        # existed previously saved, change to new name
        newPathRootPart = "%s_%s" % (filenamePrefix, newTailNumberInt)
        # 'crifanLib/demo/input/image/20191219_172616_drawRect_40x40_2'
        newFilename = newPathRootPart + pointSuffix
        # 'crifanLib/demo/input/image/20191219_172616_drawRect_40x40_2.jpg'

    return newFilename
```

调用：

```python
  notExistFile = "crifanLib/demo/input/image/some_not_exist_filename.jpg"
  nextFilename = findNextNumberFilename(notExistFile)
  print("notExistFile=%s -> nextFilename=%s" % (notExistFile, nextFilename))
  # notExistFile=crifanLib/demo/input/image/some_not_exist_filename.jpg -> nextFilename=crifanLib/demo/input/image/some_not_exist_filename.jpg

  realExistFile = "crifanLib/demo/input/image/20191219_172616_drawRect_40x40.jpg"
  nextUntilNotExistFilename = findNextNumberFilename(realExistFile)
  print("realExistFile=%s -> nextUntilNotExistFilename=%s" % (realExistFile, nextUntilNotExistFilename))
  # realExistFile=crifanLib/demo/input/image/20191219_172616_drawRect_40x40.jpg -> nextUntilNotExistFilename=crifanLib/demo/input/image/20191219_172616_drawRect_40x40_2.jpg
```

## 从文件名后缀推断出MIME类型

用库：
* mime
  * GitHub
    * https://github.com/liluo/mime

安装mime：

```bash
pip install mime
```

代码：

```python
import mime

fileMimeType = mime.Types.of(curAudioFullFilename)[0].content_type
```

* 输入文件：`'Lots of Hearts.mp3'`
* 输出信息：`'audio/mpeg'`
