# collections集合

根据官网：

[collections --- 容器数据类型 — Python 3.8.1 文档](https://docs.python.org/zh-cn/3/library/collections.html#collections.UserDict)

介绍，集合有很多种，列出供了解：

* `namedtuple()`：创建命名元组子类的工厂函数
* `deque`：类似列表(list)的容器，实现了在两端快速添加(append)和弹出(pop)
* `ChainMap`：类似字典(dict)的容器类，将多个映射集合到一个视图里面
* `Counter`：字典的子类，提供了可哈希对象的计数功能
* `OrderedDict`：字典的子类，保存了他们被添加的顺序
* `defaultdict`：字典的子类，提供了一个工厂函数，为字典查询提供一个默认值
* `UserDict`：封装了字典对象，简化了字典子类化
* `UserList`：封装了列表对象，简化了列表子类化
* `UserString`：封装了列表对象，简化了字符串子类化

待以后用到了，再详细总结。
