---
title:  "iOS) UINavigationBar 의 large title 색 설정"
categories:
- iOS

date:   2021-05-07 17:51:00 +0900
author_profile: false
---
navigationBar 에서 large title 의 color 를 바꾸려 하였다.

uinavigationbar 개발문서에서 bar's appearance 가 아닌 bar object 위의 appearance 의 속성을 보고싶었다.

<img src ="https://user-images.githubusercontent.com/69136340/117424822-2c995e00-af5d-11eb-8e17-b81f4d6c1586.png" width ="500">

- lagacy customizations

<img src ="https://user-images.githubusercontent.com/69136340/117424852-34590280-af5d-11eb-94f4-d328b5e5ae53.png" width ="500">

- largeTitleTextAttributes

<img src ="https://user-images.githubusercontent.com/69136340/117424858-358a2f80-af5d-11eb-9157-38cdf93a74cc.png" width ="500">

- font, text color, text shadow color and text shadow offset 에 대한 특정 속성 설정이 가능하다.

```swift
self.navigationController?.navigationBar.largeTitleTextAttributes = [.foregroundColor: UIColor(hex: reminderColor)]
```

---

[Apple Developer Documentation - UINavigationBar](https://developer.apple.com/documentation/uikit/uinavigationbar)
