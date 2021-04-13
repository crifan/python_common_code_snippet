# logging日志

## 彩色日志+日志初始化

自己的库：[crifanLogging.py](https://github.com/crifan/crifanLibPython/blob/master/python3/crifanLib/crifanLogging.py)

已实现常用的功能，包括：

* 彩色日志
* 初始化

使用方式 = 典型调用代码：

先下载我的库：

* [crifanLogging.py](https://github.com/crifan/crifanLibPython/blob/master/python3/crifanLib/crifanLogging.py)
  * https://github.com/crifan/crifanLibPython/blob/master/python3/crifanLib/crifanLogging.py

对于文件：`somePythonFile.py`

调用和初始化代码：

```python
import crifanLogging

CurFilePath = os.path.abspath(__file__)
# print("CurFilePath=%s" % CurFilePath)
CurFilename = os.path.basename(CurFilePath)
# 'autoSearchGame_YingYongBao.py'
CurFileNoSuffix, pointSuffix = os.path.splitext(CurFilename)

CurFolder = os.path.dirname(CurFilePath)
# print("CurFolder=%s" % CurFolder)

LogFolder = os.path.join(CurFolder, "logs")

def initLog():
    curDatetimeStr = utils.getCurDatetimeStr() # '20200316_155954'
    utils.createFolder(LogFolder)
    curLogFile = "%s_%s.log" % (CurFileNoSuffix, curDatetimeStr)
    logFullPath = os.path.join(LogFolder, curLogFile)
    crifanLogging.loggingInit(logFullPath)

def main():
    initLog()
```

即可生成log文件：`logs/somePythonFile.log`

注：相关函数：

* `createFolder`
  * [新建文件夹](https://book.crifan.com/books/python_common_code_snippet/website/common_code/file_system/folder.html#%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9)
* `getCurDatetimeStr`
  * [getCurDatetimeStr 生成当前日期时间字符串](https://book.crifan.com/books/python_common_code_snippet/website/common_code/date_time.html#getcurdatetimestr-%E7%94%9F%E6%88%90%E5%BD%93%E5%89%8D%E6%97%A5%E6%9C%9F%E6%97%B6%E9%97%B4%E5%AD%97%E7%AC%A6%E4%B8%B2)
