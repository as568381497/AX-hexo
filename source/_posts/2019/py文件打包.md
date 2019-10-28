---
title: py文件打包及web打开本地程序
date: 2019-10-28 15:48:36
tags: [技术,打包]
category: [其他技术,Python]
---

一句话开头
```
py文件 → exe可执行文件 → 可执行安装包

```

最近回顾下将本地py文件打成安装包的步骤，一共两步，先将程序主程序py文件打包成exe可执行程序，然后将exe文件打包成可安装的安装包程序。


### 1.将py文件转换为exe可执行文件

首先确定本地有"pyinstaller"，没有的话执行以下代码安装"pyinstaller"。

```
pip install pyinstaller
```

当“pyinstaller”下载完成，并且准备打包后，终端进入程序主程序的文件夹，然后终端执行以下代码打包。

```
pyinstaller --onefile --windowed xxxx.py

```

用 "pyinstaller"执行一句话就可以全自动将py文件全部打包为exe文件，打包完成后会在当前的文件夹生成一个dic的文件夹，可执行程序就在里面，文件名是主程序的文件名称,当然想在其他位置，将后面的文件位置和进入的文件夹位置换一下就好了。

### 2.将exe文件和其他文件打包为安装文件

这一步也需要借助一个软件 —— “Inno Setup”,[这里是下载地址](http://www.jrsoftware.org/download.php/is.exe)

在安装完成后，打开"inno Setup 编译器",然后在打开的界面里面选择 "用‘脚本向导创建新的脚本文件’"这一选项，后面就可以根据向导制定一个自己exe文件的打包脚本，一个注意的地方就是选择自己的exe文件后，还可以选择其他需要打包的文件或者文件夹一起打包进去，当全部选择完成后，点击完成，就可以完成当前的j打包脚本的创建了，然后程序会询问是否运行，点击否，先保存脚本文件，然后再点击运行就可以了。

当脚本执行完毕，自己程序的安装包就生成好了


### 扩充 —— web端打开本地程序

具体的实现效果就是用户点击网页某个事件，然后网页弹出询问窗口后打开本地程序。

在上面打包的基础上，本地会保存一个后缀为 "iss" 的打包脚本，在这个脚本的内容后面加上下面代码

```
[Registry]
Root: HKCR; SubKey: WebPrinter; ValueData: "DICOM Protocol"; ValueType: string; Flags: CreateValueIfDoesntExist UninsDeleteKey;
Root: HKCR; SubKey: WebPrinter; ValueName: "URL Protocol"; Flags: CreateValueIfDoesntExist; ValueType: string;
Root: HKCR; SubKey: WebPrinter\DefaultIcon; ValueData: {app}\{#MyAppExeName}; Flags: CreateValueIfDoesntExist; ValueType: string;
Root: HKCR; SubKey: WebPrinter\shell\open\command; ValueData: "{app}\{#MyAppExeName} ""%L"""; Flags: CreateValueIfDoesntExist; ValueType: string;

```

这是在注册表里面添加打开的协议，其中："WebPrinter"是协议名称,“ValueName”是协议中key的值，"ValueData"是协议中velue的值，可以自己更改掉。

然后执行这个脚本程序再次打包就可以了。

然后在web端，需要调用的地方添加一下代码，其中"WebPrinter"是需要调用的协议，冒号后面是具体的传值。
```
href="WebPrinter:xxxxx"
```

具体的方式y可以有：
1.在“a”标签中调用:

```
<a heef="WebPrinter:xxxx"></a>
```

2.js代码中调用

```
location.href = "WebPrinter:xxxx";
```
