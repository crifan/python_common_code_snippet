# BeautifulSoup

用网络库下载到页面源码后，就是去解析（HTML等）内容了。

Python中最常用的HTML（和XML）解析库之一就是：`BeautifulSoup`

## 从xml转换出soup

背景：

iOS自动化期间，常会涉及到，获取到当前页面源码，是xml字符串，需要转换为soup，才能后续操作

所以整理出通用转换逻辑

```python
def xmlToSoup(xmlStr):
    """convert to xml string to soup
        Note: xml is tag case sensitive -> retain tag upper case -> NOT convert tag to lowercase


    Args:
        xmlStr (str): xml str, normally page source
    Returns:
        soup
    Raises:
    """
    # HtmlParser = 'html.parser'
    # XmlParser = 'xml'
    XmlParser = 'lxml-xml'
    curParser = XmlParser
    soup = BeautifulSoup(xmlStr, curParser)
    return soup
```

举例：

（1）

```python
    curPageXml = self.get_page_source()
    soup = CommonUtils.xmlToSoup(curPageXml)
```

获取到xml字符串后，去转换为soup

## 是否包含符合特定条件的soup节点

```python
    def isContainSpecificSoup(soupList, elementName, isSizeValidCallback, matchNum=1):
        """
            判断BeautifulSoup的soup的list中，是否包含符合条件的特定的元素：
                只匹配指定个数的元素才视为找到了
                元素名相同
                面积大小是否符合条件
        Args:
            elementName (str): element name
            isSizeValidCallback (function): callback function to check whether element size(width * height) is valid or not
            matchNum (int): sould only matched specific number consider as valid
        Returns:
            bool
        Raises:
        """
        isFound = False

        matchedSoupList = []

        for eachSoup in soupList:
            # if hasattr(eachSoup, "tag"):
            if hasattr(eachSoup, "name"):
                # curSoupTag = eachSoup.tag
                curSoupTag = eachSoup.name
                if curSoupTag == elementName:
                    if hasattr(eachSoup, "attrs"):
                        soupAttr = eachSoup.attrs
                        soupWidth = int(soupAttr["width"])
                        soupHeight = int(soupAttr["height"])
                        curSoupSize = soupWidth * soupHeight # 326 * 270
                        isSizeValid = isSizeValidCallback(curSoupSize)
                        if isSizeValid:
                            matchedSoupList.append(eachSoup)

        matchedSoupNum = len(matchedSoupList)
        if matchNum == 0:
            isFound = True
        else:
            if matchedSoupNum == matchNum:
                isFound = True

        return isFound
```

说明：

判断soup内，是否有符合特定条件的soup

举例：

（1）iOS的弹框，有上角带关闭按钮时，去判断一个弹框，是否符合对应条件，以便于判断是否可能是弹框

```python
nextSiblingeSoupGenerator = possibleCloseSoup.next_siblings
nextSiblingeSoupList = list(nextSiblingeSoupGenerator)

hasLargeImage = CommonUtils.isContainSpecificSoup(nextSiblingeSoupList, "XCUIElementTypeImage", self.isPopupWindowSize)
isPossibleClose = hasLargeImage
```

相关函数

```python
def isPopupWindowSize(self, curSize):
    """判断一个soup的宽高大小是否是弹框类窗口(Image,Other等）的大小"""
    # global FullScreenSize
    FullScreenSize = self.X * self.totalY
    curSizeRatio = curSize / FullScreenSize # 0.289
    PopupWindowSizeMinRatio = 0.25
    # PopupWindowSizeMaxRatio = 0.9
    PopupWindowSizeMaxRatio = 0.8
    # isSizeValid = curSizeRatio >= MinPopupWindowSizeRatio
    # is popup like window, size should large enough, but should not full screen
    isSizeValid = PopupWindowSizeMinRatio <= curSizeRatio <= PopupWindowSizeMaxRatio
    return isSizeValid
```

（2）

```python
hasNormalButton = CommonUtils.isContainSpecificSoup(nextSiblingeSoupList, "XCUIElementTypeButton", self.isNormalButtonSize)
```

相关函数：

```python
def isNormalButtonSize(self, curSize):
    """判断一个soup的宽高大小是否是普通的按钮大小"""
    NormalButtonSizeMin = 30*30
    NormalButtonSizeMax = 100*100
    isNormalSize = NormalButtonSizeMin <= curSize <= NormalButtonSizeMax
    return isNormalSize
```

## 查找元素，限定条件是符合对应的几级的父元素的条件

背景：

很多时候，需要对于iOS的app的页面的源码，即xml中，查找符合特定情况的的元素

这些特定情况，往往是parent或者前几层级的parent中，元素符合一定条件，往往是type，以及宽度是屏幕宽度，高度是屏幕高度等等

所以提取出公共函数，用于bs的find查找元素

```python
    def bsChainFind(curLevelSoup, queryChainList):
        """BeautifulSoup find with query chain

        Args:
            curLevelSoup (soup): BeautifulSoup
            queryChainList (list): str list of all level query dict
        Returns:
            soup
        Raises:
        Examples:
            input: 
                [
                    {
                        "tag": "XCUIElementTypeWindow",
                        "attrs": {"visible":"true", "enabled":"true", "width": "%s" % ScreenX, "height": "%s" % ScreenY}
                    },
                    {
                        "tag": "XCUIElementTypeButton",
                        "attrs": {"visible":"true", "enabled":"true", "width": "%s" % ScreenX, "height": "%s" % ScreenY}
                    },
                    {
                        "tag": "XCUIElementTypeStaticText",
                        "attrs": {"visible":"true", "enabled":"true", "value":"可能离开微信，打开第三方应用"}
                    },
                ]
            output:
                soup node of 
                    <XCUIElementTypeStaticText type="XCUIElementTypeStaticText" value="可能离开微信，打开第三方应用" name="可能离开微信，打开第三方应用" label="可能离开微信，打开第三方应用" enabled="true" visible="true" x="71" y="331" width="272" height="18"/>
                    in :
                    <XCUIElementTypeWindow type="XCUIElementTypeWindow" enabled="true" visible="true" x="0" y="0" width="414" height="736">
                        <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="0" y="0" width="414" height="736">
                            <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="0" y="0" width="414" height="736">
                                <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="0" y="0" width="414" height="736">
                                    <XCUIElementTypeButton type="XCUIElementTypeButton" enabled="true" visible="true" x="0" y="0" width="414" height="736">
                                        <XCUIElementTypeStaticText type="XCUIElementTypeStaticText" enabled="true" visible="false" x="47" y="288" width="0" height="0"/>
                                        <XCUIElementTypeStaticText type="XCUIElementTypeStaticText" value="可能离开微信，打开第三方应用" name="可能离开微信，打开第三方应用" label="可能离开微信，打开第三方应用" enabled="true" visible="true" x="71" y="331" width="272" height="18"/>
                                        <XCUIElementTypeStaticText type="XCUIElementTypeStaticText" value="取消" name="取消" label="取消" enabled="true" visible="true" x="109" y="409" width="36" height="22"/>
                                        <XCUIElementTypeStaticText type="XCUIElementTypeStaticText" value="继续" name="继续" label="继续" enabled="true" visible="true" x="269" y="409" width="36" height="22"/>
                                    </XCUIElementTypeButton>
                                </XCUIElementTypeOther>
                            </XCUIElementTypeOther>
                        </XCUIElementTypeOther>
                    </XCUIElementTypeWindow>
        """
        foundSoup = None
        if queryChainList:
            chainListLen = len(queryChainList)

            if chainListLen == 1:
                # last one
                curLevelFindDict = queryChainList[0]
                curTag = curLevelFindDict["tag"]
                curAttrs = curLevelFindDict["attrs"]
                foundSoup = curLevelSoup.find(curTag, attrs=curAttrs)
            else:
                highestLevelFindDict = queryChainList[0]
                curTag = highestLevelFindDict["tag"]
                curAttrs = highestLevelFindDict["attrs"]
                foundSoupList = curLevelSoup.find_all(curTag, attrs=curAttrs)
                if foundSoupList:
                    childrenChainList = queryChainList[1:]
                    for eachSoup in foundSoupList:
                        eachSoupResult = CommonUtils.bsChainFind(eachSoup, childrenChainList)
                        if eachSoupResult:
                            foundSoup = eachSoupResult
                            break

        return foundSoup
```

举例：

（1）

```python
"""
    微信-小程序 弹框 警告 尚未进行授权：
        <XCUIElementTypeButton type="XCUIElementTypeButton" enabled="true" visible="true" x="0" y="0" width="375" height="667">
            <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="32" y="240" width="311" height="187">
                <XCUIElementTypeImage type="XCUIElementTypeImage" enabled="true" visible="false" x="32" y="240" width="311" height="187"/>
                <XCUIElementTypeStaticText type="XCUIElementTypeStaticText" value="警告" name="警告" label="警告" enabled="true" visible="true" x="52" y="265" width="271" height="21"/>
                <XCUIElementTypeTextView type="XCUIElementTypeTextView" value="尚未进行授权，请点击确定跳转到授权页面进行授权。" enabled="true" visible="true" x="60" y="300" width="255" height="57"/>
                <XCUIElementTypeButton type="XCUIElementTypeButton" name="取消" label="取消" enabled="true" visible="true" x="32" y="376" width="156" height="51"/>
                <XCUIElementTypeButton type="XCUIElementTypeButton" name="确定" label="确定" enabled="true" visible="true" x="187" y="376" width="156" height="51"/>
            </XCUIElementTypeOther>
        </XCUIElementTypeButton>
"""
warningChainList = [
    {
        "tag": "XCUIElementTypeButton",
        "attrs": {"visible":"true", "enabled":"true", "width": "%s" % self.X, "height": "%s" % self.totalY}
    },
    {
        "tag": "XCUIElementTypeOther",
        "attrs": {"visible":"true", "enabled":"true"}
    },
    {
        "tag": "XCUIElementTypeStaticText",
        "attrs": {"visible":"true", "enabled":"true", "value":"警告"}
    },
]
warningSoup = CommonUtils.bsChainFind(soup, warningChainList)
```

相关：

找到元素后，再去点击：

```python
if warningSoup:
    parentOtherSoup = warningSoup.parent
    confirmSoup = parentOtherSoup.find(
        "XCUIElementTypeButton",
        attrs={"visible":"true", "enabled":"true", "name": "确定"}
    )
    if confirmSoup:
        self.clickElementCenterPosition(confirmSoup)
        foundAndProcessedPopup = True
```

（2）

```python
"""
    系统弹框 拍照或录像：
        <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="8" y="530" width="398" height="133">
            <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="8" y="530" width="398" height="133">
                <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="8" y="530" width="398" height="133">
                    <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="false" x="0" y="0" width="398" height="133">
                        <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="0" y="0" width="398" height="133">
                            <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="0" y="0" width="398" height="133">
                                <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="0" y="0" width="398" height="133">
                                    <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="0" y="0" width="398" height="133">
                                        <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="0" y="0" width="398" height="133">
                                            <XCUIElementTypeTable type="XCUIElementTypeTable" enabled="true" visible="true" x="0" y="0" width="398" height="133">
                                                <XCUIElementTypeCell type="XCUIElementTypeCell" enabled="true" visible="true" x="0" y="0" width="398" height="45">
                                                    <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="0" y="43" width="398" height="2"/>
                                                    <XCUIElementTypeImage type="XCUIElementTypeImage" enabled="true" visible="false" x="13" y="6" width="1" height="32"/>
                                                    <XCUIElementTypeStaticText type="XCUIElementTypeStaticText" value="拍照或录像" name="拍照或录像" label="拍照或录像" enabled="true" visible="true" x="15" y="11" width="87" height="22"/>
                                                    <XCUIElementTypeImage type="XCUIElementTypeImage" enabled="true" visible="false" x="351" y="6" width="32" height="32"/>
                                                </XCUIElementTypeCell>
                                                <XCUIElementTypeCell type="XCUIElementTypeCell" enabled="true" visible="true" x="0" y="44" width="398" height="45">
                                                    <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="0" y="88" width="398" height="1"/>
                                                    <XCUIElementTypeImage type="XCUIElementTypeImage" enabled="true" visible="false" x="13" y="50" width="1" height="33"/>
                                                    <XCUIElementTypeStaticText type="XCUIElementTypeStaticText" value="照片图库" name="照片图库" label="照片图库" enabled="true" visible="true" x="15" y="56" width="70" height="21"/>
                                                    <XCUIElementTypeImage type="XCUIElementTypeImage" enabled="true" visible="false" x="351" y="50" width="32" height="33"/>
                                                </XCUIElementTypeCell>
                                                <XCUIElementTypeCell type="XCUIElementTypeCell" enabled="true" visible="true" x="0" y="88" width="398" height="45">
                                                    <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="false" x="0" y="132" width="398" height="1"/>
                                                    <XCUIElementTypeImage type="XCUIElementTypeImage" enabled="true" visible="false" x="13" y="94" width="1" height="33"/>
                                                    <XCUIElementTypeStaticText type="XCUIElementTypeStaticText" value="浏览" name="浏览" label="浏览" enabled="true" visible="true" x="14" y="100" width="36" height="21"/>
                                                    <XCUIElementTypeImage type="XCUIElementTypeImage" name="UIDocumentPicker-more" enabled="true" visible="false" x="351" y="94" width="32" height="33"/>
                                                </XCUIElementTypeCell>
                                                <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="0" y="132" width="398" height="1"/>
                                            </XCUIElementTypeTable>
                                        </XCUIElementTypeOther>
                                    </XCUIElementTypeOther>
                                </XCUIElementTypeOther>
                            </XCUIElementTypeOther>
                        </XCUIElementTypeOther>
                    </XCUIElementTypeOther>
                </XCUIElementTypeOther>
            </XCUIElementTypeOther>
        </XCUIElementTypeOther>
        <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="8" y="671" width="398" height="57">
            <XCUIElementTypeButton type="XCUIElementTypeButton" name="取消" label="取消" enabled="true" visible="true" x="8" y="671" width="398" height="57"/>
        </XCUIElementTypeOther>
"""
photoCameraChainList = [
    {
        "tag": "XCUIElementTypeOther",
        "attrs": {"enabled":"true", "visible":"true"}
    },
    {
        "tag": "XCUIElementTypeTable",
        "attrs": {"enabled":"true", "visible":"true", "x":"0", "y":"0"}
    },
    {
        "tag": "XCUIElementTypeStaticText",
        "attrs": {"enabled":"true", "visible":"true", "value":"拍照或录像"}
    },
]
photoCameraSoup = CommonUtils.bsChainFind(soup, photoCameraChainList)
```


（3）iOS 设置 无线局域网 列表页 找 当前已连接的WiFi，特征是带蓝色✅的：

```python
"""
    设置 无线局域网 列表页：
        <XCUIElementTypeTable type="XCUIElementTypeTable" enabled="true" visible="true" x="0" y="0" width="414" height="736">
        。。。
            <XCUIElementTypeCell type="XCUIElementTypeCell" name=“xxx_guest_5G, 安全网络, 信号强度 3 格，共 3 格" label=“xxx_guest_5G, 安全网络, 信号强度 3 格，共 3 格" enabled="true" visible="true" x="0" y="144" width="414" height="43">
                <XCUIElementTypeStaticText type="XCUIElementTypeStaticText" value=“xxx_guest_5G" name=“xxx_guest_5G" label=“xxx_guest_5G" enabled="true" visible="true" x="40" y="155" width="278" height="21"/>
                <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="0" y="186" width="414" height="1"/>
                <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="true" x="8" y="151" width="28" height="29">
                    <XCUIElementTypeImage type="XCUIElementTypeImage" name="UIPreferencesBlueCheck" enabled="true" visible="false" x="8" y="151" width="28" height="29"/>
                </XCUIElementTypeOther>
                <XCUIElementTypeOther type="XCUIElementTypeOther" enabled="true" visible="false" x="15" y="187" width="245" height="1"/>
                <XCUIElementTypeImage type="XCUIElementTypeImage" name="Lock" enabled="true" visible="false" x="326" y="159" width="8" height="12"/>
                <XCUIElementTypeImage type="XCUIElementTypeImage" name="WifiBars3" enabled="true" visible="false" x="346" y="153" width="16" height="25"/>
                <XCUIElementTypeButton type="XCUIElementTypeButton" name="更多信息" label="更多信息" enabled="true" visible="true" x="372" y="154" width="22" height="22"/>
            </XCUIElementTypeCell>
"""
curPageXml = self.get_page_source()
soup = CommonUtils.xmlToSoup(curPageXml)
blueCheckChainList = [
    {
        "tag": "XCUIElementTypeCell",
        "attrs": {"enabled":"true", "visible":"true", "x":"0", "width":"%s" % self.X}
    },
    {
        "tag": "XCUIElementTypeOther",
        "attrs": {"enabled":"true", "visible":"true"}
    },
    {
        "tag": "XCUIElementTypeImage",
        # "attrs": {"enabled":"true", "visible":"true", "name": "UIPreferencesBlueCheck"}
        "attrs": {"enabled":"true", "name": "UIPreferencesBlueCheck"}
    },
]
blueCheckSoup = CommonUtils.bsChainFind(soup, blueCheckChainList)
if blueCheckSoup:
```

