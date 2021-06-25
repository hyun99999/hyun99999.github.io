---
title:  "iOS) AppleDeveloper - Declaring Your Actionable Notification Types"
categories:
- iOS

date:   2021-06-24 15:15:00 +0900
author_profile: false
---
Declaring Your Actionable Notification Types - notification 을 차별화하고 notification interface 에 action buttons 더하기

notification 에 대해서 공부해보다가 애플개발자문서를 정리해보았다.

## Overview
- actionable notifications 는 전달된 알림에 대해서 관련앱을 실행하지 않고도 반응할 수 있다.
- actionable notification 의 경우 알림 인터페이스 외의 아래와 같이 하나 이상의 버튼을 표시합니다. 버튼을 탭하면 정해진 action 이 앱으로 전송되고 background 에서 작업을 처리한다.

<img src = "https://user-images.githubusercontent.com/69136340/123110415-0a10c500-d477-11eb-8438-9600eb1993c1.png" width = "250">

actionable notifications 을 지원하기 위해선 :

- app 런치 시 하나 이상의 notification catergories 를 선언한다.

- notification catergories 에 적절한 actions 을 할당한다.

- 등록한 actions 를 처리한다.

- notifications 를 생성할 때 notification payloads 에 category identifiers 를 할당한다.

> 또한 이 categories 는 notification service app extension 또는 notification content app extension 을 실행할지 결정하는데도 사용된다.

> For information about how to create a notification service app extension, see [Modifying Content in Newly Delivered Notifications](https://developer.apple.com/documentation/usernotifications/modifying_content_in_newly_delivered_notifications).

> For information about how to create a notification content app extension, see [Customizing the Appearance of Notifications](https://developer.apple.com/documentation/usernotificationsui/customizing_the_appearance_of_notifications).

## Declare Your Custom Actions and Notification Types
category 와 action objects 의 조합으로 actions 를 선언합니다.

`UNNotificationCategory` 개체는 앱에서 지원하는 notification types 를 정의하고 [`UNNotificationAction`](https://developer.apple.com/documentation/usernotifications/unnotificationaction) 개체는 각  types 에 대해 표시 할 버튼을 정의합니다.

각 `UNNotificationCategory` 개체는 해당 type 의 notifications 처리할 방법에 대한 unique identifier 와 options 가 있다. [`identifier`](https://developer.apple.com/documentation/usernotifications/unnotificationcategory/1649276-identifier) 프로퍼티는 category 개체의 가장 중요한 부분이다. notifications 를 생성할 때 notification's payload 에 같은 문자열을 포함해야한다. 시스템은 category 개체와 모든 actions 를 이 문자열을 사용해서 찾는다.

notification category 와 actions 를 연결하기 위해서 하나 이상의 `UNNotificationAction` 개체를 할당한다. 각 action 개체는 유저에게 보여줄 localized string 과 해당 작업을 처리하는 방법을 나타내는 options 을 포함한다.

아래의 코드는 두가지 actions 에 대해서 custom category 를 등록한다. title 과 options 외에도 action 에 unique identifier 가 있다.
```swift
// Define the custom actions.
// 1
let acceptAction = UNNotificationAction(identifier: "ACCEPT_ACTION",
      title: "Accept", 
      options: UNNotificationActionOptions(rawValue: 0))
let declineAction = UNNotificationAction(identifier: "DECLINE_ACTION",
      title: "Decline", 
      options: UNNotificationActionOptions(rawValue: 0))
      
// Define the notification type
// 2
let meetingInviteCategory = 
      UNNotificationCategory(identifier: "MEETING_INVITATION",
      actions: [acceptAction, declineAction], 
      intentIdentifiers: [], 
      hiddenPreviewsBodyPlaceholder: "",
      options: .customDismissAction)
      
// Register the notification type.
let notificationCenter = UNUserNotificationCenter.current()
notificationCenter.setNotificationCategories([meetingInviteCategory])
```
**주석)**

1.`UNNotificationActionOptions`

`convenience init(identifier: String, title: String, options: UNNotificationActionOptions = [])`

:`options` argument 에 들어가는 `UNNotificationActionOptions` 는 작업의 작동방식을 설명한다. 

### `UNNotificationActionOptions` Initializers
- `init(rawValue: UInt)` : Initializes an action options object using the specified raw value.

### `UNNotificationActionOptions` Constants
- `static var authenticationRequired: UNNotificationActionOptions` : The action can be performed only on an unlocked device.
- `static var destructive: UNNotificationActionOptions` : The action performs a destructive task.
- `static var foreground: UNNotificationActionOptions` : The action causes the app to launch in the foreground.

---

2.`UNNotificationCategoryOptions`

`convenience init(identifier: String, actions: [UNNotificationAction], intentIdentifiers: [String], hiddenPreviewsBodyPlaceholder: String, options: UNNotificationCategoryOptions = [])` 

:`hiddenPreviewsBodyPlaceholder` argument 는 사용자가 notification previews 를 비활성화했을 때(ex)화면 잠금 시 카카오톡은 내용 대신 알림의 갯수를 표시합니다.) 표시할 placeholder 문자열 입니다. 동일한 스레드 식별자를 가진 알림 수를 나타내기 위해서 문자열에 %u 문자를 포함한다. ex) "%u Messages" -> 2 Messages

:`options` argument 에 이 type 의 notification 을 처리하기 위한 `UNNotificationCategoryOptions` 의 값을 넣어준다.

### `UNNotificationCategoryOptions`  Initializers
- `init(rawValue: UInt)` : Initializes a notification category options object using the specified raw value.

### `UNNotificationCategoryOptions`  Constants
- `static var customDismissAction: UNNotificationCategoryOptions` : Send dismiss actions to the UNUserNotificationCenter object’s delegate for handling.
- `static var allowInCarPlay: UNNotificationCategoryOptions` : Allow CarPlay to display notifications of this type.
- `static var hiddenPreviewsShowTitle: UNNotificationCategoryOptions` : Show the notification’s title, even if the user has disabled notification previews for the app.
- `static var hiddenPreviewsShowSubtitle: UNNotificationCategoryOptions` : Show the notification’s subtitle, even if the user has disabled notification previews for the app.

---

대부분의 actions 의 결과는 유저 선택으로 이어지지만 텍스트 입력을 통해 반응이 가능하기도 하다.(예를 들어 카카오톡 메시지에 답하는 것.) 이처럼 text input action 을 만들기 위해서 `UNTextInputNotificationAction` 개체를 `UNNotificationAction` 개체 대신 만들어내기도 한다. 사용자가 입력을 위해서 버튼을 텝하면 시스템은 편집 가능한 text field 를 표시한다. 그리고 입력한 text 가 앱에 action 을 report 할 때 포함된다.

### `UNTextInputNotificationAction` 
사용자 입력 텍스트를 허용하는 action.

essentials)

- `init(identifier: String, title: String, options: UNNotificationActionOptions, textInputButtonTitle: String, textInputPlaceholder: String)` : Creates an action object that accepts text input from the user.

## Include a Notification Category in the Payload
시스템은  payload 에 유효한 category identifier 가 포함된 notification 만 표시한다. category identifier 를 사용하여 notification interface 에 action buttons 를 추가한다. 

- local notification

local notification 에 category 를 할당하기 위해서 `UNMutableNotificationContent` 개체의 `categoryIdentifier` 프로퍼티에 적절한 문자열을 할당해야한다. 아래의 코드는 local notification 을 위한 content 생성이다. 기본 정보 외에도 아래의 코드는 [`userInfo`](https://developer.apple.com/documentation/usernotifications/unnotificationcontent/1649869-userinfo) dictionary 에 커스텀 데이터를 추가하여 나중에 사용할 수 있도록 한다.

```swift
let content = UNMutableNotificationContent()
content.title = "Weekly Staff Meeting"
content.body = "Every Tuesday at 2pm"
content.userInfo = ["MEETING_ID" : meetingID, 
                    "USER_ID" : userID ]
content.categoryIdentifier = "MEETING_INVITATION"
```
- remote notification

remote notification 에 category identifier 를 추가하기 위해서 아래의 코드와 같이 JSON payload 의 aps dictionary 에 category key 를 포함 시킨다.

```swift
{
   "aps" : {
      "category" : "MEETING_INVITATION"
      "alert" : {
         "title" : "Weekly Staff Meeting"
         "body" : "Every Tuesday at 2pm"
      },
   },
   "MEETING_ID" : "123456789",
   "USER_ID" : "ABCD1234"

} 
```

## Handle the Selected Action
유저가 action 을 선택하면 시스템은 app 을 background 에서 시작하고 `UNUserNotificationCenter` 개체에 알린다. `userNotificationCenter(_:didReceive:withCompletionHandler:)` 메서드를 통해서 선택된 action 을 식별하고 적절한 반응을 할 수 있다.

아래의 코드는 delegate method 의 구현이다. 이 메서드는 [actionIdentifier](https://developer.apple.com/documentation/usernotifications/unnotificationresponse/1649548-actionidentifier) 프로퍼티를 사용해서 초대에 대한 수락과 거절할지 여부를 결정하는 내용이다. action handling 이 끝난 후 항상 completion handler 를 호출해라.

```swift
func userNotificationCenter(_ center: UNUserNotificationCenter,
       didReceive response: UNNotificationResponse,
       withCompletionHandler completionHandler: 
         @escaping () -> Void) {
       
   // Get the meeting ID from the original notification.
   let userInfo = response.notification.request.content.userInfo
   let meetingID = userInfo["MEETING_ID"] as! String
   let userID = userInfo["USER_ID"] as! String
        
   // Perform the task associated with the action.
   switch response.actionIdentifier {
   case "ACCEPT_ACTION":
      sharedMeetingManager.acceptMeeting(user: userID, 
                                    meetingID: meetingID)
      break
        
   case "DECLINE_ACTION":
      sharedMeetingManager.declineMeeting(user: userID, 
                                     meetingID: meetingID)
      break
        
   // Handle other actions…
 
   default:
      break
   }
    
   // Always call the completion handler when done.    
   completionHandler()
}
```

**important)**

action 에 대한 응답이 disk 에 있는 파일 액세스와 관련이 있는 경우 다른 접근 방식으로 고려해라. 잠겨있는 기기에 대한 사용자의 응답은 앱에서 사용할 수 없는 [complete](https://developer.apple.com/documentation/foundation/fileprotectiontype/1616200-complete) option 으로 파일이 암호화 된다. 이 경우 변경 사항을 임시 저장하고 나중에 앱 데이터 구조에 통합해라.

### 출처
출처ㅣ[Declaring Your Actionable Notification Types](https://developer.apple.com/documentation/usernotifications/declaring_your_actionable_notification_types)
