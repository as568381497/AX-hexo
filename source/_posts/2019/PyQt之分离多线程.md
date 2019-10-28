---
title: PyQt之分离多线程
date: 2019-10-15 15:08:14
tags: [技术,PyQt4]
category: [PyQt4]
---

PyQt中实现多线程处理任务，本文单独创建新的多线程类处理任务

### 1.创建线程类

继承“QThread类"创建自己的线程类，在类中添加回传的信号

```
class MyThread(QtCore,QThread):
    trigger = QtCore.pyqtSignal(int)
```

### 2.实现需要任务

在自己的线程类中，实现“run”方法，在方法中写需要多线程处理的任务，然后使用
```
self.trigger.emit(int)//发送信号并传出的参数
```
来发送信号以调用其他方法来刷新UI等操作


### 3.实现收到信号后实现的方法
```
//收到信号后实现的方法，这里收到信号后也是子线程调用处理
def fation(self, int):
    //具体实现
```

### 4.在需要创建子线程处理任务的地方创建类
```
//创建类并指定接收到信号时实现方法
thred = MyThread()
//绑定收到信号后实现方法
thred.trigger.connect(self.fation)
//启动子线程
thred.staet()
```

最后分享一个我自己简单封装的异步请求类

[这个是地址](https://github.com/as568381497/CXPyQt4HTTP)

