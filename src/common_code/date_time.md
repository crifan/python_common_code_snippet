# 日期时间

详见：

https://github.com/crifan/crifanLibPython/blob/master/crifanLib/crifanDatetime.py

## getCurDatetimeStr 生成当前日期时间字符串

```python
def getCurDatetimeStr(outputFormat="%Y%m%d_%H%M%S"):
    """
    get current datetime then format to string

    eg:
        20171111_220722

    :param outputFormat: datetime output format
    :return: current datetime formatted string
    """
    curDatetime = datetime.now() # 2017-11-11 22:07:22.705101
    curDatetimeStr = curDatetime.strftime(format=outputFormat) #'20171111_220722'
    return curDatetimeStr
```

调用举例：

```python
curDatetimeStr = getCurDatetimeStr() # '20191219_143400'
```

## datetime转时间戳

```python
import time

def datetimeToTimestamp(self, datetimeVal, withMilliseconds=False) :
    """
        convert datetime value to timestamp
        eg:
            "2006-06-01 00:00:00.123" -> 1149091200
            if with milliseconds -> 1149091200123
    :param datetimeVal:
    :return:
    """
    timetupleValue = datetimeVal.timetuple()
    timestampFloat = time.mktime(timetupleValue) # 1531468736.0 -> 10 digits
    timestamp10DigitInt = int(timestampFloat) # 1531468736
    timestampInt = timestamp10DigitInt

    if withMilliseconds:
        microsecondInt = datetimeVal.microsecond # 817762
        microsecondFloat = float(microsecondInt)/float(1000000) # 0.817762
        timestampFloat = timestampFloat + microsecondFloat # 1531468736.817762
        timestampFloat = timestampFloat * 1000 # 1531468736817.7621 -> 13 digits
        timestamp13DigitInt = int(timestampFloat) # 1531468736817
        timestampInt = timestamp13DigitInt

    return timestampInt
```

## 获取当前时间戳

```python
from datetime import datetime

def getCurTimestamp(withMilliseconds=False):
    """
    get current time's timestamp
        (default)not milliseconds -> 10 digits: 1351670162
        with milliseconds -> 13 digits: 1531464292921
    """
    curDatetime = datetime.now()
    return datetimeToTimestamp(curDatetime, withMilliseconds)
```

## 时间戳精确到毫秒

```python
from datetime import datetime,timedelta
timestampStr = datetime.now().strftime("%Y%m%d_%H%M%S_%f")
# 20180712_154134_660436
```
