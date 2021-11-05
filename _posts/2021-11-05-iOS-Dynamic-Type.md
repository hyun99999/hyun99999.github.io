---
title:  "iOS) Dynamic Type"
categories:
- iOS

date:   2021-11-05  17:30:00 +0900
author_profile: false
---
Dynamic Type 이란! 쉽게 말해서 우리가 기기에서 아래와 같이 `텍스트 크기` 를 설정할 수 있는데요. 이 때 사용자가 지정한 텍스트 크기에 따라 동적으로 크기가 변하는 것을 말해요!

> 설정 > 디스플레이 및 밝기 > 텍스트 크기
> 

<img src="https://user-images.githubusercontent.com/69136340/140479925-68eb3367-ea56-4e99-a043-00967d7d3a7f.PNG" width ="200">

Dynamic Type 을 적용하지 않으면 기기에서 텍스트 크기를 변경했는데 적용이 되지 않는 것이지요!

또한, 더 큰 사이즈도 설정이 가능한데요

> 설정 > 손쉬운 사용 > 디스플레이 및 텍스트 크기 > 더 큰 텍스트
> 

추가로 5개의 더 큰 사이즈도 선택할 수 있는 것을 볼 수 있어요!

<img src ="https://user-images.githubusercontent.com/69136340/140479943-666f56b3-bb75-46dc-ae74-8aab872bd5de.jpeg" width="200">

## 🔍 HIG 살펴보기

HIG 문서에서 말하는 `Dynamic Type` 은 이러합니다.

> Dynamic Type 은 글자를 읽는이가 선호하는 텍스트 사이즈를 선택할 수 있도록 추가적인 flexibility(유연성)을 제공합니다.
> 

Dynamic Type 은 `xSmall`, `Small`, `Medium`, `Large(Default)`, `xLarge`, `xxLarge`, `xxxLarge` 이렇게 총 7가지 사이즈를 제공합니다.

그리고 Dynamic Type 에 따른 텍스트 스타일의 weight, size, leading(행간) 값을 다음의 표를 통해서 볼 수 있어요!


<img src ="https://user-images.githubusercontent.com/69136340/140480027-308d8ce3-9eeb-40de-adea-c3404e1a5dfe.png" width =" 400">

시스템은 접근성이 필요한 사용자들을 위해서 더 큰 크기의 사이즈도 제공합니다! (위에서 언급한 더 큰 텍스트가 이것이죠!)

<img src="https://user-images.githubusercontent.com/69136340/140480034-40f98501-8c1a-4e2b-b202-661b4e0c9718.png" width ="400">

**출처:** 

[Typography - Visual Design - iOS - Human Interface Guidelines - Apple Developer](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/typography/)

## 🔍 [WWDC 17: Buildeing Apps with Dynamic Types](https://developer.apple.com/videos/play/wwdc2017/245/)

WWDC 에서 발표할 때 Dynamic Type 을 적용하기 위한 세 가지 목표를 제시해줍니다.

- Text is large enough for the user to read
- Text is fully readable
- App UI looks beautiful

세 가지 목표를 기반으로 WWDC 에서 제시된 방법들에 대해서 살펴봐요!

### 1.Fitting Large Text on Screen

흰색 테두리가 사용자가 볼 수 있는 사이즈라고 해볼까요?

<img src ="https://user-images.githubusercontent.com/69136340/140480164-addadc29-636a-4225-95a5-f741e04dedfe.png" width ="450">

### 2.Avoid Constant Values Based on Defualy Text Size

텍스트 크기를 기반으로 한 상수값은 폰트사이즈가 커지면서 다음과 같이 겹치는 문제를 가지게 됩니다!

(우리가 일반적 top과 bottom 으로 오토레이아웃을 잡는데 이건 문제가 되지 않아요!)

<img src="https://user-images.githubusercontent.com/69136340/140480256-2554dcbc-0e07-469d-91ac-e19c4cde07e2.png" width ="500">

그래서 First Label 의 `First Baseline` 과 Second Label 의 `Last Bseline` 을 기준으로 오토레이아웃을 잡아주라고 합니다.

<img width="250" alt="8" src="https://user-images.githubusercontent.com/69136340/140480290-e9484251-8df9-470f-8584-20561c717993.png">

코드)

<img src="https://user-images.githubusercontent.com/69136340/140480356-230c0db4-0a32-410c-ac0f-fb49bb720606.png" width ="500">

### 3.Side-By-Side Text

이번에는 노란색 박스가 사용자가 보는 스크린이라고 해봅시다.

<img src="https://user-images.githubusercontent.com/69136340/140480417-9cdb9cd4-9f02-489f-baf6-35753a8b6bdb.png" width ="350">

## 🔍 Dynamic Type 적용

- Interface Builder 로 적용하기
- 코드로 적용하기

### Interface Builder 로 적용하기

- Font 에서 `Text Style` 을 설정해주고, `Automatically Adjusts Font` 를 체크해주면 적용이 끝납니다!

<img width="600" alt="5" src="https://user-images.githubusercontent.com/69136340/140480606-08501850-120e-4669-bbb7-6f65bde79ff2.png">

- UIButton 은 iOS 15 부터 기본적으로 Dynamic Type 을 지원하기 시작했어요. 그래서 아래와 같이 Font 에서 지정만하면 적용할 수 있어요.

<img width="250" alt="11" src="https://user-images.githubusercontent.com/69136340/140480639-6835cd15-80e8-40fc-b417-7a0523295214.png">

### 코드로 적용하기

iOS 15 에서 UIButton 이 Dynamic Type 을 지원하면서 그전에는 복잡했던 코드가 쉬워졌다...! 👍

```swift
import UIKit

class ViewController: UIViewController {
    @IBOutlet weak var label: UILabel!
    @IBOutlet weak var button: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()

        // ✅ text style 적용
        label.font = UIFont.preferredFont(forTextStyle: .largeTitle)
        // ✅ Automatically Adjust Font 체크
        label.adjustsFontForContentSizeCategory = true

        // ✅ text style 적용
        button.titleLabel?.font = UIFont.preferredFont(forTextStyle: .largeTitle)
        // ✅ Automatically Adjust Font 체크
        button.titleLabel?.adjustsFontForContentSizeCategory = true
    }
}
```

### 커스텀 폰트에도 적용

지금까지는 시스템이 제공하는 폰트를 사용했는데 커스텀 폰트의 경우는 어떻게 대응할 수 있을까요?

```swift
label.adjustsFontForContentSizeCategory = true

// 1. 폰트 크기 비율에 따라 리사이징
label.font = UIFontMetrics.default.scaledFont(for: customFont)
// 2. build-in 폰트 사이즈에 매핑하여 리사이징
label.font = UIFontMetrics(forTextStyle: .largeTitle).scaledFont(for: customFont)
```

## 🔍  폰트 사이즈 구하기

[preferredContentSizeCategory](https://developer.apple.com/documentation/uikit/uiapplication/1623048-preferredcontentsizecategory) 로 사용자의 폰트사이즈를 구할 수 있어요!

이를 통해서 앞서 looks beautiful 하게 vertical 로 만들라는데..! 이거 어캐해!! 했는데 분기 처리가 가능하겠어요!

<img src="https://user-images.githubusercontent.com/69136340/140480669-af4b4e48-feab-4aa1-81a3-94367dc729ff.png" width ="450">

`preferredContentSizeCategory` 를 사용해서 시뮬레이터의 text style 이 large(default) 인지 확인해보았어요!

```swift
if traitCollection.preferredContentSizeCategory == UIContentSizeCategory.large {
            print("large")
}
// large
        
if UIApplication.shared.preferredContentSizeCategory == UIContentSizeCategory.large {
            print("large")
}
// large
```

### **UIContentSizeCategory**

<img width="500" alt="스크린샷 2021-11-05 오후 5 26 35" src="https://user-images.githubusercontent.com/69136340/140480776-befeb73b-6afd-400c-8b53-d35a55962711.png">

## 🔍 Dynamic Type 쉽게 테스트하기

### Simulator

[WWDC 2019 - Visual Design and Accessibility](https://developer.apple.com/videos/play/wwdc2019/244/) 에서 Accessibility 를 소개하면서 Dynamic Type 을 테스트하는 방법을 언급했는데 알아보자구요

> Xcode > Open Developer Tool > Accessibility Inspector
> 

<img width="300" alt="13" src="https://user-images.githubusercontent.com/69136340/140480786-ab518298-8af6-4192-bda1-54c44e6dd270.png">

- 시뮬레이터를 설정해주고 우측 상단의 버튼을 눌러주면 시뮬레이터의 Dynamic Type 을 손쉽게 설정할 수 있어요!

<img width="350" alt="14" src="https://user-images.githubusercontent.com/69136340/140480816-e83714a5-deeb-4a6c-80af-176585c32d4f.png">


### Accessibility

시뮬레이터가 아닌 Xcode 내에서 개발할 때도 손쉽게 설정할 수 있는데요.

스토리보드 화면 하단의 Accessibility 에서 설정하면 된답니다!

<img width="300" alt="15" src="https://user-images.githubusercontent.com/69136340/140480831-d1695c4c-c687-444b-bdac-0fbb8bc269e3.png">

출처 :

[[iOS] Dynamic Type](https://eunjin3786.tistory.com/363)

[Dynamic Type](https://velog.io/@minni/Dynamic-Type-egjn26z5)

[Dynamic Type을 사용하여 폰트 크기 조정하기](https://junyng.tistory.com/30)
