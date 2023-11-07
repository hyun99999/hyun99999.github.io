---
title:  "iOS) UISheetPresentationController dimming view alpha 수정해보기(실패)"
categories:
- iOS

date:   2023-10-10  23:43:00 +0900
author_profile: false
---
## 👉 내용

- dimming view 의 alpha 값을 수정해보겠습니다.
- 결과적으로 alpha 값을 원하는대로 수정하지 못했고, 뷰 계층의 접근방법과 문제를 해결하기 위해 고민한 과정을 글로써 작성하였습니다.

## 👉 dimming view alpha 수정

우선, 개발자 문서에서는 **alpha** 에 대한 안내가 없기 때문에 view 에 접근해서 프로퍼티를 수정해보기 위해 view hierarchy 를 확인해보았습니다.

<img width="700" alt="11" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/862f023a-a759-44c1-a26e-41537276c62e">

view hierarchy 를 확인해보면 background alpha 기본값은 0.2 로 설정되어 있습니다. 이를 수정해보겠습니다.

UIDimmingView 에 접근하기 위해서

**UIWindowScene > UIWindow > UITransitionView > UIDimmingView** 인 것을 알아야 합니다.

다음의 코드로 접근해서 속성을 확인해보겠습니다.

```swift
// ✅ UIWindowScene > UIWindow > UITransitionView > UIDimmingView
let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene
let windows = windowScene?.windows.first?.subviews

print(windows)
// Optional([<UITransitionView: 0x117f1b160; frame = (0 0; 393 852); autoresize = W+H; layer = <CALayer: 0x600000278560>>, <UITransitionView: 0x117f33140; frame = (0 0; 393 852); autoresize = W+H; layer = <CALayer: 0x6000002d2d40>>, <UITransitionView: 0x11748eed0; frame = (0 0; 393 852); autoresize = W+H; layer = <CALayer: 0x600000334b00>>])
```

view hierachy 에 표시된 대로 세 개의 UITransitionView 중 마지막의 UIDimmingView 에 접근해보겠습니다.

<img width="400" alt="12" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/19a44935-706b-401b-bf68-44c0e330529d">

```swift
// ✅ UIWindowScene > UIWindow > UITransitionView > UIDimmingView
let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene
let transitionView = windowScene?.windows.first?.subviews[2]
let dimmingView = transitionView?.subviews[0]

dimmingView?.alpha = 0.8

print(transitionView?.subviews)
// Optional([<UIDimmingView: 0x11ad97b60; frame = (-393 -852; 1179 2556); alpha = 0.8; opaque = NO; gestureRecognizers = <NSArray: 0x600000ecf840>; backgroundColor = UIExtendedSRGBColorSpace 0 0 0 0.2; layer = <CALayer: 0x60000031b8a0>>, <UIDropShadowView: 0x11ada0260; frame = (0 548; 393 304); gestureRecognizers = <NSArray: 0x600000e8c270>; layer = <CALayer: 0x600000336200>>])
```

UIDimmingView 에 접근할 수 있게 되었습니다!

다음과 같이 view 자체를 0.8 로 설정하였습니다. 그러나 background 의 alpha 값이 0.2이기 때문에 의도한 UI 가 아니지만 수정되었습니다!

<img width="700" alt="13" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/2724add5-640d-4733-ba7c-08f59fb67859">

이번에는 background 색상에 검정색에 alpha 값을 0.4 주어서 수정하여 의도한 UI 가 완성되도록 해보겠습니다.

```swift
let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene
let transitionView = windowScene?.windows.first?.subviews[2]
let dimmingView = transitionView?.subviews[0]
dimmingView?.backgroundColor = .black.withAlphaComponent(0.4)
```

<img width="700" alt="14" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/34afec64-017a-4b00-9331-81ce8be1aac4">

하지만, 이런식의 접근을 위해서는 **viewDidLoad** 에서 호출할 수 없습니다. 해당 라이프 사이클까지는 당연히 뷰가 아직 그려지기 전이니 UITransitionView 가 두 개만 있습니다.

그래서 **viewWillAppear** 단계 이후부터 우리가 다루고자하는 UITransitionView 에  접근할 수 있습니다.

해당 라이프 사이클에서 alpha 값을 조정하게 되면 다음과 같이 부자연스럽게 UI 가 변할 수 밖에 없었습니다.

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/e0b1df11-68a4-4cc1-8951-fea5520838f9" width ="250">

## 🚨 트러블 슈팅

아직 만들어지지 않은 뷰가 이렇게 뷰 계층을 가질 것이다 라는 전제를 가지고 뷰에 접근해보았습니다.

그 결과 특정 라이프 사이클 이후에 뷰에 접근할 수 있었고, 부자연스럽게 UI 가 그려지는 것에 이는 사용할 수 없다고 판단했습니다. 😭

## 👉 접근

UITransitionView 를 보니 present() 메소드를 통해서 뷰를 보여줄 때 UISheetPresentationController 설정 없이도 아래와 같이 다음 화면을 띄어줄 때도 dimming view 가 있던 것이 생각났습니다.

```swift
nextViewController.modalPresentationStyle = .pageSheet
```

역시나 동일한 UIDimmingView 였습니다.

<img width="700" alt="16" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/9a3efb5c-1278-41e5-93ec-5f311defe46c">

**문제의 관점을 바꾸어서 modalPresentationStyle.pageSheet 일 때 dimming view 의 alpha 값을 변경해야하는 문제임을 알게되었습니다.**

이는 더이상 UISheetPresentationController 와 관련된 문제가 아닌 UIPresentationController 클래스를 상속받아 presentationController 을 커스텀해야하는 문제임을 알게되었습니다.
