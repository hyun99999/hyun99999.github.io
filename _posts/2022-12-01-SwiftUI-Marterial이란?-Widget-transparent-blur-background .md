---
title:  "SwiftUI) Marterial 이란? (Widget transparent/blur background..?) "
categories:
- iOS

date:   2022-12-01  18:47:00 +0900
author_profile: false
---
위젯의 배경에 투명도를 부여해서 단순 불투명한 배경을 만드는 것이 아니라 아래와 같은 블러처리 같은 느낌을 구현해보고자 했습니다.

(애초에 다크모드에 따른 위젯의 기본 배경도 변경되고 이는 바꿀 수 없습니다. 대신, 위에 색을 가진 뷰를 하나 얹는 것이죠...)

<img width="200" alt="1" src="https://user-images.githubusercontent.com/69136340/205013376-5bd88a3b-0938-4b61-885a-4835bf6e3a02.png">

결론적으로 얘기하면 **써드파티의 앱이 구현하도록 지원하지 않고 있습니다.**

비슷한 질문은 여러 포럼에서 등장하였지만, 지원하지 않는 기능라고 답변을 주고 있었습니다. 뿐만 아니라 위젯의 배경자체를 투명하게 만들 수 없었습니다.

(위젯 자체의 기본 배경을 라이트모드는 하얀색, 다크모드는 검정색으로 지원하고 있었습니다. 또한, background 와 관련된 modifier 를 어디에 위치시키든 조정할 수 없었습니다🥲)

[Blurred widget background - SwiftUI - Hacking with Swift forums](https://www.hackingwithswift.com/forums/swiftui/blurred-widget-background/2680)

[WidgetKit background like the syst... | Apple Developer Forums](https://developer.apple.com/forums/thread/651040)

[WidgetKit blur](https://developer.apple.com/forums/thread/655990)

그러던 중 위젯의 배경을 블러처리할 수는 없지만, 다른 뷰 위에 있는 뷰에게 blur effect 를 줄 수 있는 기능을 찾게되어서 소개해드리려고 합니다.

***간단하게 이해하고 들어가보자면, view 와 background 사이에 translucent layer 와 같은 material 을 넣게 되는 개념입니다.***

그렇다면 Marterial 개념에 대해서 개발자 문서를 참고해서 알아봅시다!

## 👉 [Marterial](https://developer.apple.com/documentation/swiftui/material)

A background material type.(iOS 15.0+…)

```swift
struct Material
```

### Overview

background(_:ignoresSafeAreaEdges:) modifier 를 사용하여 material 을 추가하여 뒤에 나타는 뷰에 blur 효과를 적용할 수 있습니다.

```swift
ZStack {
    Color.teal
    Label("Flag", systemImage: "flag.fill")
        .padding()
        .background(.regularMaterial)
}
```

위의 예시에서, ZStack 은 청록색 위에 label 을 레이어 합니다. background modifier 는 label 아래에 material 를 삽입하여 padding 을 포함하여 label 을 덮는 배경 부분을 흐리게 만듭니다.

(좌측은 light, 우측은 dark)

<img width="700" alt="스크린샷 2022-12-01 오후 6 13 31" src="https://user-images.githubusercontent.com/69136340/205015095-6e08767c-2f63-4836-8244-2f793dd72d80.png">

**material 은 view 가 아니지만, modified view 와 background 사이에 translucent layer 를 삽입하는 것과 같습니다.**

<img width="400" alt="2" src="https://user-images.githubusercontent.com/69136340/205015160-4c457e20-be99-401c-839d-c0da5c77f916.png">

여기서 중요한 점은 material 이 제공하는 blur effect 는 단순한 불투명도가 아닙니다. 대신, heavily frosted glass effect 를 생성하는 platform-specific blending 을 사용합니다. 

이는 이미지와 같은 복잡한 배경을 사용하면 더 쉽게 확인할 수 있습니다.

```swift
ZStack {
    Image("chili_peppers")
        .resizable()
        .aspectRatio(contentMode: .fit)
    Label("Flag", systemImage: "flag.fill")
        .padding()
        .background(.regularMaterial)
}
```

<img src="https://user-images.githubusercontent.com/69136340/205015202-5c740269-c56b-4b28-9cef-c6bb57b59165.png" width ="300">

physical materials 의 경우, 배경색이 통과하는 정도는 thickness 에 따라 다릅니다. 이 효과 또한 light 와 dark appearance(다크모드 유무)에 따라 달라집니다.

<img width="400" alt="4" src="https://user-images.githubusercontent.com/69136340/205015231-bf5ae5ef-624a-40b0-b615-33f127bb85c3.png">


만약에 특정 모양의 material 이 필요한 경우, `background(_:in:fillStyle:)` modifier 를 사용할 수 있습니다.

아래 코드처럼 모서리가 둥근 material 을 만들 수 있습니다.

```swift
ZStack {
    Color.teal
    Label("Flag", systemImage: "flag.fill")
        .padding()
        .background(.regularMaterial, in: RoundedRectangle(cornerRadius: 8))
}
```

<img src="https://user-images.githubusercontent.com/69136340/205015382-09ee0e59-79c6-4ce7-828f-3760598b34a4.png" width = "500">

material 을 추가할 때 foreground elements 는 대비를 향상시키는 fregbround 와 background color 의 컨텍스트별 혼합인 vibrancy(생동감) 을 나타냅니다. 하지만, `foregroundStyle(_:)` 을 사용하여 hierarchical 스타일을 제외하고, custom foreground style 을 설정하면 vibrancy 가 비활성화됩니다.

<img width="700" alt="스크린샷 2022-12-01 오후 6 13 20" src="https://user-images.githubusercontent.com/69136340/205015553-3ce0521b-fc96-4475-ad64-611b57bf967e.png">
