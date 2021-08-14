---
title:  "iOS) Modal 의 Life Cycle"
categories:
- iOS

date:   2021-08-14  17:19:00 +0900
author_profile: false
---
# 📌 Modal presentation style

- iOS 13 부터 default 값이 `automatic` 으로 변경되었다. `automatic` 은 대부분의 뷰컨트롤러의 경우 `pagingSheet` 스타일에 매핑된다.

# 📌  Life Cycle

### pagingSheet

presenting view controller 가 뒤에 보이기 때문에 제거되지 않는다.

<img src ="https://user-images.githubusercontent.com/69136340/129439824-ae9fefa6-67ac-4071-b40d-e58b36e56be3.jpeg" width ="800">

### currentContext, fullScreen

presenting view controller 은 presentation 이 끝난 후 제거된다.

<img src ="https://user-images.githubusercontent.com/69136340/129439825-1979f789-f1d2-4606-a4cf-ca29aef9ad80.jpeg" width ="800">

### overCurrentContext, overFullScreen

presented view controller 의 아래의 뷰는 presentation 이 끝날 때 뷰 계층에서 제거되지 않는다. 따라서 불투명한 컨텐츠로 채우지 않으면 아래의 컨텐츠가 보인다.

<img src ="https://user-images.githubusercontent.com/69136340/129439826-5ed6144a-0df1-4105-9d2a-6cd58cb42b9c.jpeg" width ="800">

### 참조

[Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uimodalpresentationstyle)

[iOS) ViewController의 특징과 생명주기](https://o-o-wl.tistory.com/43)

[[iOS] iOS13 Modal Style 및 Life Cycle](https://jinnify.tistory.com/64)
