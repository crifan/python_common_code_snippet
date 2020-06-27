# 通用逻辑

## 多次运行一个函数，直到成功运行

执行一个函数（可能有多个可变数量的参数），且尝试多次，直到成功或超出最大此时，最终实现是：

```python
def multipleRetry(functionInfoDict, maxRetryNum=5, sleepInterval=0.1, isShowErrWhenFail=True):
    """
    do something, retry mutiple time if fail

    Args:
        functionInfoDict (dict): function info dict contain functionCallback and [optional] functionParaDict
        maxRetryNum (int): max retry number
        sleepInterval (float): sleep time of each interval when fail
        isShowErrWhenFail (bool): show error when fail if true
    Returns:
        bool
    Raises:
    """
    doSuccess = False
    functionCallback = functionInfoDict["functionCallback"]
    functionParaDict = functionInfoDict.get("functionParaDict", None)

    curRetryNum = maxRetryNum
    while curRetryNum > 0:
        if functionParaDict:
            doSuccess = functionCallback(**functionParaDict)
        else:
            doSuccess = functionCallback()
        
        if doSuccess:
            break
        
        time.sleep(sleepInterval)
        curRetryNum -= 1

    if not doSuccess:
        if isShowErrWhenFail:
            functionName = str(functionCallback)
            # '<bound method DevicesMethods.switchToAppStoreSearchTab of <src.AppCrawler.AppCrawler object at 0x1053abe80>>'
            logging.error("Still fail after %d retry for %s", maxRetryNum, functionName)
    return doSuccess
```

说明：

`functionCallback`函数类型都要符合：返回值是bool类型才可以

调用举例：

（1）没有额外参数

```python
foundAndClickedWifi = CommonUtils.multipleRetry({"functionCallback": self.iOSFromSettingsIntoWifiList})
```

其中：

```python
def iOSFromSettingsIntoWifiList(self):
。。。
    foundAndClickedWifi = self.findAndClickElement(query=wifiTextQuery, timeout=0.1)
    return foundAndClickedWifi
```

类似例子：

```python
isSwitchOk = self.multipleRetry({"functionCallback": self.switchToAppStoreSearchTab})
```

对比之前原始写法：

```python
isSwitchOk = self.switchToAppStoreSearchTab()
```

其他类似例子：

```python
foundAndClickedDownload = self.multipleRetry({"functionCallback": self.appStoreStartDownload})
```

详见：

【已解决】AppStore自动安装iOS的app：逻辑优化加等待和多试几次

（2）有额外参数，参数个数：2个

```python
searchInputQuery = {"type":"XCUIElementTypeSearchField", "name":"App Store"}
isInputOk = CommonUtils.multipleRetry(
    {
        "functionCallback": self.wait_element_setText,
        "functionParaDict": {
            "locator": searchInputQuery,
            "text": appName,
        }
    }
)
```

对比之前原始写法：

```python
searchInputQuery = {"type":"XCUIElementTypeSearchField", "name":"App Store"}
isInputOk = self.wait_element_setText(searchInputQuery, appName)
```

其中wait_element_setText的定义是：

```python
def wait_element_setText(self, locator, text):
```

对应着之前传入时的：

```python
"functionParaDict": {
    "locator": searchInputQuery,
    "text": appName,
}
```

（3）有额外参数，且加上multipleRetry的额外参数

```python
isSwitchOk = CommonUtils.multipleRetry(
    {"functionCallback": self.switchToAppStoreSearchTab},
    maxRetryNum = 10,
    sleepInterval = 0.5,
)
```

以及类似的：

```python
isIntoDetailOk = self.multipleRetry(
    {
        "functionCallback": self.appStoreSearchResultIntoDetail,
        "functionParaDict": {
            "appName": appName,
        }
    },
    sleepInterval=0.5
)
```

之前原始写法：

```python
isIntoDetailOk = self.appStoreSearchResultIntoDetail(appName)
```

注：

此处是后来加上的

`sleepInterval=0.5`

是因为后来遇到了，即使尝试了5次，依旧没找到，所以增加了没找到的延迟等待时间。

详见：

【已解决】AppStore自动安装iOS的app：逻辑优化加等待和多试几次

（4）

```python
isIntoDetailOk = CommonUtils.multipleRetry(
    {
        "functionCallback": self.appStoreSearchResultIntoDetail,
        "functionParaDict": {
            "appName": appName,
        }
    },
    maxRetryNum = 10,
    sleepInterval = 0.5,
)
```
