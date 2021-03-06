# 蹭网助手
[![](https://badge.juejin.im/entry/5bc32a2f6fb9a05d09657aeb/likes.svg?style=flat-square)](https://juejin.im/entry/5bc32a2f6fb9a05d09657aeb/detail)

# 运行
> 当然里面很多事根据个人配置的,所以代码仅供学习

```
$ npm install

$ node index.js
```

今天下了个wifi共享精灵, 连上了别人的网, 从此搞别无网生活
于是顺手访问了下 192.168.0.1 
居然不用输管理员密码就进入后台了...
![image.png](https://upload-images.jianshu.io/upload_images/2245742-e4f0b90e2eb87ff0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

于是和朋友聊天, 像搞个实时监控, 看看如果对方连上网, 我就低调点, 当然都是搞的玩的

# 需要相关技术

 - NodeJS
 - axios --- 用于请求路由器接口
 - wunderbar --- 命令行绘制图表(装x
 - snoretoast --- 系统通知程序( 我看vue-cli用的就是这个,不然还不知道

# 总体原理
 - 通过请求api获取路由器数据
 - 通过判断联网设备,来判断是否有出了自己之外的人连上, 通过判断前后的长度差,来判断是否有新设备加入

# 代码
```javascript
const axios = require('axios') // http请求库
const wunderbar = require('@gribnoysup/wunderbar') // 命令行图表库
const WindowsToaster = require('node-notifier').WindowsToaster // 弹框通知库
let notifier = new WindowsToaster({ // 配置SnoreToast
  withFallback: false,
  customPath: './SnoreToast.exe'
});

let my = '' // 自己设备的ip
let num = 0 // 当前联入设备总数
let allDown = '' // 总下载
let allUp = '' // 总上传
let downSpeed = [] // 下载速度数组
let deviceLength = 0 // 除去自己的设备数

function getData() {
  // 请求设备网络使用数据
  axios.get('http://192.168.0.1/goform/getQos?random=0.2296175026847045')
    .then(res => {
      let result = res.data
      my = result.localhost
      downSpeed = []
      upSpeed = []
      result.qosList.map(v => {
        // 将数据遍历到数组中, 图表需要的数据格式 {value: 你的数据, label: 数据的标题, color: 数据的颜色} 后两项不是必须的
        downSpeed.push({
          value: v.qosListDownSpeed,
          label: v.qosListHostname
        })
      })
    })
    .catch(err => {
      console.log(error)
    })
  // 请求设备联网总数据
  axios.get('http://192.168.0.1/goform/getStatus?0.3297709679659049')
    .then(res => {
      let result = res.data
      allDown = result.statusDownSpeed
      allUp = res.data.statusUpSpeed
      num = res.data.statusOnlineNumber
    })
    .catch(err => {
      console.log(err)
    })
}

// 绘图方法
const printData = () => {
  const { chart, legend, scale, __raw } = wunderbar(downSpeed, {
    min: 0,
    length: 42,
    format: (n) => `${n}KB/s`
  });

  // 清空命令行
  process.stdout.write('\n');
  process.stdout.write('\033[0f');
  process.stdout.write('\033[2J');

  // 绘制图表
  console.log()
  console.log('==========================================');
  console.log(`当前时间:${new Date().toLocaleTimeString()}`)
  console.log(`本机当前IP: ${my}\t 当前联网设备:${num}\t 总下载速度:${allDown}KB/s\t 总上传速度:${allUp}KB/s\t`);
  console.log('==========================================');
  console.log(chart);
  console.log();
  console.log(legend);
  console.log();
};

// 定时刷新
setInterval(async () => {
  await getData()
  // 判断是否有新设备加入
  if (num - 2 > deviceLength) {
    deviceLength = num - 2
    // 桌面通知
    notifier.notify({
      title: '有人连进来了',
      message: '低调点!低调点!低调点!',
      icon: './xu.jpg'
    },
      function (error, response) {
        console.log(response)
      });
  }
  await printData()
}, 5000)
```

# 效果
![image.png](https://upload-images.jianshu.io/upload_images/2245742-2e2e3798ee9fcbc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![讲真的,我喜欢这种质感的命令行](https://upload-images.jianshu.io/upload_images/2245742-c4e201c3751b939d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 参考资料

 - 命令行图表: **[wunderbar](https://github.com/gribnoysup/wunderbar)**
 - 数字格式化库: http://numeraljs.com/#format
 - 系统弹框工具: **[snoretoast](https://github.com/KDE/snoretoast)**
 - NodeJS操控系统弹框库: **[node-notifier](https://github.com/mikaelbr/node-notifier)**
 - HTTP请求工具: **[axios](https://github.com/axios/axios)**
