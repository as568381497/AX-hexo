---
title: WEB端接入阿里多人音视频通信
date: 2020-09-30 15:32:07
tags: [WEB,SDK,音视频]
category: [WEB]
---

###  1.序
前段时间做完了一个项目，其中有个视频会议的功能，其中要实现的核心是多对多音视频通话功能，用介入第三方SDK，也就是接入了阿里的音视频RTC实现，觉得还是可以记录一下。

当时其实考察了很多其他的SDK，比如腾讯的视频会议，还有阿里的视频会议，但是，实际操作完后，发现这两个SDK还在开发中，不完善，而且其实具体的实现方式，还是跳转回他们自己视频会议的页面，觉得这样的接入方式限制太多，于是考虑自己实现所有功能，也就是换了阿里的音视频会议的RTC。

### 2.概述
因为涉及到的场景是多对多人视频通话，阿里这边的实现方式是，通过一个key值，可以理解为一个房间号，或者通道号，把进入这个房间（通道）中的人的音视频信息共享，都向这个房间传音视频流，同时从这个房间拉取别人的音视频流。

### 3.接入流程
在这里接入流程有两个，第一个是SDK的接入流程，另外一个是功能实现的流程。


#### 3.1 SDK接入
官方给的SDK的接入流程是三步：
1.开通服务
2.创建应用
3.集成客户端

在这里，我主要是再介绍下这里面一些注意的地方:
1.第一个是开通服务后，测试是不收费的，也就是说，可以先接入调试，不需要先付费，
2.第二步，需要在阿里内置的控制台创建一个应用，名字可随意，主要是生成完应用后，会给你一个app key 和 app id ，这两个很重要，是接入SDK的凭证，做完这两步，然后在下载页面，直接下载自己平台的SDK就好了，我这边是下载的是WEB端的SDK。

接入SDK后，也就是将SDK包下载下来，我这边是WEB端，主要是三步：
1.首先下载SDK，然后将AliWebRtcSDK包保存到本地项目下
2.最后在项目相应的前端页面文件中3.
3.对aliyun-webrtc-sdk.js文件进行引用

#### 3.2 SDK功能实现

1.接入SDK后，首先要初始化SDK,获得SDK的对象实例。

```bash
var aliWebrtc = new AliRtcEngine();
```
推荐在进入视频页面的时候，先预览下视频状态，也就是预览视频,预览成功时，正常执行，预览失败，则说明有其他问题，跳出提示。
```bash
aliWebrtc.startPreview(
video    // html中的video元素
).then(()=>{
}).catch((error) => {
// 预览失败
});
```

2.加入频道,预览成功后，加入制定的房间
```bash
aliWebrtc.joinChannel({
userid,         // 用户ID
channel,        // 频道
appid,          // 应用ID
nonce,          // 随机码
timestamp,      // 时间戳
gslb,           // gslb服务地址
token,          // 令牌
},displayName).then(()=>{
// 入会成功
} ,(error)=>{
// 入会失败，打印错误内容，可以看到失败原因
console.log(error.message);
});
```

将我们之前申请的appid 和appkey放上去，然后根据文档提示，将参数补全，这里要重点说下两个参数，一个是timestamp,他是说明你这个加入频道的请求是否过期，从你发出请求的那个时间开始，过期了是会进入房间失败的，第二个是token，他是有其他的参数一起生成，如果进入频道失败，很大的可能就是这个参数生成错误。

3.发布或取消发布本地流。
```bash
//开始推音视频流
aliWebrtc.publish().then(()=>{
} ,(error)=>{
console.log(error.message);
});
//取消推流
aliWebrtc.unPublish().then(()=>{
} ,(error)=>{
console.log(error.message);
});
```
当我们进入房间成功后，就可以设置是否推流了。

4.订阅onPublisher回调或onUnPublisher回调
```bash
//拉取音视频流
aliWebrtc.on('onPublisher',(publisher) =>{
//远程发布者userId
console.log(publisher.userId);
//远程发布名字
console.log(publisher.displayName);
//远程流内容，streamConfigs是数组格式
console.log(publisher.streamConfigs);
});

//用户取消推流
aliWebrtc.on('onUnPublisher',(publisher) =>{
//远程发布者userId
console.log(publisher.userId);
//远程发布名字
console.log(publisher.displayName);
});

```
这里，可以让用户自己选择是否开始打开视频（是否推流），然后拉取别的用户的音视频流，你需要根据用户数量来设置展示视频的view，然后在拉取视频流的回调里面将回调里面的内容放在view里面展示，就可以了，然后在“onUnPublisher”的回调里面，获取到那个用户取消推流了，就把那个用户取消掉就好了,这里说下，进入频道传的那个“userid”就是在这里用的，最好用唯一的，不然会出现奇观的问题。

5.订阅或取消订阅远程流

```bash
aliWebrtc.subscribe(userId).then((userId)=>{
aliWebrtc.setDisplayRemoteVideo(
userId,       // 用户ID
video,        // html中用于显示stream对象的video元素
1             // 1表示摄像头流（大流和小流），2表示屏幕分享流
)
},(error)=>{
console.log(error.message);
});
//取消订阅
aliWebrtc.unSubscribe(userId).then(() => {
},(error)=>{
console.log(error.message);
});
```
可以在这个里面拿到数据后，用"setDisplayRemoteVideo"指定播放view，开始播放


6.离开频道

```bash
aliWebrtc.leaveChannel().then(()=>{
} ,(error)=>{
console.log(error.message);
});
```

7.其他
打开/关闭摄像头
```bash
//关闭摄像头
this.isOpenVieo = false;

if (this.isScreen == true) {
aliWebrtc.configLocalScreenPublish = false;
} else {
aliWebrtc.configLocalAudioPublish = false;
aliWebrtc.configLocalCameraPublish = false;
}


aliWebrtc.unPublish().then(() => {}, (error) => {
//                    console.log(error.message);
});
//打开摄像头
this.isOpenVieo = true;


if (this.isScreen == true) {
aliWebrtc.configLocalScreenPublish = true;
} else {
aliWebrtc.configLocalAudioPublish = true;
aliWebrtc.configLocalCameraPublish = true;
}



aliWebrtc.publish().then(() => {}, (error) => {
//                    console.log(error.message);
});
```

打开/关闭 共享桌面
```bash
if (this.isScreen) {
//关闭共享
this.isScreen = false;
aliWebrtc.configLocalAudioPublish = true;
aliWebrtc.configLocalCameraPublish = true;
aliWebrtc.configLocalScreenPublish = false;
} else {
//打开共享
this.isScreen = true;
aliWebrtc.configLocalAudioPublish = false;
aliWebrtc.configLocalCameraPublish = false;
aliWebrtc.configLocalScreenPublish = true;
}

//热切换
aliWebrtc.publish().then((res) => {
setTimeout(() => {
//                    console.log("发布流成功");
this.isOpenVieo = true;
}, 2000)

}, (error) => {

console.log("[推流失败]" + error.message, "danger");
});
```
错误处理
```bash
//error 处理
aliWebrtc.on("onError", (error) => {
//10010 屏幕共享未知错误
//10011 屏幕共享在选择页面取消选择 屏幕共享被禁止
//10012 屏幕共享在网页底部悬浮窗单击停止共享  屏幕共享已取消

if (error.errorCode == 10012 || error.errorCode == 10011) {
this.isScreen = true;
this.openScreen();
}

//console.log(error.errorCode);
});
```

到这里，基本上所有的功能都接入完了，最后，要注意的是，退出频道的时候会有个延迟，在其他用户哪里可能会白屏一会，暂时还没找到原因。
