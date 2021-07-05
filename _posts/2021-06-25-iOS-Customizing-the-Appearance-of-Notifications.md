---
title:  "iOS) AppleDeveloper - Customizing the Appearance of Notifications"
categories:
- iOS

date:   2021-06-25 15:50:00 +0900
author_profile: false
---
Customizing the Appearance of Notifications - notifications interface 를 커스텀 해보자

# Apple Developer - Customizing the Appearance of Notifications

### Overview

iOS 장비가  alert 가 포함된 notification 을 받으면 시스템은 alert 내용을 두 단계로 표시합니다. 처음에는 title, subtitle, two to four lines of body text 가 포함된 축약된 banner 을 표시합니다. 축약된 banner 를 누르면 notification-based actions 과 전체 notification interface 를 표시합니다. 시스템은 축약 된 배너에 대한 인터페이스를 제공하지만 ntification content app extension 을 사용하여 전체 interface 를 커스텀할 수 있습니다.

<p align = "center"><img src = "https://user-images.githubusercontent.com/69136340/123389618-05fab980-d5d5-11eb-87d0-a34baab4550e.png" width ="300"></p>

notification content app extension 은 custom notification interface 를 표시하는 뷰컨트롤러를 관리한다. 이 뷰컨트롤러는 notifications 에 대한 기본 시스템 인터페이스를 대체하거나 추가할 수 있다. 뷰컨트롤러에서 내가 할 수 있는 것은 다음과 같다.

- alert's title, subtitle, body text 의 위치를 커스텀
- interface 요소의 fonts 또는 styling 대체
- 앱별 데이터
- custom image 또는 branding 을 포함

알림이 적시에 전달되도록 가능한 네트워크를 통해 데이터를 검색하는 것과 같이 장기 실행 작업을 수행하지 않도록 한다.

### Add the Notification Content App Extension to Your Project

To add a notification content app extension to your iOS app:

1.Choose File > New > Target in Xcode.

<p align = "center"><img src = "https://user-images.githubusercontent.com/69136340/123389723-21fe5b00-d5d5-11eb-928a-57cdf1a4f859.png" width ="500"></p>


2.Select Notification Content Extension from iOS Application Extension.

<p align = "center"><img src = "https://user-images.githubusercontent.com/69136340/123389791-36425800-d5d5-11eb-8628-c0a1f27276b4.png" width ="600"></p>


3.Click Next.

4.Provide a name for your app extension.

5.Click Finish.

Note) 하나 이상의 notification content app extension 을 사용하려면 Info.plist 파일에서 지정해주면 된다. 

[Apple Developer Documentation - Declare the Supported Notification Types](https://developer.apple.com/documentation/usernotificationsui/customizing_the_appearance_of_notifications#2950661)

### Add Views to Your View Controller

<p align = "center"><img src = "https://user-images.githubusercontent.com/69136340/123389888-4d814580-d5d5-11eb-883e-773a15e11649.png" width ="400"></p>

Xcode 가 제공하는 template 은 storyboard 와 view controller 를 포함한다. 

label 뿐만 아니라 image views 와 vies 를 사용할 수 있다. 또한 iOS12 이상부터는 interactive controls 를 사용할 수 있다. (buttons or switches)

important) 추가적인 view controllers 를 app extension 에 추가하거나 stroyboard 파일을 추가하지 마시오. app extension 은 무조건 하나의 view controller 포함해야한다.

### Configure Your View Controller

뷰컨트롤러의  `didReceive(_:)` 메서드를 사용해서 labels 과 다른 views 를 업데이트 한다. notification payload 에는 뷰컨트롤를 구성할 때 사용할 데이터를 포함한다. 

아래의 코드는 title, body text 를 notification payload 에서 조회하고 등록하는 코드이다.

```swift
func didReceive(_ notification: UNNotification) {
   self.bodyText?.text = notification.request.content.body
   self.headlineText?.text = notification.request.content.title
}
```

두번째 notification 이 도착하면 시스템은 다시 새로운 notification payload 와 함께 `didReceive(_:)` 메서드를 호출합니다.

### Declare the Supported Notification Types

notification content app extension 이 제공하는 인터페이스를 위해서 notification types 를 지정한다. notification 을 수신하면 시스템은 notification's category 값과 app 의 모든 notification content app extensions 를 매칭합니다. 일치하는 항목을 찾으면 해당 app extension 을 로드합니다.

notification contetn app extension 의 info.plist 파일에서 확장이 가능한 category strings 로 `UNNotificationExtensionCategory` 키를 구성한다. category strings 를 사용해서 앱에서 수신할 수 있는 notifications 를 구분한다. 대소문자를 구분한다.

아래는 notification content app extension 의 info.plist 파일에서 두개의 다른 notificatoin types 를 지원하는 것이다. 

<p align = "center"><img src = "https://user-images.githubusercontent.com/69136340/123393499-1f9e0000-d5d9-11eb-822e-6938ddb2a8bb.png" width ="400"></p>

note) 처음에는 `UNNotificationExtensionCategory` 키의 값이 문자열이므로 하나만 지원할 수 있다. 여러 type 을 지원하려면 유형을 문자열 배열로 변경해라.

- local notification 의 경우

category string 을 `UNMutableNotificationContent` 의 [category identifier](https://developer.apple.com/documentation/usernotifications/unmutablenotificationcontent/1649862-categoryidentifier) 프로퍼티에 넣는다.

- remote notification 의 경우

문자열을 JSON payload 의 category 키에 넣는다.

---

app's notification types 를 선언하는 것의 자세한 내용은 [Declaring Your Actionable Notification Types.](https://developer.apple.com/documentation/usernotifications/declaring_your_actionable_notification_types) 를 참조하자.

### Hide the Default Notification Interface


### 출처
출처ㅣ[Customizing the Appearance of Notifications](https://developer.apple.com/documentation/usernotificationsui/customizing_the_appearance_of_notifications)
