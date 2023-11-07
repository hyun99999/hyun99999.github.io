---
title:  "iOS) UISheetPresentationController 에서 키보드를 사용해보자"
categories:
- iOS

date:   2023-10-10  23:43:00 +0900
author_profile: false
---
text field 를 통해 키보드를 사용하게 되면 어떻게 되는지 알아보겠습니다. 우선, 결과는 WWDC21 **[Customize and resize sheets in UIKit](https://developer.apple.com/videos/play/wwdc2021/10063/)** 에서 등장합니다.

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/69ce2a28-41ea-498e-bf76-c888d96acf77" width ="350">

키보드와 관련된 스크립트를 살펴보겠습니다.

<img width="700" alt="1-1" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/62f3852f-e6f8-4e0a-8ba5-01693bc90e39">

**medium** 높이의 sheet 는 **automatic keyboard avoidance** 를 지원해서 keyboard 를 계산해서 커지거나 축소된다고 합니다.

***우리는 결과를 알지만, custom detent 의 높이에도 자연스럽게 적용되는지 확인해보겠습니다.***

## 👉 확인해보자

두 가지를 살펴보겠습니다.

첫 번째는 **UISheetPresentationController** 의 detents 프로퍼티에 두 가지 detent 를 설정하겠습니다.

- 코드

```swift
let sheetVC = SheetVC()
let detentIdentifier = UISheetPresentationController.Detent.Identifier("tagSheet")
let customDetent = UISheetPresentationController.Detent.custom(identifier: detentIdentifier) { _ in
    return 270
}
                
if let sheet = sheetVC.sheetPresentationController {
    sheet.detents = [customDetent, .large()] // detent 설정
    // ...
}
```

- 결과

키보드가 활성되는 순간 **large** detent 로 높이가 설정되었습니다.

**large** detent 를 제거해보겠습니다.

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/221bfb44-5a89-4fb1-a9f4-56e9892a52af" width ="250">

두 번째는 detents 에 custom detent 만 설정하겠습니다.

- 코드

```swift
// ...
                
if let sheet = tagSheet.sheetPresentationController {
    sheet.detents = [customDetent] // detent 설정
    // ...
}
```

- 결과

키보드가 활성화되었을 때 키보드 만큼 위로 올라가는 것을 확인할 수 있었습니다.

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/120914b3-4d34-4c63-ab2c-346fb3609e22" width ="250">

## ❓ 궁금증

detents 가 두 개 보다 많을 때는 어떻게 작동할까요?

custom detent(높이 270), medium, large 를 사용해보겠습니다.

```swift
// ...
                
if let sheet = tagSheet.sheetPresentationController {
    sheet.detents = [customDetent, .medium(), .large()] // detent 설정
    // ...
}
```

스크롤에 따라서 세 개의 detent 모두 확인할 수 있었고, 키보드가 활성화되면 가장 큰 detnet 에 맞춰서 **automatic keyboard avoidance** 가 적용되는 것을 확인하였습니다.

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/08a37e7f-8dcc-4596-9088-54f1b51cc1ca" width ="250">

❗️ **결과적으로, 다음과 같이 알 수 있었습니다.**

- custom detent 도 automatic keyboard avoidance 가 적용됩니다.
- detent 가 두 개 이상이라면, 가장 큰 detent 에 맞춰서 automatic keyboard avoidance 가 적용됩니다.

## 👉 요구사항

다음의 요구사항을 구현해보겠습니다.

- sheet 를 띄우는 동시에 키보드도 띄우고, sheet 가 사라질 때 키보드도 사라지게 해봅시다.

UISheetPresentationController 를 사용할 때 뷰 컨트롤러의 라이프 사이클에 대해서 알아야 자연스럽게 적용할 수 있을 것 같습니다.

### View Life Cycle

우선, UISheetPresentationController 역할을 하는 뷰 컨트롤러에 다음과 같이 현재 라이프 사이클을 알 수 있도록 UI 를 작성하여 보겠습니다.

그리고 detent 를 변경하고, dimming view 를 터치하여 바텀시트를 내리거나 grabber 를 사용하여 바텀시트를 내릴 때 어떤 라이프 사이클인지 확인해보겠습니다.

- grabber 잡고 내릴때 **viewWillDisappear** 호출되고, medium detent 로 돌려놓으면 **viewWillAppear -> viewDidAppear** 호출됩니다.

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/66f787d1-d78d-4dce-b1a7-4dbfe975ef14" width ="250">

결과물로도 확인해보겠습니다.

- **viewWillAppear(_:)** 에서 키보드를 활성화하는 것이 가장 자연스러웠습니다.
- grabber 를 통해 일정 높이 이상 sheet 를 내리게 되면 keyboard 가 사라졌습니다.(view life cycle 과는 별개로 automatic keyboard avoidance 가 적용되기 때문인 것 같습니다.)
- dimming view 를 터치하여 sheet 를 한번에 내릴때 역시 별도의 작업 없이 자연스러웠습니다.

```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    
    textFiled.becomeFirstResponder()
}
```

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/8b306865-36a8-4e10-a783-fa24833a24b5" width ="250">

## 🚨트러블 슈팅
- viewWillAppear(_:) 에서 키보드를 띄우다보니 grabber 을 잡고 동작하다가 다시 detent 로 돌려놓으면 viewWillAppear(_:) 가 호출되어서 키보드가 원치 않는 상황에 다시금 등장하게 되었습니다.

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/8087c5c1-ac48-404a-8eee-8c1213fd0925" width ="250">

- flag 역할을 할 수 있는 변수를 사용하여 해결하였습니다.

```swift
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
    
        // flag 역할
        if !keyboardOn {
            adjectiveTextFiled.becomeFirstResponder()
            keyboardOn = true
        }
    }
```

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/ce4c20ca-35a7-40ad-82a7-d720ef417849" width ="250">
