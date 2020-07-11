# dict字典

## 删除dict中某个键（和值）

* 常见写法：
    ```python
    del yourDict["keyToDelete"]
    ```
* 更加Pythonic的写法：
    ```python
    yourDict.pop("keyToDelete")
    ```

注意：

为了防止出现`KeyError`，注意确保要删除的key都是存在的，否则就要先判断存在，再去删除。

## OrderedDict

### 想要获取OrderedDict的最后一个item（的key和value）

```python
next(reversed(someOrderedDict.items()))
```

另外，只需要获取最后一个元素的key，则可以：

```python
next(reversed(someOrderedDict.keys()))
```

或：

```python
next(reversed(someOrderedDict))
```

详见：

【已解决】Python中获取OrderedDict中最后一个元素

## 合并2个dict的值

（1）如果无需保留原有（第一个dict）的值，则用update即可：

```python
firstDict.update(secondDict)
```

支持：`Python >=3.5`

（2）如果要保留之前的dict的值，则用**展开

```python
thirdDict = (**firstDict, **secondDict)
```

支持：`Python 2`和 `Python <=3.4`

详见：

【已解决】Python中如何合并2个dict字典变量的值

## 有序字典OrderedDict的初始化

```python
from collections import OrderedDict

orderedDict = OrderedDict()
```

后续正常作为普通dict使用

```python
>>> from collections import OrderedDict
>>> orderedDict = OrderedDict()
>>> orderedDict["key2"] = "value2"
>>> orderedDict["key1"] = "value1"
>>> orderedDict["key3"] = "value3"
>>> orderedDict
OrderedDict([('key2', 'value2'), ('key1', 'value1'), ('key3', 'value3')])
```

## dict的递归的合并更新

```python

def recursiveMergeDict(aDict, bDict):
    """
    Recursively merge dict a to b, return merged dict b
    Note: Sub dict and sub list's won't be overwritten but also updated/merged

    example:
(1) input and output example:
input:
{
  "keyStr": "strValueA",
  "keyInt": 1,
  "keyBool": true,
  "keyList": [
    {
      "index0Item1": "index0Item1",
      "index0Item2": "index0Item2"
    },
    {
      "index1Item1": "index1Item1"
    },
    {
      "index2Item1": "index2Item1"
    }
  ]
}

and

{
  "keyStr": "strValueB",
  "keyInt": 2,
  "keyList": [
    {
      "index0Item1": "index0Item1_b"
    },
    {
      "index1Item1": "index1Item1_b"
    }
  ]
}

output:

{
  "keyStr": "strValueB", 
  "keyBool": true, 
  "keyInt": 2,
  "keyList": [
    {
      "index0Item1": "index0Item1_b", 
      "index0Item2": "index0Item2"
    }, 
    {
      "index1Item1": "index1Item1_b"
    }, 
    {
      "index2Item1": "index2Item1"
    }
  ]
}

(2) code usage example:
import copy
cDict = recursiveMergeDict(aDict, copy.deepcopy(bDict))

Note:
bDict should use deepcopy, otherwise will be altered after call this function !!!

    """
    aDictItems = None
    if (sys.version_info[0] == 2): # is python 2
      aDictItems = aDict.iteritems()
    else: # is python 3
      aDictItems = aDict.items()

    for aKey, aValue in aDictItems:
      # print("------ [%s]=%s" % (aKey, aValue))
      if aKey not in bDict:
        bDict[aKey] = aValue
      else:
        bValue = bDict[aKey]
        # print("aValue=%s" % aValue)
        # print("bValue=%s" % bValue)
        if isinstance(aValue, dict):
          recursiveMergeDict(aValue, bValue)
        elif isinstance(aValue, list):
          aValueListLen = len(aValue)
          bValueListLen = len(bValue)
          bValueListMaxIdx = bValueListLen - 1
          for aListIdx in range(aValueListLen):
            # print("---[%d]" % aListIdx)
            aListItem = aValue[aListIdx]
            # print("aListItem=%s" % aListItem)
            if aListIdx <= bValueListMaxIdx:
              bListItem = bValue[aListIdx]
              # print("bListItem=%s" % bListItem)
              recursiveMergeDict(aListItem, bListItem)
            else:
              # print("bDict=%s" % bDict)
              # print("aKey=%s" % aKey)
              # print("aListItem=%s" % aListItem)
              bDict[aKey].append(aListItem)

    return bDict
```

调用举例：

```python

templateJson = {
  "author": "Crifan Li <admin@crifan.com>",
  "description": "gitbook书的描述",
  "gitbook": "3.2.3",
  "language": "zh-hans",
  "links": { "sidebar": { "主页": "http://www.crifan.com" } },
  "plugins": [
    "theme-comscore",
    "anchors",
    "-lunr",
    "-search",
    "search-plus",
    "disqus",
    "-highlight",
    "prism",
    "prism-themes",
    "github-buttons",
    "splitter",
    "-sharing",
    "sharing-plus",
    "tbfed-pagefooter",
    "expandable-chapters-small",
    "ga",
    "donate",
    "sitemap-general",
    "copy-code-button",
    "callouts",
    "toolbar-button"
  ],
  "pluginsConfig": {
    "callouts": { "showTypeInHeader": false },
    "disqus": { "shortName": "crifan" },
    "donate": {
      "alipay": "https://www.crifan.com/files/res/crifan_com/crifan_alipay_pay.jpg",
      "alipayText": "支付宝打赏给Crifan",
      "button": "打赏",
      "title": "",
      "wechat": "https://www.crifan.com/files/res/crifan_com/crifan_wechat_pay.jpg",
      "wechatText": "微信打赏给Crifan"
    },
    "ga": { "token": "UA-28297199-1" },
    "github-buttons": {
      "buttons": [
        {
          "count": true,
          "repo": "gitbook_name",
          "size": "small",
          "type": "star",
          "user": "crifan"
        },
        {
          "count": false,
          "size": "small",
          "type": "follow",
          "user": "crifan",
          "width": "120"
        }
      ]
    },
    "prism": { "css": ["prism-themes/themes/prism-atom-dark.css"] },
    "sharing": {
      "all": [
        "douban",
        "facebook",
        "google",
        "instapaper",
        "line",
        "linkedin",
        "messenger",
        "pocket",
        "qq",
        "qzone",
        "stumbleupon",
        "twitter",
        "viber",
        "vk",
        "weibo",
        "whatsapp"
      ],
      "douban": false,
      "facebook": true,
      "google": false,
      "hatenaBookmark": false,
      "instapaper": false,
      "line": false,
      "linkedin": false,
      "messenger": false,
      "pocket": false,
      "qq": true,
      "qzone": false,
      "stumbleupon": false,
      "twitter": true,
      "viber": false,
      "vk": false,
      "weibo": true,
      "whatsapp": false
    },
    "sitemap-general": {
      "prefix": "https://book.crifan.com/gitbook/gitbook_name/website/"
    },
    "tbfed-pagefooter": {
      "copyright": "crifan.com，使用<a \"href=\"https://creativecommons.org/licenses/by/4.0/deed.zh\">署名4.0国际(CC \"BY 4.0)协议</a>发布",
      "modify_format": "YYYY-MM-DD HH:mm:ss",
      "modify_label": "最后更新："
    },
    "theme-default": { "showLevel": true },
    "toolbar-button": {
      "icon": "fa-file-pdf-o",
      "label": "下载PDF",
      "url": "http://book.crifan.com/books/gitbook_name/pdf/gitbook_name.pdf"
    }
  },
  "root": "./src",
  "title": "Gitbook的书名"
}

currentJson = {
  "description": "crifan整理的Python各个方面常用的代码段，供需要的参考。",
  "pluginsConfig": {
    "github-buttons": { "buttons": [{ "repo": "python_common_code_snippet" }] },
    "sitemap-general": {
      "prefix": "https://book.crifan.com/gitbook/python_common_code_snippet/website/"
    },
    "toolbar-button": {
      "url": "http://book.crifan.com/books/python_common_code_snippet/pdf/python_common_code_snippet.pdf"
    }
  },
  "title": "Python常用代码段"
}


 bookJson = recursiveMergeDict(templateJson, copy.deepcopy(currentJson))

```
