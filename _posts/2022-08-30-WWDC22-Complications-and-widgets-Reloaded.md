---
title:  "WWDC22) Complications and widgets: Reloaded"
categories:
- iOS

date:   2022-08-30  22:05:00 +0900
author_profile: false
---
[Complications and widgets: Reloaded - WWDC22 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2022/10050/)

***본 글은 WWDC 를 보고, 번역 및 요약 그리고 실행해보는 스터디 프로젝트의 일환입니다.***

<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/187438030-754672f8-2e8e-4739-a113-5e5bb62bd34d.png">

### 내용

- watchOS Lock Screen 과 complications 로 액세서리 위젯을 쓸 수 있는 WidgetKit 의 API 추가사항
- SwiftUI 추가사항

## 🔥 Overview

<img width="700" alt="2" src="https://user-images.githubusercontent.com/69136340/187438061-3dee177b-46d3-4ab1-ab95-1696f4c224d8.png">

- 컴플리케이션의 타임라인
- 위젯과 컴플리케이션에 입힐 색에 대한 API
- Project setup - 직접 위젯을 만드는 방법과 기존 위젯을 watchOS 에 확장하여 옮기는 작업
- Making glanceable views - 더 작은 view 의 대부분을 만드는 방법
- 위젯이 보일 여러 privacy 환경에 대한 이야기

## 🔥 Complication timeline

<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/187438136-15ce03e8-5ba2-44a6-8c04-3f5850284407.png">

컴플리케이션은 watchOS 의 주요 요소로 시계 페이스에 quick, glaceable information 을 보여줍니다. 또한, 즉시 접근할 수 있는 high-value 의 정보를 전달하고 탭 하여 앱의 관련 위치로 이동시켜줍니다.

<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/187438153-6b893aa5-ffdd-4e0d-855c-3e187926db36.png">

watchOS 2에서는 ClockKit 가 컴플리케이션을 만들 수 있도록 하였고 그때부터 시작되었습니다.

풍부한 컴플리케이션과 그래픽 컨텐츠와 새로운 families 세트와 함께 watchOS 5 가 소개되었습니다.

SwiftUI 컴플리케이션과 다수의 컴플리케이션은 watchOS 7에서 소개되어 컴플리케이션에 다양한 옵션을 제공하며 발전시킬 수 있게 해주었습니다.

<img width="700" alt="5" src="https://user-images.githubusercontent.com/69136340/187438574-ec02b30c-f327-413f-8d05-81adaa4d78a1.png">

오늘 컴플리케이션은 WidgetKit 과 재창조되어 iOS 16 과 watchOS 9 의 WidgetKit 으로 두 플랫폼에서 훌륭한 컴플리케이션과 위젯을 구축해 코드를 한 번 쓰면 기존 홈 화면 위젯과 인프라를 구축할 수 있게 됩니다.

<img width="700" alt="6" src="https://user-images.githubusercontent.com/69136340/187438232-bc7cd1fd-abf5-4da6-bf47-da2fca4685c5.png">

이를 위해서  기존 WidgetFamily 유형에 accessory 라는 접두어를 가진 위의  새로운 위젯군을 추가했습니다. 

### 1️⃣ accessoryRectangular

<img width="700" alt="7" src="https://user-images.githubusercontent.com/69136340/187438257-afe91bc2-b011-4ed7-8433-be579344cc31.png">

새로운 accessoryRectangular 는 기존의 ClockKit 의graphicRectangular 와 유사하게 텍스트의 여러라인이나 작은 차트를 표현하는데 쓸 수 있습니다. 

### 2️⃣ accessoryCircular

<img width="700" alt="8" src="https://user-images.githubusercontent.com/69136340/187439105-0a0c5823-9e5c-4065-82e9-5045532210da.png">

accessoryCircular 는 간단한 정보, progress view 에 좋습니다. 이 또한 ClockKit 의 graphicCircular 를 대체합니다.

### 3️⃣ accessoryInline

<img width="700" alt="9" src="https://user-images.githubusercontent.com/69136340/187438712-aecb64e5-5be6-4e8d-9e0c-4784c4f09228.png">

accessoryInline 는 텍스트만 되는 슬롯이기 때문에 많은 페이스와 iOS 시간 위에 보여집니다.

슬롯 사이즈는 다양한 사이즈로 존재하는데 어떻게 최선으로 이용할지는 뒤에서 알아보겠습니다.

### 4️⃣ accessoryCorner

<img width="700" alt="10" src="https://user-images.githubusercontent.com/69136340/187438750-da621fe8-99eb-49af-8f0d-fea1c8afafab.png">

watchOS 를 특정짓는 것은 새로운 accessoryCorner 입니다. 위젯 콘텐츠의 작은 원을 게이지 및 텍스트와 섞습니다.

앞으로의 이야기는 iOS 와 watchOS 사이의 공통 families 에 초점을 맞춘 것입니다.

새 watchOS 의 자세한 사항을 더 알고 싶다면 해당 세션을 시청하면 됩니다.

<img width="700" alt="11" src="https://user-images.githubusercontent.com/69136340/187439178-3ef9ac73-daeb-44ad-8784-2b0f2f3a3536.png">

색상과 렌더링 모드에 대해서 이야기해봅시다.

## 🔥 Colors

accessory widget 이 몇 가지 다양한 외관을 보여주는 것을 눈치챘을지도 모릅니다.

(앞서 iOS16 에서 새롭게 보여지는 반투명 위젯들)

시스템은 accessory family widget 들의 모습을 제어하고 우리들은 rendering style 에 적응하는 걸 도와줄 도구를 주었습니다.

여기 위젯이 보여졌을 때 다른 세 가지 rendering mode set 이 있습니다. 왼쪽에서부터 Full color, accented, vibrant 입니다.

<img width="700" alt="12" src="https://user-images.githubusercontent.com/69136340/187439202-49508e9e-a349-4253-8422-5824c6807791.png">

WidgetRenderingMode 를 도입해 세 가지 다른 표현을 합니다.

<img width="700" alt="13" src="https://user-images.githubusercontent.com/69136340/187439385-c91fb43a-807a-4b3f-883b-5b47a7117433.png">

아래의 사용된 예시를 보게되면 widgetRenderingMode keypath 를 활용하여 Environment 에서 이 값에 접근할 수 있습니다. 이후 코드에서 콘텐츠가 적합해 보이도록 switch 문에서 변경할 수 있습니다.

### 1️⃣ Full color

<img width="700" alt="14" src="https://user-images.githubusercontent.com/69136340/187439289-3c103b4f-9917-43aa-8848-072249f7f5a5.png">

watchOS 의 full color 모드에서 콘텐츠는 여러분이 지정한대로 보여집니다. 

기존의 많은 컴플리케이션은 날씨 게이지의 변화나 activity ring 의 색깔처럼 다채로운 색을 띕니다.

### 2️⃣ accented

<img width="700" alt="15" src="https://user-images.githubusercontent.com/69136340/187439532-3c97b832-000c-461f-85bd-9c508e72c048.png">

accented 모드에서는 독립된 색의 두 그룹으로 나뉘게 됩니다. 두 그룹은 차분한 색으로 원래의 불투명도만 보존합니다.

우리는 시스템이 어떻게 .widgetAccentable() view modifier 로 view 를 그룹짓는지 알 수 있었습니다. 혹은 위젯 렌더링 모드 environment 값에 근거해 flattend 되었을 때 완벽하게 보이도록 컨텐츠를 전환하는지 알 수 있었습니다.

(맨 좌측이 Accented)

<img width="700" alt="16" src="https://user-images.githubusercontent.com/69136340/187439626-3250f5de-29e3-40ce-9c88-e8a535c166fb.png">

일부는 검정 배경인 반면에 다른 건 watchOS 9 에서 full color 배경입니다. 이것이 watchOS 9 의 새로운 점입니다. 

Vibrant 에 대한 이야기를 시작해보겠습니다.

<img width="700" alt="17" src="https://user-images.githubusercontent.com/69136340/187439689-00370fcc-d6dd-40ba-8045-6208efe51bb1.png">

iOS 의 Vibrant 모드에서는 컨텐츠는 흐릿해진 다음 Lock Screen 배경에 맞게 적절한 색으로 바뀝니다.

시스템은 greyscale 의 컨텐츠를 주목할 수 있도록 그려냅니다. 그리고 이것은 환경에 적합하게 나타납니다.

추가적으로 Lock Screen 은 vibrant rendering mode 를 색상으로 물들이도록 설정될 수 있습니다. 밝은 컬러는 결국 주로 불투명해지고 더 밝아집니다. 반면, 어두운 컬러는 배경에서 약간의 광택만으로 흐릿하게 나타납니다.

<img width="700" alt="18" src="https://user-images.githubusercontent.com/69136340/187439759-0ade6bb9-1244-46ea-96e7-cf6b588f0ca6.png">

가독성을 보장하기 위해서 Vibrant 모드에서 투명 컬러는 사용을 피하고 대신 어두운색이나 검정을 사용하여 가독성은 유지하면서 콘텐츠는 눈에 덜 띄게 합니다.

이러한 미묘한 차이인 일관된 배경을 위젯에 주기 위해서 AccessoryWidgetBackground view 를 도입했습니다.

<img width="700" alt="19" src="https://user-images.githubusercontent.com/69136340/187439777-7f376d30-c102-4d3f-8930-59933d4db8fc.png">

background view 는 다양한 위젯 렌더링 모드에서 여러 모습을 가집니다.

## ✅ Project setup

기존 앱인 Emoji Rangers 에 새로운 위젯 그룹군을 추가하려고 합니다.

<img width="700" alt="20" src="https://user-images.githubusercontent.com/69136340/187440067-aab5a15d-08e2-461e-af00-4c990cbf0efb.png">

위와 같이 Widget Extension target 을 프로젝트에 추가하여 시작할 수 있습니다.(Emoji Rangers 는 이미 있기 때문에 새 위젯과 컴플리케이션을 추가하는 것에 대해서 이야기하겠습니다.)

<img width="700" alt="21" src="https://user-images.githubusercontent.com/69136340/187439954-0725ba3f-d1d9-4370-9f54-3a844ec05ee9.png">

Emoji Rangers 프로젝트는 Emoji Rangers 를 추적하고, 홈 화면 위젯 사용으로 Ranger 의 건강과 재충전 시간의 최신 정보를 알려줍니다.

### watch Extension target

기존 iOS target(CharacterDetailExtension) 과 코드를 공유하는 새로운 watchOS target 을 추가할 것입니다.

<img width="700" alt="22" src="https://user-images.githubusercontent.com/69136340/187440201-7c442013-ef34-4b7d-b096-64bca37c0780.png">

CharacterDetailExtension 인 widget extension target 을 복사하여 이름을 바꾸었습니다.

Bundle Identifier 를 변경하고 Base SDK 도 변경하여 watch App 을 위한 extension 으로 변경하겠습니다.

<img width="700" alt="23" src="https://user-images.githubusercontent.com/69136340/187440226-5517b56d-f03e-4c4b-b333-cbbea130a12c.png">

WatchApp 에 새로운 extension(CharacterDetailExtension watch)을 임베드 하겠습니다.

<img width="700" alt="24" src="https://user-images.githubusercontent.com/69136340/187440364-b90591ce-4a2d-4259-82d0-b9466a540494.png">

(이 과정은 기존에 만든 iOS target 과 코드를 공유하기 위함이다.

자, 이제 watchOS 를 구축하는 코드를 작성해봅시다.

<img width="700" alt="25" src="https://user-images.githubusercontent.com/69136340/187440399-d8b01874-08e2-4e48-b9a0-fa512d950777.png">

EmojiRangerWidget 코드를 살펴보면 timeline provider 를 볼 수 있는데 시스템이 콘텐츠를 다시 로드할 때 사용하는 것입니다.

이미 iOS 홈 화면 위젯을 지원합니다. 우리는 small 과 medium family 에 새 그룹군을 추가할 겁니다.

<img width="700" alt="26" src="https://user-images.githubusercontent.com/69136340/187440584-b65afc09-45e7-46df-bb1f-36a1fb0c2d8f.png">

accessory families 가 추가되었습니다.

추가적으로 system families 는 watch 에서는 사용이 불가능해서 supprtedFamilies 를 지정하기 위해 플랫폼 매크로를 사용해야합니다.

<img width="700" alt="27" src="https://user-images.githubusercontent.com/69136340/187440655-1879092f-2bee-49ac-8977-5e2a5ec94d57.png">

preview provider 에 새 families 를 위한 preview 를 추가해봅시다.(길이가 길어 코드로 대체합니다.)

```swift
EmojiRangerWidgetEntryView(entry: SimpleEntry(date: Date(), relevance: nil, character: .spouty))
                .previewContext(WidgetPreviewContext(family: .accessoryCircular))
                .previewDisplayName("Circular")
EmojiRangerWidgetEntryView(entry: SimpleEntry(date: Date(), relevance: nil, character: .spouty))
                .previewContext(WidgetPreviewContext(family: .accessoryRectangular))
                .previewDisplayName("Rectangular")
EmojiRangerWidgetEntryView(entry: SimpleEntry(date: Date(), relevance: nil, character: .spouty))
                .previewContext(WidgetPreviewContext(family: .accessoryInline))
                .previewDisplayName("Inline")

#if os(iOS)

// ...

#endif
```

다음으로 watchOS 를 성공적으로 빌드하기 전에 새로운 intentRecommendation API 를 구현해야 합니다.

Intent 가 iOS 의 위젯 편집 UI 에서 configurable 할 때 watchOS 에서는 preconfigured list 를 제공해야합니다. 이는 IntentTimelineProvider 에서 새로운 recommendations 메소드를 오버라이딩하여 제공할 수 있습니다.

### 🔥 recommendations

[Apple Developer Documentation - recommendations](https://developer.apple.com/documentation/widgetkit/intenttimelineprovider/recommendations()-5ltr5)

(intent 는 아래처럼 위젯을 꾹 눌러 위젯 편집을 통해 제공하는 옵션을 의미합니다. 즉, configurable widget 이라는 이야기 입니다. widget 에서는 configurable 하도록 intent 를 설정할 수 있습니다.

반면, watchOS 에서는 preconfigured list 를 제공해야 하는데 이는 IntentTimelineProvider 의 새로운 recommendations 메소드를 오버라이딩하여 제공할 수 있다는 말입니다.

그리고 preconfigured list 는 컴플리케이션에서 사용자와 가장 관련성이 높은 데이터를 표시하기위한 목적이고 사용가능한 컴플리케이션 목록에서 사용자에게 추천됩니다.

즉, 애플워치에서 컴플리케이션을 편집할 때 사용자에게 추천되는 preconfigured complications 을 만드는 역할입니다.)

<img width="500" alt="28" src="https://user-images.githubusercontent.com/69136340/187440787-c7491a06-4399-4af6-b070-2ef4a0af331c.png">

[Apple Developer Documentation - Making a Configurable Widget](https://developer.apple.com/documentation/WidgetKit/Making-a-Configurable-Widget)

이 문서에는 configuable 한 widget 을 만들 수 있고, watch 에 preconfigured 한 컴플리케이션을 제공하는 방법에 대해서 소개하고 있습니다.

---

<img width="700" alt="29" src="https://user-images.githubusercontent.com/69136340/187440899-5e96d7d4-6965-4b1d-9b33-8e7d10b73fb4.png">

다음과 같이 오버라이딩해주겠습니다.

<img width="700" alt="30" src="https://user-images.githubusercontent.com/69136340/187440916-3fa4ea33-77a0-4512-b7bb-b130a5286e19.png">

preview 의 widget 을 확인해보겠습니다.

<img width="700" alt="31" src="https://user-images.githubusercontent.com/69136340/187441078-dd9e3574-862a-428c-91c4-2151eadbc43f.png">

small widget 에 대한 컨텐츠가 새 폼팩터(accessoryCircular) 안에 잘 맞지 않습니다. 새로운 widget families 는 iOS widgets 보다 작기 때문에 여러분의 컴플리케이션 컨텐츠를 고려해야 합니다.

자, 이제 컴플리케이션을 눈에 띄게 할 새로운 view 에 대해서 이야기 해봅시다. 

## ✅ Making glanceable views

### 1️⃣ accessoryCircular

zaccessoryCircular case 에 대한 코드도 추가해 봅시다. 

<img width="700" alt="32" src="https://user-images.githubusercontent.com/69136340/187441233-5602a1ac-13e7-4856-9970-1bcd675c2f68.png">

이 progressView 는 Ranger 가 다시 전투 준비가 되는지 사용자에게 보여주는 역할을 할 것입니다.

문제는 이 progress view 가 통용되도록 활성화하기 위해 연달아 timeline entries 를 많이 요구할 것이라는 점입니다. 

(widget 이 언제 실행되어 정보를 최신화할지 timeline 을 알아야 하기 때문에 timeline entry 가 존재합니다.)

대신! 우리는 SwiftUI 의 새로운 자동 업데이트 ProgressView 를 사용할 수 있습니다. 이로써 Ranger 가 완전히 회복될 날짜 간격을 가지고, 이것은 시스템이 progress view 의 업데이트 유지를 timeline entry 단 하나만 필요하다는 의미입니다.

<img width="700" alt="33" src="https://user-images.githubusercontent.com/69136340/187441290-f5d287c7-8db9-46e6-9c9c-5d53d898b8a6.png">

### 2️⃣ accessoryRectangular

이제 Retangluar family 를 추가한 후, Ractangular preview 를 선택해봅시다.

<img width="700" alt="35" src="https://user-images.githubusercontent.com/69136340/187441446-a01b8360-846b-4890-89dc-07a11b5fc4e3.png">

공간이 있으니 컴플리케이션 스타일의 세 줄 view 를 만들어 봅시다. 캐릭터 이름, 레벨, 완전히 치유될 때까지의 시간으로 구성할 것이고, 캐릭터 이름은 헤드라인 서체와 색상을 조정하는 widgetAccentable modifier 를 추가할 것입니다.

<img width="700" alt="36" src="https://user-images.githubusercontent.com/69136340/187441488-ed37e3e4-2d72-46fe-baf7-dccd6294e199.png">

vibrant 에서는 view 가 근사해 보이는데 watch 의 다른 렌더링 모드에서는 어떻게 보이는지 살펴봅시다.

<img width="700" alt="37" src="https://user-images.githubusercontent.com/69136340/187441550-a9c59965-2c26-4c92-a6d3-0cbae1f66a04.png">

캐릭터 이름이 accent color 를 어떻게 가지는지 볼 수 있습니다. widgets 과 complications 를 편안하게 느끼게 만들고 default font parameter 를 사용하고 font styles 를 사용도록 하는 것이 중요합니다.

<img width="700" alt="38" src="https://user-images.githubusercontent.com/69136340/187441583-47c55a60-1d8a-44e8-83e7-351d6b0273e5.png">

iOS 와 watchOS 는 서체와 사이즈가 다릅니다. iOS 는 regular 디자인이고, watchOS 는 rounded 디자인입니다. 위젯과 컴플리케이션은 다른 것들과 가까이 스크린에 자리할 것입니다. 그래서 일관되게 보이는 font styles Title, Headline, Body, Caption 사용하기를 권장합니다.

<img width="700" alt="39" src="https://user-images.githubusercontent.com/69136340/187441710-d489c7ea-5271-4af6-aec0-5d68f114a7c2.png">

Avatar 를 남은 공간에 추가하였습니다. iPhone 에서는 어떻게 보이는지 살펴봅시다.

<img width="300" alt="40" src="https://user-images.githubusercontent.com/69136340/187441758-1db5365b-6be4-41c2-a1c7-b01cdb3a3ca0.png">

좋습니다! 마지막으로 텍스트 라인과 선택적으로 이미지를 게시하는 accessoryInline 을 추가해 봅시다.

### 3️⃣ accessoryInline

system-defined coloring 과 font 에 따라 그려졌다는 것에 주목해 봅시다.

<img width="700" alt="41" src="https://user-images.githubusercontent.com/69136340/187441895-1feeb70f-0167-43d5-9546-b98c827b1ba0.png">
위의 코드에서 히어로 이름과 재충전 카운트다운을 보여주겠습니다.

그런데 이 텍스트는 watch 슬롯에서 너무 깁니다. 여러분들께 **VeiwThatFits** 를 보여드릴 때입니다.

길이를 축약해서 여러 view 를 지원할 수 있고, ViewThatFits 는 끊거나 잘라내기 없이 가능한 공간에 맞춰서 처음 콘텐츠 view 를 선택합니다.

<img width="700" alt="42" src="https://user-images.githubusercontent.com/69136340/187442015-3c1f09e9-473e-4c90-b9a6-723f9e884de4.png">

너무 길지도 모르겠네요. 세번째 대안을 적용해보겠습니다.

<img width="700" alt="43" src="https://user-images.githubusercontent.com/69136340/187442067-b8f33c09-bc62-4573-a7a6-0ca70efe28f3.png">

이렇게 아바타와 재충전 카운트다운을 통해 보여줄 수도 있습니다.

(*더 알고 싶다면 Compose Custom layouts with SwiftUI 세션을 참고해주세요.)*

## ✅ Privacy

지금까지 위젯과 컴플리케이션의 활성화 상태를 논의했습니다. 하지만, 플랫폼을 넘어서 콘텐츠를 편집 중이거나 밝기가 어두운지를 고려해야 합니다. 

<img width="700" alt="44" src="https://user-images.githubusercontent.com/69136340/187442131-635633a2-cfa7-4e96-86d2-320442a3c23a.png">

iOS Lock Screen 에서 기본 행동은 기기가 잠겨있는 동안에도 셀 왼쪽 상단의 콘텐츠(하이라이트된 셀)를 보여주는 겁니다.

<img width="700" alt="45" src="https://user-images.githubusercontent.com/69136340/187442179-ff17f392-2fa3-4ef8-bad3-4b3b116b523f.png">

이것은 Settings 에서 설정이 가능하고 사용자들은 알림처럼 잠긴 상태에서도 민감한 내용을 삭제할 수 있습니다.

<img width="700" alt="46" src="https://user-images.githubusercontent.com/69136340/187442260-b15d56ca-6542-42fd-ad32-2c5b57ac941b.png">

watchOS 에서는 watch 를 차고 있는 한 기기가 잠금 해제된 상태로 있습니다. 비활성화일 때 watch 는 어두운 밝기 표현의 콘텐츠와 낮은 업데이트 케이던스로 always-on 으로 변합니다.

기본적으로 콘텐츠는 제일 하단 왼쪽의 상태로 삭제와 같은 수정이 되지 않습니다.

<img width="700" alt="47" src="https://user-images.githubusercontent.com/69136340/187442369-136b7bc8-32ad-4d19-ae1c-e3bf07ac331a.png">

잠금 화면에서처럼 사용자들이 always-on 상태에서 컴플리케이션 콘텐츠가 삭제되도록 설정할 수 있습니다.

위의 모든 상태를 고려해 여러분의 컴플리케이션과 위젯이 잘 동작할 수 있도록 해야 합니다.

<img width="700" alt="48" src="https://user-images.githubusercontent.com/69136340/187442474-a950475f-5ded-42e8-91a1-7b5ddd0e4588.png">

watch 에서는 위젯은 always-on 디스플레이 경험을 지원해야 합니다. `\.isLuminanceReduced` environment 값으로 콘텐츠를 늘 always-on 상태에 대응할 수 있도록 할 수 있습니다.

ClockKit 에서 모든 timeline entry 에 always-on 콘텐츠를 준비할 수 있습니다.

<img width="300" alt="49" src="https://user-images.githubusercontent.com/69136340/187442549-e1066339-46c7-48d1-a493-4da80df82f94.png">

always-on 에서 시간 관련 텍스트와 progress view 가 always-on 의 낮은 업데이트 케이던스를 지원하려고 감소한 fidelity mode(정확도 모드)로 바뀔 것입니다. 이 모드를 지원하기 위해 environment 값을 사용하여 시간에 민감한 콘텐츠를 삭제하고, 업데이트 빈도를 낮추기 위해 콘텐츠 최적화하도록 해야 합니다.

<img width="700" alt="50" src="https://user-images.githubusercontent.com/69136340/187442629-e2aa4202-b314-4d87-aa10-df7119898ef1.png">

redaction(민감한 사항 편집)에 관한 이야기를 해볼게요. 기본적으로 privacy mode 는 TimelineProvider 가 생성하는 placeholde view 의 redaction 버전을 보여줄 것입니다.

일부 element 가 민감하고 다른 element 는 redacted 할 필요가 없다면 redacted view 만 표시하기 위해 `.privacySensitive` modifier 를 사용할 수 있습니다.

이 예시에서 위젯의 heart rate 를 redact 했지만 이미지는 unredact 된 채로 남습니다.

<img width="700" alt="51" src="https://user-images.githubusercontent.com/69136340/187442699-a9616fe3-3671-48da-ab2c-d4bf4b0ff99f.png">

SwiftUI 의 새로운 기능을 더 알고 싶다면 세션을 시청해주세요.

<img width="700" alt="52" src="https://user-images.githubusercontent.com/69136340/187442737-aabbf8f8-c2aa-41cc-bf2d-04a1b0261c68.png">
