
最近打算做了个音频自定义播放器，录音跟音频播放的功能点踩各种莫名其妙的坑，社区也有不少人在提问，特写此文祭天<br>
基于[WEPY](https://tencent.github.io/wepy/)开发。<br>
### 录音功能 

相关api [wx.getRecorderManager](https://developers.weixin.qq.com/miniprogram/dev/api/getRecorderManager.html)

#### WXML 模板 <br>
... 除了低版本样式兼容，没啥坑。<br>
#### JS 交互逻辑

#### 录音坑之一
部分手机无法上传录音文件。<br>
原因：服务端上传文件大小限制<br>
解决：sampleRate,encodeBitRate两者有对应要求，具体看文档，尽量调质中低音质,公司财大气粗忽略，顶配服务器跑起来。<br>
音质越高文件文件越大，相同参数ios系统的录音文件更大。
```js
const recorderManager = wepy.getRecorderManager()
const options = {
    duration: 600000, // 是的没看错 10分钟...
    sampleRate: 8000,
    encodeBitRate: 20000,
    ...
}
```

#### 录音坑之二
部分用户录音之后无法试听。<br>
原因：用户拒绝授权，录音代码无做校验(不严谨哈)；苹果手机用户开了静音功能(就是左上角那个开关，这真无力吐槽吖);内存不足，开启蓝牙；
解决：录音开始前先查看麦克风授权情况，无授权不录音。代码如下<br>设置播放实例```obeyMuteSwitch```属性（暂只支持ios）..<br>第三，我也不知道，一般建议重启。。。就是这么美妙。
```js
...
methods = {
    // 开始录音
    recording() {
      wepy.getSetting().then((res) => {
        if (!res.authSetting['scope.record']) {
          wepy.authorize({scope: 'scope.record'}).then(() => {
            recorderManager.start(options)
            this.startTimer()
            this.$apply()
          }, (e) => {
            wepy.openSetting()
          })
        } else {
          recorderManager.start(options)
          this.startTimer()
          this.$apply()
        }
      })
    },
}
...
```
#### 录音坑之三
录音时长不准(该参数列表需要)<br>
原因：手机卡顿，延迟导致部分用户录音跟计时器不同步(你永远不知道用户用的是啥手机，我只能说蓝绿厂大坑)<br>
解决：调用```onStop```方法回调录音时长。按理来说最长也就600s，但是后台看到有段录音时长是10000多s，目前还不知道啥原因，求解。
```js
 onLoad () {
    recorderManager.onStop(({tempFilePath, duration})=>{
        //do something
        this.duration = parseInt(duration / 1000)
    }) 
 }
 // 计时器
  startTimer (){
      // do something
  }
```
#### 录音坑之四
录音不完整<br>
原因：录音过程中自动锁屏功能，来电等外部原因导致录音中断。<br>
解决：提醒用户保持小程序运行状态；按住录音。不过我们10分钟，我怕用户手抽筋;wx.setKeepScreenOn()接口。

### 音频播放功能 
相关api [wx.createInnerAudioContext](https://developers.weixin.qq.com/miniprogram/dev/api/createInnerAudioContext.html)

#### JS 交互逻辑
audio组件不好用吗？自定义好看多啦~<br>
播放的坑相对较少一点，建议页面只注册一个播放器，动态修改音源，相关事件只在页面onLoad注册<br>
*动态修改音源，无法获取当前音源duration,异步!？
```js
const innerAudioContext = wepy.createInnerAudioContext()
...
onLoad ({classId, date}) {
    innerAudioContext.onEnded(() => {
      //do something
    })
    innerAudioContext.onPlay(() => {
       
    })
    innerAudioContext.onTimeUpdate(()=>{
        //update  Progress bar
    })
    ...
}
// 计时器
startTimer (){
    // do something
}
```
#### 播放坑
听不到(好想除了听不到也没啥坑了吧)，canpaly状态需要主动触发<br>
原因：同上，ios用户开了静音模式自己；网络原因，进度条同定时器更新，文件没有缓存至可播放状态，导致进度条播放不同步；https,https,https..部分ios无法播放https协议的资源。
解决：进度条再```onTimeUpdate```方法中更新。采用http的资源。

### 后续
10分钟的录音，用户试听的时候缺少拖放功能，不方便。待完善。。。<br>
看下热度。开源自定义播放器。

