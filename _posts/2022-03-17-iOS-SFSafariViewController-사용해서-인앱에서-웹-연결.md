---
title:  "iOS) SFSafariViewController 사용해서 인앱에서 웹 연결"
categories:
- iOS

date:   2022-03-17  16:38:00 +0900
author_profile: false
---
### 내용

- 인앱에서 웹 연결
- 화면전환 다루기

## 🔌 [SFSafariViewController](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller)

웹 브라우징을 위한 visible standard interface 를 제공하는 객체

**Overview**

Reader, AutoFill, Fraudulent Website Detection 및 컨텐츠 차단과 같은 Safari 기능이 포함되어있습니다. iOS 9 부터는 쿠키 및 기타 웹사이트 데이터를 사파리와 공유합니다. SFSafariViewController 와의 활동과 상호작용은 앱에 보여지지 않습니다. (앱에서 AutoFill data, browsing history, or website data 는 접근할 수 없습니다.) 그래서 앱과 Safari 간에 데이터를 보호할 필요가 없습니다. 원한다면 iOS 11 이상에서 [ASWebAuthenticationSession](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession) 를 사용하세요.

⚠️ **Important**

App Store Review  Guidelines 에 따르면 이 view controller 는 정보를 유저에게 시각적으로 표시하는데 사용해야합니다. 다른 뷰 혹은 레이어에 의해서 숨겨지거나 가려져서는 안됩니다. 또한 앱은 사용자 동의 없이 SFSafariViewController 를 사용하여 사용자를 추적할 수  없습니다.

UI features 는 다음이 포함됩니다.

- A read-only address field with a security indicator and a Reader button
- An Action button that invokes an activity view controller offering custom services from your app, and activities, such as messaging, from the system and other extensions
- A Done button, back and forward navigation buttons, and a button to open the page directly in Safari
- On devices that support 3D Touch, automatic Peek and Pop for links and detected data

### Choosing the Best Web Viewing Class

- 앱을 통해서 웹사이트를 볼 수 있다면 SFSafariViewController 클래스를 사용하세요.
- 앱이 웹 컨텐츠 표시를 커스텀하거나 상호작용 또는 제어하는 경우는 [WKWebView](https://developer.apple.com/documentation/webkit/wkwebview) 를 사용하세요.

SFSafariViewController 를 적용하고 사용자가 링크를 눌러 이동하면 앱 내에서 웹 컨텐츠를 볼 수 있습니다. `Done` 을 탭하면 사용자는 웹 컨텐츠가 로드되기 전 표시되었던 뷰 컨트롤러로 돌아갑니다.

---

기본적으로 다음과 같이 push 로 화면전환이 이루어집니다.

그런데 해당 화면전환을 변경하고싶다면 `UIViewControllerTransitioningDelegate` 프로토콜을 채택해서 화면전환을 해당 뷰컨트롤러에서 담당한다고 명시하면됩니다!

<img src="https://user-images.githubusercontent.com/69136340/158759397-ae729ade-3ef3-4abf-be77-1d1c2a268fd3.MP4" width = "250">

```swift
guard let url = URL(string: "https://www.naver.com") else { return }
let safariVC = SFSafariViewController(url: url)
// 🔥 delegate 지정 및 presentation style 설정.
safariVC.transitioningDelegate = self
safariVC.modalPresentationStyle = .pageSheet
                
present(safariVC, animated: true, completion: nil)

// 🔥 UIViewControllerTransitioningDelegate 채택.
extension MypageVC: UIViewControllerTransitioningDelegate { }
```

### 결과

<img src="https://user-images.githubusercontent.com/69136340/158759321-9f98859f-42b2-41fd-ac66-afcc7e9ccef9.MP4" width ="250">

**출처:**

[Apple Developer Documentation](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller)

[iOS ) App에서 Web페이지를 여는 방법 정리](https://zeddios.tistory.com/375)
