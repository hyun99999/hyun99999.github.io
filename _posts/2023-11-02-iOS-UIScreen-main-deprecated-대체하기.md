---
title:  "iOS) UIScreen.main deprecated 대체하기"
categories:
- iOS

date:   2023-11-02  15:24:00 +0900
author_profile: false
---
UIScreen.main 이 iOS 16 에서 Deprecated 되었습니다.

대체 방안을 알아보겠습니다!

<img width="500" alt="1" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/5cddb1f3-31e8-4dd7-a188-7d98749154bd">

기기의 screen scale 을 얻기위해서 보통 다음의 방법을 사용합니다.

```swift
// deprecated 예정
UIScreen.main.screen.scale

// windowScene 을 활용한 접근
view.window?.windowScene?.screen.scale

// UIApplication 을 활용한 접근
private let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene
windowScene?.screen.scale
```

이번에는 다른 방법에 대해서 알아보려 합니다.

### UITraitCollection 활용한 접근

```swift
UITraitCollection.current.displayScale
```

코드로 변경하였습니다. 이 장점은 `UIApplication` 을 통해서 접근하는 screen 의 경우 visionOS 를 지원하지 않습니다.

<img width="350" alt="2" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/f65b99cd-555e-4652-8898-f67a2dd21329">

(출처 : https://developer.apple.com/documentation/uikit/uiwindowscene/3198089-screen)

하지만, UITraitCollection 의 displayScale 은 visionOS 를 beta 로 지원하고 있습니다.

<img width="500" alt="3" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/33218122-eb96-4c20-bc4d-ccd435e239f8">

(출처: https://developer.apple.com/documentation/uikit/uitraitcollection/1623519-displayscale)

***개발자 문서를 살펴보겠습니다.***

“non-Retina 디스플레이스에서는 1.0을 나타내고, Retina 디스플레이스에서는 2.0 을 나타냅니다. 기본적으로 trait collection 의 display scale 은 0.0입니다.”

하지만, scale 3.0 의 값을 가지는 시뮬레이터에서 확인을 해봐도 그렇고 Retina 디스플레이는 2.0 혹은 3.0 을 가집니다.

아마도 visionOS 베타에서 정식으로 적용되면 개발자 문서도 내용이 바뀌지 않을까 싶습니다.

바뀌게 된다면 예전 내용과 확인해보는 것도 재밌겠네요!
