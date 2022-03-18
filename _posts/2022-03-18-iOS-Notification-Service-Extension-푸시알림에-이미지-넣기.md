---
title:  "iOS) Notification Service Extension - 푸시알림에 이미지 넣기"
categories:
- iOS

date:   2022-03-18  18:38:00 +0900
author_profile: false
---
`Notification Service Extension` 을 활용해서 `payload` 의 값을 가공하는 과정이고 개발자문서는 아래를 참고 하면 됩니다.

[Apple Developer Documentation](https://developer.apple.com/documentation/usernotifications/modifying_content_in_newly_delivered_notifications)

### 내용

- `Notification Service Extension` 으로 전달된 payload 의 정보를 가공해보자!
- 궁극적으로, payload 의 body 값인 URL 을 통해서 사진을 다운로드 받아 notification 에 보여주자!

### 순서

- 1️⃣ `Notification Service Extension` 추가
- 2️⃣ `Notification Service Extension` 에서 페이로드 컨텐츠 수정
- 📬 번외)  `FirebaseMessaging` 사용해서 이미지를 자동으로 채워보자

## 📬 Notification Service Extension 추가

- [File] → [New] → [Target...] 에서 `Notification Service Extension` 을 추가

<img src="https://user-images.githubusercontent.com/69136340/158991342-da5cebf0-c0dc-47ba-b673-810c26b38a16.png" width ="600">
<img src="https://user-images.githubusercontent.com/69136340/158991398-9f1946dc-ee4b-4a54-b4ab-94202195e45c.png" width ="600">

- Activate 눌러 스키마를 사용하도록 합니다.

<img src="https://user-images.githubusercontent.com/69136340/158991367-16e06578-e2a2-442c-badc-beac541aa5d4.png" width ="300">

처음에 다음과 같이 `NotificationService` 클래스가 만들어집니다.

```swift
import UserNotifications

class NotificationService: UNNotificationServiceExtension {

    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?

    override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
        self.contentHandler = contentHandler
        bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)
        
        if let bestAttemptContent = bestAttemptContent {
            // Modify the notification content here...
            bestAttemptContent.title = "\(bestAttemptContent.title) [modified]"
            
            contentHandler(bestAttemptContent)
        }
    }
    
    override func serviceExtensionTimeWillExpire() {
        // Called just before the extension will be terminated by the system.
        // Use this as an opportunity to deliver your "best attempt" at modified content, otherwise the original push payload will be used.
        if let contentHandler = contentHandler, let bestAttemptContent =  bestAttemptContent {
            contentHandler(bestAttemptContent)
        }
    }

}
```

## 📬 Notification Service Extension 에서 페이로드 컨텐츠 수정

여기서 잠깐! 제가 서버로부터 받기로한 페이로드는 다음과 같습니다.

```
apns: {
  payload: {
    aps: {
      alert: {
        'title': 'certification title text',
        'body': 'certification body text',
      },
      'category': 'certification',  // notification identifier.
      'thread-id': 'certification', // notification grouping 을 위해 필요한 key.
      'mutable-content': 1,         // notification service extension 사용하려면 1.
      },
  },

  fcm_options: {
    image: 'https://www.imageUrl',
  },
}
```

- `mutable-content` : notification service app extension flag 이다. 만약 `1` 이라면, 시스템이 notification 이 전달되기 전에 notification service app extension 으로 보낸다. (`1` 로 설정해야 payload 데이터를 notification service extension 에서 다룰 수 있다!)

출처 : 

[Apple Developer Documentation](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification#2943363)

- `fcm_options` : 파이어베이스의 예시를 따라서 네이밍했고, notification service extension 에서 가공할 데이터들을 받기위해서 따로 묶었습니다.

---

우리는 페이로드의 `image` 의 주소값을 가지고 이미지를 다운로드 한 후 푸시알림에서 보여줄 것입니다.

- 다음과 같이 `UNNotificationAttachment` 를 만들어서 `UNMutableNotificationContent` 인 `bestAttemptContent` 를 설정해주면됩니다.

```swift
let attachment = try UNNotificationAttachment(identifier: "image", url: imageFileURL, options: nil)
bestAttemptContent.attachments = [attachment]
```

UNNotificationAttachment 의 이니셜라이저를 개발자 문서를 통해 알아봅시다.

### UNNotificationAttachment

- [init(identifier:url:options:)](https://developer.apple.com/documentation/usernotifications/unnotificationattachment/1649987-init)

```swift
convenience init(identifier: String, 
             url URL: URL, 
             options: [AnyHashable : Any]? = nil) throws
```

**Parameters**

- `identifier` : attachment 의 고유 식별자. 후에 attachment 를 식별하는데 사용된다. 빈 문자열을 지정하면 이 메서드는 고유한 식별자 문자열을 생성합니다.
- `URL` : notification 에 첨부한 file 의 URL 입니다. 반드시 file URL 이어야 하고 파일은 현재 프로세스에서 읽을 수 있어야 합니다. 이 파라미터는 nil 이 아녀야 합니다. 지원되는 파일 형식 목록은 [Supported File Types](https://developer.apple.com/documentation/usernotifications/unnotificationattachment#1682051) 를 참고하세요.(이미지의 경우 kUTTypeJPEG, kUTTypeGIF, kUTTypePNG 를 지원하고 10MB 가 최대 사이즈입니다.)
- `options` : 첨부 파일과 관련된 옵션 사전입니다. 결과 썸네일에 사용할 clipping rectangle 과 같은 첨부 파일에 대한 meta information 을 지정합니다.
- `error` : 문제가 발생했는지 여부를 나타내는 error 객체. 시스템이 성공적으로 attachment 를 생성하면 이 매개변수는 nil 입니다. 에러가 발생하면 attachment 가 생성되지 않은 이유에 대한 정보가 포함된 error 객체로 설정됩니다. 해당 매개변수에 nil 을 지정할 수 있습니다.

**Return Value**

지정된 file 또는 nil 에 대한 정보(attachment 가 만들어지지 않은 경우 nil)를 포함하는 attachment 객체를 리턴합니다.

**Discussion**

이 메서드는 지정된 파일을 읽을 수 있고 파일 형식이 지원되는지 확인합니다. 오류가 발생하면 적절한 error 객체를 제공합니다.

attachment 가 포함된 notifiation request 을 예약하면 프로세스에서 쉽게 액세스 할 수 있도록 시스템에서 attachment 의 파일을 새 위치로 옮깁니다. 이동 후 파일에 액세스하는 방법은 [UNUserNotificationCenter](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter) 객체를 사용하는 것입니다.

---

이때 url 에는 페이로드 안의 url 이 아닌 실제 임시로 저장된 파일의 경로를 가져와야합니다.

- 먼저, URL 을 가지고 파일을 저장한 뒤 경로로 `NUNotificationAttachment` 를 리턴하는 메서드를 만들어 봅시다.

```swift
// MARK: - UNNotificationAttachment

extension UNNotificationAttachment {
    static func saveImageToDisk(identifier: String, data: Data, options: [AnyHashable : Any]? = nil) -> UNNotificationAttachment? {
        let fileManager = FileManager.default
        let folderName = ProcessInfo.processInfo.globallyUniqueString
        let folderURL = NSURL(fileURLWithPath: NSTemporaryDirectory()).appendingPathComponent(folderName, isDirectory: true)!
        
        do {
            try fileManager.createDirectory(at: folderURL, withIntermediateDirectories: true, attributes: nil)
            let fileURL = folderURL.appendingPathExtension(identifier)
            try data.write(to: fileURL)
            let attachment = try UNNotificationAttachment(identifier: identifier, url: fileURL, options: options)
            return attachment
        } catch {
            print("saveImageToDisk error - \(error)")
        }
        return nil
    }
}
```

### didReceive 메서드에 적용해보자

```swift
override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
    self.contentHandler = contentHandler
    bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)
        
    if let bestAttemptContent = bestAttemptContent,
        // ✅payload 에 따라서 키 값은 달라진다.
        let fcmOptionsUserInfo = bestAttemptContent.userInfo["fcm_options"] as? [String: Any] {
        guard let imageURLString = fcmOptionsUserInfo["image"] as? String else {
                contentHandler(bestAttemptContent)
                return
        }
        let imageURL = URL(string: imageURLString)!

        // 🔥 download image.
        guard let imageData = try? Data(contentsOf: imageURL) else {
            contentHandler(bestAttemptContent)
            return
        }

        // 🔥 set UNNotificationAttachment.
        guard let attachment = UNNotificationAttachment.saveImageToDisk(identifier: "certificationImage.jpg", data: imageData, options: nil) else {
            contentHandler(bestAttemptContent)
            return
        }
        bestAttemptContent.attachments = [attachment]
        contentHandler(bestAttemptContent)
    }
}
```

- attachment 를 만들었으니 확인해봅시다. 잊지않고 다음과 같이 빌드해야합니다!

<img src="https://user-images.githubusercontent.com/69136340/158991654-5ac65555-ed14-4d13-9ae0-3466806ecb6a.png" width ="500">

## ⚠️ 왜... 안되지..?

```swift
// 1. identifier 를 certificationImage 위와 같이 설정.
// ❗️ Error Domain=UNErrorDomain Code=101 "Unrecognized attachment file type" UserInfo={NSLocalizedDescription=Unrecognized attachment file type}
// 에러 발생해서 확장자를 명시해주어서 해결했다!
guard let attachment = UNNotificationAttachment.saveImageToDisk(identifier: "certificationImage", data: imageData, options: nil) else {
     contentHandler(bestAttemptContent)
     return
}

// 2. identifier 를 certificationImage.jpg 설정.
guard let attachment = UNNotificationAttachment.saveImageToDisk(identifier: "certificationImage.jpg", data: imageData, options: nil) else {
     contentHandler(bestAttemptContent)
     return
}
```

## 📬 FirebaseMessaging 사용해서 이미지를 자동으로 채워보자

FirebaseMessaging 을 사용해서 작업도 해보겠습니다. 더 간편합니다.

본 프로젝트에서는 FirebaseMessaing 을 사용해서 Notification Servcie Extension 에서도 적용이 될 줄 알았는데 아니었습니다.

<img src="https://user-images.githubusercontent.com/69136340/158991737-3019da32-704c-4236-97b2-2ecae57ab8b9.png" width = "500">

- 기존의 pod 파일을 다음을 추가해서 수정해주었습니다.

```swift
target 'SparkNotificationService' do
  use_frameworks!
  
  pod 'Firebase/Messaging'
end
```

<img src="https://user-images.githubusercontent.com/69136340/158992022-856bb842-bf94-493a-8a0d-af10ee4f66a9.png" width ="600">

### [FIREMessagingExtensionHelper](https://firebase.google.com/docs/reference/swift/firebasemessaging/api/reference/Classes/FIRMessagingExtensionHelper)

`FIREMessagingExtensionHelper` 를 사용하여 간단하게 위의 작업을 대체할 수 있습니다.

```swift
override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
    self.contentHandler = contentHandler
    bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)

    // 🔥 FirebaseMessaging
    guard let bestAttemptContent = bestAttemptContent else { return }
    FIRMessagingExtensionHelper().populateNotificationContent(bestAttemptContent, withContentHandler: contentHandler)
}
```

위의 문서를 확인하면 payload 의 body 에 파라미터를 image 로 설정하라고 명시되어 있다. 그래서 수정없이 사용할 수 있었다.

### 결과

<img src="https://user-images.githubusercontent.com/69136340/158991802-b082e19e-6d04-4a8c-b949-39e827896dfc.jpeg" width ="300">

**참고:** 

[Apple Developer Documentation](https://developer.apple.com/documentation/usernotifications/modifying_content_in_newly_delivered_notifications)

[Firebase Cloud Messaging for iOS: Push Notifications](https://www.raywenderlich.com/20201639-firebase-cloud-messaging-for-ios-push-notifications)

[[Swift] iOS Push Image도 받아보기(Feat.FCM) 2/2](https://yoonandro.tistory.com/98)

[Custom Push Notification with image and interactions on iOS - Swift 4](https://medium.com/@lucasgoesvalle/custom-push-notification-with-image-and-interactions-on-ios-swift-4-ffdbde1f457)
