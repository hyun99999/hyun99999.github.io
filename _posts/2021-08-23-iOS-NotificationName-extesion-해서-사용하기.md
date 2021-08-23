---
title:  "iOS) Notification.Name extesion 해서 사용하기"
categories:
- iOS

date:   2021-08-23  11:06:00 +0900
author_profile: false
---
notification 이 프로젝트 내에서  퍼져있기 때문에 이름이 가끔 겹치기도 하고 내가 무슨 이름을 사용했는지 기억이 안나기도 했다.

그래서 extension 으로  Notification.Name 을 확장시켜서 사용해보기로 했다.

- NotificaationName+Extention

```swift
extension Notification.Name {
    
    static let testNotification = Notification.Name(rawValue: "test")
}
```

- 사용할 때

```swift
// post
NotificationCenter.default.post(name: .testNotification, object: nil)

// observer
NotificationCenter.default.addObserver(self, selector: #selector(testMethod), name: .testNotification, object: nil)
```

출처 :

[[iOS] NotificationCenter 깔끔하게 쓰기](https://velog.io/@sso0022/iOS-NotificationCenter-깔끔하게-쓰기)
