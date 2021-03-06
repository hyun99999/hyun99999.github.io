---
title:  "iOS) APNs(Apple Push Notification Service)"
categories:
- iOS

date:   2021-06-17 16:11:00 +0900
author_profile: false
---
서버 push 알림을 구현해보자

# APNsTutorial-iOS
🤨 Apple Push Notification service tutorial

단순히 순서를 따라서 가면 될 줄 알았는데 알아야할 것도 있었고 경우에 따라서 요구하는 파일도 달랐다.

그렇기 때문에 천천히 인내심을 가지고 읽어주면 고맙겠습니다!

먼저 어떤 서버 환경에서 푸쉬알림을 보내줄 건지 그렇다면 server provider 가 요구하는 파일은 무엇인지 알고 진행하기를 바랍니다.

p8(인증키) ,p12(인증서) 파일 모두 만드는 방법을 정리했다. APNs 키는(p8) 계정당 2개까지만 만들 수 있어서 나는 못만들어봤고 출처를 참고했다. 

(참고로 p8(인증키)은 인증갱신을 하지 않아도 되기 때문에 p12(인증서)보다 선호한다. 파이어베이스에서는 둘 다 지원한다.)

이해를 완벽하게 하지 않더라도 아래의 글을 먼저 읽어보는 걸 추천한다.

[[iOS] provider server와 APNs의 안전한 연결을 위한 두가지 방법](https://sujinnaljin.medium.com/provider-server와-apns의-안전한-연결을-위한-두가지-방법-bb82d60ea7c8)

# Apple Push Notification service (APNs) 설정하기

- 단순히 순서를 따라서 가면 될 줄 알았는데 알아야할 것도 있었고 경우에 따라서 요구하는 파일도 달랐다. 그러니 천천히 읽어주시기 바랍니다.

- 먼저 어떤 서버 환경에서 푸쉬알림을 보내줄 건지 그렇다면 server provider 가 요구하는 파일은 무엇인지 알고 진행하기를 바랍니다.
---
p8(인증키) ,p12(인증서) 파일 모두 만드는 방법을 정리했다. APNs 키는(p8) 계정당 2개까지만 만들 수 있어서 나는 못만들어봤고 출처를 참고했다. 

(참고로 p8(인증키)은 인증갱신을 하지 않아도 되기 때문에 p12(인증서)보다 선호한다. 파이어베이스에서는 둘 다 지원한다.)

이해를 완벽하게 하지 않더라도 아래의 글을 먼저 읽어보는 걸 추천한다.

[[iOS] provider server와 APNs의 안전한 연결을 위한 두가지 방법](https://sujinnaljin.medium.com/provider-server와-apns의-안전한-연결을-위한-두가지-방법-bb82d60ea7c8)


❗️이제 푸쉬알림을 사용하기 위한 과정을 알아보자
# Apple Push Notification service (APNs) 설정하기

APNs 란? iOS 에서 지원하는 푸쉬알림이다. 

## 1. CSR 발급

> CSR (Certificate Signing Request : 인증서 서명 요청)

- 키체인을 연다. 키체인 접근 - 인증서 지원 - 인증 기관에서 인증서 요청

<p align="center"><img width = "400" src ="https://user-images.githubusercontent.com/69136340/122346284-be947d80-cf83-11eb-8f8d-86fb0b973ac7.png"></p>

- 이메일 주소와 일반 이름을 입력해주고 요청항목을 디스크에 저장됨 이라고 체크해준다.

<p align="center"><img width = "400" src ="https://user-images.githubusercontent.com/69136340/122346344-d0762080-cf83-11eb-8b74-7dc60ec7eeaf.png"></p>

- 계속 진행하면 된다.

<p align="center"><img src = "https://user-images.githubusercontent.com/69136340/122346566-0e734480-cf84-11eb-862f-50984dd65ee5.png" width ="150"></p>

CertificateSigningRequest.certSigningRequest 명으로 파일이 생성된다.

## 2. App ID 등록 및 Push Notification 활성화

[Apple Developer](https://developer.apple.com)

apple developer 홈페이지에 접속.

(Account ➡️  Certificates, IDs & Profiles ➡️  Identifiers)

- Identifiers 등록

<p align="center"><img width = "400" src ="https://user-images.githubusercontent.com/69136340/122343478-b5ee7800-cf80-11eb-951f-154ea77f3f7a.png"></p>

- App IDs 선택

<p align="center"><img src="https://user-images.githubusercontent.com/69136340/122343591-d3bbdd00-cf80-11eb-8aa0-491802ba9a91.png" width = "600"></p>

<p align="center"><img src="https://user-images.githubusercontent.com/69136340/122343712-f817b980-cf80-11eb-8ad5-5581d1920bc4.png" width = "600"></p>

- Register an App ID

<p align="center"><img src="https://user-images.githubusercontent.com/69136340/122343875-24cbd100-cf81-11eb-8f5b-a4663c8da6dd.png" width ="600"></p>

Bundle ID 는 프로젝트에서 확인 할 수 있습니다!

- 하단에 Push Notifications 선택

<p align="center"><img src="https://user-images.githubusercontent.com/69136340/122343945-37460a80-cf81-11eb-8ee3-da0a9e72f452.png" width ="300"></p>

그리고 만들어진 App ID 를 확인하면 됩니다.

## A. p12 인증서 만들기

p8 인증키를 만들려면 B 로 가면 된다.

### A-1. Certificates 생성

- 아까 만든 Identifier 를 선택해서 하단의 Push Notification 에서 configure 를 클릭한다.

<p align="center"><img src ="https://user-images.githubusercontent.com/69136340/122346833-5eeaa200-cf84-11eb-9c66-0b78f95d1570.png" width = "300"></p>

- 용도에 맞게 인증서를 생성하면 된다. (파이어베이스에서는 Development 와 Production 모두 요구한다.)

<p align="center"><img src ="https://user-images.githubusercontent.com/69136340/122346895-6c079100-cf84-11eb-8ddc-5f30adc63101.png" width = "600"></p>

- 아까 만들어둔 CSR 파일을 업로드한다.

<p align="center"><img src ="https://user-images.githubusercontent.com/69136340/122346932-76298f80-cf84-11eb-9dc6-0de9c05d12a5.png" width = "600"></p>

- 생성한 인증서를 다운로드해준다.

<p align="center"><img src ="https://user-images.githubusercontent.com/69136340/122346998-86da0580-cf84-11eb-954d-490f15d2f73c.png" width ="600></p>


<p align="center"><img width="150" src="https://user-images.githubusercontent.com/69136340/122347045-948f8b00-cf84-11eb-9f91-733a5402d250.png"></p>

확인)

그러면 Account ➡️  Certificates, IDs & Profiles ➡️  Certificates 에서 생성한 인증서를 확인할 수 있다.

<p align="center"><img src ="https://user-images.githubusercontent.com/69136340/122344527-dbc84c80-cf81-11eb-94d1-b7c92ef92d8c.png" width ="700"></p>

### A-2. Certificate 를 키체인에 등록하기

- 다운로드받은 aps_development.cer 파일을 더블클릭해서 키체인에 등록해준다.

(이때 키도 같이 등록되는데 한번만 등록 할 수 있는 듯 하다. 궁금해서 키체인에서 삭제해봤는데 키는 등록이 안된다. identifier 를 새로 등록해주고 인증서도 다시 만들자 등록 할 수 있었다.)

(일부 글에서 p8 파일을 만들기 위해서 apple developer 에서 key 를 등록해주는데 그때만 다운로드가 가능했다. p12 파일에 대해서 키는 이때만 등록가능한 것 같다.)

### A-3. p12 파일 생성

우리가 Push Notification 을 받기 위해선  **pem** 형태의 인증서가 필요하다.(경우에 따라서는 p12 형태의 파일만 요구되기도 한다.) 그렇기 위해서 먼저 p12 형태의 파일을 만들어야 하는데, 이 p12 파일을 생성하기 위해선 **아까 키체인에 등록했던** **인증서**와 **키**가 필요하다.

- 키체인을 등록할 때 로컬항목에 지정해두었다면 기본키체인 - 로그인 - 인증서 에서 인증서와 개인키를 확인 할 수 있다.
- 인증서와 개인키 두개모두 선택하고 내보내기한다.

<p align="center"><img src = "https://user-images.githubusercontent.com/69136340/122344577-e97dd200-cf81-11eb-99af-33a9113593cf.png" width ="400"></p>

- .p12 파일로 설정하고 저장해준다.

<p align="center"><img src = "https://user-images.githubusercontent.com/69136340/122344676-01edec80-cf82-11eb-8a33-4102c480a9c4.png" width ="400"></p>

- 암호를 입력해주는데 나중에도 필요하다 기억해두자.

<p align="center"><img src = "https://user-images.githubusercontent.com/69136340/122344726-0fa37200-cf82-11eb-882a-c78787b2e267.png" width = "400"></p>

<p align="center"><img src ="https://user-images.githubusercontent.com/69136340/122344789-1f22bb00-cf82-11eb-80e5-cba981ed5f99.png" width = "150"></p>

*provider server 에 인증서와 개인 키를 모두 설치해야하는데, 서버 개발환경에 따라 전달할 파일이 달라진다고 한다. php 서버에는 .pem 을 전달해야하고, 나머지는 .p12 만 전달해도 되는 경우가 있는 듯하다.

firebase 같은경우는 두가지 지원한다.

참고) pem 파일이 요구된다면 아래 주소를 참조해서 통해서 만들어주면 된다.

[https://babbab2.tistory.com/57](https://babbab2.tistory.com/57)

## B. p8 인증키 만들기

위의 App ID 등록 방법과 똑같이 해주고 다른점은 Development 와 Production certificates 만 만들지 않으면 된다. 왜냐면 우린 인증서가 아닌 인증키를 사용할거니까.

<p align="center"><img src ="https://user-images.githubusercontent.com/69136340/122344451-c7844f80-cf81-11eb-9c1a-27f894553a90.png" width = "700"></p>

<p align="center"><img width="700" alt="스크린샷 2021-06-17 오후 2 32 49" src="https://user-images.githubusercontent.com/69136340/122344414-bcc9ba80-cf81-11eb-9b0a-b2fe1a7a9a9b.png"></p>

<p align="center"><img width="700" alt="스크린샷 2021-06-17 오후 2 33 04" src="https://user-images.githubusercontent.com/69136340/122344343-a7ed2700-cf81-11eb-94b3-1280f9bc32ed.png"></p>

<p align="center"><img src="https://user-images.githubusercontent.com/69136340/122344317-a28fdc80-cf81-11eb-9aa8-e9a39c82f827.png" width ="700"></p>

노란색 경고처럼 한번만 다운로드 가능하다.

**그 다음은 파이어베이스에서 p8 파일을 인증키로 등록해주면 된다.**

**출처**

[[React Native] 🔥 Firebase 로 푸쉬 알림 구현하기 - (2) iOS 앱에서 푸시 알림 띄우기!](https://velog.io/@mayinjanuary/React-Native-Firebase-로-푸쉬-알림-구현하기-2-IOS-앱-세팅하기)

## 3. 프로젝트에서 iOS  Push Notification, Backgrounds Mode 활성화

- 자신의 iPhone 에서 돌릴 경우에 해당 개발자계정과 device 를 설정한다.
- Signing & Capabilities 탭에서 +Capability 를 선택해서 Push Notifications 를 추가한다.

<p align="center"><img width="400" src="https://user-images.githubusercontent.com/69136340/122344141-65c3e580-cf81-11eb-8f24-a94d1a176667.png"></p>

- 같은 방법으로 Background Modes 도 추가하고 Background fetch 와 Remote notifications 를 설정한다.

<p align="center"><img width="400" alt="스크린샷 2021-06-16 오후 9 33 58" src="https://user-images.githubusercontent.com/69136340/122344205-7a07e280-cf81-11eb-90c9-9505c0b31fcf.png"></p>

<p align="center"><img src ="https://user-images.githubusercontent.com/69136340/122344225-855b0e00-cf81-11eb-9359-a734978001d0.png" width = "300"></p>

## 4. firebase 에서 푸시 알림을 구현한다면?

- p8 키를 쓰는 예시. 인증키는 1년마다 갱신하지 않아도 되기 때문에 보통 p8 를 쓰는 예시가 많았다.

[https://velog.io/@mayinjanuary/React-Native-Firebase-로-푸쉬-알림-구현하기-2-IOS-앱-세팅하기](https://velog.io/@mayinjanuary/React-Native-Firebase-%EB%A1%9C-%ED%91%B8%EC%89%AC-%EC%95%8C%EB%A6%BC-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-2-IOS-%EC%95%B1-%EC%84%B8%ED%8C%85%ED%95%98%EA%B8%B0)

- p12 인증서를 쓰는 예시. 인증서는 1년마다 갱신해야 한다.

[https://faith-developer.tistory.com/156](https://faith-developer.tistory.com/156)

자 실제로 한번 푸시알림을 보내보겠습니다.
  
## 5. iOS 앱에 Firebase 추가하기

iOS 앱을 내 Firebase 프로젝트와 안전하게 연결하기 위해선, 인증서 파일을 다운받아야 하고, 그걸 내 iOS 앱 안에 세팅해야 합니다. 

- 프로젝트를 생성해주고
- ios 앱을 추가해줍니다.

<p align="center"><img width="300" alt="스크린샷 2021-06-19 오전 1 17 00" src="https://user-images.githubusercontent.com/69136340/122633638-90927300-d114-11eb-99ad-9bc116fa5c93.png"></p>

- 앱을 등록해줍니다. (iOS 번들 ID 는 프로젝트파일에서 확인가능하다.)

<p align="center"><img width="500" alt="스크린샷 2021-06-19 오전 1 18 36" src="https://user-images.githubusercontent.com/69136340/122633648-9ee08f00-d114-11eb-9a7d-5d2ff5dafd2a.png"></p>

- GoogleService-info.plist 다운로드 해서 프로젝트에 추가해준다.
<p align="center"><img width="550" alt="스크린샷 2021-06-19 오전 9 41 10" src="https://user-images.githubusercontent.com/69136340/122633702-d94a2c00-d114-11eb-9636-bc0c2bea6948.png"></p>

<p align="center"><img width="500" alt="스크린샷 2021-06-19 오전 9 41 37" src="https://user-images.githubusercontent.com/69136340/122633707-e0713a00-d114-11eb-9227-54004e313fe6.png"></p>

- Friebase SDK 를 추가한다.

<p align="center"><img width="550" alt="스크린샷 2021-06-19 오전 9 45 15" src="https://user-images.githubusercontent.com/69136340/122633736-00086280-d115-11eb-8886-fedcad3d4cf6.png"></p>

- 추가적으로 추후에 메시지 수신을 위해서 `Firebase/Messaging` 필요하니 지금 같이 추가해서 install 해주자.

<p align="center"><img width="500" alt="스크린샷 2021-06-19 오후 2 30 23" src="https://user-images.githubusercontent.com/69136340/122633766-2b8b4d00-d115-11eb-87eb-4c37ac65d34b.png"></p>

- AppDelegate.swift 파일에 `import Firebase` 와  `FirebaseApp.configure()` 를 추가해준다.

<p align="center"><img width="550" alt="스크린샷 2021-06-19 오전 9 46 57" src="https://user-images.githubusercontent.com/69136340/122633754-1f06f480-d115-11eb-85cf-f84af3ffefce.png"></p>

## 6. 알림 권한 요청

- iOS 에서는 사용자의 동의 없는 notification 을 수신하지 못하도록 했기때문에 권한을 요청해야한다.

 보통 앱이 처음 실행될 때 물어보거나 푸시알림을 설정하는 단계에서 물어보면 된다. 아래 코드는 앱을 런치할 때 권한을 요청하기 위해서 `application(_:didFinishLaunchingWithOptions:)` 메서드에 추가해 줬다.

- AppDelegate.swift

`UNAuthorizationOptions` 으로 푸시 권한을 설정해준다. (alert : 알람 권한, badge : badge 업데이트 권한, sound : 소리 권한)
    
```swift
// Tells the delegate that the launch process is almost done and the app is almost ready to run.
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.

    // 기존에 적었던 파이어베이스 권한 설정
        FirebaseApp.configure()
        
    // 푸시 권한 설정
        UNUserNotificationCenter.current().delegate = self

        let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]
        UNUserNotificationCenter
          .current()
          .requestAuthorization(
            options: authOptions,completionHandler: { (_, _) in }
          )
        application.registerForRemoteNotifications()
        
        return true
    }
```

- 파이어베이스의 노티가 수신될 수 있도록 한다.

```swift
extension AppDelegate : UNUserNotificationCenterDelegate {
    
    // 1. Asks the delegate how to handle a notification that arrived while the app was running in the foreground.
    func userNotificationCenter(_ center: UNUserNotificationCenter,willPresent notification: UNNotification,withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.alert, .sound])
    }

    // 2. Asks the delegate to process the user's response to a delivered notification.
    func userNotificationCenter(_ center: UNUserNotificationCenter,didReceive response: UNNotificationResponse,withCompletionHandler completionHandler: @escaping () -> Void) {
        completionHandler()
    }
}
```
    
**주석)**
    
1 : `userNotificationCenter(_:willPresent:withCompletionHandler:)` 메서드는 foreground 에서 러닝중에 앱에 도착하는 notification 을 다룬다.
    
`completionHandler()` 은 notification 에 대해서 presentation option 을 가지고 실행될 블록이다. `UNNotificationPresentationOptions(.sound, .badge 등의 옵션) -> Void` 을 통해 이 블록에는 리턴값이 없고 `UNNotificationPresentationOptions` 파라미터를 사용함을 알 수 있다.
    
2 : `userNotificationCenter(_:didReceive:withCompletionHandler:)` 메서드는 도착한 notification 에 대한 user 의 반응을 다룬다.

completionHandler 의 반환값이 () -> Void `completionHandler()` 이기 때문에 completionHandler 마지막에 항상 `completionHandler()` 를 호출해주어야 한다.
    
이 메서드는 아래와 같이 notification 에서 action 에 대한 설정을 할 수 있다.
    
<img width="300" alt="스크린샷 2021-06-19 오후 10 03 35" src="https://user-images.githubusercontent.com/69136340/122643334-34e2dc80-d14a-11eb-957d-3a968e1d1026.png">

더 자세한 내용은 아래 주소를 참조.
    
[Apple Developer - Declaring Your Actionable Notification Types](https://developer.apple.com/documentation/usernotifications/declaring_your_actionable_notification_types)

- provider server 가 device token(현재 기기에 있는 앱의 주소) 을 알아야 알림을 전달할 수 있다. AppDelegate.swift 파일에 다음과 같이 코드를 추가하자.

(이 글을 따라간다면 파이어베이스에서 앱전체에 푸시 알림을 보내기 때문에 서버에 device token 을 전달하는 메서드를 구현하지 않아도 작동된다.)

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.

                // ...

                // request the device token
                UIApplication.shared.registerForRemoteNotifications()
        return true
}

// Tells the delegate that the app successfully registered with APNs.
func application(_ application: UIApplication,
            didRegisterForRemoteNotificationsWithDeviceToken 
                deviceToken: Data) {

     // sendDeviceTokenToServer() 커스텀 메서드를 만들어주어서 서버로 deviceToken 을 보내면 된다.
   // self.sendDeviceTokenToServer(data: deviceToken)
}

// Sent to the delegate when APNs cannot successfully complete the registration process.
func application(_ application: UIApplication,
            didFailToRegisterForRemoteNotificationsWithError 
                error: Error) {
   // Try again later.
}
```
---
    
**warning**

iOS 14.0 에서 alert 옵션이 추천되지 않았다. 그래서 UNNotificationPresentationOptions 에 어떤 속성들이 존재하는지 살펴보았다.

<p align ="center"><img width="800" alt="_2021-06-19__2 41 56" src="https://user-images.githubusercontent.com/69136340/122633809-69887100-d115-11eb-9409-07e47354a62b.png"></p>

<p align ="center"><img width="400" alt="_2021-06-19__2 41 28" src="https://user-images.githubusercontent.com/69136340/122633808-668d8080-d115-11eb-9a13-98a558c81476.png"></p>

badge : 앱 아이콘 우측상단에 표시되는 알림 숫자가 badge 에 해당한다. foreground 일 경우는 알림을 바로 확인하니까 특별한 경욱 아니면 불필요하다.

sound : notification 과 관련되 소리를 실행한다.

alert : notification 으로부터 content 를 제공받아 alert 알람을 보여준다.
    
iOS 14 부터 추가된 옵션들이다. alert 는 더이상 추천되지 않는다.
    
banner : 배너처러 notification 을 보여준다.

list : Notification Center 에서 보여준다. (말 그대로 알림센터다. 잠금 화면에서 화면의 중간을 스와이프 업하거나 다른 화면에서 스크린 위의 중앙을 스와이프다운하면 나오는 '알림센터'다.)

- 앱을 실행하게 되면 notifications 에 대한 권한을 요청한다. 그리고 앱설저에서 확인해보면 Allow Notificiations 가 설정되어있는 것을 확인 할 수 있다.

<p align ="center"><img width="250" src="https://user-images.githubusercontent.com/69136340/122634920-edddf280-d11b-11eb-9c2e-02ebe3e020e8.png"><img width="250" src="https://user-images.githubusercontent.com/69136340/122634866-bbcc9080-d11b-11eb-9584-05e8b703e987.png">
</p>

## 7. Firebase iOS 앱 내에 인증서 등록
  
- 파이어베이스에 등록했던 프로젝트의 앱을 선택한다.
  
<p align ="center"><img width="300" alt="스크린샷 2021-06-19 오전 9 59 57" src="https://user-images.githubusercontent.com/69136340/122633883-de5bab00-d115-11eb-8a5a-552d9c6f80a4.png"></p>

- 클라우드 메시징에서 인증서를 등록한다.

<p align ="center"><img width="500" alt="스크린샷 2021-06-19 오전 10 00 53" src="https://user-images.githubusercontent.com/69136340/122633889-e582b900-d115-11eb-8b3c-b3d644ccb5c8.png"></p>

- 인증키 즉 p8 파일을 사용하면 좋다고 하지만 나는 p12 파일만 있기때문에 APN 인증서에 인증서를 업로드 할 것이다.

**A-1. Certificates 생성** 단계에서 개발 APN 과 프로덕션 APN 을 만들어주면 된다. 그리고 키체인에서 내보내기한 p12 파일을 업로드해준다.

<p align ="center"><img width="500" alt="스크린샷 2021-06-19 오전 10 06 04" src="https://user-images.githubusercontent.com/69136340/122633901-f6cbc580-d115-11eb-88e3-46837a7dcf86.png"></p>

## 8. Firebase 콘솔에서 테스트 메세지 보내기

- Cloud Messaging 들어간다.

<p align ="center"><img width="400" alt="스크린샷 2021-06-19 오전 10 11 24" src="https://user-images.githubusercontent.com/69136340/122633917-1367fd80-d116-11eb-8f8a-5f6a87890f09.png"></p>

- 메시지 보내기

<p align ="center"><img width="400" alt="스크린샷 2021-06-19 오전 10 13 27" src="https://user-images.githubusercontent.com/69136340/122633920-17941b00-d116-11eb-99be-6b9ac2ead210.png"></p>

- 알림 제목과 텍스트를 설정해준다.

<p align ="center"><img width="550" alt="스크린샷 2021-06-19 오후 3 08 54" src="https://user-images.githubusercontent.com/69136340/122633925-1c58cf00-d116-11eb-9acb-4407fa3035ec.png"></p>

- 파이어베이스에 등록한 App 을 선택한다.

<p align ="center"><img width="550" alt="스크린샷 2021-06-19 오후 3 10 50" src="https://user-images.githubusercontent.com/69136340/122633928-1f53bf80-d116-11eb-8aad-cc15041c0869.png"></p>

- 예약할 옵션을 선택하고 다음을 눌러 진행을 해도 되고 검토를 해도 된다.

<p align ="center"><img width="550" alt="스크린샷 2021-06-19 오후 3 12 51" src="https://user-images.githubusercontent.com/69136340/122633935-24187380-d116-11eb-871f-f5af5c19bacf.png">

- 검토하게되면 다음과 같은 팝업창이 뜬다. 게시해보자.

<p align ="center"><img width="450" alt="_2021-06-19__3 14 09" src="https://user-images.githubusercontent.com/69136340/122633936-27abfa80-d116-11eb-8cee-361ab5e4a074.png"></p>

아래의 세가지 경우 모두 정상적으로 도착한다면 테스트 메시지 보내기 성공이다.

- 앱이 켜져있을 때 (Foreground)

- 앱이 켜져있지만 background 에 있을 때 (Background)

- 앱이 꺼져있을 때 (Quit)

<p align ="center"><img width="250" src ="https://user-images.githubusercontent.com/69136340/122633981-6d68c300-d116-11eb-87a0-1d1202fa5b48.png"></p>
