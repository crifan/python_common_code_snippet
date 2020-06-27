# Pillow

Python中图像处理用的最多是：`Pillow`

* Pillow
  * 继承自：`PIL`
    * `PIL` = `Python Imaging Library`
  * 官网资料：
    * [Image Module — Pillow (PIL Fork) 7.0.0 documentation](https://pillow.readthedocs.io/en/stable/reference/Image.html)
    * [Image Module — Pillow (PIL Fork) 3.1.2 documentation](https://pillow.readthedocs.io/en/3.1.x/reference/Image.html)

## 从二进制生成Image

```python
if isinstance(inputImage, bytes):
  openableImage = io.BytesIO(inputImage)
  curPillowImage = Image.open(openableImage)
```

pillow变量是：

```bash
# <PIL.PngImagePlugin.PngImageFile image mode=RGBA size=3543x3543 at 0x1065F7A20>
# <PIL.JpegImagePlugin.JpegImageFile image mode=RGB size=1080x1920 at 0x1026D7278>
```

详见：

* 【已解决】Python如何从二进制数据中生成Pillow的Image
* 【已解决】Python的Pillow如何从二进制数据中读取图像数据

## 从Pillow的Image获取二进制数据

```python
import io

imageIO = io.BytesIO()
curImg.save(imageIO, curImg.format)
imgBytes = imageIO.getvalue()
```

详见：

【已解决】Python的Pillow如何返回图像的二进制数据
