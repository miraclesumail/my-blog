---
title: '前端与app和rn通信'
description: 'app'
pubDate: 'Jan 22 2026'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

<style>
    .small-code pre {
    font-size: 13px !important;  /* 强制覆盖默认大小 */
    line-height: 1.5 !important; /* 调整行高，让排版更紧凑 */
    padding: 1rem !important;    /* 缩小边距（可选） */
  }

  p {
    margin: 0 !important;
  }

  li {
     font-size: 16px !important;  /* 强制覆盖默认大小 */
    line-height: 1.4 !important; /* 调整行高，让排版更紧凑 */
    margin-bottom: 8px;
  }
</style>

1. 🤖 [rn和web通信](#rn)
2. ⚙️ [rn里面oc添加原生模块](#module)
3. 🔋 [安卓端kotin添加相同模块](#android)
4. 🤸 [swift添加可调用模块](#swift)

#### <a name="rn">🤖 rn和web通信</a>

<p>react native中的code</p>

<div class="small-code">

```js
// 定义通信的统一数据结构
interface BridgeMessage {
  type: string;
  payload?: Record<string, any>;
}

const BridgeWebView = () => {
    // RN 端发送消息给 Web (不再使用 injectJavaScript)
    const sendMessageToWeb = useCallback((type: string, payload?: any) => {
        if (!webViewRef.current) return;

        // 1. 序列化数据
        const message = { type, payload };
        const jsonString = JSON.stringify(message);

        // 2. 🌟 直接使用 postMessage API，纯数据传输！
        webViewRef.current.postMessage(jsonString);
    }

    /**
   * 🌟 核心：RN 接收来自 Web 的消息
   */
    const handleMessageFromWeb = (event: WebViewMessageEvent) => {
        try {
        // 1. 获取 Web 传来的字符串并解析
        const dataStr = event.nativeEvent.data;
        const message: BridgeMessage = JSON.parse(dataStr);

        console.log('RN 收到 Web 消息:', message);

        // 2. 根据不同的 type 处理业务逻辑
        switch (message.type) {
            case 'WEB_READY':
            // 接收到 Web 初始化完成的信号
            setIsWebReady(true);
            break;
            
            case 'SHOW_ALERT':
            Alert.alert('来自 Web 的呼唤', message.payload?.text);
            break;
            
            case 'GET_USER_INFO':
            // 模拟 Web 请求获取用户信息，RN 获取后传回给 Web
            const userInfo = { name: '张三', token: 'abc123456' };
            sendMessageToWeb('SET_USER_INFO', userInfo);
            break;

            default:
            console.warn('未知的消息类型:', message.type);
        }
        } catch (error) {
        console.error('解析 Web 消息失败:', error);
        }
        };
    }

    return (
    <View style={styles.container}>
      {/* 顶部测试按钮：RN 侧主动发消息给 Web */}
      <View style={styles.header}>
        <Button 
          title="RN 给 Web 发消息" 
          disabled={!isWebReady} // 必须等 Web 准备好再发
          onPress={() => sendMessageToWeb('CHANGE_COLOR', { color: '#ffeb3b' })} 
        />
      </View>

      <WebView
        ref={webViewRef}
        // 这里用本地 HTML 字符串演示，实际项目中通常是 uri: 'https://你的网页地址'
        source={{ html: webHtmlContent }}
        style={styles.webview}
        onMessage={handleMessageFromWeb}
        javaScriptEnabled={true}
        // 可选：在 Web 加载最开始就注入一段代码，防止 window.ReactNativeWebView 还没准备好
        injectedJavaScriptBeforeContentLoaded={`
          window.isNativeApp = true;
        `}
        startInLoadingState={true}
        renderLoading={() => (
          <ActivityIndicator style={styles.loader} size="large" />
        )}
      />
    </View>
```

</div>

<p>web端的code</p>

<div class="small-code">

```js
    /**
     * 🌟 核心 1：Web 向 RN 发送消息
     * @param type 消息类型
     * @param payload 携带的数据
     */
    function sendToRN(type, payload = {}) {
      if (window.ReactNativeWebView) {
        // 必须序列化成字符串才能传给 RN
        const message = JSON.stringify({ type, payload });
        window.ReactNativeWebView.postMessage(message);
        logMessage(`已发送 [${type}] 给 RN`);
      } else {
        logMessage('错误: 不在 React Native 环境中');
      }
    }

    window.addEventListener('message', (event) => {
      // event.detail 就是 RN 那边传过来的 JSON 对象
      const message = event.detail; 

      // 根据不同的 type 处理业务逻辑 带上前缀更好
      switch (message.type) {
        case 'SET_USER_INFO':
          logMessage(`✨ 解析得到用户名: ${message.payload.name}`);
          break;
        case 'CHANGE_COLOR':
          document.body.style.backgroundColor = message.payload.color;
          break;
      }
      logMessage(`收到 RN 消息: ${JSON.stringify(message)}`);

    });

    // 页面逻辑：点击按钮触发
    function sendAlertToRN() {
      sendToRN('SHOW_ALERT', { text: '你好 RN，我是 Web 端！' });
    }

    function requestUserInfo() {
      sendToRN('GET_USER_INFO');
    }

    // 🌟 核心 3 (最佳实践)：Web 页面加载完毕后，主动告诉 RN "我准备好了"
    // 防止 RN 在 Web JS 还没解析完时就发消息，导致消息丢失
    window.onload = () => {
      sendToRN('WEB_READY');
    };
```

</div>

#### <a name="module">🤖 rn里面oc添加原生模块</a>

<p>下面展示了如何在oc中添加一个名为CustomModule的模块</p>
<div class="small-code">

```objc
// CustomModule.h
#import <React/RCTBridgeModule.h>

// 定义一个继承自 NSObject 的类，并遵循 RCTBridgeModule 协议
@interface CustomModule : NSObject <RCTBridgeModule>
@end

```



</div>

<p>.m文件展示了该模块有哪些方法可被js调用</p>

<div class="small-code">

```objc
// CustomModule.m
#import "CustomModule.h"
#import <React/RCTLog.h> // 用于在控制台打印日志

@implementation CustomModule

RCT_EXPORT_MODULE();

// 2. 导出一个普通的单向方法
RCT_EXPORT_METHOD(doSomething:(NSString *)name location:(NSString *)location)
{
  // 这里的逻辑运行在原生端
  RCTLogInfo(@"原生端被调用了！收到的名字是: %@，地点是: %@", name, location);
}

// 3. 导出一个带有回调函数 (Callback) 的方法
RCT_EXPORT_METHOD(doSomethingWithCallback:(NSString *)name callback:(RCTResponseSenderBlock)callback)
{
  // 模拟一些处理过程
  NSString *result = [NSString stringWithFormat:@"成功处理了: %@", name];
  
  // 回调给 JS 端。通常数组第一个参数是 error 对象，第二个是结果 (类似 Node.js 的 error-first callback)
  callback(@[[NSNull null], result]);
}

// 2. 导出方法：获取设备名称，并通过 Promise 返回给 JS
RCT_EXPORT_METHOD(getDeviceName:(RCTPromiseResolveBlock)resolve
                       rejecter:(RCTPromiseRejectBlock)reject)
{
  @try {
    // 调用 iOS 原生 API 获取设备名 (例如 "iPhone 15 Pro")
    NSString *deviceName = [[UIDevice currentDevice] name];
    
    // 成功时，调用 resolve 将字符串传回 JS
    resolve(deviceName);
  } @catch (NSException *exception) {
    // 失败时，抛出错误给 JS 端的 catch
    reject(@"DEVICE_ERROR", @"获取设备名失败", nil);
  }
}

// 4. 导出一个支持 Promise (async/await) 的方法【推荐】
RCT_EXPORT_METHOD(doSomethingWithPromise:(NSString *)name
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
  if ([name isEqualToString:@"error"]) {
    // 模拟发生错误
    NSError *error = [NSError errorWithDomain:@"com.mycompany.error" code:500 userInfo:nil];
    reject(@"no_events", @"发生了一个原生错误", error);
  } else {
    // 成功返回数据
    NSString *result = [NSString stringWithFormat:@"Promise成功返回: %@", name];
    resolve(result);
  }
}

@end


```
</div>

<p>如何在rn里面调用ios定义的模块</p>


<div class="small-code">

```js
import {nativeModules} from 'react-native'

const {CustomModule} = nativeModules

const handleGetDeviceName = async () => {
    try {
      // 1. 调用带 Promise 返回值的原生方法
      const name = await CustomModule.getDeviceName();
      setDeviceName(name);
      console.log(name, '设备名 jsjsjweb端');

      // 2. 调用普通方法传参给原生
      // RNDeviceInfo.printMessage('你好，iOS 原生，我是 JS！');
    } catch (error) {
      console.error(error);
    }
  };

  // 1. 调用普通方法
  const handlePress = () => {
    console.log('才能参谋参谋');
    // 这行代码会触发 iOS 端的 RCTLogInfo 打印日志
    CustomModule.doSomething('张三', '北京');
  };

  // 2. 调用带有 Callback 的方法
  const handleCallback = () => {
    CustomModule.doSomethingWithCallback(
      '李四',
      (error: Error, result: string) => {
        if (error) {
          console.error(error);
        } else {
          Alert.alert('Callback 结果', result);
        }
      },
    );
  };
```

</div>


#### <a name="android">🤖 安卓端kotin添加相同模块</a>


<p>在MainApplication.kt下加入下面代码</p>

<div class="small-code">

```js
// ⭐️ 在 RN 0.84 中，所有的配置和包注册都集中在这个 reactHost 里
  override val reactHost: ReactHost by lazy {
    getDefaultReactHost(
      context = this.applicationContext,
      // ⭐️ 核心修改点：在 PackageList 后面加上 .apply { add(你的包) }
      packageList = PackageList(this).packages.apply {
        // 将你之前写好的 CustomPackage 注册到这里
        add(CustomPackage())
      }
    )
  }
```

</div>

<p>自定义一个CustomModule.kt方法和上面的ios对应</p>

<div class="small-code">

```js

// 继承 ReactContextBaseJavaModule
class CustomSwiftModule(reactContext: ReactApplicationContext) : ReactContextBaseJavaModule(reactContext) {

    // 1. 定义模块名：这个名字就是 JS 端 NativeModules 后面点出来的名字
    override fun getName(): String {
        return "CustomModule"
    }

    // 2. 导出普通方法：加上 @ReactMethod 注解，RN 就会自动把它暴露给 JS
    @ReactMethod
    fun doSomething(name: String, location: String) {
        // Android 打印日志用 Log.d，可以在 Logcat 里看到
        Log.d("CustomSwiftModule", "Android 原生端被调用啦！收到名字：$name，地点：$location")
    }

    // 3. 导出 Promise 方法
    // 注意：在 Android 中，Resolve 和 Reject 被封装进了一个统一的 Promise 对象里，且永远作为最后一个参数传进来
    @ReactMethod
    fun doSomethingWithCallback(name: String, promise: Promise) {
        if (name.isEmpty()) {
            // 失败：reject 需要传 code 和 message，可选传 Throwable(Exception)
            promise.reject("INVALID_NAME", "你传的名字是空的", Exception("Name is empty"))
        } else {
            // 成功：直接 resolve 数据
            val result = "Hello $name, 这是来自 Kotlin 的问候！"
            promise.resolve(result)
        }
    }

    @ReactMethod
    fun getDeviceName(successCallback: Promise, failCallback: Promise) {
        try {
           

            // 3. 返回结果
            promise.resolve(maxVal)
        } catch (e: Exception) {
            promise.reject("error", "计算失败: ${e.message}")
        }
    }
}
```

</div>


<p>CustomPackage里面把CustomModule添加进去</p>

<div class="small-code">

```js
class CustomPackage : ReactPackage {

    // 注册没有 UI 视图的原生模块（就是我们写的这个）
    override fun createNativeModules(reactContext: ReactApplicationContext): List<NativeModule> {
        return listOf(CustomModule(reactContext))
    }

    // 注册自定义原生 UI 组件（我们目前没写 UI 组件，返回空列表即可）
    override fun createViewManagers(reactContext: ReactApplicationContext): List<ViewManager<*, *>> {
        return emptyList()
    }
}
```

</div>

#### <a name="swift">🤸 swift添加可调用模块</a>

<p>在AppDelegate的目录下添加CustomSwiftModule</p>

<div class="small-code">

```swift
// CustomSwiftModule.swift
import Foundation
import React    // <--- 加上这一行！！！

// 加上 @objc(模块名) 标签，让 Objective-C 能够识别这个类
@objc(CustomSwiftModule)
class CustomSwiftModule: NSObject {

    // 1. 写一个没有返回值的普通方法
    // 注意 @objc 里的名字要和后面暴露给 JS 的名字严格对应
    @objc(doSomething:location:)
    func doSomething(name: String, location: String) -> Void {
        // 这里的代码运行在原生 iOS 端
        print("Swift 原生模块被调用啦！收到名字：\(name)，地点：\(location)")
    }

    // 2. 写一个支持 Promise 的方法（推荐！JS 可以用 async/await）
    @objc(doSomethingWithPromise:resolver:rejecter:)
    func doSomethingWithPromise(name: String,
                                resolve: @escaping RCTPromiseResolveBlock,
                                reject: @escaping RCTPromiseRejectBlock) -> Void {
        
        if name.isEmpty {
            // 失败时的情况：创建一个原生 NSError
            let error = NSError(domain: "com.yourapp.error", code: 400, userInfo: nil)
            reject("INVALID_NAME", "你传的名字是空的", error)
        } else {
            // 成功时的情况：返回一段字符串给 JS 端
            let result = "Hello \(name), 这是来自 Swift 的问候！"
            resolve(result)
        }
    }
  
  @objc(findMax:resolver:rejecter:)
  func findMax(numbers: [Double], resolve: @escaping RCTPromiseResolveBlock, reject: @escaping RCTPromiseRejectBlock) -> Void {
    if numbers.isEmpty {
      let error = NSError(domain: "MaxFinder", code: 404, userInfo: [NSLocalizedDescriptionKey: "数组不能为空"])
            reject("no_data", "Array is empty", error)
            return
    }
    
    // 获取最大值
        if let maxVal = numbers.max() {
          resolve(maxVal) // 返回给 RN
        } else {
          reject("error", "计算失败", nil)
        }
  }

    // 3. 必须实现的方法：告诉 RN 这个模块是否需要在主线程初始化
    @objc
    static func requiresMainQueueSetup() -> Bool {
        return true // 如果你的模块要操作 UI 就返回 true，只做数据处理可以返回 false
    }
}


```
</div>

<p>在AppDelegate的目录下添加CustomSwiftModule.m</p>

<div class="small-code">

```objc
// CustomSwiftModule.m
#import <React/RCTBridgeModule.h>

// 1. 导出模块 (绑定刚刚写的 CustomSwiftModule 类)
@interface RCT_EXTERN_MODULE(CustomSwiftModule, NSObject)

// 2. 导出普通方法
RCT_EXTERN_METHOD(doSomething:(NSString *)name location:(NSString *)location)

// 3. 导出 Promise 方法
RCT_EXTERN_METHOD(doSomethingWithPromise:(NSString *)name
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)

// 导出方法名，注意这里的参数冒号要和 Swift 里的参数对应
RCT_EXTERN_METHOD(findMax:(NSArray *)numbers
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)

@end

```

</div>
