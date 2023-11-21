 ---
title:  "iOS) viewIsAppearing 대해 알아보자 - (적잘한 시점에서 키보드 활성화하기)"
categories:
- iOS

date:   2023-11-21  20:06:00 +0900
author_profile: false
---
### 내용

- 트러블 슈팅을 살펴보고 viewIsAppearing 에 대해서 알아보자
- 이를 사용해서 트러블 슈팅을 해결해보자

## 🚨 트러블 슈팅

- 문제  상황

iOS 17 - medium, custom detent 모두 **viewWillAppear** 에서 키보드를 활성화해도 원하는 동작 함.(custom detent 는 iOS 16이상부터 사용 가능)

iOS 15 - medium detent 를 설정하고 **viewWillAppear** 에서 키보드를 활성화시키면 다음과 같이 automatic keyboard avoidance 가 적용되지 않는 것 같습니다.

iOS 17 에서는 medium 이어도 정상적으로 키보드가 뷰를 밀어내는 반면 iOS 15 에서는 그렇지 못했습니다.

<img src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/d7531add-35e7-40df-948d-1ebd5f3a2b48" width ="250">

iOS 버전이 다름에서 오는 차이는 어쩔 수 없으니 해결을 위해서 우선, 키보드를 활성화시키는 타이밍이 문제라고 생각했습니다. viewDidAppear 에서 활성화하니 반응이 늦어 적용하기 힘들겠다고 생각했습니다.

<img src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/d77a32a8-e31d-4044-9f48-6c4c369fae22" width ="250">

## ✅ 해결

WWDC23 에서 소개된 **viewIsAppearing** 을 사용해서 해결해보기로 하였습니다.

### viewIsAppearing

[viewIsAppearing(_:) | Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uiviewcontroller/4195485-viewisappearing)

<img width="500" alt="3" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/08bfa6e0-0474-460b-b913-8bc48b7cc0b5">

viewWillAppear(_:) 호출 후 뷰 컨트롤러의 뷰가 나타날 때마다 이 메소드를 호출합니다. viewWillAppear(_:) 와 달리 시스템은 뷰 컨트롤러의 뷰를 뷰 계층에 추가한 후 이 메소드를 호출하고, superview 는 뷰를 배치합니다. **시스템이 호출할 때 즘이면 뷰 컨트롤러와 뷰 모두 trait collections 를 수신하고 뷰는 정확한 geometry(size, safe area 등)를 가집니다.**

이 메소드를 override 해서 뷰 표시와 관련된 커스텀 동작을 수행할 수도 있습니다. **예를 들어,** 뷰 또는 뷰 컨트롤러의 trait collections 를 기반으로 뷰를 구성하거나 업데이트할 수 있습니다. 또는 scroll position 계산은 view’s size 와 geometry 에 따라 달라지기 때문에, 뷰가 appear 할 때 collection/table view 를 programmatically scroll 해서  선택한 셀이 표시되도록 할 수 있습니다.

**Choosing Appropriate** 

시스템이 viewWillAppear(_:) 를 호출한 후에 이 메소드를 호출하지만, 두 콜백은 모두 같은 CATransaction 에서 발생합니다. 즉, 두 메소드에서의 변경이 동시에 사용자에게 보여지게 되는 것입니다.

<img width="250" alt="4" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/7025b35f-6a8c-4d4e-8e78-adc5ea733128">


**viewWillAppear(_:) 를  호출할 때는 traits 와 geometry 가 최신 상태가 아니지만, viewIsAppearing(_:) 을 호출할 때는 최신 상태이므로 뷰를 업데이트할 때 사용할수 있습니다.** 

다음과 같은 경우에만 viewWillAppear(_:) 사용하세요.

- alognside animations 을 추가하기 위해 transitionCoordinator 에 접근할 때와 같이 view transition 이 시작되기 전에 콜백이 필요합니다. alongside animations 는 프레임워크가 뷰 컨트롤러 transition animations 과 동시에 수행하도록 지시하는 애니메이션입니다.
- 뷰 컨트롤러 또는 뷰의 traits, hierarchy, or geometry 의존하지 않는 작업을 수행할 때 콜백이 필요합니다. viewWillAppear(_:) 에서 database notifications 를 등록하거나 viewDidDisappear(_:) 에서 등록 해제가 이에 해당합니다.

다른 모든 경우에는 **viewIsAppearing(_:)** 을 사용해서 뷰를 업데이트할 수 있습니다.

<img width="500" alt="5" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/8c0aec4e-77c4-4a13-8441-6b4363fe343f">

시스템은 뷰가 layoutSubviews() 를 실행할 때마다 viewWillLayoutSubviews() 와 viewDidLayoutSubviews() 와 같은 레이아웃 메소드를 호출합니다. 이는 뷰가 전환되는 중이거나 표시되는 동안 여러 번 발생할 수 있습니다.

**하지만, 시스템은 뷰가 보여질 때 viewIsAppearing(_:) 을 한 번만 호출하고, 레이아웃이 필요없는 경우에도 이를 호출합니다.** 

뷰 컨트롤러가 뷰 계층에 추가하는 방법과 발생하는 메시지의 순서에 대해서 좀 더 자세히 알아보기 위해 다음 문서를 살펴볼 수 있습니다. [Displaying and managing views with a view controller](https://developer.apple.com/documentation/uikit/view_controllers/displaying_and_managing_views_with_a_view_controller)

---

WWDC23 > [What's new in UIKit](https://developer.apple.com/videos/play/wwdc2023/10055/) 에서도 몇 부분을 발췌해보겠습니다.(2분 23초)
WWDC23 에서 UIViewController.viewIsAppearing(_:) 새로운 view controller callback 이 생겼습니다.

<img width="400" alt="6" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/fdb5ebcd-4389-414f-9739-6f7210b0142b">

개발자 문서에서도 보았던 흐름도입니다. 몇 가지를 중요하게 강조하였습니다.

<img width="500" alt="7" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/6a6bfd53-60da-4a60-970d-fb05ccbafc78">

- viewWillAppear(_:)가 호출되는 시점을 살펴보게 되면 trait collection 을 사용하거나 뷰의 크기와 geometry 구조에 의존하기에 이른 것을 볼 수 있습니다.
- transition animaties 후에 마지막 transaction 에서 viewDidAppear(_:)가 호출되는 시점을 주목해보겠습니다. transaction 이 완료되기 전까지는 viewDidAppear(_:)의 변화가 눈에 보이지 않는다는 것이고, 보이게 하고 싶기에는 너무 늦었다는 뜻입니다.
반면에 viewIsAppearing(_:)은 viewWillAppear(_:)과 동일한 transaction에서 호출됩니다. 즉, 두 개의 콜백에서의 변화는 동시에 사용자 눈에 보이게 된다는 것입니다. 전환의 첫 프레임부터 가능합니다.
- viewIsAppearing(_:)과 layout call back(viewWillLayoutSubviews(), viewDidLayoutSubviews) 사이에는 주요한 차이가 있습니다.
뷰의 layoutSubviews() 실행 때마다 layout callback 이 이루어지는데 전환 중에 여러번 혹은 나중에 뷰가 보일 때 언제든지 일어날 수 있습니다. 하지만, viewIsAppearing(_:)은 전환 중에 한 번만 호출되고 뷰에 레이아웃이 필요 없더라도 호출됩니다.

---

**정리해보자면**

- **viewIsAppearing** 은 viewWillAppear 호출 후에 뷰 컨트롤러의 뷰를 뷰 계층에 추가할 때 호출됩니다.
- 같은 transaction 에서 호출되기 때문에 두 콜백에서의 변경이 동시에 사용자에게 첫 프레임부터 보여지게 됩니다.
- **viewIsAppearing** 은 viewWillAppear 와 달리 뷰 컨트롤러와 뷰 모두 최신 상태의 trait collection 을 수신하고 뷰는 geometry(size, safe area 등)을 가집니다.
- 뷰가 layoutSubviews() 를 실행할 때마다 viewWillLayoutSubviews() 와 viewDidLayoutSubviews() 와 같은 레이아웃 콜백을 호출합니다. 이는 뷰가 전환되는 중이거나 표시되는 동안 여러 번 발생할 수 있습니다. 하지만, 시스템은 뷰가 보여질 때 **viewIsAppearing** 을 한 번만 호출하고, 레이아웃이 필요없는 경우에도 이를 호출합니다.
- 사용하는 예시)
    - triat collection 을 기반으로 뷰를 구성하거나 업데이트
    - 뷰가 보여질 때 collection/table view 를 programmatically scroll 해서 해당 셀이 보여지도록 scroll position 을 설정할 수 있습니다.

---

***적용해보겠습니다.***

```swift
override func viewIsAppearing(_ animated: Bool) {
    super.viewIsAppearing(animated)

    textFiled.becomeFirstResponder()
}
```

- iOS 15, 17 모두 원하는 대로 작동하게 되었습니다.

<img src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/b0b3f5de-a95b-41ba-ad83-2bb58867d717" width ="250">
