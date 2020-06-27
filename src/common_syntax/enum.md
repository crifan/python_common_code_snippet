# enum枚举

## 枚举基本用法

### 枚举定义

举例1：

```python
from enum import Enum

class BatteryState(Enum):
    Unknown = 0
    Unplugged = 1
    Charging = 2
    Full = 3
```

举例2：

```python
import enum

class ScreenshotQuality(enum.Enum):
    Original = 0
    Medium = 1
    Low = 2
```

举例3：

```python
class SentenceInvalidReason(Enum):
    NONE = "none"
    UNKNOWN = "unknown"
    EMPTY = "empty
    TOO_SHORT = "too short"
    TOO_LONG = "too long"
    TOO_MANY_INVALID_WORD = "contain too many invalid words"
```

### 初始化创建枚举值

直接传入对应的（此处是int）值即可：

```python
batteryStateInt = 2
curBattryStateEnum = BatteryState(batteryStateInt)
```

log输出是：

```bash
curBattryStateEnum=BatteryState.Charging
```

### 获取枚举的名称

```python
curBattryStateName = curBattryStateEnum.name
```

输出：`'Charging'`

### 获取枚举的值

```python
curBattryStateValue = curBattryStateEnum.value
```
输出：`2`

类似，直接从定义中获取值：

```python
gScreenQuality = ScreenshotQuality.Low.value # 2
```

## 枚举高级用法

### 给枚举中添加函数

```python
class TipType(enum.Enum):
    NoTip = "NoTip"
    TenPercent = "TenPercent"
    FifthPercent = "FifthPercent"
    TwentyPercent = "TwentyPercent"

    # @property
    def getTipPercent(self):
        tipPercent = 0.0
        if self == TipType.NoTip:
            tipPercent = 0.0
        elif self == TipType.TenPercent:
            tipPercent = 0.10
        elif self == TipType.FifthPercent:
            tipPercent = 0.15
        elif self == TipType.TwentyPercent:
            tipPercent = 0.20
        gLog.debug("self=%s -> tipPercent=%s", self, tipPercent)
        return tipPercent
```

调用：

```python
tipPercent = initiatorTipType.getTipPercent()
# tipPercent=0.1
```

## 注意事项

### 字符串枚举定义最后不要加逗号

enum定义期间不要加（多余的）逗号：

```python
class ScreenshotQuality(enum.Enum):
    Original = 0,
    Medium = 1,
    Low = 2,
```

否则`value`就是`tuple`**元祖**了：

```python
gScreenQuality = ScreenshotQuality.Low.value # 实际上是 (2,)
print("gScreenQuality=%s" % gScreenQuality) # gScreenQuality=2
print("type(gScreenQuality)=%s" % type(gScreenQuality)) # type(gScreenQuality)=<class 'tuple'>
```

![enum_comma_str_to_tuple](../assets/img/enum_comma_str_to_tuple.png)
