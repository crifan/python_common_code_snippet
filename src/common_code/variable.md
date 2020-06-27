# 变量

## 判断变量类型

优先用`isinstance`，而不是`type`

```python
>>> isinstance(2, float)
False
>>> isinstance('a', (str, unicode))
True
>>> isinstance((2, 3), (str, list, tuple))
True
```
