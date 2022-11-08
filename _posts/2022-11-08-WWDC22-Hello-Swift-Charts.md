---
title:  "WWDC22) Go further with Complications in WidgetKit"
categories:
- iOS

date:   2022-11-08  20:43:00 +0900
author_profile: false
---
[Go further with Complications in WidgetKit - WWDC22 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2022/10051)

_**본 글은 WWDC 를 보고, 번역 및 요약 그리고 실행해보는 스터디 프로젝트의 일환입니다.**_

<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/188642836-f2245b51-6ab9-40a2-9333-2c3e8ee6cadf.png">

### 내용

- watchOS 에서만 가능한 widgetKit 의 기능과 ClockKit  컴플리케이션을 WidgetKit 에 마이그레이션 하는 방법

[Developer Apple - widgets](https://developer.apple.com/kr/widgets/)

> 이제 WidgetKit을 사용하여 iPhone 잠금 화면의 위젯과 watchOS의 컴플리케이션을 생성할 수 있습니다.


> 이제 WidgetKit을 사용하여 Apple Watch용 컴플리케이션과 iPhone용 잠금 화면의 위젯을 빌드하고, … iOS 16 및 watchOS 9용 코드를 한 번 작성하고 기존의 홈 화면 위젯에 인프라를 공유해 보십시오.


## ✅ Overview

<img width="700" alt="2" src="https://user-images.githubusercontent.com/69136340/188643118-ac64770d-e51b-439c-8368-fb8005120141.png">

- watchOS 에서만 가능한 WidgetKit 기능
- ClockKit complications 을 WidgetKit 에 마이그레이션 하는 방법

### Coffe Tracker sample app

- 하루 종일 물을 제외한 음료 섭취 횟수를 기록해 시간이 지날수록 쌓이는 체내 카페인량을 추적하는 앱

(coffe tracker 샘플 앱을 예시로 세션을 진행하겠습니다.)

## ✅ Unique to watchOS

**🧑‍🏭 :** watchOS 에서만 나타나는 특징을 살펴봅시다.

<img width="637" alt="3" src="https://user-images.githubusercontent.com/69136340/188643143-1f7147fe-5d21-478e-a19c-4dc007c44126.png">

***iOS 16 에서는 컴플리케이션 스타일 위젯을 iPhone 잠금 화면에 가지고 왔으며 watchOS 9 에서는 WidgetKit 을 watch complications 에 가져왔습니다.***

<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/188643214-646009a8-ea9b-4e48-be05-1c7f1d180815.png">

- watch specific family

watch face 에서는 screen corner 에 독특한 컴플리케이션을 구성해 보았는데요. 이때문에 accessoryCorner 라는 특유의 WidgetKit family 가 필요합니다.

- Auxiliary(보조자) content

<img width="250" alt="5" src="https://user-images.githubusercontent.com/69136340/188643260-0188ad2b-1dca-4d89-b41e-1e96f06e7e58.png">

unique 한 표현은 보조 콘텐츠로 SwiftUI 뷰에서 구체화됩니다. 즉, 워치 페이스에 의해 렌더링됩니다.

<img width="250" alt="6" src="https://user-images.githubusercontent.com/69136340/188643327-2e9db0fd-3f18-444f-bd8e-e0e518120bf9.png">

코너의 circular 부분은 표준 SwiftUI 렌더링으로 보조 콘텐츠는 코너의 곡선 부분입니다.

<img width="250" alt="7" src="https://user-images.githubusercontent.com/69136340/188643361-c685d197-4cca-43d8-bdf5-b6551446a805.png">

또는 Infograph 페이스의 다이얼이 있습니다.

- Mutiple representations

<img width="700" alt="8" src="https://user-images.githubusercontent.com/69136340/188643409-fc5e521b-7a6b-406a-9ea0-fb69244e67d4.png">

acessoryInline family 는 독특한 행동을 합니다. 페이스에 따라 렌더링되는 방식이 다양합니다. 어떨때는 평평하고, 어떨때는 다이얼에 맞게 곡선형입니다.

## ✅ How to Support Unique Features

iOS 16에서는 complication-styled widget families 로 accessoryRectangular, accessoryCircular, acessoryInline 외에도 accessoryCorner 라는 family 를 개발하였습니다.

<img width="700" alt="9" src="https://user-images.githubusercontent.com/69136340/188643472-c3f9b22e-6d57-4bd8-b000-6de554f280db.png">

크고 둥근 콘텐츠인 하단에 나타나는 지도, 심장박동을 나타내거나 상단 코너에 나타나는 커피 추적기나 달 형상처럼 곡선 label 이나 게이지가 있는 작고 둥근 콘텐츠로 나타납니다.

inner auxiliary content 를 보여주는 여부를 제어하기 위해 watchOS 9에는 새로운 view modifier 가 추가되었습니다.

## ✅ Building a Corner Complication

<img width="700" alt="10" src="https://user-images.githubusercontent.com/69136340/188643530-552a3436-90bf-42c4-85e8-640d827a2f44.png">

- SF Symbol 과 background 를 갖춘 ZStack
- SwiftUI 콘텐츠는 다른 코너 컴플리케이션의 디자인과 맞추기 위해서 자동으로 원 모양으로 고정됩니다.

### ❗️ AccesoryWidgetBackground()

(출처: WWDC22 Complications and widgets: Reloaded)

background view 는 다양한 위젯 렌더링 모드에서 여러 모습을 가집니다.

<img width="700" alt="11" src="https://user-images.githubusercontent.com/69136340/188643609-05f9f91d-de26-4dfd-b199-1c68093b0dbb.png">

---

### 🦉 New in watchOS 9.0

<img width="700" alt="12" src="https://user-images.githubusercontent.com/69136340/188643693-96fa665b-cef8-4ca0-b823-3756f507613d.png">

- inner curved content 를 추가하기 위해서 새로운 watchOS 9 의 `widgetLabel` 를 사용.

워치 페이스는 family 와 워치 페이스 스타일에 적합한 제어를 끌어내기 위해 modifier 의 contents 를 추출합니다. 그리고 circular content 가 자동으로 축소되어 공간을 만듭니다.

**🧑‍🏭 : accessoryCorner 에서는 text, gauge 구체화가, widgetLabel 에서는 progressView 구체화가 가능합니다.**

## ✅ How `WidgetLabel` used on the AccessoryCircular

accessoryCorner 에서 widgetLabel 을 유일하게 지원하지 않습니다. accessoryCircular 에서는 어떻게 사용될까요?

<img width="700" alt="13" src="https://user-images.githubusercontent.com/69136340/188643769-5840eb70-a2a7-4da5-bda6-e6a8732d4872.png">

Infograph 워치 페이스에서는 corner complications 외에도 다이얼 내부에 4개의 circular complications 가 있습니다.

**🧑‍🏭 : 중간 상단의 커피 추적기 컴플리케이션은 우리가 이전에 만났던 코너 컴플리케이션과 매우 유사합니다. 다이얼 안쪽 텍스트에 어떻게 추가할 까요?**

<img width="700" alt="14" src="https://user-images.githubusercontent.com/69136340/188643860-c8d64239-1d51-43d1-967f-eb9b39e7c32d.png">

위의 디자인을 위하여 corner complication 에 있는 widgetLabel 의 Gauge 를 옮기는 것이 더 좋을 것이라 판단했습니다.

<img width="700" alt="15" src="https://user-images.githubusercontent.com/69136340/188643995-987dbc1b-6749-4f89-a36f-d00cc5f0b144.png">

Infograph 의 중앙 상단을 이용하기 위해서 원형 콘텐츠에 맞지 않을 긴 베젤 영역에 추가 텍스트를 넣기 위해 widgetLabel 에 gauge 를 추가하였습니다.

이제 메인 뷰와 텍스트 위에 너무 정보가 많습니다. 

<img width="700" alt="16" src="https://user-images.githubusercontent.com/69136340/188644020-620bebb9-7083-4325-9746-01b29a757343.png">

- circular content 를 커피 컵 심볼로 변환해 이 부분을 지웠습니다.
- 베젤이 없는 circular complication 으로 바꾸면 카페인 정보가 전부 사라집니다.

우리는 두 사례 모두에게 추가할 수 있는 API 가 있습니다.

### 🦉 New in watchOS 9.0

<img width="700" alt="17" src="https://user-images.githubusercontent.com/69136340/188644070-1de2d069-d0bb-40af-94d2-a4ce910d110e.png">

**🧑‍🏭 : showsWidgetLabel 이라는 Environment 프로퍼티를 뷰에 추가하여 컴플리케이션을 업데이트 합니다. 이는 complication 이 widgetLabel 의 콘텐츠가 보이는 워치 페이스에 위치할 때마다 참이 될 것입니다.**

<img width="700" alt="18" src="https://user-images.githubusercontent.com/69136340/188644101-5934e313-4379-49a6-9a5c-4d58df43a58f.png">

showsWidgetLabel 의 값에 따라 조건문을 사용하여 콘텐츠를 바꿀 수 있습니다.

**🧑‍🏭 : accessoryCircular family 가 시계 페이스를 나타내는 두 가지 방법을 보여드렸습니다. 한 가지 더 알고 계셔야 할 방법이 있습니다.**

## ✅ Extra Large

Extra Large 워치 페이스는 시간을 보기에 좋은 방법입니다. 그리고 하나의 큰 circular complication 을 지원합니다.

<img width="700" alt="19" src="https://user-images.githubusercontent.com/69136340/188644142-b3c06850-5cb6-4f70-86cf-ef1f101bb0d3.png">

그리고 accessoryCircular family 를 사용하고 페이스 스타일에 맞게 콘텐츠를 자동으로 확대합니다.

<img width="700" alt="20" src="https://user-images.githubusercontent.com/69136340/188644191-d9e379ec-7044-471e-a25e-ed9cba674dc2.png">

**단, 이때 canvas size 확대를 통해 complication 을 꽉 채우려고 하지 마세요. 콘텐츠는 보통의 circular family 와 동일하되 약간 커야 합니다.**

워치 페이스에서 사용되는 위젯 families 는 accessoryRectangular 와 accessoryInline 도 있습니다. widgetLabel 이 보이는 rectangular 를 가진 페이스는 없습니다. 

accessoryInline family 는 이미 widgetLabel 로 작동하고 있습니다. 워치 페이스는 inline content 의 Images 와 Texts 를 추출하고 페이스 모습에 맞춰 스스로 렌더링합니다.

## ✅ Migration

두 가지 부분으로 나뉩니다.

<img width="700" alt="21" src="https://user-images.githubusercontent.com/69136340/188644241-e15421aa-12cd-48e6-bb6f-d4a60f4f3f6e.png">

- WidgetKit 을 적용할 때 시스템은 새 콘텐츠에 대해 ClockKit 데이터 소스를 묻지 않고 face editing picker 에 새로운 complications 만 보여줄 것입니다.
- WidgetKit 에서 기존의 ClockKit complication 코드를 다시 쓰고 mapping 을 제공함으로써 페이스 적용 컴플리케이션으로 업그레이드하는 방법을 알려줍니다.

### 🦉 New in watchOS 9.0

<img width="700" alt="22" src="https://user-images.githubusercontent.com/69136340/188644276-6180f7ea-3c67-430d-84bb-c31c1e1be9fc.png">

watchOS 9 은 WidgetKit 을 워치에 가져왔을 뿐만 아니라 모든 페이스에 풍부한 컴플리케이션을 업데이트해주었고, 이것은 complication families 의 수를 드라마틱하게 줄여줄 수 있었습니다. 12개에서 4개로요!

<img width="700" alt="23" src="https://user-images.githubusercontent.com/69136340/188644311-29694dd9-54d3-4ddd-897d-d6248493d4bd.png">

- rectangular 와 corner 는 accessoryRectangular 와 accessoryCircular 로 매핑
- 세 개의 graphic circular 스타일인 ClockKit families 는 모두 accessoryCircular Widgetkit family 가 되었다.
- accessoryInline family 는 기존 utilitarianSmallFlat 이나 utilitarianLarge 가 사용되는 곳에 쓰인다.
- 그리고 utilitarianSmall 이었던 곳이 accessoryCorner family 로 많이 업데이트 되었습니다.

<img width="700" alt="24" src="https://user-images.githubusercontent.com/69136340/188644352-a8a30bb4-e00c-4e94-8460-e10235e823c2.png">

- WidgetKit 에서는 SwiftUI 뷰와 레이아웃이 ClockKit 의 템플릿으로 대체됩니다.
- WidgetKit 의 타임라인과 엔트리는 친숙합니다. 실제로 이것은 ClockKit 자체에서 영감을 받은 것입니다.(호오…) **이것은 컴플리케이션 데이터 소스가 정적 또는 intent 기반의 WidgetKit 구성에 잘 마이그레이션 한다는 것이죠.**

<img width="700" alt="25" src="https://user-images.githubusercontent.com/69136340/188644419-16af6c96-fe85-40f1-bac1-e28e0bcd36e7.png">

WidgetKit 을 지원하는 configurations 타입과 일반적인 family 지원에 대해서 알고싶다면 위의 세션을 확인해 보세요.

## ✅ Migration API

ClockKit 에 컴플리케이션을 자동으로 마이그레이션하도록 마지막 API 를 추가하였습니다.

<img width="700" alt="26" src="https://user-images.githubusercontent.com/69136340/188644460-540e235b-fc10-469c-8d2e-f356bf45795d.png">

기존의 컴플리케이션을 사용자 상호작용 없이 자동으로 WidgetKit 으로 업그레이드 할 수 있습니다.

<img width="700" alt="27" src="https://user-images.githubusercontent.com/69136340/188644515-c60fe7f6-b92a-4188-b229-fe2b6fab3dda.png">

1️⃣ 여러분의 앱이 워치에서 업데이트되면

2️⃣ Watch Faces 가 앱의 번들에서 위젯 표시를 확인할 겁니다.

3️⃣ 발견할 경우 ClockKit 컴플리케이션 데이터 소스를 공개해 기존 컴플리케이션에 대한 마이그레이션을 생성합니다.

4️⃣ 더 나아가면 CLKComplicationDataSource 는 당신의 ClockKit 공유 페이스를 누군가 받게 될 때 마이그레이션을 요청하게되고 이때만 작동할 것입니다.

### 🦉 New in watchOS 9.0

<img width="700" alt="28" src="https://user-images.githubusercontent.com/69136340/188644567-cdf0e347-8608-4a69-8cc2-f1930dab1b30.png">

WidgetKit 컴플리케이션을 다 만들었다면, 새 프로퍼티 widgetMigrator 를 추가할 수 있습니다. 이는 새 Migrator protocol 을 준수하는 객체를 제공합니다. 컴플리케이션의 데이터 소스 자체이건 다른 유형이건 상관 없습니다.

<img width="700" alt="29" src="https://user-images.githubusercontent.com/69136340/188644613-0470bb0e-7333-441d-936c-cfea9f060899.png">

CLKComplicationWidgetMigrator 프로토콜에는 기존 CLKComplicationDescriptors 의 위젯 마이그레이션 구성을 워치 페이스에 제공하는 단일 함수만 존재합니다.

<img width="700" alt="30" src="https://user-images.githubusercontent.com/69136340/188644665-840f9a27-7161-4e4c-8b23-2058808fe80e.png">

새 API 를 채택하는 가장 직접적인 방법은 데이터 소스를 새 프로토콜 Migrator 에 준수시키는 것입니다.

<img width="700" alt="31" src="https://user-images.githubusercontent.com/69136340/188644728-a923fb6f-2e1c-479a-8c61-4b21843fde51.png">

WidgetKit 컴플리케이션이 정적 구성을 사용한다면 정적 마이그레이션 구성을 제공합니다.

<img width="700" alt="32" src="https://user-images.githubusercontent.com/69136340/188644774-ae436481-1769-4c1a-9236-3daf508c70b7.png">

위젯 컴플리케이션에서 인텐트를 사용한다면 동등한 마이그레이션 구성이 됩니다.

<img width="700" alt="33" src="https://user-images.githubusercontent.com/69136340/188644843-c7028a65-d7f2-4933-9123-b2881cc3e53c.png">

인텐트 기반 마이그레이션 제공시 watch app 과 widget extension 에 intent definitions 를 추가해야만 할 것입니다. 양쪽에서 intent object 를 만들 수 있어야 하니까요. 

**🧑‍🏭 : WidgetKit 은 경험을 극적으로 단순화시키면서도 워치 컴플리케이션 만들기를 창의적이고 새롭게 만들어줍니다.**

<img width="700" alt="34" src="https://user-images.githubusercontent.com/69136340/188644887-5c9f520e-9a6c-4644-8778-6176fd71bd45.png">

<img width="700" alt="35" src="https://user-images.githubusercontent.com/69136340/188644971-dafe20a1-1f81-4eac-b5e5-82bb28d71f09.png">
