 ---
title:  "iOS) UIScreen.main deprecated 를 대체하여 오픈 소스 라이브러리 컨트리뷰터 되어보기"
categories:
- iOS

date:   2023-11-21  23:16:00 +0900
author_profile: false
---
## 계기

[iOS) YPImagePicker error :  Stored properties cannot be marked unavailable with `@available`](https://gyuios.tistory.com/295)

Xcode 15 업데이트 이후 YPImagePicker 에서 사용하려하니 에러를 만났고, 하룻동안은 빌드 없이 코드를 봤던 기억이 있습니다.

그리고 몇일 뒤 CocoaPods 를 업데이트하고 나서 빌드 오류가 나지 않아 YPImagePicker 깃허브를 들어갔고 새로운 버전이 릴리즈된 것을 확인하였습니다.

거창한 문서도 거창한 코드 변화도 아닌 필요에 의한 이제는 사용하지 않는 변수의 단순한 코드 삭제였습니다.

오픈 소스 라이브러리를 사용만 하다보니 사용자 입장에서 수동적으로 대응한 것 같다는 생각이 들어서 내가 할 수 있는 부분이 있다면 적극적으로 적극적으로 기여해보고 싶다는 생각이 들었습니다. 이번 기회를 통해서 몇 군데에 PR 을 작성해볼 수 있었습니다.

## 👉 들어가기 전

WWDC2022 ****What's new in UIKit 세션에서 다음과 같이 청천벽력같은.. 소식을 보았습니다.****

<img width="500" alt="1" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/909a4417-6ccf-4e98-9186-3a9c80112bfc">

(출처 : https://developer.apple.com/videos/play/wwdc2022/10068/?time=1121)

이렇게두고 있다가 시간이 흘러 iOS 15에서 iOS 17까지 시간이 흘렀습니다.

iOS 16까지만 지원한다고 하니 이를 대응해야겠다고 생각해서 글을 작성하였습니다.(하지만, iOS 17시뮬레이터에서도 정상적으로 동작하였습니다.)

<img width="500" alt="2" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/9c33d1ae-612d-4472-baa3-e413f462b818">

(출처: https://developer.apple.com/documentation/uikit/uiscreen/1617815-main)

<img width="400" alt="3" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/09d1d5f5-c8e5-434c-94e7-a5f969e31360">

(Xcode 에서 해당 프로퍼티에 접근하면 경고 메시지를 확인할 수 있었습니다.)

기존에는 다음과 같이 디바이스 스크린에 접근할 수 있었습니다.

```swift
UIScreen.main.bounds.width
```

(배경을 살펴보자면 사용할 수 있는 Scene 들이 많아졌습니다. 그래서 해당 Scene 들에 접근하는 방법으로 대체하였습니다.)

wwdc 일부를 발췌해보기

```swift
// 개발자 문서에서 예시를 든 코드
view.window?.windowScene?.screen.bounds

// UIApplication 을 활용한 접근
private let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene
windowScene?.screen.bounds
```

진행 중인 프로젝트를 확인해보니 이렇게 많은 곳에서 아직 UIScreen.main 을 사용하고 있었고, 이에 기여해보고자 하였습니다.

<img width="300" alt="4" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/1d8d1250-7c20-4ef5-82bc-f4c79876ac57">

## 👉 초기에 생각했던 유의할 점

1. UIScreen.mian 타입 프로퍼티는 iOS 16까지만 사용하고 곧 deprecated 된다.
2. UIWindowScene 은 iOS 13부터 사용할 수 있습니다. 
3. 오픈 소스 라이브러리의 최소 버전을 확인하여 OS분기처리가 예상됩니다.

- PinLayout
    - Cocoapods deployment_target : 12.0
- YPImagePicker
    - Cocoapods deployment_target : 12.0
- Kingfisher
    - Cocoapods deployment_target : 12.0
- lottie-ios
    - Cocoapods deployment_target : 11.0
- KakaoSDKCommon
    - 배포를 위한 목적으로 운영하기 때문에 이슈와 PR 을 지원하지 않는다고 합니다.
- IQKeyboardManager
    - Cocoapods deployment_target : 11.0
- PryntTrimmerView
    - Cocoapods deployment_target : 9.0

확인해보니 대부분이 iOS 12 를 최소 지원 버전으로 가지기 때문에 iOS 13 이후에만 UIWindowScene 을 사용하도록 코드를 수정해보겠습니다.

또한, 모든 라이브러리에서 아직 UIScreen.main 관련된 이슈와 PR 이 없기 때문에 작성해보겠습니다.

## 👉 PR 작성 및 Issue 작성

(Issue 템플릿을 사용하고 있거나 PR 템플릿 주석에서 이슈를 예시로 드는 레포지토리에서는 이슈를 만들어서 진행하였습니다.)

***다음은 코드리뷰를 거쳐 최종적으로 커밋된 코드입니다.(2023.11.21 기준)***

- PinLayout

https://github.com/layoutBox/PinLayout/pull/275/files

<img width="700" alt="1" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/280b6a25-6b9d-4f14-8e7d-1faac8b93712">

- YPImagePicker

https://github.com/Yummypets/YPImagePicker/pull/805

<img width="700" alt="2" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/800c3301-f5f3-4092-905e-d2fdfb308627">

- Kingfisher

https://github.com/onevcat/Kingfisher/pull/2152

<img width="700" alt="3" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/1b8fc8e4-eb8d-4941-8eb0-20d26bc75c7a">

- lottie-ios

https://github.com/airbnb/lottie-ios/pull/2216

<img width="700" alt="4" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/3ebc2ae7-691b-48cd-aec9-ecd0d4918d65">

- IQKeyboardManager

https://github.com/hackiftekhar/IQKeyboardManager/pull/1989

<img width="700" alt="5" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/4b9ce298-1b5b-483e-bcad-65059a8e3575">

- PrynTrimmerView

https://github.com/HHK1/PryntTrimmerView/pull/93

<img width="700" alt="6" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/d23b008b-8dd1-41e3-8dca-7eaab9421739">

## 👉 코드 리뷰 - lottie-ios

lottie-ios 에서 코드 리뷰가 달렸다..!

어렴풋이 내 PR 에 과연 리뷰를 달아줄까 반신반의하였고, UIScreen.main deprecated 된다는데? 대안은 이거야! 라고 하는 의견에 얼마나 적극적인 코드 리뷰가 달릴까 싶어서 코드의 품질보다 PR 작성에 열을 올렸었던 것 같다.

사실 설명 없이도 이해할 수 있는 것이 가장 좋은 질의 코드 아닐까 싶다.

그러던 중 내가 놓친 부분들을 리뷰 받았다.*(이제라도 잘할게요!🥺)*

- nil 인 경우(사실 없지 않을까! 하지만 값이 옵셔널이기때문에 force binding 을 하고 싶지는 않았다.).zero 를 반환하도록 했었다. 더 나은 의견이 없는지 물었고, 다음과 같이 답을 남겨두었습니다.

<img width="700" alt="7" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/31ae29a6-e12c-4f10-91b3-109518dd97e1">

- 해당 라이브러리는 tvOS 에서도 사용되었고, [main](https://developer.apple.com/documentation/uikit/uiscreen/1617815-main)문서를 다시 살펴보니 다른 os 들도 이제서야 눈에 들어왔다.(iOS 2.0-16.0, iPadOS 2.0-16.0, Mac Catalyst 13.1-16.0, tvOS 9.0-16.0 Deprecated)

<img width="700" alt="8" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/7baada4d-5be7-4346-931d-1e86315ade05">

이후 다른 PR도 필요한 경우에 수정하게 되었습니다.

마지막으로, CI 과정에서 Lint 에서 fail 을 받았고 아래를 참고해서 code format 을 맞춘 뒤에 진행하면 되겠습니다.(리뷰어가 저는 커밋해주었습니다.)

(참고 : https://github.com/airbnb/lottie-ios/blob/master/README.md#contributing)

```swift
// % sudo gem install bundle
// bundle 설치
 
% bundle exec rake format:swift
```

- blocks are indented with 4 spaces instead of 2.

<img width="700" alt="9" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/c86f036e-1893-415a-92fe-b69bd035a20e">

이후에 Test 과정에서 스냅샷을 확인한 후에 리뷰어가 window 가 nil 되는 경우를 가져왔다..!(와우 없을 줄 알았는데 이렇게 코딩하면 안되겠다는 생각이 팍 들었다.)리뷰를 남겨주었고 이를 함께 해결하기도 하였습니다.

과정이 궁금하시다면 아래 PR 에서 확인할 수 있습니다!

https://github.com/airbnb/lottie-ios/pull/2216

## 👉 느낀 점

당장의 에러가 아니기 때문에 PR 에 대한 관심이 낮을 수도 있다고 생각했다.

나 역시나 해당 이슈를 가볍게 생각해서 반신반의하며 PR 을 올렸던 것 같다. 이에 진심으로 리뷰해주자 조금 부끄러웠다.

그런데 또 뭐 어떤가 서로 의견을 교환하며 놓친 부분들을 채우고 더 나은 코드로써 보답하면 되지 않나 싶었다.

코드의 품질이 우선인데 너무 쫄아있었던 것이 아니었나싶다. 평소 내 프로젝트에 코드를 작성하듯이 적극적으로 코드를 작성하는 자세를 가져도 아무런 문제가 없다고 느꼈다.
