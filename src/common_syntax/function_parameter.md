# 函数参数

## 可变参数

之前一个用到了可变参数的函数是：

```python
def multipleRetry(self, functionInfoDict, maxRetryNum=5, sleepInterval=0.1):
    """
    do something, retry mutiple time if fail

    Args:
        functionInfoDict (dict): function info dict contain functionCallback and [optional] functionParaDict
        maxRetryNum (int): max retry number
        sleepInterval (float): sleep time of each interval when fail
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
        functionName = str(functionCallback)
        # '<bound method DevicesMethods.switchToAppStoreSearchTab of <src.AppCrawler.AppCrawler object at 0x1053abe80>>'
        logging.error("Still fail after %d retry for %s", functionName)
    return doSuccess
```

其中的：

`functionCallback(**functionParaDict)`

中的：

`**functionParaDict`

表示，dict类型的参数，内部包含多个key和value，用**去展开后，传入真正要执行的函数

几种调用中带参数的例子是：

```python
searchInputQuery = {"type":"XCUIElementTypeSearchField", "name":"App Store"}
isInputOk = self.multipleRetry(
    {
        "functionCallback": self.wait_element_setText,
        "functionParaDict": {
            "locator": searchInputQuery,
            "text": appName,
        }
    }
)
```

之前原始写法：

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

即可，给出上述细节，便于理解，传入的参数是如何用`**`展开的。

详见：

【已解决】Python中如何实现函数调用时多个可变数量的参数传递
