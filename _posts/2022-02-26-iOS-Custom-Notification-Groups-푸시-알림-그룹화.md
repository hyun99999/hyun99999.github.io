---
title:  "iOS) Custom Notification Groups - 푸시 알림 그룹화"
categories:
- iOS

date:   2022-02-26  01:49:00 +0900
author_profile: false
---
### 내용

- Notification 의 그룹화는 iOS 12 의 새로운 기능입니다.
- Grouped Notifications 대해서 알아보고 그룹화 튜토리얼을 진행해봅시다!
- 궁극적으로, Remote Notifications 의 경우에 어떻게 그룹화를 세팅해야 하는지 알아봅시다!

- 아래의 출처를 참고했습니다.

[iOS 12: Custom Notification Groups](https://medium.com/swift2go/ios-12-custom-notification-groups-1b8bb1457c5b)

[Using Grouped Notifications - WWDC18 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2018/711/)

## 💡Grouped Notifications

앱에서 보내는 알림을 그룹화하면 사용자들이 한눈에 더 많은 정보를 얻고, 여러 알림을 한 번에 관리할 수 있습니다. notification 의 그룹화는 아래의 왼쪽 사진과 같습니다.

*많이들 보셨을거에요!!* 🙂

<img src="https://user-images.githubusercontent.com/69136340/155752467-03eeddf4-3859-4e40-8c1f-3836849b0d24.png" width ="550">

iOS 12 에는 `grouped notifications` 에 대한 3가지 세팅이 있습니다.

- `Automatic`: 기본적인 설정으로 각 앱에 대해 “smart” notification groups 를 가질 수 있다. title, content, what it say, 메일 앱의 경우 sender(보낸 사람) 기준으로 합니다.
- `By App`: 기본적으로 모든 notifications 은 title, content, sender 별로 구분되지 않고, 앱별로 단일 그룹으로 누적됩니다.
- `Off`: 각 앱에 대한 각 notifications 이 개별적으로 보여지는 iOS 12 이전의 방식 고수하는 경우입니다.

- [설정] > 해당 앱으로 이동 > [알림] > [알림 그룹 설정] 에서 볼 수 있는 3가지 세팅입니다!

<img src="https://user-images.githubusercontent.com/69136340/155755494-65273aab-29a7-4829-a825-5795c68ad263.jpeg" width ="300">

## 💡Schedule Notifications

- 튜토리얼의 진행을 위해 `UNMutableNotificationContent` 을 사용해서 시뮬레이터의 Local Notifications 를 트리거 해보겠습니다.

먼저 알림을 보내려면 사용자 권한이 있어야 합니다. 아래의 코드를 `viewDidLoad` 에 추가해서 권한을 얻을 수 있습니다.

```swift
// Ask for Notification Permissions 
let center = UNUserNotificationCenter.current()
center.requestAuthorization(options: [.sound, .alert, .badge]) { granted, _ in
    DispatchQueue.main.async {
        if granted {
            UIApplication.shared.registerForRemoteNotifications()
        } else {
            // Handle error or not granted scenario
        }
    }
}
```

*(**Note**: 튜토리얼을 위한 것이므로 `viewDidLoad` 보다 더 나은 위치에서 요청 승인을 처리해야 합니다.)*

```swift
// ✅ 5초 후에 트리거될 UNMutableNotificationContent 를 만드는 코드이다.
func scheduleGroupedNotifications() {
    for i in 1...5 {
        let notificationContent = UNMutableNotificationContent()
        notificationContent.title = "Hello!"
        notificationContent.body = "Do not forget the pizza!"
        notificationContent.sound = UNNotificationSound.default

// ✅ 5초 뒤에 notification 이 올 수 있도록 UNTimeIntervalNotificationTrigger 을 통해서 트리거 생성.
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
// ✅  Schedule the notification.
        let request = UNNotificationRequest(identifier: "\(i)FiveSecond", content: notificationContent, trigger: trigger)
        let center = UNUserNotificationCenter.current()
        center.add(request) { (error: Error?) in
            if let theError = error {
                print(theError)
            }
        }
    }
}
// foreground 에서는 알림이 배너로 등장하지 않기 때문에 background 로 돌려주어야한다.
// 즉, 화면을 잠구거나 홈으로 나가주면 된다!
```
<img src="https://user-images.githubusercontent.com/69136340/155753512-655ae249-b55a-4cc1-bff9-9b939786593e.gif" width ="300">

## 💡Custom Notification Groups

어떤 상황에서는 `Automatic` 그룹화가 앱에 적합하지 않을 수 있고, 앱의 로직 내부에서 custom groups 를 생성해야 할 수도 있습니다.

특정 인물에게서 오는 알림을 그룹화해야한다고 가정한다면, 이를 위해서 `notificationContent` 객체를 업데이트를 해야합니다!

- `[threadIdentifier](https://developer.apple.com/documentation/usernotifications/unmutablenotificationcontent/1649872-threadidentifier)`: 스레드 고유 식별자입니다. 알림을 시각적으로 그룹화하는 데 사용됩니다.

추가적으로,

- `summaryArgument`: 알림이 카테고리의 요약 형식 문자열에 추가하는 문자열입니다.

위의 튜토리얼 코드를 수정해봅시다!

```swift
func scheduleGroupedNotifications() {
    for i in 1...6 {
        let notificationContent = UNMutableNotificationContent()
        notificationContent.title = "Hello!"
        notificationContent.body = "Do not forget the pizza!"
        notificationContent.sound = UNNotificationSound.default

// ✅ 짝수와 홀수를 구분해서 지정.
        if i % 2 == 0 {
// ✅ threadIdentifier 를 설정해서 그룹화.
            notificationContent.threadIdentifier = "Guerrix-Wife"
            notificationContent.summaryArgument = "your wife"
        } else {
            notificationContent.threadIdentifier = "Guerrix-Son"
            notificationContent.summaryArgument = "your son"
        }

        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)

        let request = UNNotificationRequest(identifier: "\(i)FiveSecond", content: notificationContent, trigger: trigger)
        let center = UNUserNotificationCenter.current()
        center.add(request) { (error: Error?) in
            if let theError = error {
                print(theError)
            }
        }
    }
}
```

- 다음과 같이 그룹화 된 알림과 알림 하단의 “your wife”, “your son” 으로 보이는 정보도 확인 가능합니다.

<img src="https://user-images.githubusercontent.com/69136340/155753956-e982e9ca-9bd8-4904-b8fa-7ae4099f0295.gif" width ="300">

### ❗️ Local Notification 을 통해서 Notification 의 그룹화를 진행해 봤어요! Remote Notification 의 그룹화도 알아봅시다! 😤

---

개발자 문서를 살펴보자!

# ✨Grouping Notifications

notifications 을 threads 로 구성한다.

### **Overview**

시스템 Notification Center 에서 관련 notifications 들을 그룹화합니다. thread identifier 를 추가하여 watchOS 가 앱의 notifications 을 그룹화하는 방법을 제어할 수 있습니다.

### Set the Thread Identifier

시스템은 같은 `category` 와 `thread ID` 로 notifications 을 자동으로 그룹화합니다. local notifications 의 경우, content 의 [threadIdentifier](https://developer.apple.com/documentation/usernotifications/unmutablenotificationcontent/1649872-threadidentifier) 프로퍼티를 설정합니다. **remote notifications 의 경우, `thread-id` key 를 사용합니다.**

```swift
let myPOICategory = "NearbyPlaceOfInterestCategoryIdentifier"
let myPOIThread = "NearbyPlaceOfInterestThreadIdentifier"

// Create the local notification's content.
let content = UNMutableNotificationContent()
content.title = "Grand Canyon"
content.body = "You are within 50 miles of the Grand Canyon"

// Enable grouping by adding a thread identifier.
content.threadIdentifier = myPOIThread

// Then create the request for the notification.
let request = UNNotificationRequest(identifier: myPOICategory,
                                    content: content)
```

### Display Groups in Custom Interfaces

*(watchOS 에 해당하는 내용)*

추가적으로, 커스텀 long-look 인터페이스(*watchOS 의 notifications 를 보여주는 인터페이스의 한 종류*)를 제공하면 그룹화된 notifications 에 응답하여 단일 인터페이스 내에서 여러 알림의 컨텐츠를 표시할 수 있습니다.

만약 custom notification interface 가 화면에 있고 같은 category 와 thread ID 가 있는 새 notification 이 도착하면 시스템은 [didReceive(_:)](https://developer.apple.com/documentation/watchkit/wkusernotificationinterfacecontroller/2963125-didreceive) 메서드를 다시 호출합니다. 기존 인터페이스에 들어오는 content 를 추가할 수 있도록 구현을 준비해야합니다.

 예를들어, custom interface 가 content 를 table view 로 표시하는 경우, 수신 컨텐츠에 대해 새 행을 추가할 수 있습니다.

### ❗️For remote notifications, use the `thread-id` key.

- 개발자문서를 통해서 우리는 서버에서 notification 을 받을 경우에 그룹화를 위해서는 `thread-id` 키를 이용하면 되는 것을 알았어요!

---

- remote notifications 의 경우, JSON payload 를 사용해서 사용자 기기에 notification 을 보내게 되는데 어떻게 `thread-id` 키를 이용하는지 아래의 글을 통해서 자세히 알아보자구요.

[[iOS) Generating a Remote Notification - APNs payload 알아보기](https://gyuios.tistory.com/151)](https://gyuios.tistory.com/151)

### ✨thread-id

`thread-id` 는 관련된 notifications 을 grouping 하기 위한 앱별 식별자입니다. 다음과 같이 payload 를 작성하면 됩니다!

```swift
{
   "aps" : {
      "alert" : {
         "title" : "title title",
         "subtitle" : "subtitle subtitle",
         "body" : "body body"
      },
      "thread-id" : "GROUP_A"
   }
}
```

---

- 출처
[Apple Developer Documentation - notifications](https://developer.apple.com/documentation/watchkit/notifications)
[Apple Developer Documentation - grouping notifications](https://developer.apple.com/documentation/watchkit/notifications/grouping_notifications)
[Push Notifications Tutorial: Getting Started](https://www.raywenderlich.com/11395893-push-notifications-tutorial-getting-started)
