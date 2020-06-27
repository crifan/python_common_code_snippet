# requests

## 下载图片，保存为二进制文件

```python
import requests
resp = requests.get(pictureUrl)
with open(saveFullPath, 'wb') as saveFp:
  saveFp.write(resp.content)
```

详见：

【已解决】Python的requests中如何下载二进制数据保存为图片文件
