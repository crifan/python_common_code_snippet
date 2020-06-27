# 系统

此处整理用Python处理系统相关的通用的代码。

## 系统类型

```python
import sys

def osIsWinows():
    return sys.platform == "win32"

def osIsCygwin():
    return sys.platform == "cygwin"

def osIsMacOS():
    return sys.platform == "darwin"

def osIsLinux():
    return sys.platform == "linux"

def osIsAix():
    return sys.platform == "aix"
```

## 命令行

### 获取命令行执行命令返回结果

代码：

```python
def get_cmd_lines(cmd, text=False):
    # 执行cmd命令，将结果保存为列表
    resultStr = ""
    resultStrList = []
    try:
        consoleOutputByte = subprocess.check_output(cmd, shell=True) # b'C02Y3N10JHC8\n'
        try:
            resultStr = consoleOutputByte.decode("utf-8")
        except UnicodeDecodeError:
            # TODO: use chardet auto detect encoding
            # consoleOutputStr = consoleOutputByte.decode("gbk")
            resultStr = consoleOutputByte.decode("gb18030")

        if not text:
            resultStrList = resultStr.splitlines()
    except Exception as err:
        print("err=%s when run cmd=%s" % (err, cmd))

    if text:
        return resultStr
    else:
        return resultStrList
```

## 硬件信息

## 获取当前电脑（Win或Mac）的序列号

代码：

```python
def getSerialNumber(self):
    """get current computer serial number"""
    # cmd = "wmic bios get serialnumber"
    cmd = ""
    if CommonUtils.osIsWinows():
        # Windows
        cmd = "wmic bios get serialnumber"
    elif CommonUtils.osIsMacOS():
        # macOS
        cmd = "system_profiler SPHardwareDataType | awk '/Serial/ {print $4}'"
    # TODO: add support other OS
    # AIX: aix
    # Linux: linux
    # Windows/Cygwin: cygwin

    serialNumber = ""
    lines = CommonUtils.get_cmd_lines(cmd)
    if CommonUtils.osIsWinows():
        # Windows
        serialNumber = lines[1]
    elif CommonUtils.osIsMacOS():
        # macOS
        serialNumber = lines[0] # C02Y3N10JHC8

    return serialNumber
```
