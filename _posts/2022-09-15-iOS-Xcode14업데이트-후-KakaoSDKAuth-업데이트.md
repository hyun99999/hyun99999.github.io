---
title:  "iOS) CaseIterable 을 채택한 enum"
categories:
- iOS

date:   2022-09-15  18:45:00 +0900
author_profile: false
---
Xcode 14 업데이트 후 KakaoSDKAuth에서 저장 프로퍼티에 @available 이 붙는 곳에 다음과 같은 에러 메시지가 등장하였습니다.

```swift
Stored properties cannot be marked potentially unavailable with @available
```

에러를 해결하기 위해서 KakaoSDKAuth 를 업데이트 하였습니다.

다행히 검색해보니 데브톡에서 문의를 통해 Xcode 14 배타버전 때 문제를 발견하여 업데이트를 한 지 좀 되었습니다.

[데브톡 - Xcode 14 업데이트 빌드 오류 문의](https://devtalk.kakao.com/t/swift5-7-xcode14-beta3/124083)

[Kakao SDK 변경 이력](https://developers.kakao.com/docs/latest/ko/sdk-download/ios#changelog)

아래는 Xcode 14 의 릴리즈 노트에서 가져왔습니다.

```
Stored properties in Swift can’t have type information that is potentially unavailable at runtime.
However, prior to Swift 5.7 the compiler incorrectly accepted @available attributes on stored properties when the property had either the lazy modifier or an attached property wrapper.
This could lead to crashes for apps running on older operating systems.
The Swift compiler now consistently rejects @available on all stored properties. (82713248) (FB9594187)

```

- 저장 프로퍼티는 잠재적으로 사용할 수 없는 정보를 가질 수 없지만`@available` 을 통해서 이전 운영체제에서 충돌이 일어나는 경우가 있기 때문에 Swift 컴파일러는 모든 저장 프로퍼티에 대해서 `@available` 을 일괄적으로 거부하게 되었다는 대략적인 이야기입니다.

출처: [https://developer.apple.com/documentation/xcode-release-notes/xcode-14-release-notes?changes=lat__8_1](https://developer.apple.com/documentation/xcode-release-notes/xcode-14-release-notes?changes=lat__8_1)
