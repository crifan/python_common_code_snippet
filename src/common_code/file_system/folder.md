# 文件夹=文件路径

## 新建文件夹

对于：`python 3.2+`

```python
import os

def createFolder(folderFullPath):
    """
        create folder, even if already existed
        Note: for Python 3.2+
    """
    os.makedirs(folderFullPath, exist_ok=True)
    # print("Created folder: %s" % folderFullPath)
```

或：

对于：`python 3.5+`

```python
import pathlib
pathlib.Path('/my/directory').mkdir(parents=True, exist_ok=True) 
```

## 批量删除非空文件夹

```python
import shutil

if os.path.exists(folderToDelete):
    shutil.rmtree(folderToDelete)
```

注意：

* 删除之前要先用`os.path.exists`判断是非存在该目录
    * 如果不存在就去删除，则会报错：`OSError: [Errno 2] No such file or directory`

## os.path 路径处理

```python
#!/usr/bin/python
# -*- coding: utf-8 -*-
# Author: Crifan Li
# Update: 20191219
# Function: Demo os.path common used functions

import os

def osPathDemo():
    currentSystemInfo = os.uname()
    print("currentSystemInfo=%s" % (currentSystemInfo, ))
    # currentSystemInfo=posix.uname_result(sysname='Darwin', nodename='xxx', release='18.7.0', version='Darwin Kernel Version 18.7.0: Sat Oct 12 00:02:19 PDT 2019; root:xnu-4903.278.12~1/RELEASE_X86_64', machine='x86_64')

    pathSeparatorInCurrentOS = os.path.sep
    print("pathSeparatorInCurrentOS=%s" % pathSeparatorInCurrentOS)
    # pathSeparatorInCurrentOS=/

    fullFilePath = "/Users/limao/dev/crifan/python/notEnoughUnpack/Snip20191212_113.png"
    print("fullFilePath=%s" % fullFilePath)
    # fullFilePath=/Users/limao/dev/crifan/python/notEnoughUnpack/Snip20191212_113.png

    dirname = os.path.dirname(fullFilePath)
    print("dirname=%s" % dirname)
    # dirname=/Users/limao/dev/crifan/python/notEnoughUnpack
    basename = os.path.basename(fullFilePath)
    print("basename=%s" % basename)
    # basename=Snip20191212_113.png
    joinedFullPath = os.path.join(dirname, basename)
    print("joinedFullPath=%s" % joinedFullPath)
    # joinedFullPath=/Users/limao/dev/crifan/python/notEnoughUnpack/Snip20191212_113.png
    isSame = (fullFilePath == joinedFullPath)
    print("isSame=%s" % isSame)
    # isSame=True

    root, pointSuffix = os.path.splitext(fullFilePath)
    print("root=%s, pointSuffix=%s" % (root, pointSuffix))
    # root=/Users/limao/dev/crifan/python/notEnoughUnpack/Snip20191212_113, pointSuffix=.png
    head, tail = os.path.split(fullFilePath)
    print("head=%s, tail=%s" % (head, tail))
    # head=/Users/limao/dev/crifan/python/notEnoughUnpack, tail=Snip20191212_113.png
    drive, tail = os.path.splitdrive(fullFilePath)
    print("drive=%s, tail=%s" % (drive, tail))
    # drive=, tail=/Users/limao/dev/crifan/python/notEnoughUnpack/Snip20191212_113.png

    curPath = os.getcwd()
    print("curPath=%s" % curPath)
    # curPath=/Users/limao/dev/crifan/python
    relativePath = os.path.relpath(fullFilePath)
    print("relativePath=%s" % relativePath)
    # relativePath=notEnoughUnpack/Snip20191212_113.png

    isFile = os.path.isfile(fullFilePath)
    print("isFile=%s" % isFile)
    # isFile=True
    isDirectory = os.path.isdir(fullFilePath)
    print("isDirectory=%s" % isDirectory)
    # isDirectory=False

    fileSize = os.path.getsize(fullFilePath)
    print("fileSize=%s" % fileSize)
    # fileSize=368810

    isFileOrFolderRealExist = os.path.exists(fullFilePath)
    print("isFileOrFolderRealExist=%s" % isFileOrFolderRealExist)
    # isFileOrFolderRealExist=True

if __name__ == "__main__":
    osPathDemo()
```

## 列出目录中的文件（和文件夹，且支持递归）

```python
def listSubfolderFiles(subfolder, isIncludeFolder=True, isRecursive=False):
    """os.listdir recursively

    Args:
        subfolder (str): sub folder path
        isIncludeFolder (bool): whether is include folder. Default is True. If True, result contain folder
        isRecursive (bool): whether is recursive, means contain sub folder. Default is False
    Returns:
        list of str
    Raises:
    """
    allSubItemList = []
    curSubItemList = os.listdir(path=subfolder)
    for curSubItem in curSubItemList:
        curSubItemFullPath = os.path.join(subfolder, curSubItem)
        if os.path.isfile(curSubItemFullPath):
            allSubItemList.append(curSubItemFullPath)
        else:
            if isIncludeFolder:
                if os.path.isdir(curSubItemFullPath):
                    subSubItemList = listSubfolderFiles(curSubItemFullPath, isIncludeFolder, isRecursive)
                    allSubItemList.extend(subSubItemList)

    if isIncludeFolder:
        allSubItemList.append(subfolder)

    return allSubItemList
```
