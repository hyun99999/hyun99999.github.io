---
title:  "WWDC22) Build a Watch productivity app"
categories:
- iOS

date:   2022-11-11  04:13:00 +0900
author_profile: false
---
[Build a productivity app for Apple Watch - WWDC22 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2022/10133/)

***본 글은 WWDC 를 보고, 번역 및 요약 그리고 실행해보는 스터디 프로젝트의 일환입니다.***

<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/201180765-c17da3b7-d7ee-42dd-a7fe-06c1928b2df1.png">

### 👉 세션에서 새롭게 소개된 내용

- Xcode 14 부터 새 Watch 앱은 단일 Watch target 만 가집니다.
- 이미 SwiftUI life cycle 을 사용하는 Watch 앱이라면 Xcode 14 migration tool 로 쉽게 전환할 수 있습니다.
- Xcode 14 에서는 앱 아이콘도 훨씬 쉽게 추가할 수 있습니다.
- watchOS 9 부터 Stepper, TextFieldLink, ShareLink  를 사용할 수 있습니다.
- Swift Charts 를 활용하여 차트를 생성할 수 있습니다.

<img width="700" alt="2" src="https://user-images.githubusercontent.com/69136340/201180847-4b065dc4-099a-4f0b-a0a1-ff4d04abf6c4.png">

- 새 앱을 만들고, 간단한 list 를 표시하고, 사람들이 아이템을 list 에 추가할 수 있게 하고, 편집할 수 있도록 해보겠습니다.
- 이러한 기능을 추가하면서 Watch 앱의 일반적인 app navigation 전략과 올바른 것을 고르는 법에 대해서도 알아보겠습니다.
- 친구와 아이템을 공유할 수 있도록 해보겠습니다.
- 차트를 추가해서 productivity trends 를 파악하고 동기를 얻겠습니다.
- Digital Crown 으로 차트를 스크롤 할 수 있게 해서 더 큰 데이터의 범위를 표시하겠습니다.

## 1️⃣ Create a Watch app

<img width="700" alt="3" src="https://user-images.githubusercontent.com/69136340/201180895-bdd13950-70ea-46d4-913f-4ba262c11597.png">

 WatchOS 탭에서 App 선택하고 Next 를 클릭하여 만들 수 있습니다.

<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/201180951-f3014b8c-0553-4e95-8ce9-febac81eb110.png">

이때 Watch-only 앱 을 만들지 iOS 앱이 있는 Watch 앱을 만들지에 대한 선택지는 매우 중요합니다.

### 👉 Great Watch apps

<img width="700" alt="5" src="https://user-images.githubusercontent.com/69136340/201181009-94219d48-326b-406d-b3fa-49828b52b10d.png">

- Great Watch app Workout 의 인터페이스에서 좋아하는 운동을 빨리 시작할 수 있는 것 처럼 빠른 상호 작용을 가능하게 합니다.
- Great Watch app 은 앱의 기본 목적에 주목합니다. 예를 들어 날씨 앱은 오늘의 일기 예보와 관련 현재 기상 및 간단한 10일 예측을 표시합니다.
- Great Watch app 은 iPhone 과 독립적으로 사용하도록 설계되었습니다. 예를 들어 연락처 앱은 휴대전화와 동기화되지만, iPhone 이 근처에 있지 않아도 애플워치의 연락처 정보에 액세스합니다.

### 👉 iOS Companion apps

<img width="700" alt="6" src="https://user-images.githubusercontent.com/69136340/201181062-cbb66b04-6993-4877-922f-64ef4816774e.png">

watch 앱을 위한 companon app 이 필요한 이유는 여러가지가 있습니다. 예를 들어 피트니스 앱처럼 애플 워치에서 캡처한 데이터의 과거 기록, 추세에 대한 자세한 분석을 제공하는 것이 있습니다.

우리가 만들 앱은 집중해야하는 기능과 빠른 상호 작용, 제한된 데이터를 가지기 때문에 Watch-only app 을 만들 것입니다.

**생성된 target 에 대해서 잠깐 이야기를 해보겠습니다.**

<img width="700" alt="7" src="https://user-images.githubusercontent.com/69136340/201181108-8630ba74-8d79-4d3e-a2eb-16c6f8e53741.png">

과거에는 watch 앱을 빌드했다면 프로젝트에는 watch 관련 target 이 두 개가 생깁니다.(Test 를 포함하는 경우에는 추가로 생깁니다.) 아래와 같이요.

<img width="700" alt="8" src="https://user-images.githubusercontent.com/69136340/201181176-7838a9d3-b31f-49f6-93ea-7ceb09a86d5e.png">

좀 더 자세하게 살펴보겠습니다.

<img width="600" alt="9" src="https://user-images.githubusercontent.com/69136340/201181238-03681d15-15a9-4296-b86f-981332c003c1.png">

**출처:** 

[Apple Developer Documentation](https://developer.apple.com/documentation/watchos-apps/creating-independent-watchos-apps)

storyboard, assets, lacalization-related files 가 있는 WatchKit App target 과 앱 코드가 포함된 WatchKit Extension 이 존재합니다.

이러한 이중 타겟은 watchOS 초기부터 유지된 것입니다. 이젠 여러 Watch target 가 있을 이유가 없습니다. ***Xcode 14 부터 새 Watch 앱은 단일 Watch target 만 가집니다.***

<img width="700" alt="10" src="https://user-images.githubusercontent.com/69136340/201181338-8e042706-f622-4f58-8d20-d6870a3aedc7.png">

**code, assets, localizations, Siri Intent, Widget Extensions 까지 Watch App 에 관련된 모든 것이 하나의 타겟에 속하게됩니다.**

<img width="700" alt="11" src="https://user-images.githubusercontent.com/69136340/201181401-abc30b82-d3fe-4b9a-bd84-8e829227ab15.png">

좋은 소식은 single-target Watch app 이 watchOS 7 에서 지원됩니다. 이를 통해 최신 watchOS 가 아닌 고객을 계속 지원하면서 프로젝트 구조를 단순화하고, 혼란과 중복을 줄일 수 있습니다.

또한, WatchKit Extension target 이 있는 기존 앱의 경우 계속 작동하며 Xcode 를 사용하여 앱을 업데이트하고 앱 스토어에 올릴 수 있습니다.

이미 SwiftUI life cycle 을 사용하는 Watch 앱이라면 Xcode 14 migration tool 로 쉽게 전환할 수 있습니다. 

### 👉 SwiftUI life cycle?

이전에는 다음과 같이 Life Cycle 을 선택할 수 있었습니다. **Xcode 14 부터는 모두 SwiftUI 로 설정되어집니다.**

<img src="https://user-images.githubusercontent.com/69136340/201181452-00420b34-1a39-439e-9ed0-010cac778803.png" width ="600">

**출처 :**

[Apple Developer Documentation - creating a watchOS app](https://developer.apple.com/tutorials/swiftui/creating-a-watchos-app)

---

<img width="700" alt="13" src="https://user-images.githubusercontent.com/69136340/201181580-b65676ff-84d4-475d-9447-a1a14ac422f8.png">

target 선택 > Editor > Validate Settings… 선택합니다. deployment target 이 watchOS 7 이상이라면 target collapsing option(타겟 축소 옵션)이 제공됩니다.

### 👉 Add an app icon

<img width="700" alt="14" src="https://user-images.githubusercontent.com/69136340/201181638-606b5f29-605e-4594-9477-c82ba708f582.png">

- **Xcode 14 에서는 앱 아이콘도 훨씬 쉽게 추가할 수 있습니다. 하나의 1024x1024 픽셀 이미지만 있으면 됩니다.**

<img width="700" alt="15" src="https://user-images.githubusercontent.com/69136340/201181660-96eea90b-d217-4a68-9745-c3a182e35509.png">

- 앱 아이콘 이미지는 모든 Watch 장치에 올바르게 표시되도록 크기가 조정되고 iPhone Watch 앱의 홈 화면이나 알림 앱 설정에서 테스트할 수 있습니다.

<img width="700" alt="16" src="https://user-images.githubusercontent.com/69136340/201181761-a4fd91b9-3fdc-4885-b63b-3346e7e5bffb.png">

<img width="700" alt="17" src="https://user-images.githubusercontent.com/69136340/201181735-ad2966ca-e122-417e-9360-ff0830917889.png">

- 필요한 경우 특정 작은 크기에는 custom images 를 추가할 수 있습니다. 예를 들어 앱 아이콘에 작은 디테일이 있어서 크기를 줄였을 때 보이지 않는다면 데티일이 제거된 특정 아이콘 이미지를 추가할 수 있습니다.

## 2️⃣ Add a simple list

할 일 목록을 추가해서 앱에 몇 가지 기능을 추가해 보겠습니다.

할 일 목록에 대한 데이터 모델 생성합니다.

<img width="700" alt="18" src="https://user-images.githubusercontent.com/69136340/201181979-1e393b23-048b-4485-82d2-62028c21f96e.png">

그런 다음 데이터를 저장하고 항목의 배열을 게시하는 간단한 모델을 만듭니다.

<img width="700" alt="19" src="https://user-images.githubusercontent.com/69136340/201182027-256ab3e5-14ab-4746-8e2e-32222164ee34.png">

마지막으로, 뷰가 모델에 액세스 할 수 있도록 모델을 environment object 로 추가합니다.

<img width="700" alt="20" src="https://user-images.githubusercontent.com/69136340/201182119-3b4af3f1-7aee-466c-8cb7-9a39eb23fb14.png">

이제 model 로 List 를 만듭니다. 지금은 빈 목록이 표시됩니다. 이에 대해 사람들이 추가할 수 있는 방법을 제공해야 합니다.

<img width="700" alt="21" src="https://user-images.githubusercontent.com/69136340/201182149-1ed79445-4838-4b21-8433-597c887e34ac.png">

## 3️⃣ Enable list updates

- 탭해서 목록에 새 항목을 추가할 수 있는 버튼 추가.
- watchOS 9 의 새 기능인 Text filed link 를 사용하면 여러 텍스트 입력 옵션을 호출할 수 있습니다.

<img width="700" alt="스크린샷 2022-11-11 오전 3 42 37" src="https://user-images.githubusercontent.com/69136340/201182195-f27e8aed-59b9-4cec-ac52-5b9159f4ba2b.png">

다음과 같이 간단한 문자열을 가진 TextFieldLink 를 만들거나

<img width="700" alt="23" src="https://user-images.githubusercontent.com/69136340/201182859-87ba7443-e04d-40de-98bf-8d705687411d.png">

Label 로 더 개성있게 만들 수 있습니다.

<img width="700" alt="24" src="https://user-images.githubusercontent.com/69136340/201182266-1dc25e20-7dbf-451b-ba39-aae74f1ea14d.png">

view modifier 로 버튼 모양을 바꿀 수 있습니다. foregroundColor 나 foregroundStyle, buttonStyle 등이 있습니다.

<img width="700" alt="25" src="https://user-images.githubusercontent.com/69136340/201182304-0ea5c817-9fec-4bcd-ae18-3732304ab601.png">

<img width="700" alt="26" src="https://user-images.githubusercontent.com/69136340/201182345-20876c8d-0df7-46fa-a945-d26081c334d7.png">

TextFieldLink 의 스타일과 동작을 캡슐화하기 위해 `AddItemLink` 뷰를 생성하겠습니다.

- 버튼에 사용자 정의 Label 을 사용하고 텍스트를 입력하면 리스트에 새 항목을 추가.

<img width="700" alt="27" src="https://user-images.githubusercontent.com/69136340/201182493-c23b8bf3-59da-475e-838c-a5bbaab70093.png">

**🧑‍🏭 : 이제 TextFieldLink 를 추가하기로 하였으니 위치에 대해서 생각해봅시다.**

Watch 앱의 목록에 작업을 추가할 때는 몇 가지 옵션있습니다.

- 짧은 목록의 기본작업에 button 과 navigation link 혹은 TextFieldLink 를 사용하세요.

<img width="700" alt="28" src="https://user-images.githubusercontent.com/69136340/201182773-f8078118-1c10-43dc-b87f-5be19d8c579e.png">

- 목록 끝에 항목을 추가하는 것은 기본 작업에 좋습니다. World Clock 의 도시 목록처럼 짧은 목록에서 유용합니다.

<img width="700" alt="29" src="https://user-images.githubusercontent.com/69136340/201182599-6bacb204-0d28-48ad-890e-cd83658c5359.png">

🚨 그러나, 목록이 길 것으로 예쌍되는 경우에는 목록의 끝까지 계속 스크롤해야 합니다.

- 이때는 ToolbarItem 을 사용하세요.

<img width="700" alt="30" src="https://user-images.githubusercontent.com/69136340/201182703-b37b57ac-1054-4228-8efe-ee8f931a101f.png">

ToolbarItem 을 추가하려면 List 에 toolbar modifier 를 넣고, action view 를 콘텐츠로 사용하세요.(예를 들어, 위에서 만든 `AddItemLink`)

<img width="633" alt="31" src="https://user-images.githubusercontent.com/69136340/201182639-05750b12-0ce9-4518-9bcc-81efd8c33bf9.png">
1%84%92%E1%85%AE_4.08.53.png)

그러면 자동으로 단일 toolbar item 이 위치합니다.

**🧑‍🏭 : 저는 할 일 목록을 짧게 쓰려고 하는데 결국에는 그렇게 안되더라구요! 그래서 ToolbarItem 에 TextFieldLink 를 넣어서 쉽게 액세스할 수 있게 하겠습니다.**

다음과 같이 결과를 확인할 수 있습니다. environmentObject 는 부모 혹은 조상 뷰에서 제공하는 observable object 이기 때문에 `ItemListModel` 을 사용할 수 있었습니다.

<img width="700" alt="스크린샷 2022-11-11 오전 3 43 19" src="https://user-images.githubusercontent.com/69136340/201183026-1f1efee2-e56b-40b4-af86-2a55c1e081a8.png">

## 4️⃣ App navigation structure

**🧑‍🏭 : 설명만 있는 리스트를 만드는 것은 간단하지만 그다지 유용하지 않습니다. 항목을 완료로 표시할 수 있고, 우선 순위를 설정하는 방법도 필요할 수 있습니다. 이를 위한 상세 뷰를 추가해보겠습니다.**

### 👉 Watch 의 SwiftUI 에서 navigation structure options

- list-detail  관계를 가진 뷰에는 hierarchical navigation 이 사용됩니다.
- watchOS 9 부터 SwiftUI NavigationStack 으로 이런 인터페이스를 만들 수 있습니다.

<img width="700" alt="33" src="https://user-images.githubusercontent.com/69136340/201183117-3fe18d36-e39b-4cae-aee5-807961046dd9.png">

- page-based navigation 은 모든 뷰가 또래인 flat structure 에 사용됩니다.

<img width="700" alt="34" src="https://user-images.githubusercontent.com/69136340/201183215-e2f80ecb-7a26-4caf-bb75-b58df7fd82f2.png">

**예시)** page-based navigation 의 가장 좋은 예는 Workout 앱의 운동 중 뷰입니다. 스와이프하며 운동 제어, 지표, 재생 제어를 쉽게 전환할 수 있습니다.

<img width="700" alt="35" src="https://user-images.githubusercontent.com/69136340/201183373-41e37fa4-ab7b-43f9-831f-2a29aa943c51.png">

- full screen 앱에는 전체 스크린을 사용하는 단일 뷰가 있습니다. 게임이나 single main view 가 있는 앱에서 사용됩니다.
- 이를 위해서 ignoreSafeArea modifier 를 사용해서 디스플레이를 가장자리로 확장하고, visibility 값이 hidden 인 toolbar 로 navigation bar 를 숨길 수 있습니다.

<img width="700" alt="36" src="https://user-images.githubusercontent.com/69136340/201183408-3c7aedbf-72ab-4e89-a51e-91d3b3c2d4f8.png">

- modal sheet 는 현재 뷰 위로 미끄러지는 full-screen 뷰 입니다.
- 현재 워크플로우의 일부로 완료해야하는 중요한 작업에 사용할 수 있습니다.

<img width="700" alt="37" src="https://user-images.githubusercontent.com/69136340/201183615-e57afb38-283a-41a6-beec-21666cb89b12.png">

**예시)** mail 은 `hierarchical style` 로 메시지의 목록을 표시하고 각 메시지 또는 스레드를 detail view 로 표시합니다.

<img width="700" alt="38" src="https://user-images.githubusercontent.com/69136340/201183681-4bfdcfc8-803a-416c-b21a-2aad7d59e21b.png">

메시지 세부 정보에서 수행할 수 있는 작업이 있지만 목록으로 돌아가기 전에 수행해야만 하는 작업은 없습니다. 

<img width="700" alt="39" src="https://user-images.githubusercontent.com/69136340/201183746-16daa59d-245b-47a4-8f98-7c03562b1d95.png">

목록으로 돌아가서 New Message 를 탭하면

<img width="700" alt="40" src="https://user-images.githubusercontent.com/69136340/201183783-5b91e745-09d1-4f4c-b77a-58c5947d45c5.png">

mail 은 modal sheet 를 사용해서 New Message 뷰를 표시합니다.

<img width="700" alt="41" src="https://user-images.githubusercontent.com/69136340/201183932-cd9f90f6-86fa-479f-99e3-8c399844e517.png">

SwiftUI 의 NavigationStack 에 대한 내용을 포함하여 programmatic navigation 에 대해 자세히 알아보려면 SwiftUi cookbook for navigation 세션을 확인하세요.

<img width="700" alt="42" src="https://user-images.githubusercontent.com/69136340/201183972-d8df7bff-ab61-45b2-a9a7-8168b4d9216f.png">

### 👉 이제 만들어봅시다!

모달 시트를 표시하려면 sheet presentation state 를 제어하는 프로퍼티가 필요합니다.

- `showDetails` 프로퍼티가 true 일 때 custom modal sheet content 를 표시하게 됩니다.

<img width="700" alt="43" src="https://user-images.githubusercontent.com/69136340/201184029-b0de4cbc-9632-4270-9935-c1dcf7817721.png">

toolbar 를 추가하기 위해서 아래와 같이 작성할 수 있습니다.

confirmationAction 과 cancellationAction, destructiveAction 등을 사용하여 toolbar item 의 위치를 설정할 수 있습니다.

<img width="700" alt="44" src="https://user-images.githubusercontent.com/69136340/201184098-4c595204-f9e7-408f-994c-f5135eeedf14.png">

우리는 edit item 을 하고 있었으므로 detail view 에서 modal sheet 를 사용할 것입니다.(작업을 완료하고 done 을 탭하기 전까지 한 작업에 집중할 것이기 때문입니다.)

### 👉 update list item struct

- 이제 detail view 로 어떻게 이동할지 결정했으므로 list item 의 구조를 업데이트 하겠습니다.

예상 작업, 생선 날짜 및 완료 날짜를 저장하는 새로운 프로퍼티가 생겼습니다.

<img width="700" alt="45" src="https://user-images.githubusercontent.com/69136340/201184130-ae9fc07f-13c5-4f13-b611-9acbfaff39ca.png">

이러한 세부정보를 보고 편집할 수 있는 방법을 제공해보겠습니다.

설명을 편집하는 TextField 와 작업 완료 여부를 표시하는 Toggle 이 있는 상세 뷰를 만듭니다.

<img width="700" alt="46" src="https://user-images.githubusercontent.com/69136340/201184245-88d85212-c714-4546-a08c-64823dd58b9e.png">

예상 작업량은 값이 모두 숫자이고, 유효한 값의 범위를 지정할 수 있습니다.

### 👉 watchOS 9 부터 Stepper 사용 가능

Stepper 는 순차적인 값을 편집할 수 있게 세분화된 제어를 제공할 때 훌룡한 옵션입니다.

<img width="700" alt="47" src="https://user-images.githubusercontent.com/69136340/201184315-8adbb7f0-21ea-4457-8b75-8e009661369d.png">

또한, Stepper 를 사용하여 논리적, 순차적으로 편집할 수 있지만 반드시 숫자일 필요는 없습니다. 예를 들어 항목에 대한 예상 스트레스 수준을 기록하고 싶을 수 있습니다.

<img width="700" alt="48" src="https://user-images.githubusercontent.com/69136340/201184363-ebcfca49-45a4-492c-8d2e-7e75565d301d.png">

(참고 : StressStepper 의 자식 뷰인 Stepper 가 값을 수정하기 위해서 $ 를 붙여서 사용하게 됩니다.)

## 5️⃣ Share with a friend

**🧑‍🏭 : 목록에 있는 항목을 친구와 공유해보면 좋을 것 같습니다.**

detail view 에 항목을 공유할 수 있도록 share sheet 를 사용해보겠습니다.

<img width="700" alt="49" src="https://user-images.githubusercontent.com/69136340/201184494-ae5db89d-b979-4a72-8873-f25f82d9d64b.png">

친구 목록에서 선택해서 메시지를 편집하여 도움을 요청할 수 있습니다.

<img width="700" alt="50" src="https://user-images.githubusercontent.com/69136340/201184539-b7ed2e46-7f0b-4d7f-8433-f0daec13a1e9.png">

<img width="700" alt="51" src="https://user-images.githubusercontent.com/69136340/201184623-103f1dc4-d1d4-4e98-bb7a-6d57b52b79d6.png">

이를 위해서 watchOS 9 부터 SwiftUI 새로운 도구인 ShareLink 를 사용할 수 있습니다.

<img width="700" alt="52" src="https://user-images.githubusercontent.com/69136340/201184686-20f52ef5-c732-43cb-a97c-2f9cd4f354f8.png">

- item 으로 ShareLink 를 만들어서 공유할 수 있습니다. 메시지의 초기 텍스트를 subject 와 message 로 사용자 정의할 수 있습니다.
- 공유할 때 share sheet 에 표시할 미리보기도 preview 로 제공할 수 있습니다.(위의 공유 시트를 보게 되면 리스트의 아이템이었던 “Design new app” 이 미리보기로 제공되었다.)
- ShareLink 를 사용하여 iOS, macOS 및 watchOS 의 SwiftUI 앱에서 공유할 수 있습니다.

Meet Transferable 세션을 꼭 확인해 보세요. ShareLink 에 대한 자세한 내용과 옵션을 알 수 있습니다.

<img width="700" alt="53" src="https://user-images.githubusercontent.com/69136340/201184732-722cdbfb-566e-4810-819c-2fffec8b1743.png">

## 6️⃣ Add a chart

**🧑‍🏭 : 이제 항목을 완료한 시간을 추적하고, 작업을 완료하기 위해 도움을 요청할 수 있으므로 생산성을 보기 위해 차트를 추가하고 싶습니다.**

단일 데이터와 고유한 값이 있기 때문에 막대 차트를 사용하겠습니다.

<img width="700" alt="54" src="https://user-images.githubusercontent.com/69136340/201184764-051a7e28-8671-4699-beb6-b152d0f7c119.png">

앱의 navigation structure 에 chart view 를 추가하는 것으로 시작하겠습니다.

list-detail 관계가 없기 때문에 page-based navigation strategy 선택하여 언제든지 목록과 차트 사이를 스와이프할 수 있도록 하겠습니다.

<img width="700" alt="55" src="https://user-images.githubusercontent.com/69136340/201184818-737f2698-6f77-47ac-a997-5b847abf2771.png">

이를 위해서 목록 뷰를 캡슐화하는 `ItemList` 부터 만들겠습니다.

<img width="700" alt="56" src="https://user-images.githubusercontent.com/69136340/201184853-58886873-f70b-4b43-a829-82a0b38881f1.png">

차트 뷰에 대한 구조를 만들겠습니다. 우선, navigation structure 에 집중하기 위해서 임시로 이미지를 넣겠습니다.

<img width="700" alt="57" src="https://user-images.githubusercontent.com/69136340/201184896-ae2160de-4607-45bf-a3ad-722cfe1d73e1.png">

page-style 의 TabView 로 설정하였습니다.

<img width="700" alt="58" src="https://user-images.githubusercontent.com/69136340/201184941-750a29c8-f6f5-4bc0-b409-57eb9a7de884.png">

SwiftUI Canvas 를 사용해서 차트를 그릴 순 있지만 watchOS 9 부터는 Swift Charts 가 있습니다. 이는 iOS, macOS, tvOS 에서도 사용할 수 있으므로 SwiftUI 를 사용하는 모든 곳에서 차트를 재사용할 수 있습니다.

우리는 이 차트를 날짜별로 완료된 항목 수를 표시하려고 합니다.

그리고 데이터를 집계하는 `createData(_:)` 메서드도 작성합니다.

<img width="700" alt="59" src="https://user-images.githubusercontent.com/69136340/201184986-7d3c8e63-34fb-4eb9-be08-a70e178b6666.png">

(같은 날짜의 data 갯수만큼 완료되었다고 설정하는 메서드)

data 로 계열을 정의하고, 날짜를 x 값으로, y 는 완료된 항목의 수로 차트를 표시하겠습니다.

<img width="700" alt="60" src="https://user-images.githubusercontent.com/69136340/201185024-409c88db-6338-4892-8884-2c1d38c91066.png">

chartXAxis modifier 를 사용하여 AxisValueLabel 의 스타일을 지정하였습니다.(axis 는 축)

세로 눈금선을 원하지 않기 때문에 AxisGridLine 를 생략했습니다.

<img width="700" alt="61" src="https://user-images.githubusercontent.com/69136340/201185090-5b511597-5123-4608-83f9-f57db262c90f.png">

chartYAxis modifier 로 y 축을 정의하겠습니다.

AxisGridLine 으로차트와 어울리는 눈금선 스타일을 지정하였습니다.

AxisValueLabel 의 format 을 정수로 지정하고 상단 레이블은 생략하여 차트 상단이 잘리는 것을 방지했습니다.

<img width="648700" alt="62" src="https://user-images.githubusercontent.com/69136340/201185139-477b8ce8-c6bd-4c55-9dd7-937b1e9cf401.png">

Swift Charts 로 할 수 있는 것을 자세히 알아보려면 아래의 세션을 확인하세요.

<img width="700" alt="63" src="https://user-images.githubusercontent.com/69136340/201185164-02a6208c-6423-4368-8a1a-bf55f8e12818.png">

## 7️⃣ Scroll with the Digital Crown

**🧑‍🏭 : 더 많은 데이터를 보여주고 싶습니다. 스크롤이 가능하도록 해보겠습니다.**

이를 달성하기 위해 digitalCrownRotation modifier 사용할 것입니다. Digital crown 이벤트에 대한 콜백을 설정할 수 있습니다. 이를 통해 차트에 대한 사용자 정의 스크롤 동작을 구현하겠습니다.

<img width="700" alt="64" src="https://user-images.githubusercontent.com/69136340/201185209-d068df5c-d1f0-40cc-b263-73fd893c9501.png">

- `highlightedDateIndex` 를 통해 현재 스크롤 위치에 대한 인덱스를 사용하겠습니다.
- 차트를 스크롤할 때 현재 crown 위치를 표시할 수 있도록 `crownOffset` 에 저장하겠습니다. crown 이 움직이는 동안 데이터 포인트 사이의 값입니다.
- 활발하게 스크롤하는 경우를 추적하기 위해서 `isCrownIdle` 을 사용합니다. 이를 사용하여 crown 스크롤이 멈추고 시작될 때 약간의 애니메이션을 추가하겠습니다.

digitalCrownRotation modifier 를 추가할 수 있습니다.

<img width="700" alt="65" src="https://user-images.githubusercontent.com/69136340/201185236-60300290-95a7-4feb-acb9-9b55857cd239.png">

- 클로저를 가지는 파라미터는 `onChange` 콜백 핸들러입니다. 여기서 `isCrownIdle` 값을 `false` 로 설정합니다.
- 스크롤 되고 있기 때문에 `crownOffset` 값을 현재 값으로 설정해줍니다.
- `onIdle` 콜백 핸들러에서 `isCrownIdle` 값을 `true` 로 설정합니다.

**detent 라는 용어를 개발자 문서에서 사용하는데 멈춤쇠라고 해석하기에 조금 부족함이 있었다. 그러던 중 WWDC22 세션 `Build a productivity app for Apple Watch` 에서 듣게 되어서 발췌했다.**

### 👉 detent?

detent 는 기계적 용어로 움직일 만큼 충분한 힘이 가해질 때까지 무언가를 제자리에 고정시키는 메커니즘입니다.

예를 들어, 차 문을 열 때 안착되는 ‘정지’ 위치가 있다. 문을 조금 더 세게 밀어 또 다른 ‘정지’ 위치까지 더 열 수도 있다.

문을 닫으려면 ‘정지’에서 빼낼 수 있을 만큼 세게 당겨서 저항을 이겨내야 합니다. 그렇지 않으면 문은 ‘정지’ 위치로 돌아갑니다. 이것이 detent 입니다.

---

이처럼 파라미터 `detent` 는 crown 정지 톱니 역할입니다.

이제 차트에서 스크롤할 때 crown 의 위치를 표시할 수 있고, 이를 위해 `Swift Charts` 의 `RuleMark` 를 사용할 수 있습니다. `RuleMark` 는 차트위의 직선입니다. 수평선 혹은 수직선으로 임계값을 표시하거나 경사선을 표시할 수 있습니다.

<img width="700" alt="66" src="https://user-images.githubusercontent.com/69136340/201185310-b9983a0f-8fc1-4bd6-8063-7da9ec589736.png">

- crownOffsetDate 으로 crown 의 현재 위치를 표시하겠습니다.

**🧑‍🏭 : 보기 좋게 하기 위해서 crown 이 움직이지 않을 때 선을 희미하게 하고 싶습니다.**

이는 isCrownIdle 속성을 사용하여 간단한 애니메이션을 할 수 있습니다.(앞서 언급했던 스크롤이 멈추고 시작할 때의 애니메이션을 말함)

<img width="700" alt="67" src="https://user-images.githubusercontent.com/69136340/201185360-f6834858-6760-40f8-b683-50f6b386d989.png">

- chart 에 `onChange` modifier 를 추가해서 `isCrownIdle` 값이 변경될 때 `crownPositionOpacity` 값 변경이 애니메이션과 함께 하도록 합니다.
- `crownPositionOpacity` 를 활용해서 `RuleMark` `foregroundStyle` 에 반영할 수 있습니다.

스크롤할 때 차트의 막대 옆에값을 표시하기 위해서 annotation 을 추가하면 됩니다.

<img width="700" alt="68" src="https://user-images.githubusercontent.com/69136340/201185397-223db161-32cc-43fb-97c5-ee253a8f2ef5.png">

상단의 후행에 값을 추가하는데 마지막 막대에서는 상단 선행에 추가하겠습니다.

<img width="700" alt="69" src="https://user-images.githubusercontent.com/69136340/201185450-6aaa513d-0d16-4566-902c-cbed6345e109.png">

**🧑‍🏭 : 스크롤 가능한 사용자 정의 차트를 만드는 마지막 단계는 스크롤할 때 차트의 데이터 범위를 조정하는 것입니다.**

차트에 범위 데이터를 제공하기 위해서 `chartData` 변수를 생성하고 `highlightedDateIndex` 값이 변경될 때 필요한 경우 업데이트 합니다.

<img width="700" alt="70" src="https://user-images.githubusercontent.com/69136340/201185544-88af9026-05d2-4ed9-967c-243c0715ba18.png">

**(23분 8초부터 구현된 기능 확인 가능.)**

watchOS 9 에서 사용할 수 있는 새로운 SwiftUI 기능에 대해 자세히 알아보려면 아래의 세션을 시청해주세요.

<img width="700" alt="71" src="https://user-images.githubusercontent.com/69136340/201185576-88fbf129-dc85-42da-a50f-a4aa83e0cada.png">
