---
title: MediaStreamTrack.contentHint
tags: 
  - WebRTC
  - MediaStreamTrack
  - contentHint
categories: WebRTC
date: 2021-09-24 14:50:24
---


```IDL
partial interface MediaStreamTrack {
  attribute DOMString contentHint;
};
```

track内容类型提示，用于提示消费端应该如何使用这个track
默认为""，表示没有任何提示


## 音频
| 值                   | 说明                                                                                                   |
| -------------------- | ------------------------------------------------------------------------------------------------------ |
| ""                   | 没有提供任何提示，应该程序推测                                                                         |
| "speech"             | 包含语音数据。应开启噪声抑制或提高传入信号的可理解性                                                   |
| "speech-recognition" | 包含用于机器语音识别目的的数据。提高传入信号的可理解性以进行转录并关闭用于人类语音的音频处理组件。     |
| "music"              | 曲目应被视为包含音乐数据。通常，这可能意味着调整或关闭用于处理语音数据的音频处理组件，以防止音频失真。 |

## 视频

| 值       | 说明                                                           |
| -------- | -------------------------------------------------------------- |
| ""       | 没有提供任何提示，应该程序推测                                 |
| "motion" | 流畅优先，帧率优先。适用于视频通话，电影，游戏等场景           |
| "detail" | 分辨率优先。适用于具有文本内容、绘画或线条艺术的演示文稿或网页 |
| "text"   | 视频细节特别重要，并且经常出现明显的锐利边缘和颜色一致的区域，并且可以利用优化文本渲染的编码器工具。适用于文本内容，演示文稿，网页。|

## 例子
```javascript
const track = stream.getVideoTracks()[0];
if ('contentHint' in track) {
    track.contentHint = hint;
    if (track.contentHint !== hint) {
      console.log('Invalid video track contentHint: \'' + hint + '\'');
    }
} else {
    console.log('MediaStreamTrack contentHint attribute not supported');
}
```
