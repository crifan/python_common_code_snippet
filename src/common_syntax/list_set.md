# list列表和set集合

## list vs set

* set
    * 适用于检测某元素是否在集合内、对集合进行一定的数学操作
    * 不支持indexing，slicing
* list
    * 普通的数组
    * 支持indexing，slicing

## 把list换成set

```python
someSet = set([])
for eachItem in someList:
    someSet.add(eachItem)
```

## set集合转换为字符串

```python
someSetStr = ", ".join(someSet)
```

## 把列表转为python正则中的group中可能出现的选项

```python
def listToPatternGroup(curList):
    """Convert list to pattern group"""
    patternGroupList = list(map(lambda curType: "(%s)" % curType, curList)) # ['(aaa)', '(bbb)', '(ccc)', '(zzz)', '(eee)', '(yyy)', '(ddd)', '(xxx)']
    groupP = "|".join(patternGroupList) # '(aaa)|(bbb)|(ccc)|(zzz)|(eee)|(yyy)|(ddd)|(xxx)'
    return groupP
```

调用：

```python
ValidPlatformTypeList = ["iOS", "Android"]
ValidPlatformRule = listToPatternGroup(ValidPlatformTypeList) # '(iOS)|(Android)'
```

目的是用于后续的正则判断

```python
TaskFilenamePattern = "(?P<taskDate>\d+)_(?P<businessType>%s)_(?P<taskName>[a-zA-Z\d]+)_(?P<crawlerType>%s)(_(?P<platform>%s))?" % (ValidBusinessTypeRule, ValidCrawlerTypeRule, ValidPlatformRule)
```
