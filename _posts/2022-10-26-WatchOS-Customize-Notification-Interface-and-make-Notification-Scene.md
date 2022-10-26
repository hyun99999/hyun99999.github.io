---
title:  "WatchOS) Customize Notification Interface and make Notification Scene"
categories:
- iOS

date:   2022-10-26  11:14:00 +0900
author_profile: false
---
### 내용

- Notification Scene 을 위한 작업을 알아보자.
- Notification Interface 를 커스텀 해보자.
- 이때 사용되는 프로젝트는 SwiftUI Tutorials 를 사용하여 진행하겠습니다. 기본적으로 [WatchOS) Creating a watchOS App - SwiftUI Tutorials](https://gyuios.tistory.com/244) 과 이어지는 글입니다.

Xcode 14 이전에는 기존에는 아래와 같이 Notification Scene 을 추가하고,

<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/197917656-2a7eaa8a-96cc-49f6-8d4d-95225275f326.png">

위의 파일들이 추가되었지만 Xcode 14 부터는 `Include Notification Scene` 체크박스를 지원하지 않았습니다. 다른 방향으로 추가할 수 있는 법을 아신다면 공유해주시면 감사드리겠습니당 ;-;

그래서 튜토리얼에서 제공하는 프로젝트의 파일들을 바탕으로 만들어나가면서 진행해보겠습니다.

우선, Custom Notificiation Interface 을 만들기 전 마무리되었기 때문에 만들기 위해서 어떻게 해야하는지 알아보겠습니다.

아래의 글을 참고해서 진행하였습니다.

[Apple Developer Documentation - Building a WatchOS App](https://developer.apple.com/documentation/watchos-apps/building_a_watchos_app)

## 1️⃣ Notification Scene 설정

- 앱이 시작되면, window group 에 의해 정의된 view hierarchy 가 표시됩니다. 이때 notification categroies 에 대한 scenes 를 추가할 수 있습니다.

```swift
@main
struct MyWatchAppApp: App {
    var body: some Scene {
        WindowGroup {
            NavigationView {
                ContentView()
            }
        }

// ✅ WKNotificationScenen 구조체를 사용하여 notification scene 을 만들고, notification controller's class 와 category 를 지정하게 됩니다.
        WKNotificationScene(controller: NotificationController.self, category: "myNotificationCategory")
    }
}
```

시스템이 매칭되는 category 의 notification 을 수신하게 된다면 notification controller 로 지정된 dynamic view 를 표시하게 됩니다.

view 를 interactive 하게 만들려면, navigation controller 의 `isInteractive` 프로퍼티를**(라고 문서에 나와있는데 notification controller 의 프로퍼티이다.)** `true` 로 설정해야 합니다. 자세한 내용은 [Apple Developer - Customizing Your Long-Look Interface](https://developer.apple.com/documentation/watchos-apps/customizing-your-long-look-interface) 를 참조하세요.

- `isInteractive` 에 대한 내용을 조금 가져와 보자면 아래와 같이 사용해줄 수 있다.

```swift
// Create a dynamic, interactive notification interface.
override class var isInteractive: Bool {
    return true
}
// ✅ class 키워드는 static 과 같이 타입 프로퍼티를 만들어주지만, static 과 달리 서브 클래스에서 오버라이드가 가능하다.
```

## 2️⃣ Customize the Interface

notification controller 의 `didReceive(_:)` 메소드를 override 해서 들어오는 notification 으로부터 정보를 추출해야 합니다.

```swift
override func didReceive(_ notification: UNNotification) {
    content = notification.request.content
    date = notification.date
}
```

그런 다음, notification’s content 를 사용하여 notification view 를 만듭니다.

```swift
override var body: NotificationView {
    return NotificationView(content: content, date: date)
}
```

자, 이전 포스팅에 이어서 튜토리얼을 마무리 해봅시다.

## 👉 Create a Custom Notification Interface

- 마지막 세션에서 즐겨찾기 위치 중 하나에 가깝다는 notification 을 받을 때마다 landmark 정보를 표시하는 interface 를 만들겠습니다.

<img width="300" alt="2" src="https://user-images.githubusercontent.com/69136340/197917838-9701dca0-2b60-4504-9634-dc4fbca32b1e.png">

앞서 `Include Notification Scene` 체크 박스가 없었기 때문에 튜토리얼 프로젝트를 참고하여 만들어 보겠습니다.

- `NotificationView.swift` : interface 를 커스텀하기 위한 뷰/
    - landmark, title, message 에 대한 정보를 표시하는 뷰를 만들겠습니다.

<img width="500" alt="3" src="https://user-images.githubusercontent.com/69136340/197917863-979c6c85-bcec-4e37-ab77-b268be26beef.png">

- 앞서 살펴본 것 처럼 notification controller 의 body 를 override 할 때 title, message, landmark 를 초기화해줄 수 있습니다.
    - notification’s content 에 정보가 없을 수 있으니 옵셔널 변수로 선언했습니다.

<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/197917886-91be96ca-a6fe-4b20-9716-4c90780a6271.png">

- `NotificationController.swift` : notification 을 수신했을 때 dynamic notification interface 를 구현할 수 있는 곳.
    - `didReceive(_:)` 를 override 해서 들어오는 notification 으로부터 정보를 추출할 수 있다.

<img width="700" alt="5" src="https://user-images.githubusercontent.com/69136340/197917898-03a71de2-2e60-412b-9e75-3c25608bf19f.png">

- notification 을 수신할 때 notification’s category 와 연결되는 부분을 수정하겠습니다.
    - 플랫폼 별로 대응해주기 위해서 `#if os(watchOS)` 구문을 사용해주었습니다.
    - 조건부 컴파일로 notification scene 은 watchOS 에서만 사용되도록 할 수 있습니다.
    - `NotificationController.swift` 를 controller 로 사용하고 notification’s category 는 LandmarkNear 인 경우에 notification scene 을 만들게 됩니다.

<img width="700" alt="6" src="https://user-images.githubusercontent.com/69136340/197917940-e7b4076a-06de-4508-920e-011f561e70ac.png">

## 👉 Test payload

- `PushNotificationPayload.apns` : notificiation controller 에서 예상하는 데이터를 전달할 수 있는 payload.
    - 서버에서 전송하는 remote notification 을 테스트하기 위한 파일이다.

<img width="500" alt="7" src="https://user-images.githubusercontent.com/69136340/197917963-26e82925-53a1-4793-83a9-ea3211d4970a.png">

```swift
{
    "aps": {
        "alert": {
            "title": "Silver Salmon Creek",
            "body": "You are within 5 miles of Silver Salmon Creek."
        },
        "category": "LandmarkNear",
        "thread-id": "5280"
    },

    "landmarkIndex": 1,
    
    "WatchKit Simulator Actions": [
        {
            "title": "First Button",
            "identifier": "firstButtonAction"
        }
    ],
    
    "customKey": "Use this file to define a testing payload for your notifications. The aps dictionary specifies the category, alert text and title. The WatchKit Simulator Actions array can provide info for one or more action buttons in addition to the standard Dismiss button. Any other top level keys are custom payload. If you have multiple such JSON files in your project, you'll be able to select them when choosing to debug the notification interface of your Watch App."
}
```

- `category` : notification identifier 이다. 이를 통해서 notification scene 을 구분하여 만들 수 있다.
- `thread-id` : notification grouping 을 위해 필요한 key.
- `landmarkIndex` : 로컬에 있는 landmark 목록 중에서 index 를 통해서 값에 접근하기 위한 key.

### 👉 빌드해보자!

iOS 와 달리 watch 에서 빌드하자마자 별도의 권한 설정없이 알림을 보낼 수 있는 권한을 요청하게 됩니다.

<img src="https://user-images.githubusercontent.com/69136340/197917992-c9a41137-23fa-41c9-b5f8-7777d41a9145.png" width ="200">

***아래의 글에서 notification 을 테스트 하는 방법을 알아봅시다.***

