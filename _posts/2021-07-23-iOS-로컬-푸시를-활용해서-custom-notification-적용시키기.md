---
title:  "iOS) 로컬 푸시를 활용해서 custom notification 적용시키기"
categories:
- iOS

date:   2021-07-23  01:41:00 +0900
author_profile: false
---
local push 를 통해서 custom notification interface 를 적용해보자.

- 먼저, local push 를 등록해보자

```swift
import UserNotifications

class ViewController: UIViewController {

    let userNotificationCenterr = UNUserNotificationCenter.current()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        requestNotificationAuthorization()
        sendNotification(seconds: 10)
    }

    func requestNotificationAuthorization() {
        let authOptions = UNAuthorizationOptions(arrayLiteral: .alert, .badge, .sound)
 //1.
        userNotificationCenterr.requestAuthorization(options: authOptions) {
            success, error in
            print("success: \(success)")
            if let error = error {
                print("Error: \(error)")
            }
        }
    }

    func sendNotification(seconds: Double) {
        let notificationContent = UNMutableNotificationContent()
        
        notificationContent.title = "알림 테스트 타이틀"
        notificationContent.body = "알림 테스트 바디"

//2.
        notificationContent.categoryIdentifier = "TEST_NOTIFICATION"
        
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: seconds, repeats: false)
        let request = UNNotificationRequest(identifier: "testNotification", content: notificationContent, trigger: trigger)
        
        userNotificationCenterr.add(request) { error in
            if let error = error {
                print("Notification Error: \(error)")
            }
        }
    }
}
```

주석)

1: 권한요청

<img src ="https://user-images.githubusercontent.com/69136340/126676577-06dc77bc-b0ec-4c4f-9c7c-5e4236ea5a7b.png" width = "250">

2: categoryIdentifier 를 설정해서 custom notification 을 지정해줄 것이다.

---

extension 으로 만든 스토리보드와 뷰컨트롤러를 이용해서 cutom interface 를 만들 수 있다.

- info.plist

  <img src ="https://user-images.githubusercontent.com/69136340/126676700-b6fcc55e-e53a-467d-8de1-b3ee5a62eecf.png" width ="600">

UNNotificationExtensionContentHidden

- interface 에서 default content 를 숨길 수 있다. (YES: 숨기기)

UNNotificationExtensionCategory

- 키를 통해서 앱에서 수신할 수 있는 notification 을 구분한다.

UNNotificationExtensionInitialContentSizeRatio

- The initial size of the view controller's view for an app extension, expressed as a ratio of its height to its width.
- 0 과 1 사이의 숫자 값이다. cutom interface 의 aspect ratio 를 표현한다. default 는 1이다.

- MainInterface.storyboard

<img src ="https://user-images.githubusercontent.com/69136340/126676702-4c94997c-6f30-4051-9897-609fb19c9cbc.png" width ="250">

- NotificationViewController.swift

```swift
class NotificationViewController: UIViewController, UNNotificationContentExtension {

    @IBOutlet var titleLabel: UILabel?
    @IBOutlet weak var bodyLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any required interface initialization here.
    }
    
    func didReceive(_ notification: UNNotification) {
        
        self.titleLabel?.text = notification.request.content.title
        self.bodyLabel.text = notification.request.content.body
    }

}
```

<img src ="https://user-images.githubusercontent.com/69136340/126676843-10f99464-f32a-4922-8333-b3e67ff41efd.png" width ="250">
