---
title:  "iOS) Modal ์ Life Cycle"
categories:
- iOS

date:   2021-08-14  17:19:00 +0900
author_profile: false
---
# ๐ Modal presentation style

- iOS 13 ๋ถํฐ default ๊ฐ์ด `automatic` ์ผ๋ก ๋ณ๊ฒฝ๋์๋ค. `automatic` ์ ๋๋ถ๋ถ์ ๋ทฐ์ปจํธ๋กค๋ฌ์ ๊ฒฝ์ฐ `pagingSheet` ์คํ์ผ์ ๋งคํ๋๋ค.

# ๐  Life Cycle

### pagingSheet

presenting view controller ๊ฐ ๋ค์ ๋ณด์ด๊ธฐ ๋๋ฌธ์ ์ ๊ฑฐ๋์ง ์๋๋ค.

<img src ="https://user-images.githubusercontent.com/69136340/129439824-ae9fefa6-67ac-4071-b40d-e58b36e56be3.jpeg" width ="800">

### currentContext, fullScreen

presenting view controller ์ presentation ์ด ๋๋ ํ ์ ๊ฑฐ๋๋ค.

<img src ="https://user-images.githubusercontent.com/69136340/129439825-1979f789-f1d2-4606-a4cf-ca29aef9ad80.jpeg" width ="800">

### overCurrentContext, overFullScreen

presented view controller ์ ์๋์ ๋ทฐ๋ presentation ์ด ๋๋  ๋ ๋ทฐ ๊ณ์ธต์์ ์ ๊ฑฐ๋์ง ์๋๋ค. ๋ฐ๋ผ์ ๋ถํฌ๋ชํ ์ปจํ์ธ ๋ก ์ฑ์ฐ์ง ์์ผ๋ฉด ์๋์ ์ปจํ์ธ ๊ฐ ๋ณด์ธ๋ค.

<img src ="https://user-images.githubusercontent.com/69136340/129439826-5ed6144a-0df1-4105-9d2a-6cd58cb42b9c.jpeg" width ="800">

### ์ฐธ์กฐ

[Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uimodalpresentationstyle)

[iOS) ViewController์ ํน์ง๊ณผ ์๋ช์ฃผ๊ธฐ](https://o-o-wl.tistory.com/43)

[[iOS] iOS13 Modal Style ๋ฐ Life Cycle](https://jinnify.tistory.com/64)
