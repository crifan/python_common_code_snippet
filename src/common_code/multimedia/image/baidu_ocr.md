# 百度OCR

详见：

https://github.com/crifan/crifanLibPython/blob/master/crifanLib/crifanBaiduOcr.py

---


在做安卓和iOS的移动端自动化测试期间，会涉及到**从图像中提取文字**，用的是百度OCR。

其中有些通用的功能，整理出函数，贴出供参考。

## 百度OCR初始化

```python
import os
import re
import base64
import requests
import time
import logging
from collections import OrderedDict
from PIL import Image, ImageDraw

class BaiduOCR():
    # OCR_URL = "https://aip.baidubce.com/rest/2.0/ocr/v1/general_basic" # 通用文字识别
    # OCR_URL = "https://aip.baidubce.com/rest/2.0/ocr/v1/general"        # 通用文字识别（含位置信息版）
    # OCR_URL = "https://aip.baidubce.com/rest/2.0/ocr/v1/accurate_basic" # 通用文字识别（高精度版）
    OCR_URL = "https://aip.baidubce.com/rest/2.0/ocr/v1/accurate"       # 通用文字识别（高精度含位置版）


    TOKEN_URL = 'https://aip.baidubce.com/oauth/2.0/token'


    RESP_ERR_CODE_QPS_LIMIT_REACHED = 18
    RESP_ERR_TEXT_QPS_LIMIT_REACHED = "Open api qps request limit reached"


    RESP_ERR_CODE_DAILY_LIMIT_REACHED = 17
    RESP_ERR_TEXT_DAILY_LIMIT_REACHED = "Open api daily request limit reached"


    API_KEY = 'SOxxxxxxxxxxnu'
    SECRET_KEY = 'wlxxxxxxxxxxxxxxxxxxxpL'


    def initOcr(self):
        self.curToken = self.baiduFetchToken()


    def baiduFetchToken(self):
        """Fetch Baidu token for OCR"""
        params = {
            'grant_type': 'client_credentials',
            'client_id': self.API_KEY,
            'client_secret': self.SECRET_KEY
        }


        resp = requests.get(self.TOKEN_URL, params=params)
        respJson = resp.json()


        respToken = ""


        if ('access_token' in respJson.keys() and 'scope' in respJson.keys()):
            if not 'brain_all_scope' in respJson['scope'].split(' '):
                logging.error('please ensure has check the  ability')
            else:
                respToken = respJson['access_token']
        else:
            logging.error('please overwrite the correct API_KEY and SECRET_KEY')


        # '24.8691f3c6dedd0d0d0b30a9dfec604d52.2592000.1578465979.282335-17921535'
        return respToken
```

## 百度OCR图片转文字

```python
def baiduImageToWords(self, imageFullPath):
    """Detect text from image using Baidu OCR api"""

    # # Note: if using un-paid = free baidu api, need following wait sometime to reduce: qps request limit
    # time.sleep(0.15)

    respWordsResutJson = ""

    # 读取图片二进制数据
    imgBinData = readBinDataFromFile(imageFullPath)
    encodedImgData = base64.b64encode(imgBinData)

    paramDict = {
        "access_token": self.curToken
    }

    headerDict = {
        "Content-Type": "application/x-www-form-urlencoded"
    }

    # 参数含义：http://ai.baidu.com/ai-doc/OCR/vk3h7y58v
    dataDict = {
        "image": encodedImgData,
        "recognize_granularity": "small",
        # "vertexes_location": "true",
    }
    resp = requests.post(self.OCR_URL, params=paramDict, headers=headerDict, data=dataDict)
    respJson = resp.json()

    logging.debug("baidu OCR: imgage=%s -> respJson=%s", imageFullPath, respJson)

    if "error_code" in respJson:
        logging.warning("respJson=%s" % respJson)
        errorCode = respJson["error_code"]
        # {'error_code': 17, 'error_msg': 'Open api daily request limit reached'}
        # {'error_code': 18, 'error_msg': 'Open api qps request limit reached'}
        # the limit count can found from
        # 文字识别 - 免费额度 | 百度AI开放平台
        # https://ai.baidu.com/ai-doc/OCR/fk3h7xu7h
        # for "通用文字识别（高精度含位置版）" is "50次/天"
        if errorCode == self.RESP_ERR_CODE_QPS_LIMIT_REACHED:
            # wait sometime and try again
            time.sleep(1.0)
            resp = requests.post(self.OCR_URL, params=paramDict, headers=headerDict, data=dataDict)
            respJson = resp.json()
            logging.debug("baidu OCR: for errorCode=%s, do again, imgage=%s -> respJson=%s", errorCode, imageFullPath, respJson)
        elif errorCode == self.RESP_ERR_CODE_DAILY_LIMIT_REACHED:
            logging.error("Fail to continue using baidu OCR api today !!!")
            respJson = None

    """
    {
    "log_id": 6937531796498618000,
    "words_result_num": 32,
    "words_result": [
        {
        "chars": [
            ...
    """
    if "words_result" in respJson:
        respWordsResutJson = respJson

    return respWordsResutJson
```

调用：

```python
wordsResultJson = self.baiduImageToWords(imgPath)

respJson = self.baiduImageToWords(screenImgPath)
```

### 返回结果举例

#### 安卓游戏 暗黑觉醒 首充豪礼

图片：

![anheijuexing_first_charge](../../../assets/img/anheijuexing_first_charge.jpg)

返回解析后出`json`格式的文字信息：

```json
{
  "log_id": 9009770747370640007,
  "words_result_num": 12,
  "words_result": [
    {
      "chars": [
        {
          "char": "首",
          "location": { "width": 94, "top": 105, "left": 989, "height": 158 }
        },
        {
          "char": "充",
          "location": { "width": 94, "top": 105, "left": 1086, "height": 158 }
        },
        {
          "char": "豪",
          "location": { "width": 95, "top": 105, "left": 1183, "height": 158 }
        },
        {
          "char": "礼",
          "location": { "width": 77, "top": 105, "left": 1281, "height": 158 }
        }
      ],
      "location": { "width": 370, "top": 105, "left": 989, "height": 158 },
      "words": "首充豪礼"
    },
    {
      "chars": [
        {
          "char": "×",
          "location": { "width": 30, "top": 161, "left": 1887, "height": 61 }
        }
      ],
      "location": { "width": 60, "top": 161, "left": 1887, "height": 61 },
      "words": "×"
    },
    {
      "chars": [
        {
          "char": "充",
          "location": { "width": 43, "top": 273, "left": 758, "height": 73 }
        },
        {
          "char": "值",
          "location": { "width": 43, "top": 273, "left": 803, "height": 73 }
        },
        {
          "char": "元",
          "location": { "width": 67, "top": 273, "left": 912, "height": 73 }
        },
        {
          "char": "可",
          "location": { "width": 44, "top": 273, "left": 979, "height": 73 }
        },
        {
          "char": "领",
          "location": { "width": 43, "top": 273, "left": 1023, "height": 73 }
        },
        {
          "char": "总",
          "location": { "width": 44, "top": 273, "left": 1067, "height": 73 }
        },
        {
          "char": "价",
          "location": { "width": 23, "top": 273, "left": 1111, "height": 73 }
        },
        {
          "char": "值",
          "location": { "width": 89, "top": 273, "left": 1134, "height": 73 }
        },
        {
          "char": "8",
          "location": { "width": 36, "top": 273, "left": 1259, "height": 73 }
        },
        {
          "char": "8",
          "location": { "width": 36, "top": 273, "left": 1326, "height": 73 }
        },
        {
          "char": "8",
          "location": { "width": 35, "top": 273, "left": 1371, "height": 73 }
        },
        {
          "char": "钻",
          "location": { "width": 43, "top": 273, "left": 1444, "height": 73 }
        },
        {
          "char": "豪",
          "location": { "width": 44, "top": 273, "left": 1510, "height": 73 }
        },
        {
          "char": "华",
          "location": { "width": 43, "top": 273, "left": 1555, "height": 73 }
        },
        {
          "char": "大",
          "location": { "width": 43, "top": 273, "left": 1599, "height": 73 }
        },
        {
          "char": "礼",
          "location": { "width": 27, "top": 273, "left": 1643, "height": 73 }
        }
      ],
      "location": { "width": 911, "top": 273, "left": 758, "height": 73 },
      "words": "充值元可领总价值888钻豪华大礼"
    },
    {
      "chars": [
        {
          "char": "送",
          "location": { "width": 65, "top": 369, "left": 832, "height": 107 }
        }
      ],
      "location": { "width": 107, "top": 369, "left": 832, "height": 107 },
      "words": "送"
    },
    {
      "chars": [
        {
          "char": "绝",
          "location": { "width": 38, "top": 390, "left": 974, "height": 65 }
        },
        {
          "char": "版",
          "location": { "width": 38, "top": 390, "left": 1032, "height": 65 }
        },
        {
          "char": "萌",
          "location": { "width": 38, "top": 390, "left": 1092, "height": 65 }
        },
        {
          "char": "宠",
          "location": { "width": 38, "top": 390, "left": 1150, "height": 65 }
        },
        {
          "char": "、",
          "location": { "width": 31, "top": 390, "left": 1184, "height": 65 }
        },
        {
          "char": "专",
          "location": { "width": 39, "top": 390, "left": 1230, "height": 65 }
        },
        {
          "char": "属",
          "location": { "width": 38, "top": 390, "left": 1289, "height": 65 }
        },
        {
          "char": "神",
          "location": { "width": 38, "top": 390, "left": 1368, "height": 65 }
        },
        {
          "char": "兵",
          "location": { "width": 39, "top": 390, "left": 1408, "height": 65 }
        }
      ],
      "location": { "width": 524, "top": 390, "left": 934, "height": 65 },
      "words": "绝版萌宠、专属神兵"
    },
    {
      "chars": [
        {
          "char": "绝",
          "location": { "width": 20, "top": 515, "left": 378, "height": 33 }
        }
      ],
      "location": { "width": 33, "top": 515, "left": 378, "height": 33 },
      "words": "绝"
    },
    {
      "chars": [
        {
          "char": "珍",
          "location": { "width": 33, "top": 516, "left": 1992, "height": 42 }
        }
      ],
      "location": { "width": 33, "top": 516, "left": 1992, "height": 42 },
      "words": "珍"
    },
    {
      "chars": [
        {
          "char": "版",
          "location": { "width": 20, "top": 545, "left": 379, "height": 34 }
        }
      ],
      "location": { "width": 31, "top": 545, "left": 379, "height": 34 },
      "words": "版"
    },
    {
      "chars": [
        {
          "char": "额",
          "location": { "width": 26, "top": 776, "left": 1225, "height": 44 }
        },
        {
          "char": "外",
          "location": { "width": 26, "top": 776, "left": 1264, "height": 44 }
        },
        {
          "char": "礼",
          "location": { "width": 27, "top": 776, "left": 1291, "height": 44 }
        },
        {
          "char": "包",
          "location": { "width": 27, "top": 776, "left": 1317, "height": 44 }
        }
      ],
      "location": { "width": 125, "top": 776, "left": 1225, "height": 44 },
      "words": "额外礼包"
    },
    {
      "chars": [
        {
          "char": "首",
          "location": { "width": 38, "top": 830, "left": 935, "height": 64 }
        },
        {
          "char": "充",
          "location": { "width": 38, "top": 830, "left": 994, "height": 64 }
        },
        {
          "char": "元",
          "location": { "width": 38, "top": 830, "left": 1092, "height": 64 }
        },
        {
          "char": "充",
          "location": { "width": 38, "top": 830, "left": 1286, "height": 64 }
        },
        {
          "char": "9",
          "location": { "width": 31, "top": 830, "left": 1339, "height": 64 }
        },
        {
          "char": "8",
          "location": { "width": 31, "top": 830, "left": 1377, "height": 64 }
        },
        {
          "char": "元",
          "location": { "width": 38, "top": 830, "left": 1444, "height": 64 }
        }
      ],
      "location": { "width": 549, "top": 830, "left": 935, "height": 64 },
      "words": "首充元充98元"
    },
    {
      "chars": [
        {
          "char": "战",
          "location": { "width": 42, "top": 970, "left": 373, "height": 69 }
        },
        {
          "char": "斗",
          "location": { "width": 42, "top": 970, "left": 437, "height": 69 }
        },
        {
          "char": "1",
          "location": { "width": 35, "top": 970, "left": 515, "height": 69 }
        },
        {
          "char": "5",
          "location": { "width": 35, "top": 970, "left": 537, "height": 69 }
        },
        {
          "char": "0",
          "location": { "width": 34, "top": 970, "left": 580, "height": 69 }
        },
        {
          "char": "0",
          "location": { "width": 35, "top": 970, "left": 622, "height": 69 }
        },
        {
          "char": "0",
          "location": { "width": 34, "top": 970, "left": 666, "height": 69 }
        }
      ],
      "location": { "width": 327, "top": 970, "left": 373, "height": 69 },
      "words": "战斗15000"
    },
    {
      "chars": [
        {
          "char": "战",
          "location": { "width": 43, "top": 969, "left": 1648, "height": 73 }
        },
        {
          "char": "斗",
          "location": { "width": 43, "top": 969, "left": 1713, "height": 73 }
        },
        {
          "char": "1",
          "location": { "width": 36, "top": 969, "left": 1793, "height": 73 }
        },
        {
          "char": "6",
          "location": { "width": 36, "top": 969, "left": 1816, "height": 73 }
        },
        {
          "char": "0",
          "location": { "width": 35, "top": 969, "left": 1861, "height": 73 }
        },
        {
          "char": "0",
          "location": { "width": 36, "top": 969, "left": 1904, "height": 73 }
        },
        {
          "char": "0",
          "location": { "width": 29, "top": 969, "left": 1949, "height": 73 }
        }
      ],
      "location": { "width": 330, "top": 969, "left": 1648, "height": 73 },
      "words": "战斗16000"
    }
  ]
}
```

其中：

* 首充豪礼
    * 都能完整检测出来：已经是效果很不错了
    * 当然偶尔也会有失误，比如 偶尔
        * 只解析出部分内容：首充豪
        * 或个别字错了：首充豪机
            * 本身图片上 礼 也的确很像 机
                * 作为OCR犯此错误，完全可以理解

#### 安卓游戏 暗黑觉醒 公告弹框

图片：

![anheijuexing_announcement_popop](../../../assets/img/anheijuexing_announcement_popop.jpg)

返回结果json：

```json
{
  "log_id": 2793391773289550472,
  "words_result_num": 23,
  "words_result": [
    {
      "chars": [
        {
          "char": "公",
          "location": { "width": 28, "top": 125, "left": 634, "height": 48 }
        },
        {
          "char": "告",
          "location": { "width": 29, "top": 125, "left": 691, "height": 48 }
        }
      ],
      "location": { "width": 92, "top": 125, "left": 634, "height": 48 },
      "words": "公告"
    },
    {
      "chars": [
        {
          "char": "最",
          "location": { "width": 21, "top": 240, "left": 535, "height": 36 }
        }
      ],
      "location": { "width": 33, "top": 240, "left": 535, "height": 36 },
      "words": "最"
    },
    {
      "chars": [
        {
          "char": "亲",
          "location": { "width": 26, "top": 233, "left": 922, "height": 42 }
        },
        {
          "char": "爱",
          "location": { "width": 26, "top": 233, "left": 959, "height": 42 }
        },
        {
          "char": "的",
          "location": { "width": 26, "top": 233, "left": 986, "height": 42 }
        },
        {
          "char": "觉",
          "location": { "width": 26, "top": 233, "left": 1024, "height": 42 }
        },
        {
          "char": "醒",
          "location": { "width": 26, "top": 233, "left": 1063, "height": 42 }
        },
        {
          "char": "勇",
          "location": { "width": 26, "top": 233, "left": 1088, "height": 42 }
        },
        {
          "char": "士",
          "location": { "width": 25, "top": 233, "left": 1127, "height": 42 }
        },
        {
          "char": ":",
          "location": { "width": 21, "top": 233, "left": 1148, "height": 42 }
        }
      ],
      "location": { "width": 253, "top": 233, "left": 922, "height": 42 },
      "words": "亲爱的觉醒勇士:"
    },
    {
      "chars": [
        {
          "char": "新",
          "location": { "width": 26, "top": 266, "left": 535, "height": 44 }
        },
        {
          "char": "新",
          "location": { "width": 27, "top": 266, "left": 588, "height": 44 }
        },
        {
          "char": "服",
          "location": { "width": 26, "top": 266, "left": 628, "height": 44 }
        },
        {
          "char": "公",
          "location": { "width": 27, "top": 266, "left": 668, "height": 44 }
        },
        {
          "char": "告",
          "location": { "width": 26, "top": 266, "left": 709, "height": 44 }
        }
      ],
      "location": { "width": 201, "top": 266, "left": 535, "height": 44 },
      "words": "新新服公告"
    },
    {
      "chars": [
        {
          "char": "承",
          "location": { "width": 26, "top": 282, "left": 984, "height": 43 }
        },
        {
          "char": "载",
          "location": { "width": 26, "top": 282, "left": 1023, "height": 43 }
        },
        {
          "char": "六",
          "location": { "width": 26, "top": 282, "left": 1063, "height": 43 }
        },
        {
          "char": "大",
          "location": { "width": 26, "top": 282, "left": 1090, "height": 43 }
        },
        {
          "char": "种",
          "location": { "width": 26, "top": 282, "left": 1116, "height": 43 }
        },
        {
          "char": "族",
          "location": { "width": 26, "top": 282, "left": 1155, "height": 43 }
        },
        {
          "char": "的",
          "location": { "width": 26, "top": 282, "left": 1181, "height": 43 }
        },
        {
          "char": "重",
          "location": { "width": 26, "top": 282, "left": 1220, "height": 43 }
        },
        {
          "char": "生",
          "location": { "width": 26, "top": 282, "left": 1260, "height": 43 }
        },
        {
          "char": "之",
          "location": { "width": 26, "top": 282, "left": 1286, "height": 43 }
        },
        {
          "char": "使",
          "location": { "width": 27, "top": 282, "left": 1325, "height": 43 }
        },
        {
          "char": "命",
          "location": { "width": 26, "top": 282, "left": 1351, "height": 43 }
        },
        {
          "char": ",",
          "location": { "width": 21, "top": 282, "left": 1385, "height": 43 }
        },
        {
          "char": "超",
          "location": { "width": 26, "top": 282, "left": 1416, "height": 43 }
        },
        {
          "char": "现",
          "location": { "width": 26, "top": 282, "left": 1456, "height": 43 }
        },
        {
          "char": "实",
          "location": { "width": 26, "top": 282, "left": 1483, "height": 43 }
        },
        {
          "char": "3",
          "location": { "width": 22, "top": 282, "left": 1517, "height": 43 }
        },
        {
          "char": "D",
          "location": { "width": 21, "top": 282, "left": 1531, "height": 43 }
        },
        {
          "char": "魔",
          "location": { "width": 26, "top": 282, "left": 1560, "height": 43 }
        },
        {
          "char": "M",
          "location": { "width": 61, "top": 284, "left": 1583, "height": 40 }
        },
        {
          "char": "M",
          "location": { "width": 35, "top": 284, "left": 1639, "height": 40 }
        },
        {
          "char": "O",
          "location": { "width": 31, "top": 284, "left": 1669, "height": 40 }
        },
        {
          "char": "A",
          "location": { "width": 25, "top": 284, "left": 1696, "height": 40 }
        },
        {
          "char": "R",
          "location": { "width": 25, "top": 284, "left": 1716, "height": 40 }
        },
        {
          "char": "P",
          "location": { "width": 25, "top": 284, "left": 1735, "height": 40 }
        },
        {
          "char": "G",
          "location": { "width": 25, "top": 284, "left": 1755, "height": 40 }
        },
        {
          "char": "幻",
          "location": { "width": 26, "top": 282, "left": 1588, "height": 43 }
        },
        {
          "char": "手",
          "location": { "width": 26, "top": 282, "left": 1784, "height": 43 }
        },
        {
          "char": "游",
          "location": { "width": 26, "top": 282, "left": 1823, "height": 43 }
        }
      ],
      "location": { "width": 867, "top": 282, "left": 984, "height": 43 },
      "words": "承载六大种族的重生之使命,超现实3D魔 MMOARPG幻手游"
    },
    {
      "chars": [
        {
          "char": "不",
          "location": { "width": 37, "top": 337, "left": 923, "height": 40 }
        },
        {
          "char": "负",
          "location": { "width": 23, "top": 337, "left": 959, "height": 40 }
        },
        {
          "char": "觉",
          "location": { "width": 25, "top": 337, "left": 996, "height": 40 }
        },
        {
          "char": "醒",
          "location": { "width": 23, "top": 337, "left": 1021, "height": 40 }
        },
        {
          "char": "勇",
          "location": { "width": 23, "top": 337, "left": 1058, "height": 40 }
        },
        {
          "char": "士",
          "location": { "width": 25, "top": 337, "left": 1094, "height": 40 }
        },
        {
          "char": "们",
          "location": { "width": 25, "top": 337, "left": 1119, "height": 40 }
        },
        {
          "char": "的",
          "location": { "width": 25, "top": 337, "left": 1156, "height": 40 }
        },
        {
          "char": "使",
          "location": { "width": 25, "top": 337, "left": 1192, "height": 40 }
        },
        {
          "char": "命",
          "location": { "width": 25, "top": 337, "left": 1230, "height": 40 }
        },
        {
          "char": "之",
          "location": { "width": 25, "top": 337, "left": 1254, "height": 40 }
        },
        {
          "char": "约",
          "location": { "width": 25, "top": 337, "left": 1292, "height": 40 }
        },
        {
          "char": ",",
          "location": { "width": 20, "top": 337, "left": 1324, "height": 40 }
        },
        {
          "char": "震",
          "location": { "width": 25, "top": 337, "left": 1353, "height": 40 }
        },
        {
          "char": "撼",
          "location": { "width": 25, "top": 337, "left": 1390, "height": 40 }
        },
        {
          "char": "来",
          "location": { "width": 23, "top": 337, "left": 1428, "height": 40 }
        },
        {
          "char": "袭",
          "location": { "width": 25, "top": 337, "left": 1452, "height": 40 }
        },
        {
          "char": "。",
          "location": { "width": 18, "top": 337, "left": 1485, "height": 40 }
        }
      ],
      "location": { "width": 580, "top": 337, "left": 923, "height": 40 },
      "words": "不负觉醒勇士们的使命之约,震撼来袭。"
    },
    {
      "chars": [
        {
          "char": "最",
          "location": { "width": 20, "top": 364, "left": 535, "height": 33 }
        }
      ],
      "location": { "width": 33, "top": 364, "left": 535, "height": 33 },
      "words": "最"
    },
    {
      "chars": [
        {
          "char": "新",
          "location": { "width": 26, "top": 389, "left": 535, "height": 43 }
        },
        {
          "char": "违",
          "location": { "width": 26, "top": 389, "left": 588, "height": 43 }
        },
        {
          "char": "规",
          "location": { "width": 25, "top": 389, "left": 627, "height": 43 }
        },
        {
          "char": "发",
          "location": { "width": 26, "top": 389, "left": 666, "height": 43 }
        },
        {
          "char": "言",
          "location": { "width": 26, "top": 389, "left": 704, "height": 43 }
        },
        {
          "char": "处",
          "location": { "width": 26, "top": 389, "left": 743, "height": 43 }
        },
        {
          "char": "理",
          "location": { "width": 26, "top": 389, "left": 770, "height": 43 }
        },
        {
          "char": "机",
          "location": { "width": 26, "top": 389, "left": 808, "height": 43 }
        },
        {
          "char": "制",
          "location": { "width": 26, "top": 389, "left": 848, "height": 43 }
        }
      ],
      "location": { "width": 343, "top": 389, "left": 535, "height": 43 },
      "words": "新违规发言处理机制"
    },
    {
      "chars": [
        {
          "char": "【",
          "location": { "width": 21, "top": 387, "left": 997, "height": 43 }
        },
        {
          "char": "普",
          "location": { "width": 25, "top": 387, "left": 1023, "height": 43 }
        },
        {
          "char": "纳",
          "location": { "width": 26, "top": 387, "left": 1061, "height": 43 }
        },
        {
          "char": "山",
          "location": { "width": 25, "top": 387, "left": 1087, "height": 43 }
        },
        {
          "char": "谷",
          "location": { "width": 26, "top": 387, "left": 1126, "height": 43 }
        },
        {
          "char": "1",
          "location": { "width": 21, "top": 387, "left": 1148, "height": 43 }
        },
        {
          "char": "2",
          "location": { "width": 21, "top": 387, "left": 1160, "height": 43 }
        },
        {
          "char": "0",
          "location": { "width": 21, "top": 387, "left": 1187, "height": 43 }
        },
        {
          "char": "服",
          "location": { "width": 26, "top": 387, "left": 1204, "height": 43 }
        },
        {
          "char": "】",
          "location": { "width": 21, "top": 387, "left": 1238, "height": 43 }
        },
        {
          "char": "将",
          "location": { "width": 25, "top": 387, "left": 1269, "height": 43 }
        },
        {
          "char": "于",
          "location": { "width": 26, "top": 387, "left": 1308, "height": 43 }
        },
        {
          "char": "0",
          "location": { "width": 20, "top": 387, "left": 1329, "height": 43 }
        },
        {
          "char": "7",
          "location": { "width": 21, "top": 387, "left": 1355, "height": 43 }
        },
        {
          "char": "月",
          "location": { "width": 25, "top": 387, "left": 1385, "height": 43 }
        },
        {
          "char": "0",
          "location": { "width": 21, "top": 387, "left": 1406, "height": 43 }
        },
        {
          "char": "8",
          "location": { "width": 21, "top": 387, "left": 1420, "height": 43 }
        },
        {
          "char": "日",
          "location": { "width": 25, "top": 387, "left": 1451, "height": 43 }
        },
        {
          "char": "0",
          "location": { "width": 21, "top": 387, "left": 1471, "height": 43 }
        },
        {
          "char": "0",
          "location": { "width": 21, "top": 387, "left": 1497, "height": 43 }
        },
        {
          "char": ":",
          "location": { "width": 21, "top": 387, "left": 1510, "height": 43 }
        },
        {
          "char": "1",
          "location": { "width": 21, "top": 387, "left": 1523, "height": 43 }
        },
        {
          "char": "5",
          "location": { "width": 21, "top": 387, "left": 1535, "height": 43 }
        },
        {
          "char": "震",
          "location": { "width": 25, "top": 387, "left": 1567, "height": 43 }
        },
        {
          "char": "撼",
          "location": { "width": 26, "top": 387, "left": 1592, "height": 43 }
        },
        {
          "char": "开",
          "location": { "width": 25, "top": 387, "left": 1631, "height": 43 }
        },
        {
          "char": "启",
          "location": { "width": 26, "top": 387, "left": 1656, "height": 43 }
        },
        {
          "char": "。",
          "location": { "width": 21, "top": 387, "left": 1678, "height": 43 }
        }
      ],
      "location": { "width": 711, "top": 387, "left": 997, "height": 43 },
      "words": "【普纳山谷120服】将于07月08日00:15震撼开启。"
    },
    {
      "chars": [
        {
          "char": "各",
          "location": { "width": 23, "top": 438, "left": 987, "height": 40 }
        },
        {
          "char": "位",
          "location": { "width": 25, "top": 438, "left": 1022, "height": 40 }
        },
        {
          "char": "勇",
          "location": { "width": 25, "top": 438, "left": 1059, "height": 40 }
        },
        {
          "char": "士",
          "location": { "width": 25, "top": 438, "left": 1095, "height": 40 }
        },
        {
          "char": "请",
          "location": { "width": 25, "top": 438, "left": 1120, "height": 40 }
        },
        {
          "char": "拿",
          "location": { "width": 23, "top": 438, "left": 1157, "height": 40 }
        },
        {
          "char": "起",
          "location": { "width": 25, "top": 438, "left": 1181, "height": 40 }
        },
        {
          "char": "手",
          "location": { "width": 23, "top": 438, "left": 1230, "height": 40 }
        },
        {
          "char": "中",
          "location": { "width": 23, "top": 438, "left": 1255, "height": 40 }
        },
        {
          "char": "武",
          "location": { "width": 25, "top": 438, "left": 1291, "height": 40 }
        },
        {
          "char": "器",
          "location": { "width": 23, "top": 438, "left": 1316, "height": 40 }
        },
        {
          "char": ",",
          "location": { "width": 20, "top": 438, "left": 1348, "height": 40 }
        },
        {
          "char": "与",
          "location": { "width": 25, "top": 438, "left": 1389, "height": 40 }
        },
        {
          "char": "我",
          "location": { "width": 23, "top": 438, "left": 1414, "height": 40 }
        },
        {
          "char": "们",
          "location": { "width": 25, "top": 438, "left": 1449, "height": 40 }
        },
        {
          "char": "一",
          "location": { "width": 23, "top": 438, "left": 1487, "height": 40 }
        },
        {
          "char": "同",
          "location": { "width": 23, "top": 438, "left": 1524, "height": 40 }
        },
        {
          "char": "踏",
          "location": { "width": 25, "top": 438, "left": 1548, "height": 40 }
        },
        {
          "char": "上",
          "location": { "width": 23, "top": 438, "left": 1584, "height": 40 }
        },
        {
          "char": "王",
          "location": { "width": 25, "top": 438, "left": 1621, "height": 40 }
        },
        {
          "char": "者",
          "location": { "width": 25, "top": 438, "left": 1645, "height": 40 }
        },
        {
          "char": "觉",
          "location": { "width": 23, "top": 438, "left": 1683, "height": 40 }
        },
        {
          "char": "醒",
          "location": { "width": 25, "top": 438, "left": 1718, "height": 40 }
        },
        {
          "char": "之",
          "location": { "width": 23, "top": 438, "left": 1756, "height": 40 }
        },
        {
          "char": "路",
          "location": { "width": 25, "top": 438, "left": 1780, "height": 40 }
        },
        {
          "char": "!",
          "location": { "width": 20, "top": 438, "left": 1813, "height": 40 }
        }
      ],
      "location": { "width": 853, "top": 438, "left": 987, "height": 40 },
      "words": "各位勇士请拿起手中武器,与我们一同踏上王者觉醒之路!"
    },
    {
      "chars": [
        {
          "char": "限",
          "location": { "width": 21, "top": 486, "left": 535, "height": 35 }
        }
      ],
      "location": { "width": 35, "top": 486, "left": 535, "height": 35 },
      "words": "限"
    },
    {
      "chars": [
        {
          "char": "时",
          "location": { "width": 26, "top": 512, "left": 534, "height": 42 }
        },
        {
          "char": "新",
          "location": { "width": 26, "top": 512, "left": 597, "height": 42 }
        },
        {
          "char": "服",
          "location": { "width": 26, "top": 512, "left": 622, "height": 42 }
        },
        {
          "char": "活",
          "location": { "width": 25, "top": 512, "left": 661, "height": 42 }
        },
        {
          "char": "动",
          "location": { "width": 26, "top": 512, "left": 699, "height": 42 }
        }
      ],
      "location": { "width": 202, "top": 512, "left": 534, "height": 42 },
      "words": "时新服活动"
    },
    {
      "chars": [
        {
          "char": "【",
          "location": { "width": 22, "top": 538, "left": 927, "height": 43 }
        },
        {
          "char": "游",
          "location": { "width": 26, "top": 538, "left": 955, "height": 43 }
        },
        {
          "char": "戏",
          "location": { "width": 26, "top": 538, "left": 994, "height": 43 }
        },
        {
          "char": "特",
          "location": { "width": 26, "top": 538, "left": 1020, "height": 43 }
        },
        {
          "char": "色",
          "location": { "width": 27, "top": 538, "left": 1060, "height": 43 }
        },
        {
          "char": "】",
          "location": { "width": 19, "top": 538, "left": 1095, "height": 43 }
        }
      ],
      "location": { "width": 187, "top": 538, "left": 927, "height": 43 },
      "words": "【游戏特色】"
    },
    {
      "chars": [
        {
          "char": "火",
          "location": { "width": 20, "top": 612, "left": 537, "height": 33 }
        }
      ],
      "location": { "width": 33, "top": 612, "left": 537, "height": 33 },
      "words": "火"
    },
    {
      "chars": [
        {
          "char": "1",
          "location": { "width": 21, "top": 588, "left": 992, "height": 43 }
        },
        {
          "char": ".",
          "location": { "width": 21, "top": 588, "left": 1005, "height": 43 }
        },
        {
          "char": "超",
          "location": { "width": 26, "top": 588, "left": 1036, "height": 43 }
        },
        {
          "char": "宏",
          "location": { "width": 26, "top": 588, "left": 1076, "height": 43 }
        },
        {
          "char": "伟",
          "location": { "width": 26, "top": 588, "left": 1101, "height": 43 }
        },
        {
          "char": "世",
          "location": { "width": 26, "top": 588, "left": 1141, "height": 43 }
        },
        {
          "char": "界",
          "location": { "width": 26, "top": 588, "left": 1167, "height": 43 }
        },
        {
          "char": "观",
          "location": { "width": 26, "top": 588, "left": 1206, "height": 43 }
        },
        {
          "char": "、",
          "location": { "width": 21, "top": 588, "left": 1228, "height": 43 }
        },
        {
          "char": "六",
          "location": { "width": 26, "top": 588, "left": 1271, "height": 43 }
        },
        {
          "char": "大",
          "location": { "width": 26, "top": 588, "left": 1297, "height": 43 }
        },
        {
          "char": "种",
          "location": { "width": 26, "top": 588, "left": 1337, "height": 43 }
        },
        {
          "char": "族",
          "location": { "width": 26, "top": 588, "left": 1363, "height": 43 }
        },
        {
          "char": "秘",
          "location": { "width": 26, "top": 588, "left": 1403, "height": 43 }
        },
        {
          "char": "密",
          "location": { "width": 26, "top": 588, "left": 1429, "height": 43 }
        },
        {
          "char": "等",
          "location": { "width": 26, "top": 588, "left": 1468, "height": 43 }
        },
        {
          "char": "你",
          "location": { "width": 26, "top": 588, "left": 1493, "height": 43 }
        },
        {
          "char": "探",
          "location": { "width": 26, "top": 588, "left": 1533, "height": 43 }
        },
        {
          "char": "索",
          "location": { "width": 26, "top": 588, "left": 1559, "height": 43 }
        },
        {
          "char": ";",
          "location": { "width": 18, "top": 588, "left": 1595, "height": 43 }
        }
      ],
      "location": { "width": 642, "top": 588, "left": 971, "height": 43 },
      "words": "1.超宏伟世界观、六大种族秘密等你探索;"
    },
    {
      "chars": [
        {
          "char": "爆",
          "location": { "width": 27, "top": 635, "left": 534, "height": 44 }
        },
        {
          "char": "严",
          "location": { "width": 27, "top": 635, "left": 588, "height": 44 }
        },
        {
          "char": "禁",
          "location": { "width": 26, "top": 635, "left": 628, "height": 44 }
        },
        {
          "char": "代",
          "location": { "width": 27, "top": 635, "left": 668, "height": 44 }
        },
        {
          "char": "充",
          "location": { "width": 27, "top": 635, "left": 708, "height": 44 }
        },
        {
          "char": "公",
          "location": { "width": 26, "top": 635, "left": 735, "height": 44 }
        },
        {
          "char": "告",
          "location": { "width": 27, "top": 635, "left": 774, "height": 44 }
        }
      ],
      "location": { "width": 273, "top": 635, "left": 534, "height": 44 },
      "words": "爆严禁代充公告"
    },
    {
      "chars": [
        {
          "char": "2",
          "location": { "width": 22, "top": 642, "left": 992, "height": 43 }
        },
        {
          "char": ".",
          "location": { "width": 21, "top": 642, "left": 1015, "height": 43 }
        },
        {
          "char": "全",
          "location": { "width": 26, "top": 642, "left": 1032, "height": 43 }
        },
        {
          "char": "民",
          "location": { "width": 26, "top": 642, "left": 1072, "height": 43 }
        },
        {
          "char": "打",
          "location": { "width": 26, "top": 642, "left": 1099, "height": 43 }
        },
        {
          "char": "宝",
          "location": { "width": 27, "top": 642, "left": 1138, "height": 43 }
        },
        {
          "char": "得",
          "location": { "width": 26, "top": 642, "left": 1165, "height": 43 }
        },
        {
          "char": "神",
          "location": { "width": 27, "top": 642, "left": 1204, "height": 43 }
        },
        {
          "char": "装",
          "location": { "width": 26, "top": 642, "left": 1231, "height": 43 }
        },
        {
          "char": ",",
          "location": { "width": 21, "top": 642, "left": 1267, "height": 43 }
        },
        {
          "char": "自",
          "location": { "width": 26, "top": 642, "left": 1297, "height": 43 }
        },
        {
          "char": "由",
          "location": { "width": 26, "top": 642, "left": 1337, "height": 43 }
        },
        {
          "char": "交",
          "location": { "width": 26, "top": 642, "left": 1364, "height": 43 }
        },
        {
          "char": "易",
          "location": { "width": 27, "top": 642, "left": 1403, "height": 43 }
        },
        {
          "char": "换",
          "location": { "width": 27, "top": 642, "left": 1429, "height": 43 }
        },
        {
          "char": "货",
          "location": { "width": 27, "top": 642, "left": 1469, "height": 43 }
        },
        {
          "char": "币",
          "location": { "width": 26, "top": 642, "left": 1496, "height": 43 }
        },
        {
          "char": ",",
          "location": { "width": 21, "top": 642, "left": 1532, "height": 43 }
        },
        {
          "char": "极",
          "location": { "width": 26, "top": 642, "left": 1563, "height": 43 }
        },
        {
          "char": "速",
          "location": { "width": 26, "top": 642, "left": 1589, "height": 43 }
        },
        {
          "char": "致",
          "location": { "width": 26, "top": 642, "left": 1629, "height": 43 }
        },
        {
          "char": "富",
          "location": { "width": 27, "top": 642, "left": 1668, "height": 43 }
        },
        {
          "char": ";",
          "location": { "width": 22, "top": 642, "left": 1689, "height": 43 }
        }
      ],
      "location": { "width": 719, "top": 642, "left": 992, "height": 43 },
      "words": "2.全民打宝得神装,自由交易换货币,极速致富;"
    },
    {
      "chars": [
        {
          "char": "3",
          "location": { "width": 21, "top": 693, "left": 994, "height": 42 }
        },
        {
          "char": ".",
          "location": { "width": 21, "top": 693, "left": 1014, "height": 42 }
        },
        {
          "char": "神",
          "location": { "width": 25, "top": 693, "left": 1031, "height": 42 }
        },
        {
          "char": "装",
          "location": { "width": 25, "top": 693, "left": 1069, "height": 42 }
        },
        {
          "char": "极",
          "location": { "width": 26, "top": 693, "left": 1094, "height": 42 }
        },
        {
          "char": "速",
          "location": { "width": 25, "top": 693, "left": 1133, "height": 42 }
        },
        {
          "char": "进",
          "location": { "width": 26, "top": 693, "left": 1170, "height": 42 }
        },
        {
          "char": "阶",
          "location": { "width": 26, "top": 693, "left": 1195, "height": 42 }
        },
        {
          "char": ",",
          "location": { "width": 20, "top": 693, "left": 1229, "height": 42 }
        },
        {
          "char": "炼",
          "location": { "width": 25, "top": 693, "left": 1259, "height": 42 }
        },
        {
          "char": "化",
          "location": { "width": 26, "top": 693, "left": 1296, "height": 42 }
        },
        {
          "char": "玩",
          "location": { "width": 25, "top": 693, "left": 1334, "height": 42 }
        },
        {
          "char": "法",
          "location": { "width": 25, "top": 693, "left": 1359, "height": 42 }
        },
        {
          "char": "打",
          "location": { "width": 26, "top": 693, "left": 1397, "height": 42 }
        },
        {
          "char": "造",
          "location": { "width": 25, "top": 693, "left": 1436, "height": 42 }
        },
        {
          "char": "独",
          "location": { "width": 25, "top": 693, "left": 1461, "height": 42 }
        },
        {
          "char": "特",
          "location": { "width": 26, "top": 693, "left": 1497, "height": 42 }
        },
        {
          "char": "个",
          "location": { "width": 25, "top": 693, "left": 1536, "height": 42 }
        },
        {
          "char": "性",
          "location": { "width": 25, "top": 693, "left": 1561, "height": 42 }
        },
        {
          "char": "神",
          "location": { "width": 26, "top": 693, "left": 1599, "height": 42 }
        },
        {
          "char": "装",
          "location": { "width": 26, "top": 693, "left": 1624, "height": 42 }
        },
        {
          "char": ";",
          "location": { "width": 21, "top": 693, "left": 1657, "height": 42 }
        }
      ],
      "location": { "width": 686, "top": 693, "left": 994, "height": 42 },
      "words": "3.神装极速进阶,炼化玩法打造独特个性神装;"
    },
    {
      "chars": [
        {
          "char": "4",
          "location": { "width": 19, "top": 746, "left": 996, "height": 38 }
        },
        {
          "char": ".",
          "location": { "width": 19, "top": 746, "left": 1008, "height": 38 }
        },
        {
          "char": "独",
          "location": { "width": 22, "top": 746, "left": 1036, "height": 38 }
        },
        {
          "char": "创",
          "location": { "width": 23, "top": 746, "left": 1070, "height": 38 }
        },
        {
          "char": "十",
          "location": { "width": 23, "top": 746, "left": 1106, "height": 38 }
        },
        {
          "char": "二",
          "location": { "width": 23, "top": 746, "left": 1141, "height": 38 }
        },
        {
          "char": "星",
          "location": { "width": 23, "top": 746, "left": 1165, "height": 38 }
        },
        {
          "char": "使",
          "location": { "width": 23, "top": 746, "left": 1199, "height": 38 }
        },
        {
          "char": "助",
          "location": { "width": 23, "top": 746, "left": 1235, "height": 38 }
        },
        {
          "char": "战",
          "location": { "width": 23, "top": 746, "left": 1270, "height": 38 }
        },
        {
          "char": "系",
          "location": { "width": 23, "top": 746, "left": 1305, "height": 38 }
        },
        {
          "char": "统",
          "location": { "width": 23, "top": 746, "left": 1328, "height": 38 }
        },
        {
          "char": ",",
          "location": { "width": 19, "top": 746, "left": 1360, "height": 38 }
        },
        {
          "char": "五",
          "location": { "width": 22, "top": 746, "left": 1400, "height": 38 }
        },
        {
          "char": "大",
          "location": { "width": 23, "top": 746, "left": 1435, "height": 38 }
        },
        {
          "char": "魔",
          "location": { "width": 23, "top": 746, "left": 1457, "height": 38 }
        },
        {
          "char": "幻",
          "location": { "width": 22, "top": 746, "left": 1494, "height": 38 }
        },
        {
          "char": "星",
          "location": { "width": 23, "top": 746, "left": 1529, "height": 38 }
        },
        {
          "char": "使",
          "location": { "width": 23, "top": 746, "left": 1564, "height": 38 }
        },
        {
          "char": "技",
          "location": { "width": 23, "top": 746, "left": 1599, "height": 38 }
        },
        {
          "char": "能",
          "location": { "width": 23, "top": 746, "left": 1623, "height": 38 }
        },
        {
          "char": "逆",
          "location": { "width": 23, "top": 746, "left": 1659, "height": 38 }
        },
        {
          "char": "转",
          "location": { "width": 23, "top": 746, "left": 1693, "height": 38 }
        },
        {
          "char": "战",
          "location": { "width": 23, "top": 746, "left": 1728, "height": 38 }
        },
        {
          "char": "局",
          "location": { "width": 22, "top": 746, "left": 1765, "height": 38 }
        },
        {
          "char": ";",
          "location": { "width": 15, "top": 746, "left": 1794, "height": 38 }
        }
      ],
      "location": { "width": 821, "top": 746, "left": 989, "height": 38 },
      "words": "4.独创十二星使助战系统,五大魔幻星使技能逆转战局;"
    },
    {
      "chars": [
        {
          "char": "5",
          "location": { "width": 21, "top": 795, "left": 992, "height": 41 }
        },
        {
          "char": ".",
          "location": { "width": 21, "top": 795, "left": 1013, "height": 41 }
        },
        {
          "char": "六",
          "location": { "width": 25, "top": 795, "left": 1043, "height": 41 }
        },
        {
          "char": "大",
          "location": { "width": 25, "top": 795, "left": 1067, "height": 41 }
        },
        {
          "char": "化",
          "location": { "width": 25, "top": 795, "left": 1103, "height": 41 }
        },
        {
          "char": "神",
          "location": { "width": 25, "top": 795, "left": 1128, "height": 41 }
        },
        {
          "char": "任",
          "location": { "width": 25, "top": 795, "left": 1166, "height": 41 }
        },
        {
          "char": "意",
          "location": { "width": 23, "top": 795, "left": 1204, "height": 41 }
        },
        {
          "char": "搭",
          "location": { "width": 25, "top": 795, "left": 1228, "height": 41 }
        },
        {
          "char": "配",
          "location": { "width": 25, "top": 795, "left": 1264, "height": 41 }
        },
        {
          "char": ",",
          "location": { "width": 20, "top": 795, "left": 1297, "height": 41 }
        },
        {
          "char": "助",
          "location": { "width": 23, "top": 795, "left": 1327, "height": 41 }
        },
        {
          "char": "力",
          "location": { "width": 25, "top": 795, "left": 1364, "height": 41 }
        },
        {
          "char": "培",
          "location": { "width": 25, "top": 795, "left": 1400, "height": 41 }
        },
        {
          "char": "养",
          "location": { "width": 25, "top": 795, "left": 1438, "height": 41 }
        },
        {
          "char": "个",
          "location": { "width": 25, "top": 795, "left": 1463, "height": 41 }
        },
        {
          "char": "性",
          "location": { "width": 23, "top": 795, "left": 1501, "height": 41 }
        },
        {
          "char": "化",
          "location": { "width": 25, "top": 795, "left": 1525, "height": 41 }
        },
        {
          "char": "职",
          "location": { "width": 25, "top": 795, "left": 1563, "height": 41 }
        },
        {
          "char": "业",
          "location": { "width": 25, "top": 795, "left": 1599, "height": 41 }
        },
        {
          "char": "属",
          "location": { "width": 23, "top": 795, "left": 1624, "height": 41 }
        },
        {
          "char": "性",
          "location": { "width": 25, "top": 795, "left": 1661, "height": 41 }
        },
        {
          "char": ";",
          "location": { "width": 18, "top": 795, "left": 1694, "height": 41 }
        }
      ],
      "location": { "width": 719, "top": 795, "left": 992, "height": 41 },
      "words": "5.六大化神任意搭配,助力培养个性化职业属性;"
    },
    {
      "chars": [
        {
          "char": "6",
          "location": { "width": 20, "top": 844, "left": 994, "height": 42 }
        },
        {
          "char": ".",
          "location": { "width": 20, "top": 844, "left": 1014, "height": 42 }
        },
        {
          "char": "勇",
          "location": { "width": 25, "top": 844, "left": 1030, "height": 42 }
        },
        {
          "char": "者",
          "location": { "width": 26, "top": 844, "left": 1068, "height": 42 }
        },
        {
          "char": "转",
          "location": { "width": 25, "top": 844, "left": 1106, "height": 42 }
        },
        {
          "char": "职",
          "location": { "width": 25, "top": 844, "left": 1131, "height": 42 }
        },
        {
          "char": "突",
          "location": { "width": 26, "top": 844, "left": 1168, "height": 42 }
        },
        {
          "char": "破",
          "location": { "width": 26, "top": 844, "left": 1193, "height": 42 }
        },
        {
          "char": "属",
          "location": { "width": 25, "top": 844, "left": 1232, "height": 42 }
        },
        {
          "char": "性",
          "location": { "width": 25, "top": 844, "left": 1269, "height": 42 }
        },
        {
          "char": ",",
          "location": { "width": 21, "top": 844, "left": 1291, "height": 42 }
        },
        {
          "char": "职",
          "location": { "width": 25, "top": 844, "left": 1333, "height": 42 }
        },
        {
          "char": "业",
          "location": { "width": 25, "top": 844, "left": 1371, "height": 42 }
        },
        {
          "char": "时",
          "location": { "width": 25, "top": 844, "left": 1396, "height": 42 }
        },
        {
          "char": "装",
          "location": { "width": 25, "top": 844, "left": 1433, "height": 42 }
        },
        {
          "char": "免",
          "location": { "width": 38, "top": 844, "left": 1459, "height": 42 }
        },
        {
          "char": "费",
          "location": { "width": 25, "top": 844, "left": 1496, "height": 42 }
        },
        {
          "char": "领",
          "location": { "width": 25, "top": 844, "left": 1534, "height": 42 }
        },
        {
          "char": ";",
          "location": { "width": 21, "top": 844, "left": 1555, "height": 42 }
        }
      ],
      "location": { "width": 586, "top": 844, "left": 994, "height": 42 },
      "words": "6.勇者转职突破属性,职业时装免费领;"
    },
    {
      "chars": [
        {
          "char": "7",
          "location": { "width": 23, "top": 893, "left": 992, "height": 46 }
        },
        {
          "char": ".",
          "location": { "width": 22, "top": 893, "left": 1002, "height": 46 }
        },
        {
          "char": "B",
          "location": { "width": 23, "top": 893, "left": 1029, "height": 46 }
        },
        {
          "char": "O",
          "location": { "width": 22, "top": 893, "left": 1044, "height": 46 }
        },
        {
          "char": "S",
          "location": { "width": 23, "top": 893, "left": 1071, "height": 46 }
        },
        {
          "char": "S",
          "location": { "width": 23, "top": 893, "left": 1100, "height": 46 }
        },
        {
          "char": "爆",
          "location": { "width": 28, "top": 893, "left": 1118, "height": 46 }
        },
        {
          "char": "率",
          "location": { "width": 27, "top": 893, "left": 1147, "height": 46 }
        },
        {
          "char": "1",
          "location": { "width": 22, "top": 893, "left": 1171, "height": 46 }
        },
        {
          "char": "0",
          "location": { "width": 22, "top": 893, "left": 1184, "height": 46 }
        },
        {
          "char": "0",
          "location": { "width": 23, "top": 893, "left": 1212, "height": 46 }
        },
        {
          "char": "%",
          "location": { "width": 28, "top": 893, "left": 1231, "height": 46 }
        },
        {
          "char": ",",
          "location": { "width": 22, "top": 893, "left": 1255, "height": 46 }
        },
        {
          "char": "狂",
          "location": { "width": 28, "top": 893, "left": 1287, "height": 46 }
        },
        {
          "char": "爆",
          "location": { "width": 27, "top": 893, "left": 1331, "height": 46 }
        },
        {
          "char": "极",
          "location": { "width": 28, "top": 893, "left": 1358, "height": 46 }
        },
        {
          "char": "品",
          "location": { "width": 27, "top": 893, "left": 1387, "height": 46 }
        },
        {
          "char": "装",
          "location": { "width": 28, "top": 893, "left": 1428, "height": 46 }
        },
        {
          "char": "备",
          "location": { "width": 27, "top": 893, "left": 1456, "height": 46 }
        },
        {
          "char": ",",
          "location": { "width": 22, "top": 893, "left": 1480, "height": 46 }
        },
        {
          "char": "轻",
          "location": { "width": 28, "top": 893, "left": 1526, "height": 46 }
        },
        {
          "char": "松",
          "location": { "width": 28, "top": 893, "left": 1555, "height": 46 }
        },
        {
          "char": "集",
          "location": { "width": 28, "top": 893, "left": 1597, "height": 46 }
        },
        {
          "char": "齐",
          "location": { "width": 28, "top": 893, "left": 1625, "height": 46 }
        },
        {
          "char": "全",
          "location": { "width": 28, "top": 893, "left": 1653, "height": 46 }
        },
        {
          "char": "套",
          "location": { "width": 27, "top": 893, "left": 1696, "height": 46 }
        },
        {
          "char": "神",
          "location": { "width": 28, "top": 893, "left": 1724, "height": 46 }
        },
        {
          "char": "装",
          "location": { "width": 28, "top": 893, "left": 1752, "height": 46 }
        },
        {
          "char": ";",
          "location": { "width": 23, "top": 893, "left": 1775, "height": 46 }
        }
      ],
      "location": { "width": 813, "top": 893, "left": 992, "height": 46 },
      "words": "7.BOSS爆率100%,狂爆极品装备,轻松集齐全套神装;"
    },
    {
      "chars": [
        {
          "char": "8",
          "location": { "width": 25, "top": 945, "left": 994, "height": 49 }
        },
        {
          "char": ".",
          "location": { "width": 23, "top": 945, "left": 1008, "height": 49 }
        },
        {
          "char": "首",
          "location": { "width": 44, "top": 945, "left": 1029, "height": 49 }
        },
        {
          "char": "日",
          "location": { "width": 29, "top": 945, "left": 1072, "height": 49 }
        },
        {
          "char": "直",
          "location": { "width": 29, "top": 945, "left": 1102, "height": 49 }
        },
        {
          "char": "升",
          "location": { "width": 29, "top": 945, "left": 1132, "height": 49 }
        },
        {
          "char": "1",
          "location": { "width": 23, "top": 945, "left": 1158, "height": 49 }
        },
        {
          "char": "5",
          "location": { "width": 25, "top": 945, "left": 1172, "height": 49 }
        },
        {
          "char": "0",
          "location": { "width": 25, "top": 945, "left": 1201, "height": 49 }
        },
        {
          "char": "级",
          "location": { "width": 29, "top": 945, "left": 1222, "height": 49 }
        },
        {
          "char": ",",
          "location": { "width": 25, "top": 945, "left": 1247, "height": 49 }
        },
        {
          "char": "登",
          "location": { "width": 30, "top": 945, "left": 1281, "height": 49 }
        },
        {
          "char": "录",
          "location": { "width": 29, "top": 945, "left": 1327, "height": 49 }
        },
        {
          "char": "即",
          "location": { "width": 29, "top": 945, "left": 1357, "height": 49 }
        },
        {
          "char": "送",
          "location": { "width": 29, "top": 945, "left": 1387, "height": 49 }
        },
        {
          "char": ":",
          "location": { "width": 23, "top": 945, "left": 1412, "height": 49 }
        },
        {
          "char": "兽",
          "location": { "width": 29, "top": 945, "left": 1446, "height": 49 }
        },
        {
          "char": "王",
          "location": { "width": 29, "top": 945, "left": 1492, "height": 49 }
        },
        {
          "char": "天",
          "location": { "width": 29, "top": 945, "left": 1521, "height": 49 }
        },
        {
          "char": "神",
          "location": { "width": 29, "top": 945, "left": 1551, "height": 49 }
        },
        {
          "char": "、",
          "location": { "width": 25, "top": 945, "left": 1575, "height": 49 }
        },
        {
          "char": "春",
          "location": { "width": 29, "top": 945, "left": 1611, "height": 49 }
        },
        {
          "char": "日",
          "location": { "width": 30, "top": 945, "left": 1655, "height": 49 }
        },
        {
          "char": "时",
          "location": { "width": 29, "top": 945, "left": 1686, "height": 49 }
        },
        {
          "char": "装",
          "location": { "width": 29, "top": 945, "left": 1716, "height": 49 }
        },
        {
          "char": "、",
          "location": { "width": 25, "top": 945, "left": 1740, "height": 49 }
        },
        {
          "char": "雪",
          "location": { "width": 29, "top": 945, "left": 1789, "height": 49 }
        },
        {
          "char": "豹",
          "location": { "width": 28, "top": 945, "left": 1820, "height": 49 }
        }
      ],
      "location": { "width": 879, "top": 945, "left": 968, "height": 49 },
      "words": "8.首日直升150级,登录即送:兽王天神、春日时装、雪豹"
    }
  ]
}
```

## 计算文字的位置

背景：百度OCR返回的文字都是json字典，希望能从对应匹配到的单词，找到对应的位置坐标信息

代码：

```python
def calcWordsLocation(self, wordStr, curWordsResult):
    """Calculate words location from result


    Args:
        wordStr (str): the words to check
        curWordsResult (dict): the baidu OCR result of current words
    Returns:
        location, a tuple (x, y, width, height)
    Raises:
    Examples
            wordStr="首充"
            curWordsResult= {
                    "chars": [
                        {
                        "char": "寻",
                        "location": {
                            "width": 15,
                            "top": 51,
                            "left": 725,
                            "height": 24
                        }
                        },
                        ...
                        {
                        "char": "首",
                        "location": {
                            "width": 15,
                            "top": 51,
                            "left": 971,
                            "height": 24
                        }
                        },
                        {
                        "char": "充",
                        "location": {
                            "width": 15,
                            "top": 51,
                            "left": 986,
                            "height": 24
                        }
                        }
                    ],
                    "location": {
                        "width": 280,
                        "top": 51,
                        "left": 725,
                        "height": 24
                    },
                    "words": "寻宝福利大厅商城首充"
                }
            -> (971, 51, 30, 24)
    """
    (x, y, width, height) = (0, 0, 0, 0)
    matchedStr = curWordsResult["words"]
    # Note: for special, contain multilple words, here only process firt words
    foundWords = re.search(wordStr, matchedStr)
    if foundWords:
        logging.debug("foundWords=%s" % foundWords)


        firstMatchedPos = foundWords.start()
        lastMatchedPos = foundWords.end() - 1


        matchedStrLen = len(matchedStr)
        charResultList = curWordsResult["chars"]
        charResultListLen = len(charResultList)


        firstCharResult = None
        lastCharResult = None
        if matchedStrLen == charResultListLen:
            firstCharResult = charResultList[firstMatchedPos]
            lastCharResult = charResultList[lastMatchedPos]
        else:
            # Special: for 'Loading' matched ' Loading', but charResultList not include first space ' ', but from fisrt='L' to end='g'
            # so using find the corresponding char, then got its location
            # Note: following method not work for regex str, like '^游戏公告$'


            firtToMatchChar = wordStr[0]
            lastToMatchChar = wordStr[-1]


            for eachCharResult in charResultList:
                if firstCharResult and lastCharResult:
                    break


                eachChar = eachCharResult["char"]
                if firtToMatchChar == eachChar:
                    firstCharResult = eachCharResult
                elif lastToMatchChar == eachChar:
                    lastCharResult = eachCharResult


        # Note: follow no need check words, to support input ^游戏公告$ to match "游戏公告"
        # firstLocation = None
        # lastLocation = None
        # if firstCharResult["char"] == firtToMatchChar:
        #     firstLocation = firstCharResult["location"]
        # if lastCharResult["char"] == lastToMatchChar:
        #     lastLocation = lastCharResult["location"]
        firstLocation = firstCharResult["location"]
        lastLocation = lastCharResult["location"]


        # if firstLocation and lastLocation:


        # support both horizontal and vertical words
        firstLeft = firstLocation["left"]
        lastLeft = lastLocation["left"]
        minLeft = min(firstLeft, lastLeft)
        x = minLeft


        firstHorizontalEnd = firstLeft + firstLocation["width"]
        lastHorizontalEnd = lastLeft + lastLocation["width"]
        maxHorizontalEnd = max(firstHorizontalEnd, lastHorizontalEnd)
        width = maxHorizontalEnd - x


        lastTop = lastLocation["top"]
        minTop = min(firstLocation["top"], lastTop)
        y = minTop


        lastVerticalEnd = lastTop + lastLocation["height"]
        height = lastVerticalEnd - y


    return x, y, width, height
```

调用：

```python
calculatedLocation = self.calcWordsLocation(eachInputWords, eachWordsMatchedResult)
```

## 把坐标位置转成中间坐标值

作用：用于后续点击按钮中间坐标值

代码：

```python
def locationToCenterPos(self, wordslocation):
    """Convert location of normal button to center position


    Args:
        wordslocation (tuple): words location, (x, y, width, height)
            Example: (267, 567, 140, 39)
    Returns:
        tuple, (x, y), the location's center position, normal used later to click it
            Example: (337.0, 586.5)
    Raises:
    """
    x, y, width, height = wordslocation
    centerX = x + width/2
    centerY = y + height/2
    centerPosition = (centerX, centerY)
    return centerPosition
```

调用：

```python
curCenterX, curCenterY = self.locationToCenterPos(eachLocation)
```

## 检测文字是否在结果中

代码：

```python
def isWordsInResult(self, respJson, wordsOrWordsList, isMatchMultiple=False):
    """Check words is in result or not


    Args:
        respJson (dict): Baidu OCR responsed json
        wordsOrWordsList (str/list): single input str or str list
        isMatchMultiple (bool): for each single str, to match multiple output or only match one output
    Returns:
        dict, matched result
    Raises:
    """
    # Note: use OrderedDict instead dict to keep order, for later get first match result to process
    orderedMatchedResultDict = OrderedDict()


    inputWordsList = wordsOrWordsList
    if isinstance(wordsOrWordsList, str):
        inputWords = str(wordsOrWordsList)
        inputWordsList = [inputWords]


    wordsResultList = respJson["words_result"]
    for curInputWords in inputWordsList:
        curMatchedResultList = []
        for eachWordsResult in wordsResultList:
            eachWords = eachWordsResult["words"]
            foundCurWords = re.search(curInputWords, eachWords)
            if foundCurWords:
                curMatchedResultList.append(eachWordsResult)
                if not isMatchMultiple:
                    break


        orderedMatchedResultDict[curInputWords] = curMatchedResultList
    return orderedMatchedResultDict
```

调用：

```python
matchedResultDict = self.isWordsInResult(wordsResultJson, wordsOrWordsList, isMatchMultiple)
```

## 检测当前屏幕中是否包含对应文字

```python
def isWordsInCurScreen(self, wordsOrWordsList, imgPath=None, isMatchMultiple=False, isRespShortInfo=False):
    """Found words in current screen


    Args:
        wordsOrWordsList (str/list): single input str or str list
        imgPath (str): current screen image file path; default=None; if None, will auto get current scrren image
        isMatchMultiple (bool): for each single str, to match multiple output or only match one output; default=False
        isRespShortInfo (bool): return simple=short=nomarlly bool or list[bool] info or return full info which contain imgPath and full matched result.
    Returns:
        matched result, type=bool/list[bool]/dict/tuple, depends on diffrent condition
    Raises:
    """
    retValue = None


    if not imgPath:
        # do screenshot
        imgPath = self.getCurScreenshot()


    wordsResultJson = self.baiduImageToWords(imgPath)


    isMultipleInput = False
    inputWords = None
    inputWordsList = []


    if isinstance(wordsOrWordsList, list):
        isMultipleInput = True
        inputWordsList = list(wordsOrWordsList)
    elif isinstance(wordsOrWordsList, str):
        isMultipleInput = False
        inputWords = str(wordsOrWordsList)
        inputWordsList = [inputWords]


    matchedResultDict = self.isWordsInResult(wordsResultJson, wordsOrWordsList, isMatchMultiple)


    # add caclulated location and words
    # Note: use OrderedDict instead dict to keep order, for later get first match result to process
    processedResultDict = OrderedDict()
    for eachInputWords in inputWordsList:
        isCurFound = False
        # curLocatoinList = []
        # curWordsList = []
        curResultList = []


        curWordsMatchedResultList = matchedResultDict[eachInputWords]
        if curWordsMatchedResultList:
            isCurFound = True
            for curIdx, eachWordsMatchedResult in enumerate(curWordsMatchedResultList):
                curMatchedWords = eachWordsMatchedResult["words"]
                calculatedLocation = self.calcWordsLocation(eachInputWords, eachWordsMatchedResult)
                # curLocatoinList.append(calculatedLocation)
                # curWordsList.append(curMatchedWords)
                curResult = (curMatchedWords, calculatedLocation)
                curResultList.append(curResult)


        # processedResultDict[eachInputWords] = (isCurFound, curLocatoinList, curWordsList)
        processedResultDict[eachInputWords] = (isCurFound, curResultList)
    logging.debug("For %s, matchedResult=%s from imgPath=%s", wordsOrWordsList, processedResultDict, imgPath)


    if isMultipleInput:
        if isRespShortInfo:
            isFoundList = []
            for eachInputWords in processedResultDict.keys():
                isCurFound, noUse = processedResultDict[eachInputWords]
                isFoundList.append(isCurFound)
            # Note: no mattter isMatchMultiple, both only return single boolean for each input words
            retBoolList = isFoundList
            retValue = retBoolList
        else:
            if isMatchMultiple:
                retTuple = processedResultDict, imgPath
                retValue = retTuple
            else:
                # Note: use OrderedDict instead dict to keep order, for later get first match result to process
                respResultDict = OrderedDict()
                for eachInputWords in processedResultDict.keys():
                    # isCurFound, curLocatoinList, curWordsList = processedResultDict[eachInputWords]
                    isCurFound, curResultList = processedResultDict[eachInputWords]
                    # singleLocation = None
                    # singleWords = None
                    singleResult = (None, None)
                    if isCurFound:
                        # singleLocation = curLocatoinList[0]
                        # singleWords = curWordsList[0]
                        singleResult = curResultList[0]
                    # respResultDict[eachInputWords] = (isCurFound, singleLocation, singleWords)
                    respResultDict[eachInputWords] = (isCurFound, singleResult)
                retTuple = respResultDict, imgPath
                retValue = retTuple
    else:
        singleInputResult = processedResultDict[inputWords]
        # isCurFound, curLocatoinList, curWordsList = singleInputResult
        isCurFound, curResultList = singleInputResult
        if isRespShortInfo:
            # Note: no mattter isMatchMultiple, both only return single boolean for each input words
            retBool = isCurFound
            retValue = retBool
        else:
            if isMatchMultiple:
                # retTuple = isCurFound, curLocatoinList, curWordsList, imgPath
                retTuple = isCurFound, curResultList, imgPath
                retValue = retTuple
            else:
                singleResult = (None, None)
                # singleLocation = None
                # singleWords = None
                if isCurFound:
                    # singleLocation = curLocatoinList[0]
                    # singleWords = curWordsList[0]
                    singleResult = curResultList[0]
                # retTuple = isCurFound, singleLocation, singleWords, imgPath
                retTuple = isCurFound, singleResult, imgPath
                retValue = retTuple


    logging.debug("Input: %s, output=%s", wordsOrWordsList, retValue)
    return retValue
```

调用：

```python
allResultDict, _ = self.isWordsInCurScreen(allStrList, imgPath, isMatchMultiple=True)
```

## 获取当前屏幕中的文字

```python
def getWordsInCurScreen(self):
    """get words in current screenshot"""
    screenImgPath = self.getCurScreenshot()
    wordsResultJson = self.baiduImageToWords(screenImgPath)
    return wordsResultJson
```

调用：

```python
curScreenWords = self.getWordsInCurScreen()
```

## 检测当前屏幕中是否存在某些信息

```python
def checkExistInScreen(self,
        imgPath=None,
        mandatoryStrList=[],
        mandatoryMinMatchCount=0,
        optionalStrList=[],
        # optionalMinMatchCount=2,
        optionalMinMatchCount=1,
        isRespFullInfo=False
    ):
    """Check whether mandatory and optional str list in current screen or not


    Args:
        imgPath (str): current screen image file path; default=None; if None, will auto get current scrren image
        mandatoryStrList (list): mandatory str, at least match `mandatoryMinMatchCount`, or all must match if `mandatoryMinMatchCount`=0
        mandatoryMinMatchCount (int): minimal match count for mandatory list
        optionalStrList (list): optional str, some may match
        optionalMinMatchCount (int): for `optionalStrList`, the minimal match count, consider to match or not
        isRespFullInfo (bool): return full info or not, full info means match location result and imgPath
    Returns:
        matched result, type=bool/tuple, depends on `isRespFullInfo`
    Raises:
    """
    if not imgPath:
        imgPath = self.getCurScreenshot()
    logging.debug("imgPath=%s", imgPath)


    isExist = False
    # Note: use OrderedDict instead dict to keep order, for later get first match result to process
    respMatchLocation = OrderedDict()


    isMandatoryMatch = True
    isMandatoryShouldMatchAll = (mandatoryMinMatchCount <= 0)
    isOptionalMatch = True


    allStrList = []
    allStrList.extend(mandatoryStrList)
    allStrList.extend(optionalStrList)


    optionalMatchCount = 0
    mandatoryMatchCount = 0
    allResultDict, _ = self.isWordsInCurScreen(allStrList, imgPath, isMatchMultiple=True)
    for eachStr, (isFoundCur, curResultList) in allResultDict.items():
        if eachStr in mandatoryStrList:
            if isFoundCur:
                mandatoryMatchCount += 1
                respMatchLocation[eachStr] = curResultList
            else:
                if isMandatoryShouldMatchAll:
                    isMandatoryMatch = False
                    break
        elif eachStr in optionalStrList:
            if isFoundCur:
                optionalMatchCount += 1
                respMatchLocation[eachStr] = curResultList


    if mandatoryStrList:
        if not isMandatoryShouldMatchAll:
            if mandatoryMatchCount >= mandatoryMinMatchCount:
                isMandatoryMatch = True
            else:
                isMandatoryMatch = False


    if optionalStrList:
        if optionalMatchCount >= optionalMinMatchCount:
            isOptionalMatch = True
        else:
            isOptionalMatch = False


    isExist = isMandatoryMatch and isOptionalMatch
    logging.debug("isMandatoryMatch=%s, isOptionalMatch=%s -> isExist=%s", isMandatoryMatch, isOptionalMatch, isExist)


    if isRespFullInfo:
        logging.debug("mandatoryStrList=%s, optionalStrList=%s -> isExist=%s, respMatchLocation=%s, imgPath=%s", 
            mandatoryStrList, optionalStrList, isExist, respMatchLocation, imgPath)
        return (isExist, respMatchLocation, imgPath)
    else:
        logging.debug("mandatoryStrList=%s, optionalStrList=%s -> isExist=%s", 
            mandatoryStrList, optionalStrList, isExist)
        return isExist
```

调用：

```python
checkResult = self.checkExistInScreen(
    imgPath=imgPath,
    optionalStrList=strList,
    optionalMinMatchCount=1,
    isRespFullInfo=isRespFullInfo,
)
```

和：

```python
minOptionalMatchCount = 2

mandatoryList = [
    # "公告",
    "^公告$",
    '^强力推荐$', # 王城英雄, 不规则弹框
。。。
]

possibleTitleList = [
    "^游戏公告$",
]
optionalList = []
otherOptionalList = [
    "新增(内容)?",
    "游戏特色",
    "(主要)?更新(内容)?",
。。。
    "登录即送",
]
optionalList.extend(possibleTitleList)
optionalList.extend(otherOptionalList)


checkResult = self.checkExistInScreen(
    imgPath=imgPath,
    mandatoryStrList=mandatoryList,
    mandatoryMinMatchCount=1,
    optionalStrList=optionalList,
    optionalMinMatchCount=minOptionalMatchCount,
    isRespFullInfo=isRespFullInfo,
)
```

和：

```python
mandatoryList = [
    "^购买$", # 造梦西游：'购买'
    "^￥\d+元?", # 剑玲珑, 至尊屠龙
    "^\d+元", # 造梦西游：'6元券3元券3’
    "^充值$", # 剑玲珑, 至尊屠龙
]
optionalList = [
    # common
    "元宝",
    "月卡",
。。。
    # 剑玲珑
    "赠",
]

isRealPay, matchResult, imgPath  = self.checkExistInScreen(
    mandatoryStrList=mandatoryList,
    mandatoryMinMatchCount=2,
    optionalStrList=optionalList,
    optionalMinMatchCount=2,
    isRespFullInfo=True,
)
```

和：

```python
mandatoryList = [
    "^((继续)|(结束))$", # 暗黑觉醒：'继续' or '结束'
    "^\d秒$", # 暗黑觉醒：'5秒', '1秒'
]

isAutoConverstion, matchResult, imgPath  = self.checkExistInScreen(
    imgPath=imgPath,
    mandatoryStrList=mandatoryList,
    mandatoryMinMatchCount=2,
    isRespFullInfo=True,
)
```

和：

```python
mandatoryList = [
    "^充值", # 造梦西游： "充值战神榜邮件各但,挑战竞技好友"
    "首充$", # 剑玲珑，至尊屠龙
。。。
]
optionalList = [
    # common
    "组队", # 至尊屠龙, 剑玲珑
    "任务", # 至尊屠龙, 剑玲珑

    # "商城", # 剑玲珑
    "寻宝", # 剑玲珑
    "福利大厅", # 剑玲珑
。。。
    "背包",
]

isHome, matchResult, imgPath  = self.checkExistInScreen(
    mandatoryStrList=mandatoryList,
    mandatoryMinMatchCount=1,
    optionalStrList=optionalList,
    isRespFullInfo=True,
)
```

和：

```python
mandatoryList = [
    "立即登录", # 剑玲珑, 至尊屠龙
。。。
    "^登录$", # 暗黑觉醒
]
optionalList = [
    # common
    "一键((注册)|(试玩))",
    "忘记密码", # 至尊屠龙, 青云诀
    "((用户)|(账号)|(手机))登录", # 剑玲珑, 至尊屠龙,
    "点击选服", # 青云诀，暗黑觉醒
。。。
    # 造梦西游
    "游客登录", #更新：不能用游客登录，否则后续无法弹出支付页面
]

respUserLogin = self.checkExistInScreen(
    mandatoryStrList=mandatoryList,
    mandatoryMinMatchCount=1,
    optionalStrList=optionalList,
    isRespFullInfo=isRespFullInfo
)
```

## 是否存在任意一个词组

```python
def isExistAnyStr(self, strList, imgPath=None, isRespFullInfo=False):
    """Is any str exist or not


    Args:
        strList (list): str list to check exist or not
        imgPath (str): current screen image file path; default=None; if None, will auto get current scrren image
        isRespFullInfo (bool): return full info or not, full info means match location result and imgPath
    Returns:
        matched result, type=bool/tuple, depends on `isRespFullInfo`
    Raises:
    """
    if not imgPath:
        imgPath = self.getCurScreenshot()


    checkResult = self.checkExistInScreen(
        imgPath=imgPath,
        optionalStrList=strList,
        optionalMinMatchCount=1,
        isRespFullInfo=isRespFullInfo,
    )
    if isRespFullInfo:
        isExistAny, matchResult, imgPath = checkResult
        logging.debug("isExistAny=%s, matchResult=%s, imgPath=%s for %s", isExistAny, matchResult, imgPath, strList)
        return (isExistAny, matchResult, imgPath)
    else:
        isExistAny = checkResult
        logging.debug("isExistAny=%s, for %s", isExistAny, strList)
        return isExistAny
```

调用：

```python
    isExist, matchResult, imgPath = self.isExistAnyStr(buttonStrList, imgPath=imgPath, isRespFullInfo=True)
```

和：

```python
mandatoryList = [
    # 御剑仙缘
    """^(请)?点击\s?[“”"'][^“”"',]{1,6}[“”"']?""", # '请点击“战骑'，'请点击“魂兽”'，'请点击“人物”'，'请点击“领取”'，'请点击“仙盟”'，'点击“+”,放入吞噬'，'点击“进阶”,提升战'，请点击“每日必做”按
]

respResult = self.isExistAnyStr(mandatoryList, imgPath=imgPath, isRespFullInfo=isRespFullInfo)
```

和：

```python
requireManualOperationList = [
    "完成指定操作", # speical：造梦西游 的 '(请完成指定操作)梦口'  的顶部弹框
]
isRequireManual, _, imgPath = self.isExistAnyStr(requireManualOperationList, isRespFullInfo=True)
```

和：

```python
lanuchStrList = [
    "^4399手机游戏$", # 剑玲珑
    "^西瓜游戏$", # 青云诀
]
isLaunch, _, imgPath = self.isExistAnyStr(lanuchStrList, isRespFullInfo=True)
```

和：

```python
    loadingStrList = [
        # 登录类
        "正在登录", # 正在登录
        "logging",
。。。
        "游戏资源", # 青云诀:本地游戏资源已是最新
。。。
    ]

    isLoadingSth, _, imgPath = self.isExistAnyStr(loadingStrList, isRespFullInfo=True)
```

和：

```python
gotoPayStrList = [
    "^前往充值$", # 剑玲珑
    "^立即充值$", # 至尊屠龙
。。。
]
respBoolOrTuple = self.isExistAnyStr(gotoPayStrList, isRespFullInfo=isRespLocation)
```

## 判断是否所有的字符都存在

```python
def isExistAllStr(self, strList, imgPath=None, isRespFullInfo=False):
    """Is all str exist or not


    Args:
        strList (list): str list to check exist or not
        imgPath (str): current screen image file path; default=None; if None, will auto get current scrren image
        isRespFullInfo (bool): return full info or not, full info means match location result and imgPath
    Returns:
        matched result, type=bool/tuple, depends on `isRespFullInfo`
    Raises:
    """
    if not imgPath:
        imgPath = self.getCurScreenshot()
    checkResult = self.checkExistInScreen(imgPath=imgPath, mandatoryStrList=strList, isRespFullInfo=isRespFullInfo)
    if isRespFullInfo:
        isExistAll, matchResult, imgPath = checkResult
        logging.debug("isExistAll=%s, matchResult=%s, imgPath=%s for %s", isExistAll, matchResult, imgPath, strList)
        return (isExistAll, matchResult, imgPath)
    else:
        isExistAll = checkResult
        logging.debug("isExistAll=%s, for %s", isExistAll, strList)
        return isExistAll
```

调用：

```python
realPayStrList = [
    "^￥\d+元?", # 剑玲珑, 至尊屠龙
        "^充值$", # 剑玲珑, 至尊屠龙
]
return self.isExistAllStr(realPayStrList, isRespFullInfo=isRespLocation)
```
