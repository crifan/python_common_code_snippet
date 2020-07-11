# 数学

详见：

https://github.com/crifan/crifanLibPython/blob/master/crifanLib/crifanMath.py

---

## md5

### md5计算

* `md5`
  * `Python 3`中已改名`hashlib`
  * 且update参数只允许`bytes`
    * 不允许`str`

的md5代码：

```python
from hashlib import md5 # only for python 3.x

def generateMd5(strToMd5) :
    """
    generate md5 string from input string
    eg:
        xxxxxxxx -> af0230c7fcc75b34cbb268b9bf64da79
    :param strToMd5: input string
    :return: md5 string of 32 chars
    """
    encrptedMd5 = ""
    md5Instance = md5()
    # print("type(md5Instance)=%s" % type(md5Instance)) # type(md5Instance)=<class '_hashlib.HASH'>
    # print("type(strToMd5)=%s" % type(strToMd5)) # type(strToMd5)=<class 'str'>
    bytesToMd5 = bytes(strToMd5, "UTF-8")
    # print("type(bytesToMd5)=%s" % type(bytesToMd5)) # type(bytesToMd5)=<class 'bytes'>
    md5Instance.update(bytesToMd5)
    encrptedMd5 = md5Instance.hexdigest()
    # print("type(encrptedMd5)=%s" % type(encrptedMd5)) # type(encrptedMd5)=<class 'str'>
    # print("encrptedMd5=%s" % encrptedMd5) # encrptedMd5=3a821616bec2e86e3e232d0c7f392cf5
    return encrptedMd5
```


之前旧版本的`Python 2`（`<= 2.7`）版本：

* md5还是个独立模块
    * 还没有并入`hashlib`
    * 注：
        * 好像`python 2.7`中已将md5并入`hashlib`
            * 但是`update`参数还允许`str`（而不是`bytes`）
* update参数允许str

的md5代码：

```python
try:
    import md5
except ImportError:
    from hashlib import md5

def generateMd5(strToMd5) :
    encrptedMd5 = ""
    md5Instance = md5.new()
    #md5Instance=<md5 HASH object @ 0x1062af738>
    md5Instance.update(strToMd5)
    encrptedMd5 = md5Instance.hexdigest()
    #encrptedMd5=af0230c7fcc75b34cbb268b9bf64da79
    return encrptedMd5
```
