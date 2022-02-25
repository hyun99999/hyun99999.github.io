---
title:  "iOS) Generating a Remote Notification - APNs payload 알아보기"
categories:
- iOS

date:   2022-02-26  01:32:00 +0900
author_profile: false
---
### 내용

- payload 를 서버에서 작성한 뒤 이를 통해 사용자의 기기에 notifications 을 보냅니다.
- payload 에 대해서 알아보자!

# ✨Generating a Remote Notification

JSON payload 를 사용하여 사용자의 기기에 notifications 를 보냅니다.

### 💡**Overview**

remote notifications 은 JSON payload 형태로 사용자에게 중요한 정보를 전달합니다. payload 는 수행하려는 user interactions (alert, sound, or badge) 를 지정하고, 앱이 notification 에 응답하는데 필요한 custom data 를 포함합니다.

<img src="https://user-images.githubusercontent.com/69136340/155750948-608e5d2f-407e-4e6c-a958-294c211850e0.png" width ="700">

기본 remote notification payload 는 Apple-defined keys 와 custom values 가 포함됩니다. notifications 와 관련된 custom keys 와 values 를 추가할수도 있습니다. APNs 는 payload 의 총 크기가 다음 제한을 초과하는 경우 거부합니다.

- For Voice over Internet Protocol (VoIP) notifications, the maximum payload size is 5 KB (5120 bytes).
- For all other remote notifications, the maximum payload size is 4 KB (4096 bytes).

### 💡Create the JSON Payload

JSON dictionary 를 사용해서 remote notification 에 대한 `payload` 를 지정합니다. 이 dictionary 안에 Table 1 에 나열된 하나 이상의 Apple-defined keys를 포함하는 dictionary 인 `aps` 키를 포함합니다. 

시스템에 alert 를 표시하거나 sound 를 재생하거나, 앱 아이콘에 badge 를 지정하도록 지시하는 키를 포함할 수 있습니다. 또한 notification 을 백그라운드에서 silently 처리하도록 지시를 할 수도 있습니다. 자세한 정보는 [Pushing Background Updates to Your App](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/pushing_background_updates_to_your_app) 참조하세요.

Apple-defined keys 외에도 `payload` 에 custom keys 를 추가해서앱, `notification service app extension`, `notification content app extension` 에 조금의 데이터를 전달할 수 있습니다. 

### ❓notification service? notification content?

- `notification service` 는 payload 를 가로채서 notification 에 띄우고자 하는 데이터를 재구성하는 것입니다. 이미지, 비디오 등 미디어를 다운로드하여 첨부할 수 있습니다. payload 는 4KB 로 용량이 제한되기때문에 URL 로 전달해야합니다. URL 로 다운로드하는 일을 service 에서 하게 됩니다.
- `notification content` 는 long touch 를 통해서 나타나는 인터페이스를 커스텀 할 수 있습니다.
- 참고:

[[iOS-Rich Push] Notification Service Extension, Notification Content Extension을 알아보자. (푸쉬에 이미지 넣기)](https://rhammer.tistory.com/270)

---

custom keys 에는 dictionary, array, string, number, or Boolean 와 같은 기본적인 타입이 있어야 합니다. custom keys 는 앱에 전달된 `UNNotificationContent` 개체의 [userInfo](https://developer.apple.com/documentation/usernotifications/unnotificationcontent/1649869-userinfo) dictionary 에 사용할 수 있습니다.

일반적으로 코드가 notification 을 처리하는데 도움이 되도록 custom keys 를 사용합니다.

예를 들어, 코드가 앱별 데이터를 조회하는데 사용할 수 있는 식별자 문자열을 포함할 수 있습니다. 그렇다면 앱별 키를 `**aps` dictionary 의 peers 로 추가하는 식**으로 **custom keys 를 사용**하면 됩니다.

아래 Listing 1 는 사용자가 게임을 하도록 초대하는 알림 메시지를 표시하는 notification payload 를 보여줍니다. `category` 키가 이전에 등록된 `UNNotificationCategory` 개체를 식별하면 시스템은 알림에 action buttons 을 추가합니다.

예를들어, 이 `category` 에는 게임을 즉시 시작하기 위한 액션이 포함됩니다. custom key 인 `gameID` 에는 앱이 게임 초대를 검색하는데 사용할 수 있는 식별자가 들어있습니다.

- Listing 1

```swift
// 사용자가 게임을 하도록 초대하는 알림 메시지를 표시.
{
   "aps" : {
      "alert" : {
         "title" : "Game Request",
         "subtitle" : "Five Card Draw",
         "body" : "Bob wants to play poker"
      },
      "category" : "GAME_INVITATION"
   },
   "gameID" : "12345678"
}
```

Listing 2 는 앱 아이콘에 badge 를 달고 sound 를 재생하는 notification payload 를 보여줍니다. 지정된 소리 파일은 유저 기기에 이미 있어야 하고, 앱 번들 또는 앱 컨테이너의 Library/Sounds 폴더에 있어야 합니다. `messageID` 키는 notification 을 유발하는 메시지를 식별하기 위한 앱별 정보를 포함합니다. 

- Listing 2

```swift
{
   "aps" : {
      "badge" : 9,
      "sound" : "bingbong.aiff"
   },
   "messageID" : "ABCDEFGHIJ"
}
```

알림 소리 생성에 대한 자세한 내용은 [UNNotificationSound](https://developer.apple.com/documentation/usernotifications/unnotificationsound) 를 참조하세요.

⚠️ **Important**
notification 의 payload 에 고객정보나 신용카드 번호와 같은 민감한 데이터를 포함하지 마십시오. 만약 민감한 데이터를 포함해야 하는 경우, 추가하기 전에 암호화 하십시오. `notification service app extension` 을 사용해서 데이터를 해독할 수 있습니다. 더 자세한 내용은 [Modifying Content in Newly Delivered Notifications](https://developer.apple.com/documentation/usernotifications/modifying_content_in_newly_delivered_notifications) 를 참고하세요.

### 💡****Localize Your Alert Messages****

remote notifications 의 컨텐츠를 localize 하는 방법에는 두가지가 있습니다.

- Include localized strings directly in the payload.
- Add localized message strings in your app bundle, and let the system choose which strings to display.

현지화된 문자열을 payload 에 직접 배치하면 더 많은 flexibility 를 얻을 수 있지만, provider server 에서 사용자가 선호하는 언어를 추적해야합니다. 사용자의 디바이스에서 [NSLocale](https://developer.apple.com/documentation/foundation/nslocale) 의  [preferredLanguages](https://developer.apple.com/documentation/foundation/nslocale/1415614-preferredlanguages) 프로퍼티를 검사하여 사용자가 선호하는 언어를 조회할 수 있습니다.

notification 메시지가 미리 정해진 경우, 앱 번들의 `Localizable.strings` 파일에 메시지 문자열을 저장하고 `title-loc-key`, `subtitle-loc-key`, and `loc-key` payload 키를 사용하여 표시하려는 문자열을 지정할 수 있습니다. 현지화된 문자열에는 최종 문자열에 콘텐츠를 동적으로 삽입할 수 있는 placeholders 가 포함될 수 있습니다. Listing 3 은 app-provided 문자열에서 파생된 메시지가 있는 payload 의 예시를 보여줍니다. `loc-args` 키는 문자열로 대체할 변수의 배열을 포함합니다.

```swift
{
   "aps" : {
      "alert" : {
         "loc-key" : "GAME_PLAY_REQUEST_FORMAT",
         "loc-args" : [ "Shelly", "Rick"]
      }
   }
}
```

현지화된 컨텐츠를사용하는 키에 대한 자세한 내용은 개발자 문서의 [Table 2](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification#2943365) 를 참조하세요.

### 💡 ****Payload Key Reference****

[Table 1](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification#2943363) 은 `aps` dictionary 에 포함할 수 있는 키들입니다. 사용자와 상호작용하려면 alert, badge or sound keys 를 포함해야합니다.

***개발자 문서에서 [Table 1](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification#2943363) 을 확인할 수 있습니다.***

[Table 2](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification#2943365) 은 `alert` dictionary(Table 1 key 인 alert 의 key 모음) 에 포함할 수 있는 key 가 나열되어있다. 이 문자열을 사용해서 `alert` banner 에 포함할 title 과 message 를 지정합니다.

***개발자 문서에서 [Table 2](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification#2943365) 을 확인할 수 있습니다.***

- 출처
[Apple Developer Documentation - generating a remote notification](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification)
