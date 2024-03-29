bookbook JS-SDK
===========================

当前版本: 0.3.4

## 引入SDK

* 页面中引入脚本

  ```js
  <script src="https://ptio.cn/web/public/js_sdk/bookbook_sdk_0.3.4.min.js"></script>
  <script>
    function onSDKReady(callback) {
      if (window.bookbookBridge) {
        callback && callback();
      } else {
        const afterReady = () => {
          callback && callback();
          document.removeEventListener('bookbookBridgeReady', afterReady, false);
        };
        document.addEventListener('bookbookBridgeReady', afterReady, false);
      }
    }

    onSDKReady(() => {
      window.bookbookBridge.config(
        appkey, // appkey 由步步阅读团队分配
        { // 配置项，可选
          actId: 123 // 活动id，由第三方自行维护
        }
      );
      window.bookbookBridge.ready(bb => {
        // 初始化完成
      });
    });
  </script>
  ```


## 配置和初始化

```js
bookbookBridge.config(appkey); // 使用 SDK 前，必须先 config，传入 appkey
bookbookBridge.ready(bb => {
  // 初始化完成
});
```


## 常量

* `bookbookBridge.isApp` bool，是否在步步阅读的 App 中
* `bookbookBridge.appVersion` string，步步阅读 App 版本号
* `bookbookBridge.asPhone` bool，是否为手机
* `bookbookBridge.asTablet` bool，是否为平板


## 公共方法

* `bookbookBridge.compareVersion` 用法：compareVersion(a, b)，return: -1 (a < b)，0 (a = b)，1 (a > b)
* `bookbookBridge.lessThan` 用法：lessThan(targetVersion, currentVersion = appVersion)
* `bookbookBridge.notLessThan` 用法：notLessThan(targetVersion, currentVersion = appVersion)


## ready 方法返回的实例的方法

* `bb.openMiniProgram` 唤起小程序

  ```js
  bookbookBridge.ready(bb => {
    bb.openMiniProgram({
      platform: 'weixin',  // 平台，如果对应不上平台，返回错误
      userName: 'xxx',  // 小程序原始id
      path: '/pages/index',  // 小程序路径
      env: 'preview',  // dev:开发版，preview:体验版，release:正式版
      success: () => {
        // 成功回调
      }, 
      fail: () => {
        // 失败回调
      },
      complete: () => {
        // 无论成功、失败都回调
      }
    });
  });
  ```

* `bb.track` 发送 pingback 统计

  ```js
  bookbookBridge.ready(bb => {
    bb.track({
      action: 'pv'
    });
  });
  ```

* `bb.createAudioContext` 创建音频播放上下文

  ```js
  bookbookBridge.ready(bb => {
    const audio = bb.createAudioContext('main'); // 可用音轨：main、effect
    audio.src = 'https://ptio.cn/web/bookbook/p2/static/media/library-bgm.47a4fbd1.mp3'; // 音频资源地址
    audio.play(); // 播放音频
    audio.pause(); // 暂停播放
    audio.resume(); // 恢复播放
    audio.stop(); // 停止播放
    audio.destroy(); // 销毁播放器，会清除所有监听

    audio.onTimeUpdate(fn = (currentTime) => { // 监听进度事件
      // currentTime: 播放当前时间
    }) 
    audio.offTimeUpdate(fn) // 取消监听进度事件

    audio.onPlay(fn) // 监听播放事件
    audio.offPlay(fn) // 取消监听播放事件

    audio.onPause(fn) // 监听音频暂停事件
    audio.offPause(fn) // 取消暂停监听

    audio.onEnded(fn) // 监听播放完成
    audio.offEnded(fn) // 取消监听播放完成

    audio.onError(fn = (e) => { // 监听报错
      // e 的结构： {code: 20001, msg: '播放失败'}
    }) 
    audio.offError(fn) // 取消报错监听
  });
  ```

* `bb.canIUse` 能力检测

  ```js
  bookbookBridge.ready(bb => {
    bb.canIUse({
      ability: 'weixin', // weixin: 是否已安装微信
      success: () => {
        // 支持该能力
      },
      fail: (e) => {
        // 不支持该能力或状态未知
      },
      complete: (res) => {
        // res结构：{code: 0, msg: ''}
        // code：0:支持能力 1:不支持能力 -1:支持情况未知
      }
    });
  });
  ```

* `bb.share` 分享

  ```js
  bookbookBridge.ready(bb => {
    bb.share({
      shareType: 'web', // web:图文链接，image:图片，text:文本
      baseInfo: {
        title: '牛津阅读树免费看！',
        text: '我刚刚在步步阅读看了牛津阅读树！也邀你来体验～',
        imageUrl: 'https://ptio.cn/pv/books/113/2d45c42a-63a8f.jpg?imageMogr2/crop/385x436/gravity/center/quality/86/interlace/1',
        webUrl: 'https://m.buburead.com',
      },
      success: () => {
        // 成功回调
      },
      fail: () => {
        // 失败回调
      },
      complete: () => {
        // 无论成功、失败都回调
      }
    });
  });
  ```

* `bb.downloadApp` 通过应用商店下载 App

  ```js
  bookbookBridge.ready(bb => {
    bb.downloadApp({
      data: {
        // 安卓：如果 markets 中有 channel 对应的应用市场配置，优先跳应用市场，没有配置或跳转失败使用 download 中的下载链接兜底
        markets: [  // 应用市场配置
          {
            channel: 'string',  // App 发行渠道，参考步步阅读《App 发布渠道》
            appPkg: 'string',  // 应用包名
            marketPkg: 'string',  // 市场包名
          },
          ...
        ],
        download: {  // 下载兜底，必须配置
          url: 'string',  // apk 文件链接（域名有白名单限制）
          appName: 'string',  // 应用名称
          appIcon: 'string',  // 应用图标
        },
        showConfirm: bool,  // 跳市场/下载前是否弹窗确认

        // iOS：弹出应用市场下载弹窗
        iOSAppId: 'string'  // 应用商店 iTunesItemIdentifier，数字格式
      },
      success: () => {
        // 已经打开应用市场或者已经开始下载
      },
      fail: (e) => {
        // e 的结构：{code: 1, msg: ''}
        // code:
        //   -1: App 版本不支持
        //   1: 弹窗点击了取消
        //   2: 无法开启下载
      },
      complete: (res) => {
        // res 的结构：{code: 0, msg: ''} 
        // code 和 fail 中的一致
      }
    });
  });
  ```

* `bb.openUri` 使用系统默认方式打开指定 URI

  ```js
  bookbookBridge.ready(bb => {
    bb.openUri({
      uri: 'string',  // URI 有白名单限制
      success: () => {
        // 成功回调
      }, 
      fail: () => {
        // 失败回调
      },
      complete: () => {
        // 无论成功、失败都回调
      }
    });
  });
  ```

* `bb.closeWebview` 关闭当前 WebView

  ```js
  bookbookBridge.ready(bb => {
    bb.closeWebview({
      success: () => {
        // 成功回调
      }, 
      fail: () => {
        // 失败回调
      },
      complete: () => {
        // 无论成功、失败都回调
      }
    });
  });
  ```
