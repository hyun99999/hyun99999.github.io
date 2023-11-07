---
title:  "iOS) UISheetPresentationController 를 사용해서 바텀시트 만들기"
categories:
- iOS

date:   2023-09-27  11:32:00 +0900
author_profile: false
---
### 내용

- UISheetPresentationController 을 사용해서 바텀시트 만들기
- 높이를 커스텀하여 내가 원하는 바텀시트의 높이를 설정해보자
- 바텀시트의 둥글기, grabber의 유무에 대해서 설정해보자

WWDC21 **[Customize and resize sheets in UIKit](https://developer.apple.com/videos/play/wwdc2021/10063/)** 에서 소개된 UISheetPresentationController을 활용한 바텀시트에 대해서 알아보겠습니다.

<img width="250" alt="1" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/01a62c25-7a1a-466a-9064-1db4c9f52b6f">

개발자 문서를 살펴보겠습니다.

[UISheetPresentationController | Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uisheetpresentationcontroller)

iOS 15와 iPadOS 15부터 적용가능합니다. **UISheetPresentationController** 를 사용하면 뷰컨트롤러를 sheet 로 표현할 수 있습니다. 

다음과 같이 **sheetPresentationController** 프로퍼티를 통해 구성할 수 있습니다. 해당 프로퍼티는 옵셔널 형태입니다. sheet 형식으로 보여줄 때는 필요가 없기 때문에 nil 이 됩니다.

```swift
// In a subclass of UIViewController, customize and present the sheet.
func showSheet() {
    let viewControllerToPresent = MyViewController()

    if let sheet = viewControllerToPresent.sheetPresentationController {
    // ✅ 다음 프로퍼티들은 찬찬히 알아가봅시다.
        sheet.detents = [.medium(), .large()]
        sheet.largestUndimmedDetentIdentifier = .medium  // nil 기본값
        sheet.prefersScrollingExpandsWhenScrolledToEdge = false  // true 기본값
        sheet.prefersEdgeAttachedInCompactHeight = true // false 기본값
        sheet.widthFollowsPreferredContentSizeWhenEdgeAttached = true // false 기본값
    }
        
    // ✅ sheet present.
    present(viewControllerToPresent, animated: true, completion: nil)
}
```

위의 코드를 통해 시트가 자연스럽게 리셋되는 높이인 detent 를 기준으로 시트의 크기를 정하게됩니다.

🧑‍🏭 *개발자 문서에 나온 코드이니 순서대로 알아봅시다. 그만큼 중요하다는 거겠죠?*

### detent?

detent 라는 용어를 **멈춤쇠**라고 해석하기에 조금 부족함이 있다고 생각했습니다. 그러던 중 WWDC22 세션 "[Build a productivity app for Apple Watch](https://developer.apple.com/videos/play/wwdc2022/10133/)" 에서 듣게 되어서 발췌했습니다.

### detent!

**detent** 는 기계적 용어로 움직일 만큼 충분한 힘이 가해질 때까지 무언가를 제자리에 고정시키는 메커니즘입니다. 예를 들어, 차 문을 열 때 안착되는 ‘정지’ 위치가 있습니다. 문을 조금 더 세게 밀어 또 다른 ‘정지’ 위치까지 더 열 수도 있습니다.

문을 닫으려면 ‘정지’에서 빼낼 수 있을 만큼 세게 당겨서 저항을 이겨내야 합니다. 그렇지 않으면 문은 ‘정지’ 위치로 돌아갑니다. 이것이 **detent** 입니다.

시스템 **detent** 는 large, medium 이 있습니다.

<img width="700" alt="2" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/eefdcff6-eed7-4941-99a9-778211ad2571">

(출처: https://developer.apple.com/documentation/uikit/uisheetpresentationcontroller/detent)

아래에서 살펴보겠지만 `custom(identifier:resolver:)` 메서드를 사용해서 detent를 커스텀할 수 있습니다.

iPad 와 iPhone 의 sheet 는 다음과 같습니다.

<img width="700" alt="3" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/5ce6314e-e606-4f4f-960f-a6634c54a3c6">

(출처: https://developer.apple.com/videos/play/wwdc2021/10063/)

👉 ***다음은 user interaction 에 대한 매니징입니다.***

<img width="500" alt="4" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/1a30fb3c-e764-4c4f-a06a-116f3200985b">

### 👉 largestUndimmedDetentIdentifier

The largest detent that doesn’t dim the view underneath the sheet.

즉, sheet 아래의 뷰를 dim(흐리게) 만들지 않는 가장 큰 detent 를 지정할 수 있습니다.

기본값은 nil 이고, 이는 모든 detent 에서 dimming view 를 추가한다는 것을 의미합니다. 예를 들어, medium detent 에 dimming view 를 추가하지 않기 위해서 medium 으로 값을 설정할 수 있습니다.

***예를 들어, 지도 앱을 살펴보겠습니다.(지도앱의 초기 detent 는 medium이 아니긴 합니다.)***


<img src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/63367ad8-8489-46be-bb86-c389fdc0699e" width ="250">


이외에도 **dimming view** 와 관련된 중요한 점이 있습니다.

원래라면 dimming view 를 터치해서 sheet 를 없앨 수 있지만, dimming view 가 있지 않으면 유저와 상호작용 할 수 있는 **nonmodal 경험을 제공합니다.** 

### 👉 prefersScrollingExpandsWhenScrolledToEdge

A Boolean value that determines whether scrolling expands the sheet to a larger detent.

스크롤이 더 큰 detent 로 확장하는지 여부를 결정합니다.

true 기본값. 즉, 스크롤할때 더 큰 detent 로 확장할 수 있다면 가장 큰 detent 로 확장되어 스크롤이 시작됩니다.

위에서 확인한 지도 앱처럼 바로 스크롤할때 더 큰 detent 로 확장되는 것을 보여줍니다.

👉 ***다음은 appearance 에 대한 매니징입니다.***

<img width="500" alt="6" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/8419afd3-a5e2-4b7c-9b2d-f2c4e0a851ef">

### 👉 prefersGrabberVisible

A Boolean value that determines whether the sheet shows a grabber at the top.

기본값은 flase. sheet 위의 grabber 를  표시할지 여부를 결정합니다.

<img width="300" alt="7" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/d3ab37be-f157-4b5f-8e61-b0b35ed6188c">

### 👉 prefersPageSizing

A Boolean value that indicates whether the sheet sizes itself for readable content.

**iOS 17** 이상부터 사용가능합니다.

기본값은 true 입니다. 즉, 기본값은 sheet 가 읽을 수 있는 너비를 따르는 [UIModalPresentationStyle.pageSheet](https://developer.apple.com/documentation/uikit/uimodalpresentationstyle/pagesheet) 동작을 사용함을 의미합니다.

값이 false 로 설정되면, sheet는 [UIModalPresentationStyle.formSheet](https://developer.apple.com/documentation/uikit/uimodalpresentationstyle/formsheet) 동작을 사용합니다. 여기서 sheet 의 크기는 presented view controller 의 [preferredContentSize](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621476-preferredcontentsize) 를 따릅니다.

### 👉prefersEdgeAttachedInCompactHeight

A Boolean value that determines whether the sheet attaches to the bottom edge of the screen in a compact-height size class.

아래 WWDC21 영상의 일부처럼 compact-height size(iPhone의 가로모드에 해당) 에서 sheet 가 화면의 bottom edge 에만 붙는지 여부를 결정합니다.
<img width="600" alt="8" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/8882f1fd-0f4a-4331-84e5-0c712ba25eea">

기본값은 false. 전체 화면 모양으로 설정됨을 의미합니다. 

### 👉 widthFollowsPreferredContentSizeWhenEdgeAttached

A Boolean value that determines whether the sheet’s width matches its view controller’s preferred content size.

기본값은 false. sheet 의 너비가 container 의 safe area 와 동일함을 의미합니다.

true 로 설정하면 view controller 의 **preferredContentSize** 를 통해서 sheet 의 너비가 결정됩니다.

<img width="600" alt="9" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/6e5406bd-983d-472a-964c-9cf6e5fea9c8">

sheet 가 compact-width 와 regular-height size 인 경우(iPhone의 세로모드에 해당) 혹은 **prefersEdgeAttachedInCompactHeight** 가 false 인 경우에는 효과가 없습니다.

(**아래는 HIG문서에서 확인할 수 있는 디바이스들의 사이즈 클래스입니다.)**

<img width="500" alt="10" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/3a6fe75c-9bf8-4bb3-a833-4c7713b72bc0">

(출처 : https://developer.apple.com/design/human-interface-guidelines/layout)

### 👉 preferredCornerRadius

The corner radius that the sheet attempts to present with.

기본값은 nil. 하지만, 어느정도 둥글기가 적용되어 있습니다.

sheet 가 sheet stack 의 맨 앞에 있는 경우에만 유효합니다.

***👉 이번 글에서는 사용되지 않지만 그 외에도 있습니다.***

<img width="500" alt="11" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/fe3d0ee9-1bd6-4cdd-9d25-7a4cec9ef874">

## ✋ 만들어보자

자, 이제 만들어봅시다. 몇가지 목표를 가지고 적용해보겠습니다.

### 목표

- detent 를 커스텀
- corner radius 를 적용
- 스크롤을 통해서 detent 조절(두 가지 이상의 Detent를 사용해보자)
- grabber 보이지 않게 구현

### Creating a custom detent

13mini 기준으로 436 라는 높이값을 가집니다.(사실, 이는 safe area bottom 34 를 포함한 값입니다.) 이 값은 기기별로 다릅니다.

<img width="700" alt="12" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/67643db5-2f9d-4f21-b0b0-40e9eccef631">

이제, 원하는 높이의 custom detent 를 만들어봅시다.

### custom(identifier:resolver)

[custom(identifier:resolver:) | Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uisheetpresentationcontroller/detent/3976719-custom)

**iOS 16 이상**부터 사용 가능합니다.

```swift
custom(identifier:resolver:)

// Creates a custom detent for a sheet by computing its value according to the properties of the provided context.

@MainActor
static func custom(
    identifier: UISheetPresentationController.Detent.Identifier? = nil,
    resolver: @escaping (_ context: UISheetPresentationControllerDetentResolutionContext) -> CGFloat?
) -> UISheetPresentationController.Detent
```

- **identifier** : detent 의 식별자. 식별자를 지정하지 않으면 시스템에서 랜덤한 식별자가 생성됩니다.
- **resolver** : 이 클로저에서 반환되는 값은 safe area 내의 높이입니다. 예를 들어 200 반환하면 시트가 가장자리에 닿을 때 `safeAreaInsets.bottom` 을 더한 값을 반환하고(13mini 기준 200+34), 혹은 시트가 띄어져있는 경우 200을 반환합니다.
detent 가 비활성화되도록 지정하려면 `nil` 반환하면 됩니다.
클로저가 외부 입력에 따라 달라지는 경우, sheet 에서 [vaildateDetents()](https://developer.apple.com/documentation/uikit/uisheetpresentationcontroller/3975902-invalidatedetents) 를 호출하면 됩니다.
클로저를 실행하는 동안 `UISheetpresentationController` 어떤 속성도 설정하면 안됩니다.

```swift
if #available(iOS 16.0, *) {
    let detentIdentifier = UISheetPresentationController.Detent.Identifier("customDetent")
    // identifier 매개변수는 옵셔널이기 때문에 채우지 않으면 시스템이 임의의 식별자를 설정합니다.
    let customDetent = UISheetPresentationController.Detent.custom(identifier: .init("customDetent")) { _ in
        // 13mini 기준 최종적으로 safeAreaInsets.bottom을 더한 634높이를 가집니다.
        return 600
    }
                
    if let sheet = tagSheet.sheetPresentationController {
         sheet.detents = [customDetent, .large()] // detent 설정
    }
}
```

이러한 기기별로 다른 detent 높이 값을 사용할 수는 없을까요?

**detent 의 값을 가져와 이를 통해서 `custom(identifier:resolver:)` 메서드 안에서 활용하여 custom detent 를 구성할 수도 있습니다.**

### [resolvedValue(in:)](https://developer.apple.com/documentation/uikit/uisheetpresentationcontroller/detent/3976720-resolvedvalue)

**iOS 16** 이상부터 사용할 수 있습니다.

detent 값을 반환하고, 비활성화된 경우 `nil` 반환합니다. `medium` 과 `large` detent 값을 출력해보겠습니다.

```swift
let customDetent = UISheetPresentationController.Detent.custom(identifier: "customDetent") { context in
    print("medium: ",UISheetPresentationController.Detent.medium().resolvedValue(in: context))
    print("large: ", UISheetPresentationController.Detent.large().resolvedValue(in: context))
    return nil
}

// 13mini 기준
// medium:  Optional(402.08000000000004)
// large:  Optional(718.0)
```

### 구현해보자

```swift
let sheetVC = SheetVC()

// ✅ custom detent 생성
let detentIdentifier = UISheetPresentationController.Detent.Identifier("customDetent")
let customDetent = UISheetPresentationController.Detent.custom(identifier: detentIdentifier) { _ in
    // safe area bottom을 구하기 위한 선언.
    let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene
    let safeAreaBottom = windowScene?.windows.first?.safeAreaInsets.bottom ?? 0    

    // ✅ 모든 기기에서 항상 높이가 600인 detent를 만들어낼 수 있다.
    return 600 - safeAreaBottom
}
                
if let sheet = sheetVC.sheetPresentationController {
    sheet.detents = [customDetent, .large()] // detent 설정
    sheet.preferredCornerRadius = 30 // 둥글기 수정

    // ✅ grabber를 보이지 않게 구현.(UI를 위해 이미지로 대체)
    // sheet.prefersGrabberVisible = false // 기본값

    // ✅ 스크롤 상황에서 최대 detent까지 확장하는 여부 결정.
    // sheet.prefersScrollingExpandsWhenScrolledToEdge = true // 기본값
}

// ✅ 기본값 automatic. 대부분의 뷰 컨트롤러의 경우 pageSheet 스타일에 매핑.
// sheetVC.modalPresentationStyle = .pageSheet
present(sheetVC, animated: true)
```

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/e0abebbe-0b3a-41bf-8863-491cc6d20ed5" width = "250">
