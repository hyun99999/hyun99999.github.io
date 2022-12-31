---
title:  "iOS) CoreData 를 사용하여 Configurable Widget 만들기 (1/3) - 프로젝트 세팅"
categories:
- iOS

date:   2022-12-30  15:27:00 +0900
author_profile: false
---
✅ **1. 프로젝트 세팅**

- **Configurable Widget 에서 CoreData 로 데이터를 공유하는 프로젝트 세팅 진행.**
- **App Group 을 사용하여 containing app 과 app extension 의 데이터 공유.**

2. Widget 만들기

3. Configurable Widget 만들기

### ❓ 들어가기 전 - 왜 CoreData 를 사용하여 데이터를 저장하기로 했나요?

- 이전에 UserDefaults 를 사용하여 widget 과 containing app 의 데이터 공유를 해보았기 때문에 이번에는 CoreData 를 사용해서 데이터 공유를 구현해보고 싶었습니다.

[iOS) Kakao QRcode Widget 클론코딩 - Widget 데이터 공유 및 뷰 구현(SwiftUI)](https://gyuios.tistory.com/102)

- 이 글은 **명함 형태**의 데이터 모델을 프로젝트에 적용하기 위한 연습을 위해서 작성했습니다. 그래서 **명함 종류, 속한 그룹** 등과 같은 관계성을 프로젝트 내에서 가질 수 있다고 판단하고 관계형 데이터 모델을 제공할 수 있는 **CoreData** 를 사용해보고자 하였습니다.

## 1️⃣ 프로젝트 세팅

- **CoreData** 를 활용할 것이기 때문에 프로젝트를 만들 때 부터 체크해주겠습니다.(물론, 중간에 추가할 수도 있습니다.)

<img width="500" alt="1" src="https://user-images.githubusercontent.com/69136340/210040316-00343842-a751-4bc7-bb74-44de0f1be63a.png">

- Widget 을 만들 것이기 때문에 이를 위한 **Widget Extension** 을 추가해주겠습니다.

<img width="500" alt="2" src="https://user-images.githubusercontent.com/69136340/210040323-72d92c2a-6c4b-46f4-a1a1-21edda1fac4c.png">

- configurable 한 widget 을 만들기 위해서 해당  체크박스도 체크해줍니다.(이를 통해 위젯을 길게 눌러 ”위젯 편집”을 통해 사용자 구성 가능한 옵션을 제공할 수 있습니다.)

<img width="500" alt="3" src="https://user-images.githubusercontent.com/69136340/210040343-78a35be8-a239-4520-a223-9f5e9fc6c2d2.png">

## 👉 App Group

- 서로 다른 target 간에 데이터를 공유하기 위해서 App Group 을 추가해야 합니다.

<img src="https://user-images.githubusercontent.com/69136340/210040365-7d1fb172-9c07-4640-9468-3ada6374dfd9.png" width ="600">

containing app 과 contained app extension 의 **App Group** 을 활성화하여 앱에서 사용할 **App Group** 을 지정할 수 있습니다.

## 2️⃣ App Groups 를 생성하고 적용해보자

- 공유하는 target 모두에 생성해주어야 합니다.
- `group.` 을 prefix 로 가지는 포멧이 제공됩니다. **App Group Identifie**r 는 bundle identifier 처럼 유니크한 값이기 때문에 가져다가 사용하였습니다.

<img width="600" alt="5" src="https://user-images.githubusercontent.com/69136340/210040390-fcba7893-35be-4322-9c00-235caefd0ea7.png">

- Widget Extension 에도 잊지 않고 생성해주었습니다.

<img width="600" alt="6" src="https://user-images.githubusercontent.com/69136340/210040414-b27a5423-e426-4cfb-b98b-cf0bf0c296b8.png">

- 각 target 의 **entitlements** 파일이 추가되고 설정한 **App Groups** 값이 추가된 것을 확인할 수 있습니다.

<img width="600" alt="7" src="https://user-images.githubusercontent.com/69136340/210040433-de12f0dd-63cd-45f5-b2eb-09258dacf0ef.png">

### 👉 만든 App Groups 는 어디서 확인할 수 있나요?

- **Apple Developer** 의 **Identifiers** 에서 **App Groups** 를 설정하여 조회 및 편집이 가능합니다.

<img width="600" alt="8" src="https://user-images.githubusercontent.com/69136340/210040455-d86cbcd7-4115-4809-8af5-536fb18257e9.png">

## 3️⃣ App Group 을 사용하여 데이터 공유

- main 이 되는 view controller 에서 아래와 같이 **카드 이름, 이름, 배경**을 입력받고 widget extension 과 데이터를 공유하여 위젯에서 보여주도록 하겠습니다.
- 이때 CoreData 를 사용하여 데이터를 저장하겠습니다.

<img width="600" alt="9" src="https://user-images.githubusercontent.com/69136340/210040465-cf452885-6853-4890-add2-a939f31a677a.png">

- 아래는 위젯의 UI 입니다.

<img width="250" alt="10" src="https://user-images.githubusercontent.com/69136340/210040534-95706305-e003-4867-b0b9-eb854fb4a38d.png">

***다음 글에서는 공유한 데이터를 보여줄 widget 을 만들고, App Group 과 CoreData 를 활용하여 데이터를 저장하고 조회해보겠습니다.***

**참고:**
[Handling Common Scenarios](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html#//apple_ref/doc/uid/TP40014214-CH21-SW1)
