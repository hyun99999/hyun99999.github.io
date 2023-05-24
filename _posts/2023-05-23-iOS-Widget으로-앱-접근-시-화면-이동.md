 ---
title:  "iOS) Widget 으로 앱 접근 시 화면 이동"
categories:
- iOS

date:   2023-05-23  19:02:00 +0900
author_profile: false
---
### 내용

- 특정 widget 에서 앱으로 접근 시를 구분하여 분기 처리를 대응해보자.
- 위젯의 `widgetURL()` 로부터 넘겨준 URL 의 쿼리를 매개변수로 사용해보자.

우선, widget 으로 앱에 접근할 때의 상황은 크게 두 가지로 나뉩니다.

- 앱을 실행시켜서 foreground 로 진입
- background 에서 foreground 로 진입

이 상황들에 대응하기 위해서 다음의 메서드를 SceneDelegate 에서 사용할 수 있습니다.

### [scene(_:willConnectTo:options:)](https://developer.apple.com/documentation/uikit/uiscenedelegate/3197914-scene)

- delegate(UISceneDelegate) 에게 앱에 scene 추가에 대해서 알립니다.

```swift
// scene: app 에 연결되는 scene 객체.
// session: scene configuration 에 대한 세부 정보가 포함된 session 객체.
// connectionOptions: scene 을 구성하기 위한 추가 옵션. 이 객체의 정보를 사용하여 scene 생성을 일으킨 작업을 처리. 예를 들어, 사용자가 선택한 quick action 에 응답.
optional func scene(
    _ scene: UIScene,
    willConnectTo session: UISceneSession,
    options connectionOptions: UIScene.ConnectionOptions
)
```

### [scene(_:openURLContexts:)](https://developer.apple.com/documentation/uikit/uiscenedelegate/3238059-scene)

- delegate(UISceneDelegate) 에게 하나 이상의 URL을 열도록 요청합니다.

```swift
// scene: UIKit 이 URL 을 열라고 요청하는 scene
// URLContexts: 하나 이상의 UIOpenURLContext 객체. 각 객체는 하나의 URL 과 URL 을 열기 위한 추가적인 정보를 포함.
optional func scene(
    _ scene: UIScene,
    openURLContexts URLContexts: Set<UIOpenURLContext>
)
```

위젯을 열때 다음과 같이 URL 이 전달됩니다.

```swift
struct MyCardEntryView: View {
    Image("widget")
        // ✅ cardUUID 값을 쿼리형태로 URL 에 추가.
        .widgetURL(URL(string: "openMyCardWidget://?cardUUID=\(cardUUID)"))
}
```

## 1️⃣ 앱을 실행시켜서 foreground 로 진입

- `scene(_:willConnectTo:options:)` 메서드를 사용
- 앱에 scene 을 추가할 때 즉, 앱을 런치할 때에 해당 메서드로 delegate 에게 알림니다.

```swift
// SceneDelegate.swift

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    private let myCardURL = "openMyCardWidget"
    private let qrCodeURL = "openQRCodeWidget"
    
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        // urlContexts : 열기 위한 meatadata 및 URL. 
        if let url = connectionOptions.urlContexts.first?.url {
            // ✅ 전달된 url 을 통해서 분기처리.
            if url.absoluteString == qrCURL {

                // ✅ 화면 이동에 필요한 UserDefaults 추가.
                UserDefaults.standard.setValue(true, forKey: Const.UserDefaultsKey.openQRCodeWidget)
            }

            // ✅ "openMyCardWidget://?cardUUID=123456" 과 같이 전달.
            else if url.absoluteString.starts(with: myCardURL) {
                
                // ✅ queryItems 에서 "cardUUID" 값 사용.
                guard let queryItems = URLComponents(string: url.absoluteString)?.queryItems,
                      let cardUUID = queryItems.filter({ $0.name == "cardUUID" }).first?.value else { return }
              
                UserDefaults.standard.setValue(cardUUID, forKey: Const.UserDefaultsKey.widgetCardUUID)
            }
        }
    }
}
```

## 2️⃣ background 에서 foreground 로 진입

- `scene(_:openURLContexts:)` 메서드를 사용
- URL 을 여는 요청에서 URL 을 확인 후에 특정 화면으로 이동.
- 즉, 앱을 런치하지 않을 때 URL 을 다루기 위해 사용할 수 있습니다.

```swift
// SceneDelegate.swift

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        guard let url = URLContexts.first?.url,
              let urlComponents = URLComponents(string: url.absoluteString) else { return }
        
        if qrCodeURL == url.absoluteString {
            // qr code 위젯.
            }
        } else if url.absoluteString.starts(with: myCardURL) {
            // 내 명함 위젯.
        }
    }
}
```

## 👉 참고

카메라로 QR 코드를 찍어서 동적링크로 앱을 실행하는 경우에는 다른 delegate 메서드를 사용하기 때문에 알아보겠습니다.

### [scene(_:continue:)](https://developer.apple.com/documentation/uikit/uiscenedelegate/3238056-scene)

- delegate 에게 지정된 Handoff-related activity 를 처리하도록 알립니다.

```swift
// ✅ user acticty 와 관련된 데이터를 scene 에서 사용하여 작업을 이어갈 수 있다.
func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    if let url = userActivity.webpageURL {
        // ...
    }
}
```

---

**출처:** 

[Detect app launch from WidgetKit widget extension](https://stackoverflow.com/questions/63697132/detect-app-launch-from-widgetkit-widget-extension)

[WidgetKit: Advanced development - Part 2](https://medium.com/kinandcartacreated/widgetkit-advanced-development-part-2-a675a617fdc9)
