# 字符和字符串

详见：

https://github.com/crifan/crifanLibPython/blob/master/crifanLib/crifanFile.py

## Python3 str转bytes

Python 3中，把字符串转换成字节码，可以有两种写法：

* 方法1：用bytes去转换
    ```python
    convertedBytes = bytes(originStr)
    ```
* 方法2：用字符串str的编码encode
    ```python
    encodedBytes = originStr.encode()
    ```

其中：

都有额外的encoding参数：

* 由于默认都是UTF-8
    * 所以可加可不加
* 也可以根据需要，去加其他你要的编码
    * 如果要加，就是：
        ```python
        convertedBytes = bytes(originStr, "UTF-8")
        encodedBytes = originStr.encode("UTF-8")
        ```

## 字符串格式化为人类易读格式

```python

def formatSize(sizeInBytes, decimalNum=1, isUnitWithI=False, sizeUnitSeperator=""):
  """
    format size to human readable string

    example:
      3746 -> 3.7KB
      87533 -> 85.5KiB
      98654 -> 96.3 KB
      352 -> 352.0B
      76383285 -> 72.84MB
      763832854988542 -> 694.70TB
      763832854988542665 -> 678.4199PB

    refer:
      https://stackoverflow.com/questions/1094841/reusable-library-to-get-human-readable-version-of-file-size
  """
  # https://en.wikipedia.org/wiki/Binary_prefix#Specific_units_of_IEC_60027-2_A.2_and_ISO.2FIEC_80000
  # K=kilo, M=mega, G=giga, T=tera, P=peta, E=exa, Z=zetta, Y=yotta
  sizeUnitList = ['','K','M','G','T','P','E','Z']
  largestUnit = 'Y'

  if isUnitWithI:
    sizeUnitListWithI = []
    for curIdx, eachUnit in enumerate(sizeUnitList):
      unitWithI = eachUnit
      if curIdx >= 1:
        unitWithI += 'i'
      sizeUnitListWithI.append(unitWithI)

    # sizeUnitListWithI = ['','Ki','Mi','Gi','Ti','Pi','Ei','Zi']
    sizeUnitList = sizeUnitListWithI

    largestUnit += 'i'

  suffix = "B"
  decimalFormat = "." + str(decimalNum) + "f" # ".1f"
  finalFormat = "%" + decimalFormat + sizeUnitSeperator + "%s%s" # "%.1f%s%s"
  sizeNum = sizeInBytes
  for sizeUnit in sizeUnitList:
      if abs(sizeNum) < 1024.0:
        return finalFormat % (sizeNum, sizeUnit, suffix)
      sizeNum /= 1024.0
  return finalFormat % (sizeNum, largestUnit, suffix)
```

调用：

```python

def testKb():
  kbSize = 3746
  kbStr = formatSize(kbSize)
  print("%s -&gt; %s" % (kbSize, kbStr))

def testI():
  iSize = 87533
  iStr = formatSize(iSize, isUnitWithI=True)
  print("%s -&gt; %s" % (iSize, iStr))

def testSeparator():
  seperatorSize = 98654
  seperatorStr = formatSize(seperatorSize, sizeUnitSeperator=" ")
  print("%s -&gt; %s" % (seperatorSize, seperatorStr))

def testBytes():
  bytesSize = 352
  bytesStr = formatSize(bytesSize)
  print("%s -&gt; %s" % (bytesSize, bytesStr))

def testMb():
  mbSize = 76383285
  mbStr = formatSize(mbSize, decimalNum=2)
  print("%s -&gt; %s" % (mbSize, mbStr))

def testTb():
  tbSize = 763832854988542
  tbStr = formatSize(tbSize, decimalNum=2)
  print("%s -&gt; %s" % (tbSize, tbStr))

def testPb():
  pbSize = 763832854988542665
  pbStr = formatSize(pbSize, decimalNum=4)
  print("%s -&gt; %s" % (pbSize, pbStr))

def demoFormatSize():
  testKb()
  testI()
  testSeparator()
  testBytes()
  testMb()
  testTb()
  testPb()
```

