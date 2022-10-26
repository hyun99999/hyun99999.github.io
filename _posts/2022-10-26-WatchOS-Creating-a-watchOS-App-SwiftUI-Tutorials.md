---
title:  "WatchOS) Creating a watchOS App - SwiftUI Tutorials"
categories:
- iOS

date:   2022-10-26  10:44:00 +0900
author_profile: false
---
### 내용

- SwiftUI Tutorial 로 만든 Landmarks 프로젝트에 watch app 을 추가해보자.
- watch app 을 어떻게 추가하며 notification interface 를 수정할 수 있는지 알아보자.
- 아래의 튜토리얼을 참고하여 진행하였습니다.

[Apple Developer Documentation - Creating a WatchOS App](https://developer.apple.com/tutorials/swiftui/creating-a-watchos-app)

## 1️⃣ Add a WatchOS Target

<img width="400" alt="1" src="https://user-images.githubusercontent.com/69136340/197912958-ade387b0-5249-40ef-b8e9-3f5fc7f29ebf.png">

- watch app 을 만들기 위해서 target 추가해주었다.

<img width="500" alt="2" src="https://user-images.githubusercontent.com/69136340/197912966-48fbacc4-af58-43a2-8fa6-bbe8909b9bc0.png">

- 기존에 있는 Landmarks 프로젝트에 추가하는 것이기 때문에 다음과 같이 설정해주었다.

<img width="500" alt="3" src="https://user-images.githubusercontent.com/69136340/197913039-dee57abe-2dbc-4210-bfe3-cf7a4376576f.png">

- WatchLandmarks Watch App 타겟을 위한 스키마가 만들어지는데 활성화하겠냐고 묻는다.
    - 튜토리얼에서는 `Cancel` 을 선택하라고 한다. 이는 개발자 스스로가 스키마를 선택하는 과정을 경험시키 위한 것이므로 `Activate` 를 선택해도 상관없다.

<img width="400" alt="4" src="https://user-images.githubusercontent.com/69136340/197913224-e4c21cf7-ccaf-42be-80c1-e9dceecc47c6.png">

- 다음과 같이 스키마를 선택할 수 있다.

<img width="300" alt="5" src="https://user-images.githubusercontent.com/69136340/197913239-d47b87fb-2293-434d-a487-45feebf6daa4.png">

- **General > Deplymetn Info > Supports Running Without iOS App Installation** 설정해준다.
    - 이는 iOS app 없이도 실행을 지원하는 유무이다.

<img width="700" alt="6" src="https://user-images.githubusercontent.com/69136340/197913489-552b90c2-261f-4265-96ac-f2c6fe090fae.png">

## 2️⃣ ****Share Files Between Targets****

- Landmarks iOS 앱의 데이터 모델, 일부 리소스 파일과 수정없이 보여줄 수 있는 모든 뷰를 재사용할 것이다.
    - SwiftUI 로 UI 를 작성하였기 때문에 iOS 앱과 watch 앱에서 모두 자연스럽게 보여질 수 있는 것이 장점이다.

<img width="250" alt="9" src="https://user-images.githubusercontent.com/69136340/197913591-f7d7f4f8-26a8-4944-9174-8b5a3ca08d01.png">

- watchOS app 의 entry point 인 WatchLandmarks 파일을 삭제합니다. iOS 앱에 있는 entry point 를 이용할 것입니다.

<img width="700" alt="10" src="https://user-images.githubusercontent.com/69136340/197913630-1eab55a9-f99d-47fb-abc2-d88a85a9f58b.png">

- watchOS app 이 iOS app 과 공유할 수 있는 entry point 를 포함한 파일들을 선택하여 WatchLandmarks Watch App target 에도 설정해주겠습니다.
    - 데이터 모델과 리소스 파일들도 선택해준다.

<img width="700" alt="11" src="https://user-images.githubusercontent.com/69136340/197913653-be2ca9b4-8db7-4322-9e6a-9aef0d423c00.png">

- 우측의 file inspector 에서 `Target Membership` 에서 설정해준다.

<img width="300" alt="12" src="https://user-images.githubusercontent.com/69136340/197913819-c1fce748-d31c-45d3-beb8-af49e3e77e0a.png">

- WatchLandmarks Watch App 폴더의 `Assets.xcasset` 파일의 AppIcon 을 삭제해줍니다.
    - 그리고 튜토리얼 프로젝트를 다운로드해서 AppIcon 에셋들을 추가해줍니다.

<img width="700" alt="13" src="https://user-images.githubusercontent.com/69136340/197913852-0e01bca0-842b-4dcb-897a-e33ae09d8edc.png">

## 3️⃣ ****Create the Detail View****

- landmark detail 을 위해서 watch 만을 위한 뷰를 만들어보겠습니다.
- 테스트를 위한 smallest, largest watch size 에 대한 preview 를 만들고, 모든 것이 watch face 에 맞도록 `circle view` 를 수정해보겠습니다.

<img src="https://user-images.githubusercontent.com/69136340/197913893-d51f13a9-9d8f-4805-b903-cf0c9eade164.png" width="250">

- LandmarkDetail.swift 파일을 생성합니다.
    - 이 파일은 iOS app 파일과 같은 이름이지만 watch app 에서만 사용합니다.

<img width="700" alt="15" src="https://user-images.githubusercontent.com/69136340/197913982-8296b4c7-ca62-494e-ab54-47c685031be6.png">

- model data 인스턴스를 preview 에서 만들어주어서 사용하였다. body 부분에 CircleImage 뷰를 생성하였다. (iOS 프로젝트에서의 뷰를 재사용하였다.)
    - `resizable` 하게 만들었기 때문에 `scaledToFill()` 호출하여 원의 크기를 조정할 수 있다.

<img width="700" alt="16" src="https://user-images.githubusercontent.com/69136340/197914001-0f8bcf98-bd76-4921-8de1-0e1c50c7376b.png">

- 앞서 언급한 것 처럼 가장 큰 시계 모드(44mm) 와 작은 시계 모드(40mm)에 대한 미리 보기를 만듭니다.
    - 아래에서 말하겠지만 canvas 에서 제대로 해당 디바이스가 불러오지 않는다.(현재는 빌드용 Apple Watch Series 8 45mm 디바이스로 보여진다.)

<img width="1043" alt="17" src="https://user-images.githubusercontent.com/69136340/197914043-6235fdef-11e2-4f07-a1bb-7d71206992e6.png">

### ❗️Apple Watch Ultra

그런데 최근에 애플워치 울트라가 나왔죠! 그렇다면 가장 큰 워치는 49mm 의 울트라입니다. `previewDevice` 의 파라미터로 들어가는 문자열은 어떻게 확인할 수 있을까요?

[Apple Developer Documentation - previewDeviece(_:)](https://developer.apple.com/documentation/swiftui/view/previewdevice(_:))

위의 문서를 참고하시면 터미널에 다음의 명령어를 통해서 조회할 수 있습니다.

```swift
% xcrun simctl list devicetypes
```

<img width="700" alt="18" src="https://user-images.githubusercontent.com/69136340/197914155-6e69a916-5f66-4acf-b9d3-5620167376b7.png">

이 위에도 많이 있지만 우리는 울트라를 확인할거니까 맨 아래를 살펴볼게요. 다음과 같이 우리는 코드를 수정할 수 있습니다.

- 디바이스의 이름들이 모두 변경된 것 같습니다. 그래서 터미널에서 알려주는 문자열을 사용하였고, 제대로 적용되는 것을 확인할 수 있었다.

<img width="700" alt="19" src="https://user-images.githubusercontent.com/69136340/197914184-15f762b1-9217-404d-8b21-89e8165ef3b5.png">

- CircleImage 외에도 정보를 나타내는 components 를 추가해 보겠습니다. 스크롤도 할 수 있도록 scroll view 로 감싸겠습니다.

<img width="700" alt="20" src="https://user-images.githubusercontent.com/69136340/197914256-34b6042f-1c69-421c-a967-f294da2a9769.png">

- 원과 랜드마크 이름이 나타나도록 원 이미지의 크기를 조절해보겠습니다.
    - `CircleImage` 의 이미지가 잘리지 않도록 `scaledToFit` 하게 변경하였고, `padding` 도 주었습니다.

<img width="700" alt="21" src="https://user-images.githubusercontent.com/69136340/197914338-c16cc5c1-b0d3-40d0-afa5-e519e7777608.png">

- MapView 를 추가하여 위치도 확인할 수 있도록 하겠습니다.

<img width="700" alt="22" src="https://user-images.githubusercontent.com/69136340/197914384-9c60fa4a-fe92-4e63-8dfc-c776da0bfe95.png">

- back button 의 title 도 지정하겠습니다.

```swift
ScrollView {

  // ...

}.navigationTitle("Landmarks")
```

## 4️⃣ ****Add the Landmarks List****

- watch 의 `ContentView` 에 리스트를 연결해보겠습니다.

<img width="300" alt="23" src="https://user-images.githubusercontent.com/69136340/197914400-c2c99d2c-9303-48b9-9b53-f2844c74e507.png">

- `WatchLandmarks Watch App` 폴더의 `ContentView` 를 수정하겠습니다.
    - watchOS target 에 대한 content view(우리는 앞서 LandmarkDetail 을 작성하였다.) 를 iOS target 과 동일한 이름을 갖게 하였습니다. 이렇게 하면 이름과 인터페이스를 동일하게 유지하면 tragets 간의 파일을 쉽게 공유할 수 있습니다.
    - 즉, 이름과 인터페이스를 동일하게 가져가면 iOS 와 watchOS targe 모두에서 자연스럽게 같은 흐름을 공유할 수 있다는 것이다.

<img width="700" alt="24" src="https://user-images.githubusercontent.com/69136340/197914413-f2957337-8e3a-444b-a1c5-652bf2d53a2c.png">

### 👉 빌드해보자!

- 앞서 `Supports Running Without iOS App Installation` 을 체크하지 않았다면 우선 iOS 스키마를 빌드하여 아이폰에 설치하고, 워치 스키마를 빌드하여야 한다. 체크를 해주었다면 별도로 워치 스키마만 빌드해도 실행할 수 있다.

<img src="https://user-images.githubusercontent.com/69136340/197914499-d992f28d-0e19-467e-913f-0ff674436fdc.mp4" width = "300">

## 5️⃣ ****Create a Custom Notification Interface****

- 마지막 세션에서 즐겨찾기 위치 중 하나에 가깝다는 notification 을 받을 때마다 landmark 정보를 표시하는 interface 를 만들겠습니다.

<img width="300" alt="26" src="https://user-images.githubusercontent.com/69136340/197914602-b73cc4a6-7a24-49ee-97a3-ae55e13ec728.png">

### 👉 Xcode 14

기존에는 아래와 같이 Notification Scene 을 추가하고,

<img width="700" alt="27" src="https://user-images.githubusercontent.com/69136340/197914643-0d1b3d4c-f0b4-447f-8096-5d2a2ff8c981.png">

위의 파일들이 추가되었지만 Xcode 14 부터는 지원하지 않습니다. 또한, notification 을 테스트하는 방법은 튜토리얼에서 제외되어 있지만 문서를 찾아서 진행해보겠습니다.

이 부분은 따로 글을 작성하였습니다.
