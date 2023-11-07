---
title:  "iOS) DT_TOOLCHAIN_DIR cannot be used to evaluate LIBRARY_SEARCH_PATHS, use TOOLCHAIN_DIR instead"
categories:
- iOS

date:   2023-09-25  18:28:00 +0900
author_profile: false
---
```swift
DT_TOOLCHAIN_DIR cannot be used to evaluate LIBRARY_SEARCH_PATHS, use TOOLCHAIN_DIR instead
```

CocoaPods 깃허브의 이슈를 보니 Xcode 15.0(15A240d) 에서 iOS 17 시뮬레이터를 사용할 수 있게되면서 위의 에러를 겪고 있었습니다.

https://github.com/CocoaPods/CocoaPods/issues/12012

CocoaPods 1.13.0 버전의 릴리즈를 확인하니 버그가 수정되었습니다.

https://github.com/CocoaPods/CocoaPods/releases/tag/1.13.0

버전을 업데이트하여 해결하였습니다.

```swift
// 최신 버전으로 업데이트
gem install cocoapods --pre
// 그리고 업데이트해줘야함
pod update

// 버전 확인
pod --version

// 참고: https://github.com/ClintJang/cocoapods-tips/blob/master/README.md#1-2-최신버전으로-업데이트-하기
```
