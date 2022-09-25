---
title:  "iOS) Live Activities 와 Dynamic Island 뉴스 살펴보기"
categories:
- iOS

date:   2022-09-25  22:08:00 +0900
author_profile: false
---
### 내용

- New and Updates 로 등장한 뉴스로 Dynamic Island 를 살펴보자.

### 🗞 ****Live Activities now available in beta(****2022.7.27)

[Live Activities now available in beta - Latest News - Apple Developer](https://developer.apple.com/news/?id=hi37aek8)

<aside>
🧑‍🏭 `Live Activities(실시간 현황)` 을 사용해서 앱에서 일어나는 일을 Lock Screen 에서 실시간으로  알 수 있습니다.

iOS 16 의 베타 4에서 Live Activities 와 ActivityKit 프레임워크를 사용할 수 있습니다.

Live Activities 와 ActivityKit는 iOS 16 초기 공개 릴리즈에 포함되지 않습니다. 올 하반기 업데이트에서 부터 공개적으로 사용 가능하며 App Store 에 Live Activities이 포함된 앱을 제출할 수 있습니다.

</aside>

이때까지만해도 dynamic island 라는 기능에 대해서 언급이 없었습니다. 

그리고 iPhone 14 Pro 출시 후 다음과 같은 뉴스가 업데이트 되었습니다.

### 🗞 ****Develop for Live Activities with iOS 16.1 beta and Xcode 14.1 beta(2022.9.14)****

[Develop for Live Activities with iOS 16.1 beta and Xcode 14.1 beta - Latest News - Apple Developer](https://developer.apple.com/news/?id=ttuz9vwq)

<aside>
🧑‍🏭 이제 iOS 16.1 베타 및 Xcode 14.1 베타에서 새로운 ActivityKit 프레임워크를 사용하여 앱에 실시간 현황(Live Activities)을 빌드할 수 있습니다. Live Activities 는 실시간으로 업데이트 되는 앱의 콘텐츠를 계속 추적할 수 있습니다.
Live Activities 는 Lock Screen 과 Dynamic Island(iPhone 14 Pro와 14 Pro Max에서 경험할 수 있는 직관적이고 다채로운 새 다자인) 에 표시됩니다.
Live Activities 와 AcitivityKit 은 올해 말부터 사용가능한 iOS 16.1 에 포함됩니다. iOS 16.1 Release Candidate 가 사용가능하게 되면 Live Activities 를 사용하는 앱을 App Store 에 제출할 수 있게 됩니다.

</aside>

그리고 ActivityKit 에 대한 설명에서 Dynamic Island 가 언급됩니다!

Live Activities 를 표시하는 방법으로 ActivityKit 프레임워크를 사용하여 dynamic island 와 lock screen 을 사용할 수 있을 것으로 보입니다.

아직 베타지만 AcitivityKit 프레임워크 개발자 문서를 살펴보도록 해봐여!

[Apple Developer Documentation](https://developer.apple.com/documentation/activitykit)

# **ActivityKit**

Share live updates from your app as Live Activities in the Dynamic Island and on the Lock Screen.

`iOS 16.1+` / `iPadOS 16.1+` / `Mac Catalyst 16.1+`

ActivityKit 프레임워크를 사용하면 `Live Activity` 를 시작하여 dynamic island 및 lock screen 에서 라이브 업데이트를 공유할 수 있습니다. 예를 들어, 스포츠 앱을 사용하면 게임 동안 실시간 정보를 한 눈에 볼 수 있도록 `Live Activity` 를 시작할 수 있습니다.

Lock Screen 에서는 Live Activity 가 화면 하단에 나타납니다. dynamic island 를 지원하는 기기는 Live Activity 가 홈 화면의 dynamic island 에 표시되며 다른 앱을 사용하는 동안에도 나타납니다. 사람들은 dynamic island 에서 Live Activity 을 탭하여 앱을 실행하거나 길게 터치하여 더 많은 콘텐츠가 포함된 expanded view 를 표시합니다.

앱에서 ActivityKit 을 사용하여 Live Activity 를 구성, 시작, 업데이트, 종료 하고 앱의 위젯 extension 은 SwiftUI 와 WidgetKit 을 사용하여 Live Activity 의 인터페이스를 생성합니다. 이것은 Live Activity 의 presentation code 를 widget code 와 유사하게 만들고 widget 과 Live Activity  간의 코드 공유를 가능하게 합니다.

그러나, Live Activity 는 widget 과 비교하여 업데이트를 수신하는데 다른 메커니즘을 사용합니다. timeline 메커니즘을 사용하는 대신 Live Activity 는 ActivityKit 을 사용하거나 remote push notification 을 수신하여 앱에서 업데이트된 데이터를 수신합니다.

Live Activity 는 iPhone 에서만 사용가능합니다.

### 출처

[Apple Developer Documentation](https://developer.apple.com/documentation/activitykit)
