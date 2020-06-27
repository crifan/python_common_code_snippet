# sort排序

详见：

* https://github.com/crifan/crifanLibPython/blob/master/crifanLib/crifanDict.py
* https://github.com/crifan/crifanLibPython/blob/master/crifanLib/demo/crifanDictDemo.py

## 对字典根据key去排序

```python
from collections import OrderedDict

def sortDictByKey(originDict):
    """
        Sort dict by key
    """
    originItems = originDict.items()
    sortedOriginItems = sorted(originItems)
    sortedOrderedDict = OrderedDict(sortedOriginItems)
    return sortedOrderedDict
```

调用：

```python
def demoSortDictByKey():
  originDict = {
    "c": "abc",
    "a": 1,
    "b": 22
  }
  print("originDict=%s" % originDict)
  # originDict={'c': 'abc', 'a': 1, 'b': 22}
  sortedOrderedDict = sortDictByKey(originDict)
  print("sortedOrderedDict=%s" % sortedOrderedDict)
  # sortedOrderedDict=OrderedDict([('a', 1), ('b', 22), ('c', 'abc')])
```

## sort和sorted

```python
# Function: Demo sorted
#   mainly refer official doc:
#       排序指南 — Python 3.8.2 文档
#       https://docs.python.org/zh-cn/3/howto/sorting.html
# Author: Crifan Li
# Update: 20200304


from operator import itemgetter, attrgetter


print("%s %s %s" % ('='*40, "sort", '='*40))


originIntList = [5, 2, 3, 1, 4]
originIntList.sort()
sortedSelfIntList = originIntList
print("sortedSelfIntList=%s" % sortedSelfIntList)
# sortedSelfIntList=[1, 2, 3, 4, 5]


print("%s %s %s" % ('='*40, "sorted", '='*40))


intList = [5, 2, 3, 1, 4]
sortedIntList = sorted(intList)
print("sortedIntList=%s" % sortedIntList)
# sortedIntList=[1, 2, 3, 4, 5]


reversedSortIntList = sorted(intList, reverse=True)
print("reversedSortIntList=%s" % reversedSortIntList)
# reversedSortIntList=[5, 4, 3, 2, 1]


intStrDict = {5: 'A', 1: 'D', 2: 'B',  4: 'E', 3: 'B'}
dictSortedIntList = sorted(intStrDict)
print("dictSortedIntList=%s" % dictSortedIntList)
# dictSortedIntList=[1, 2, 3, 4, 5]


normalStr = "Crifan Li best love language is Python"
strList = normalStr.split()
print("strList=%s" % strList)
sortedStrList = sorted(strList, key=str.lower)
print("sortedStrList=%s" % sortedStrList)
# strList=['Crifan', 'Li', 'best', 'love', 'language', 'is', 'Python']
# sortedStrList=['best', 'Crifan', 'is', 'language', 'Li', 'love', 'Python']


studentTupleList = [
    # name, grade, age
    ('Cindy', 'A', 15),
    ('Crifan', 'B', 12),
    ('Tony', 'B', 10),
]
sortedTupleList_lambda = sorted(studentTupleList, key=lambda student: student[2]) # [2] is age
print("sortedTupleList_lambda=%s" % sortedTupleList_lambda)
# sortedTupleList_lambda=[('Tony', 'B', 10), ('Crifan', 'B', 12), ('Cindy', 'A', 15)]


# same as single function:
def getStudentAge(curStudentTuple):
    return curStudentTuple[2] # [2] is age
sortedTupleList_singleFunction = sorted(studentTupleList, key=getStudentAge)
print("sortedTupleList_singleFunction=%s" % sortedTupleList_singleFunction)
# sortedTupleList_singleFunction=[('Tony', 'B', 10), ('Crifan', 'B', 12), ('Cindy', 'A', 15)]


# same as operator itemgetter:
sortedTupleList_operator = sorted(studentTupleList, key=itemgetter(2))
print("sortedTupleList_operator=%s" % sortedTupleList_operator)
# sortedTupleList_operator=[('Tony', 'B', 10), ('Crifan', 'B', 12), ('Cindy', 'A', 15)]


class Student:
    def __init__(self, name, grade, age):
        self.name = name
        self.grade = grade
        self.age = age
    def __repr__(self):
        return repr((self.name, self.grade, self.age))


studentObjectList = [
    Student('john', 'A', 15),
    Student('jane', 'A', 15),
    Student('dave', 'A', 15),
]
sortedObjectList = sorted(studentObjectList, key=lambda student: student.age)
print("sortedObjectList=%s" % sortedObjectList)
# sortedObjectList=[('john', 'A', 15), ('jane', 'A', 15), ('dave', 'A', 15)]


# same as operator attrgetter:
sortedObjectList_operator = sorted(sortedObjectList, key=attrgetter('age'))
print("sortedObjectList_operator=%s" % sortedObjectList_operator)
# sortedObjectList_operator=[('john', 'A', 15), ('jane', 'A', 15), ('dave', 'A', 15)]
```


