## 功能简介

AI 降噪插件可以与 [TRTC Web SDK](https://www.npmjs.com/package/trtc-js-sdk) 结合使用，降低通话中的噪声，减少环境音对通话的影响。

本文将介绍在如何在开发 TRTC 应用中使用 [RTCAIDenoiser](https://www.npmjs.com/package/rtc-ai-denoiser) 插件给通过中的本地流应用 AI 降噪的方法。[点击在线体验](https://web.sdk.qcloud.com/trtc/webrtc/demo/latest/ai-denoiser/index.html) 降噪 demo 。

## 前提条件

支持的浏览器：Chrome 66+, Edge 79+, Safari 14.1+, Firefox 76+。

为了更好的使用 AI 降噪，建议您使用最新版本的 Chrome 浏览器。

**注意：**

如果您麦克风采集的有背景音乐，`RTCAIDenoiser`可能会将其当做噪音进行消除。

## 实现流程

#### Step1. 安装集成 AI 降噪插件

```shell
npm install rtc-ai-denoiser@latest
```

`RTCAIDenoiser` 插件需要和 `TRTC` 在同一作用域引入。

```javascript
import TRTC from 'trtc-js-sdk';
import RTCAIDenoiser from 'rtc-ai-denoiser';
```

#### Step2. 安装集成 AI 降噪插件

动态加载文件依赖：AI 降噪插件依赖一些文件。为保证浏览器可以正常加载和运行这些文件，你需要完成以下步骤：

将 `node_modules/rtc-ai-denoiser/assets` 目录下的`denoiser-wasm.js`文件发布至 CDN 或者静态资源服务器中，并且处于同一个公共路径下。后续创建 `RTCAIDenoiser` 实例时，需要传入上述公共路径的 URL，插件会动态加载依赖文件。

> - 如果 assets 目录下文件的 Host URL 与网页应用的 Host URL 不一致，则需要开启访问文件域名的 CORS 策略。
> - 不能把 assets 目录文件放在 HTTP 服务下，因为在 HTTPS 域名下加载 HTTP 资源会被浏览器安全策略禁止。

#### Step3. 创建 AI 降噪插件

1. 参考文档 [开始集成音视频通话](https://web.sdk.qcloud.com/trtc/webrtc/doc/zh-cn/tutorial-11-basic-video-call.html) 实现一个基本的音视频通话流程

2. 初始化 AI 降噪插件

```javascript
// 创建实例，传入assets 目录下的文件所在的公共路径
const rtcAIDenoiser = new RTCAIDenoiser({ assetsPath: './assets' });
```

3. 创建denoiserProcessor 实例

```javascript
const processor = await rtcAIDenoiser.createProcessor({
  sdkAppId,
  userId,
  userSig
});
```

4. 处理需要发布的 localStream

```javascript
// 初始化流
const localStream = TRTC.createStream({ video: true, audio: true });
await localStream.initialize();
// 给 localStream 增加降噪处理
await processor.process(localStream);
// 发布流
await client.publish(localStream);
```

5. 控制插件的开启或关闭：调用 `enable` 方法和 `disable` 方法。

```javascript
if (processor.enabled) {
  await processor.disable();
} else {
  await processor.enable();
}
```

6. 转储降噪处理过程中的音频数据：调用 `startDump` 方法开始， `stopDump` 方法结束，并监听 `ondumpend` 回调，获得音视频数据。

```javascript
processor.on('ondumpend', ({ blob, name }) => {
  const url = window.URL.createObjectURL(blob);

  let anchor = document.createElement('a');
  anchor.href = url;
  anchor.download = `${name}-${Date.now()}.wav`;
  anchor.click();
  window.URL.revokeObjectURL(url);
  anchor.href = '';
});
```

## API 参考

### RTCAIDenoiser

```javascript
const rtcAIDenoiser = new RTCAIDenoiser({ assetsPath: './assets' })
```

#### isSupported()

判断当前环境是否支持 AI 降噪

```javascript
if (!rtcAIDenoiser.isSupported()) {
  console.log('Your browser is not supported RTCAIDenoiser');
}
```

#### createProcessor(params)

创建 denoiserProcessor 实例

```javascript
const denoiserProcessor = await rtcAIDenoiser.createProcessor({
  sdkAppId,
  userId,
  userSig
});
```

**Params:**

| Name     | Type     | Description                                                  |
| -------- | -------- | ------------------------------------------------------------ |
| sdkAppId | `number` | sdkAppId <br/>在 [实时音视频控制台](https://console.cloud.tencent.com/trtc) 单击 **应用管理** > **创建应用** 创建新应用之后，即可在 **应用信息** 中获取 sdkAppId 信息。 |
| userId   | `string` | 用户ID<br/>建议限制长度为32字节，只允许包含大小写英文字母(a-zA-Z)、数字(0-9)及下划线和连词符。 |
| userSig  | `string` | userSig 签名<br/>计算 userSig 的方式请参考 [UserSig 相关](https://cloud.tencent.com/document/product/647/17275)。 |

### Processor

#### process(LocalStream)

给本地流的音频增加降噪效果。

```javascript
await denoiserProcessor.process(localStream);
```

#### get enabled

当前是否开启了 AI 降噪。

```javascript
const enabled = denoiserProcessor.enabled
```

#### enable()

开启 AI 降噪功能。

```javascript
await denoiserProcessor.enable()
```

#### disable()

关闭 AI 降噪功能。

```javascript
await denoiserProcessor.disable()
```

#### startDump()

开始转储降噪处理过程中的音频数据。最长 30 秒。

```javascript
denoiserProcessor.startDump()
```

#### stopDump()

停止转储降噪处理过程中的音频数据。最长 30 秒。

```javascript
denoiserProcessor.stopDump()
```

#### on(event, handler)

监听 processor 对外的事件。

**例：**

监听 `ondumpend` 事件之后，转储音频数据，实例代码如下。

```javascript
denoiserProcessor.on('ondumpend', ({ blob, name }) => {
  const url = window.URL.createObjectURL(blob);
  let anchor = document.createElement('a');

  anchor.href = url;
  anchor.download = `${name}-${Date.now()}.wav`;
  anchor.click();
  window.URL.revokeObjectURL(url);
  anchor.href = '';
});
```

## ChangeLog

## Version 1.1.1 @2022.11.17

### Bug fix

修复了一些 bug 。

## Version 1.1.0 @2022.11.07

### Improvement

优化插件的错误提示信息。

## Version 1.0.0 @2022.10.19

发布 rtc-ai-denoiser 插件 1.0.0 版本。


