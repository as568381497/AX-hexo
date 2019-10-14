---
title: PyQt4坑(特性)集合（二）
date: 2019-10-14 16:18:00
tags: [技术,坑集合,PyQt4]
category: [PyQt4]
---

最近开发中遇到的PyQt的坑，写出来，分享下


### 1.Label 控件中 给图片样式的显示方式

Background-image：显示方式  原始图片显示   不裁剪
Border-image：显示方式  拉伸  完整显示
image:显示方式 原始图片 最合适比例显示 


### 2.Label 图片的加载

png：png图片显示可以使用样式或者pixmap（跟jpg加载一样）显示，但是需要先把图片加载为py文件

⑴. 写入qrc文件

```
<!DOCTYPE RCC>
<RCC version="1.0">
<qresource prefix="/Images">
<file>time.png</file>
<file>gou.png</file>
<file>default.png</file>
</qresource>
</RCC>
```

⑵. 将qrc文件用PyRcc转换为py文件

⑶. 调用py文件进行加载

jpg图片加载： jpg图片加载只能用pixmap加载显示（实际上是画上去）

```
self.disPlayImage.load(self.jpgImageURL)
pixmap = QtGui.QPixmap(self.disPlayImage)
//画上去的显示方式（这是imageView的大小，不拉伸）
scaredPixmap = pixmap.scaled(self.imageView.width(), self.imageView.height(), aspectRatioMode=Qt.KeepAspectRatio)
//pixmap加载图片
self.imageView.setPixmap(scaredPixmap)
```

### 3.控件的边框显示


需要添加边框的宽，显示方式，颜色才能显示


### 4.获取屏幕大小


```
QtGui.QApplication.desktop()
View = desktop.screenGeometry()
```

### 5.子控件需要实现父控件paintEvent绘图方法才能显示图片

### 6.当子控件截取了点击事件后在点击事件里面调用父控件点击事件进行事件传递

### 7.父控件去掉子控件不是用“deleteLater”，这个方法只是在界面上去掉子控件，实际上内存中还存在，用sip的“deleter”方法删除也不安全，会删掉不应该删除的，查看源码发现，源码里面是用

```
view.setParent(None)
```

来删除控件的，这样内存中的地址才会被回收


### 8.在“main”函数里面可以获取软件存在的路径及外部传入的参数
```
sys.argv[0]//软件打开路径
sys.arvg[1]//外部传入参数
```
