---
title:  "iOS) dimming view 에 대해서"
categories:
- iOS

date:   2022-07-20  01:57:00 +0900
author_profile: false
---
`largestUndimmendDetentIdentifier` 프로퍼티에 대해서 공부하면서 개발자 문서에 이야기하는 dimming view 의 개념에 대해서 이해해보았다.

그렇다면 먼저, `largestUndimmendDetentIdentifier` 에 대해서 알아보자!

![1](https://user-images.githubusercontent.com/69136340/179747330-419ec1b0-08b4-4e0c-a079-65aab709f10a.png)

기본값은 nil 이고, 설정한 detent 보다 큰 detent 에만 dimming view 를 추가하려면 이 프로퍼티를 설정하면 된다. 그렇다면 설정한 detent 와 같거나 작은 detent 는 dimming view 를 가지지 않는다는 것인데 그것이 어떤 것을 의미하는지 HIG 와 `largestUndimmendDetentIdentifier` 프로퍼티 설정을 통해 생각해보자!

## dimming view 에 대해서 생각해보자!

HIG 가 제시하는 iOS 의 Sheet 의 가이드라인에 대해서도 읽고 시작하면 좋을 것 같다.

[HIG - Sheets](https://developer.apple.com/design/human-interface-guidelines/components/presentation/sheets/)

dimming view 는 투명도 조절 뷰 라고 해석할 수 있다. 즉, 투명도를 가진 뷰이다.

dimming view 가 없다는 것은 sheet 주변이 undimmend area 라는 것이다. 그래서 dimmig view 는 자연스럽게 다음의 동작을 가능하도록 한다.

![](https://user-images.githubusercontent.com/69136340/179747319-babcf52f-ed16-4ab1-b321-f589489ab568.gif)

`largestUndimmendDetentIdentifier` 를 medium 으로 설정하면 아래와 같이 medium sheet 에서는 dimming view 가 제거된다.

![](https://user-images.githubusercontent.com/69136340/179747496-dd16dc8c-9823-465c-933e-0fbb7d245998.gif)

이를 통해 iOS 에서 dimming view 의 역할에 대해서 다음과 같이 생각해볼 수 있었다.

-   dimming view 가 없다는 것은 nonmodal 처럼 user interaction 이 반영된다. underneath 의 컨텐츠와 상호작용할 수 있다.
-   기본적으로 시스템은 다음과 같이 사용자가 탭하면 시트를 닫는 noninteractive 한 dimming view 를 추가한다. dimming view 가 없다면 탭을 통해서 시트가 내려가지 않는다.

출처:

[Sheets](https://developer.apple.com/design/human-interface-guidelines/components/presentation/sheets/)

[How to present a Bottom Sheet in iOS 15 with UISheetPresentationController | Sarunw](https://sarunw.com/posts/bottom-sheet-in-ios-15-with-uisheetpresentationcontroller/)

_**추가적인 의견과 다른 의견도 환영입니다. 🙂**_
