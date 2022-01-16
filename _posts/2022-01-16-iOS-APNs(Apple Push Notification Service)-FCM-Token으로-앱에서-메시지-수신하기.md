---
title:  "iOS) APNs(Apple Push Notification Service) - FCM Token 으로 앱에서 메시지 수신하기"
categories:
- iOS

date:   2022-01-16  09:43:00 +0900
author_profile: false
---
아래의 게시물은 제가 이전에 작성해둔 푸시알림 APNs 설정 및 파이어베이스에 앱 추가하는 과정입니다.

[iOS) APNs(Apple Push Notification Service)](https://gyuios.tistory.com/42)

- 이전 게시물에서는 파이어베이스에서 메시지를 테스트해보았다.
- 이번 프로젝트에서는 p8 인증키로 진행했다.
- 이번 프로젝트를 진행하면서 서버분들과의 협업으로 FCM Token 을 사용해서 앱에서 메시지를 수신하는 기능을 구현해 보았다.

# 📌iOS 클라이언트 설정

```swift
import UIKit

// import
import Firebase
import FirebaseMessaging

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
          
        // Firebase 초기화 세팅.
        FirebaseApp.configure()

        // 메시지 대리자 설정
        Messaging.messaging().delegate = self

        // FCM 다시 사용 설정
        Messaging.messaging().isAutoInitEnabled = true
        
        // 푸시 알림 권한 설정 및 푸시 알림에 앱 등록
        UNUserNotificationCenter.current().delegate = self
        let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]
        UNUserNotificationCenter.current().requestAuthorization(options: authOptions, completionHandler: { _, _ in })
        application.registerForRemoteNotifications()

        // device token 요청.
        UIApplication.shared.registerForRemoteNotifications()
        
        return true
    }

    /// APN 토큰과 등록 토큰 매핑
    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        Messaging.messaging().apnsToken = deviceToken
    }
}
extension AppDelegate: MessagingDelegate {
    /// 현재 등록 토큰 가져오기.
    func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {

        // TODO: - 디바이스 토큰을 보내는 서버통신 구현
        
        sendDeviceTokenWithAPI(fcmToken: fcmToken ?? "")
    }
}
```

## ✨등록 토큰 액세스

다음과 같은 경우에 등록 토큰이 변경될 수 있다.

- 새 기기에서 앱 복원
- 사용자가 앱 삭제/재설치
- 사용자가 앱 데이터 소거

### 메시지 대리자 설정

```swift
등록 토큰을 받으려면 [FIRApp configure]를 호출한 후
메시지 대리자 프로토콜을 구현하고 FIRMessaging의 delegate 속성을 설정합니다.
예를 들어 애플리케이션 대리자가 메시지 대리자 프로토콜을 준수하는 경우
application:didFinishLaunchingWithOptions:에서 대리자를 애플리케이션 대리자로 설정할 수 있습니다.
```

**라고 파이어베이스 개발문서에서는 명시되어있다. 어렵게 생각하지 않아도 된다. delegate 속성을 설정해서 대리자를 설정할 수 있다는 말이다. 그 후에는 다음과 같이 구현해주면 된다.`extension** AppDelegate: MessagingDelegate { ... }` 

```swift
Messaging.messaging().delegate = self
```

### 현재 등록 토큰 가져오기

- 등록 토큰은 `messaging:didReceiveRegistrationToken:` 메서드를 통해 전달됩니다.
- 일반적으로 앱 시작 시 등록 토큰을 사용하여 이 메서드를 한 번 호출합니다. 이 메서드가 호출되면 다음과 같은 작업을 할 수 있습니다.
    - 새 등록 토큰이라면 애플리케이션 서버에 전송합니다.
    - 등록 토큰을 주제에 구독 처리합니다. 이 작업은 신규 구독 또는 사용자의 앱 재설치 같은 상황에서만 필요합니다.

```swift
extension AppDelegate: MessagingDelegate {
    func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {

        // TODO: - 디바이스 토큰을 보내는 서버통신 구현
        
        sendDeviceTokenWithAPI(fcmToken: fcmToken ?? "")
    }
}
```

**위처럼 일반적으로 앱 시작 시 등록 토큰을 사용해서 한 번 호출해보자. 아래는 등록 토큰에 엑세스하는 또 다른 방법이다.**

- [token(completion:)](https://firebase.google.com/docs/reference/swift/firebasemessaging/api/reference/Classes/Messaging#tokencompletion:)을 사용하여 직접 토큰을 가져올 수 있습니다. 어떤 방식으로든 토큰 가져오기에 실패할 경우 null이 아닌 오류가 제공됩니다.

```swift
Messaging.messaging().token { token, error in
  if let error = error {
    print("Error fetching FCM registration token: \(error)")
  } else if let token = token {
    print("FCM registration token: \(token)")
    self.fcmRegTokenMessage.text  = "Remote FCM registration token: \(token)"
  }
}
```

언제든지 이 메서드를 사용하여 토큰을 저장하지 않고도 토큰에 엑세스할 수 있습니다.

### APN 토큰과 등록 토큰 매핑

- 위의 작업을 모두 진행했다면 이제 APN 토큰을 명시적으로 FCM 등록 토큰에 매핑해야 한다.
- `didRegisterForRemoteNotificationsWithDeviceToken` 메서드를 재정의하여 APN 토큰을 검색한 후 `FIRMessaging`의 `[APNSToken](https://firebase.google.com/docs/reference/ios/firebasemessaging/api/reference/Classes/FIRMessaging#/c:objc(cs)FIRMessaging(py)APNSToken)` 속성을 설정합니다.

```swift
func application(application: UIApplication,
                 didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
  Messaging.messaging().apnsToken = deviceToken
}
```

## ✨자동 초기화 방지

- FCM 등록 토큰이 생성되면 라이브러리는 식별자와 구성 데이터를 Firebase에 업로드합니다. 사용자에게 먼저 명시적인 수신 동의를 얻으려면 FCM을 사용 중지하여 구성 시 토큰이 생성되지 않게 하면 됩니다.
- 이렇게 하려면 메타데이터 값을 `Info.plist`(`GoogleService-Info.plist` 아님)에 추가합니다.

`FirebaseMessagingAutoInitEnabled = NO`

- FCM을 다시 사용 설정하려면 런타임 호출을 만들면 됩니다.

```swift
Messaging.messaging().autoInitEnabled = true
```

이 값을 설정하고 나면 앱을 다시 시작해도 유지됩니다.

# 📌테스트 메시지 보내기

## ✨알림 메시지 전송

1. 대상 기기에 앱을 설치하고 실행합니다. 원격 알림 수신에 대한 허가 요청을 수락해야 합니다.
2. 앱을 기기에서 백그라운드 상태로 만듭니다.
3. [알림 작성기](https://console.firebase.google.com/project/_/notification)를 열고 **새 알림**을 선택합니다.
4. 메시지 본문을 입력합니다.

<img src="https://user-images.githubusercontent.com/69136340/149642769-43815b23-e283-47ea-b7fe-9125985c6dcd.png" width = "600">
    
5. **테스트 메시지 보내기**를 선택합니다.
6. **FCM 등록 토큰 추가** 필드에 이 가이드의 앞 섹션에서 가져온 등록 토큰을 입력합니다.

<img src="https://user-images.githubusercontent.com/69136340/149642788-f03fb5d1-1c1d-4680-965d-0bd32542bfe9.png" width = "450">
    
7. **테스트**를 클릭합니다.

# 📌앱에서 메시지 수신

## ✨알림처리

- FCM은 APN을 통해 Apple 앱을 타겟팅하는 모든 메시지를 전송합니다. UNUserNotificationCenter를 통해 APN 알림을 수신하는 방법에 대한 자세한 내용은 Apple의 [Handling Notifications and Notification-Related Actions(알림 및 알림 관련 작업 처리)](https://developer.apple.com/documentation/usernotifications/handling_notifications_and_notification-related_actions?language=objc) 문서를 참조하세요.
- [UNUserNotificationCenter delegate](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649522-delegate)를 설정하고 적절한 대리자 메서드를 구현하여 FCM에서 디스플레이 알림을 수신해야 합니다.

**라고 파이어베이스 개발자문서에서 명시하고 있다. 링크된 문서가 무엇인지 궁금할텐데 링크된 애플개발자 문서는 `해당 알림의 액션을 핸들링하는 방법`과 `foreground 에서 알림을 핸들링하는 방법` 을 명시하고 있다.** 

**짧게 언급만 하자면 해당 알림에는 `categoryIdentifier` 가 존재해서 알림에 대해서 핸들링이 가능하고, `actionIdentifier` 가 존재해서 알림의 액션도 핸들링이 가능하다.**

<img src="https://user-images.githubusercontent.com/69136340/149642793-538246c3-3dfc-4bce-90fe-4c8ccc6709c0.png" width ="300">

출처: [https://developer.apple.com/documentation/usernotifications/handling_notifications_and_notification-related_actions?language=objc](https://developer.apple.com/documentation/usernotifications/handling_notifications_and_notification-related_actions?language=objc)

```swift
// 메시지 수신
extension AppDelegate: UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        let userInfo = notification.request.content.userInfo
// With swizzling disabled you must let Messaging know about the message, for Analytics
        Messaging.messaging().appDidReceiveMessage(userInfo)
        
        completionHandler([.sound, .banner, .list])
    }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        let userInfo = response.notification.request.content.userInfo
// With swizzling disabled you must let Messaging know about the message, for Analytics
        Messaging.messaging().appDidReceiveMessage(userInfo)
        
        completionHandler()
    }
    

    func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable : Any], fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
        Messaging.messaging().appDidReceiveMessage(userInfo)
        
        completionHandler(.noData)
    }
}
```

알림에 커스텀 작업을 추가하려면 [알림 페이로드](https://firebase.google.com/docs/cloud-messaging/http-server-ref#notification-payload-support)에서 `click_action` 매개변수를 설정하세요. 이때 APN 페이로드의 `category`키에 사용할 값을 사용하세요. 커스텀 작업을 사용하려면 먼저 등록해야 합니다. 자세한 내용은 Apple의 [Local and Remote Notification Programming Guide(로컬 및 원격 알림 프로그래밍 가이드)](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SupportingNotificationsinYourApp.html#//apple_ref/doc/uid/TP40008194-CH4-SW26)를 참조하세요.

**다음은 알림 페이로드에서 캡쳐한 부분이다.**

<img src="https://user-images.githubusercontent.com/69136340/149642807-7f28496d-62cd-4945-88ad-67737a294895.png" width ="600">

출처: [https://firebase.google.com/docs/cloud-messaging/http-server-ref#notification-payload-support](https://firebase.google.com/docs/cloud-messaging/http-server-ref#notification-payload-support)

**알림 페이로드에서 `click_action` 이 APN 페이로드의 `category` 키에 해당한다고 했는데, 이것이 앞서 잠깐 언급했던 `categoryIdentifier` 로 넘어오는 문자열이고 이를 통해서 알림별 분기처리가 가능하다.**

## ✨자동 푸시 알림 처리

- 메시지가 자동 알림으로 전달되어 백그라운드 데이터 새로고침과 같은 작업을 위해 앱이 백그라운드에서 실행됩니다.
- 포그라운드 알림과 달리 이러한 알림은 `[appDelegate(_:didReceiveRemoteNotification:fetchCompletionHandler:)](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623013-application)` 메서드를 통해 처리되어야 합니다.

**라고 파이어베이스 개발자문서에 명시하고 있다. 즉, 백그라운드로 오는 알림은 위의 메서드를 통해서 처리해주어야 한다는 이야기이다.**

```swift
// 백그라운드에서 자동 푸시 알림 처리
    func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable : Any], fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
        // If you are receiving a notification message while your app is in the background,
        // this callback will not be fired till the user taps on the notification launching the application.
        
        // With swizzling disabled you must let Messaging know about the message, for Analytics
        Messaging.messaging().appDidReceiveMessage(userInfo)
        
        completionHandler(UIBackgroundFetchResult.newData)
    }
```

## ✨알림 메시지 페이로드 해석

알림 메시지의 페이로드는 키와 값으로 구성된 사전입니다. APN을 통해 전송된 알림 메시지는 다음과 같이 APN 페이로드 형식을 따릅니다.

```swift
{
    "aps" : {
      "alert" : {
        "body" : "great match!",
        "title" : "Portugal vs. Denmark",
      },
      "badge" : 1,
    },
    "customKey" : "customValue"
  }
```

---

- **서버와의 협업 없이도 푸시 알림이 잘오는지 확인해보자! 아래의 글을 참조하면 Postman 을 통해서도 푸시 알림을 받아 볼 수 있다.**

### Postman 을 이용해서 firebase notification 전송하기

[Postman을 이용하여 Firebase Notification 전송하기](https://hk5061.tistory.com/50)

---

- 출처:

[Set up a Firebase Cloud Messaging client app on Apple platforms | Firebase Documentation](https://firebase.google.com/docs/cloud-messaging/ios/client)

[Receive messages in an Apple app | Firebase Documentation](https://firebase.google.com/docs/cloud-messaging/ios/receive)
