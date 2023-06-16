<p align="center">
  <a href="https://intl.cloud.tencent.com/products/trtc">
    <img width="200" src="https://web.sdk.qcloud.com/trtc/webrtc/assets/trtc-logo.png">
  </a>
</p>

<h1 align="center">rtc-ai-denoiser</h1>

<div align="center">

The RTCAIDenoiser plugin can be used in conjunction with the [TRTC Web SDK](https://www.npmjs.com/package/trtc-js-sdk) to reduce noise during calls and reduce the impact of ambient sound on calls.

[![NPM version](https://img.shields.io/npm/v/rtc-ai-denoiser)](https://www.npmjs.com/package/rtc-ai-denoiser) [![NPM downloads](https://img.shields.io/npm/dw/rtc-ai-denoiser)](https://www.npmjs.com/package/rtc-ai-denoiser)  [![Documents](https://img.shields.io/badge/-Documents-blue)](https://github.com/FTTC/rtc-ai-denoiser)

</div>

 English | [简体中文](https://github.com/FTTC/rtc-ai-denoiser/blob/master/README.zh-cn.md)

## Overview

The RTCAIDenoiser plugin can be used in conjunction with the [TRTC Web SDK](https://www.npmjs.com/package/trtc-js-sdk) to reduce noise during calls and reduce the impact of ambient sound on calls.

This article describes how to use the [RTCAIDenoiser](https://www.npmjs.com/package/rtc-ai-denoiser) plugin in developing TRTC applications to apply AI noise reduction to streams when publish. [Click to experience](https://web.sdk.qcloud.com/trtc/webrtc/demo/api-sample/improve-ai-denoiser.html) the noise reduction demo online.

## Prerequisites

From 1 April 2023, TRTC [monthly subscription](https://www.tencentcloud.com/document/product/647/53564) Premium and higher is required to use the AI noise reduction function.

Supported browsers: Chrome 66+, Edge 79+, Safari 14.1+, Firefox 76+.

For better use of AI noise reduction, it is recommended that you use the latest version of Chrome.

**Notes:**

If there is background music captured by your microphone, `RTCAIDenoiser` may eliminate it as noise.

## Feature Description

#### Step1. Install RTCAIDenoiser

```shell
npm install rtc-ai-denoiser@latest
```

The `RTCAIDenoiser` plugin needs to be installed in the same scope as `TRTC`.

```javascript
import TRTC from 'trtc-js-sdk';
import RTCAIDenoiser from 'rtc-ai-denoiser';
```

#### Step2. Integrated RTCAIDenoiser

Dynamically loading file dependencies: The RTCAIDenoiser plugin relies on a number of files. To ensure that your browser can load and run these files properly, you need to complete the following steps.

Publish the `denoiser-wasm.js` file from the `node_modules/rtc-ai-denoiser/assets` directory to a CDN
or static resource server and under the same public path. When creating `RTCAIDenoiser` instances later, you need to pass in the URL of the above public path and the plugin will load the dependency files dynamically.

> - If the Host URL of the file in the assets directory does not match the Host URL of the web application, you need to enable the CORS policy for accessing the file domain.
> - You cannot place assets directory files under an HTTP service, as loading HTTP resources under an HTTPS domain is prohibited by browser security policies.
> -
#### Step3. Init RTCAIDenoiser

1. Reference [Quick Start Call](https://web.sdk.qcloud.com/trtc/webrtc/doc/en/tutorial-11-basic-video-call.html) to implement a basic audio/video call process.

2. init RTCAIDenoiser

```javascript
// Create an instance, passing in the public path where the files in the assets directory are located
const rtcAIDenoiser = new RTCAIDenoiser({ assetsPath: './assets' });
```

3. create denoiserProcessor instance

```javascript
const processor = await rtcAIDenoiser.createProcessor({
  sdkAppId,
  userId,
  userSig
});
```

4. handle localStreams that need to be published.

```javascript
// init stream
const localStream = TRTC.createStream({ video: true, audio: true });
await localStream.initialize();

// adding noise suppression to localStream
await processor.process(localStream);
// publish
await client.publish(localStream);
```

5. Control whether the plugin is turned on or off: call the `enable` method and the `disable` method.

```javascript
if (processor.enabled) {
  await processor.disable();
} else {
  await processor.enable();
}
```

6. Dump the audio data during the noise suppression process: call the `startDump` method to start and the `stopDump` method to end, and listen to the `ondumpend` callback to get the audio and video data.

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

## API Description

### RTCAIDenoiser

```javascript
const rtcAIDenoiser = new RTCAIDenoiser({ assetsPath: './assets' })
```

#### isSupported()

Determine whether the current environment supports RTCAIDenoiser.

```javascript
if (!rtcAIDenoiser.isSupported()) {
  console.log('Your browser is not supported RTCAIDenoiser');
}
```

#### createProcessor(params)

create denoiserProcessor instance.

```javascript
const denoiserProcessor = await rtcAIDenoiser.createProcessor({
  sdkAppId,
  userId,
  userSig
});
```

**Params:**

| Name     | Type     | Description                        |
|----------|----------|------------------------------------|
| sdkAppId | `number` | sdkAppId that your application use |
| userId   | `string` | userId that current client use     |
| userSig  | `string` | userSig signature                  |

### Processor

#### process(LocalStream)

Add noise suppression to the audio of the local stream.

```javascript
await denoiserProcessor.process(localStream);
```

#### get enabled

Whether AI noise suppression is currently turned on.

```javascript
const enabled = denoiserProcessor.enabled
```

#### enable()

Turn on AI noise reduction.

```javascript
await denoiserProcessor.enable()
```

#### disable()

Turn off AI noise reduction.

```javascript
await denoiserProcessor.disable()
```

#### startDump()

Start dumping audio data from the noise reduction process. Up to 30 seconds.

```javascript
denoiserProcessor.startDump()
```

#### stopDump()

Stop dumping audio data from the noise reduction process. Up to 30 seconds.

```javascript
denoiserProcessor.stopDump()
```

#### on(event, handler)

add event listener to events from the processor.

**eg:**

After listening to the `ondumpend` event, the audio data is dumped and the example code is as follows.

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

#### off(event, handler)
remove event listener to events from the processor.

#### destroy()
Destroy the processor to release resources and end the processor life cycle.


## ChangeLog
## Version 1.1.4 @2023.6.16
### Improvement
Optimize the error message.
## Version 1.1.3 @2023.2.10
### Improvement
Optimize the destruction logic and reduce memory usage.
## Version 1.1.2 @2022.12.29
### Improvement
Improve the noise reduction effect.
## Version 1.1.1 @2022.12.01
### Improvement
Optimize the authentication strategy.
## Version 1.1.0 @2022.11.07
### Improvement
Optimize the error message.
## Version 1.0.0 @2022.10.19
Released version 1.0.0.


