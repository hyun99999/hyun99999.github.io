---
title:  "iOS) 위젯으로 앱의 특정 뷰로 이동(widgetURL)"
categories:
- iOS

date:   2022-12-05  14:50:00 +0900
author_profile: false
---
### 👉 위젯을 통해 앱의 특정 뷰로 이동

- QR Code 위젯을 선택하면 아래와 같이 화면전환이 적용된 QR Code 인식 뷰로 이동하도록 하겠습니다.

<img width="600" alt="스크린샷 2022-12-02 오전 11 30 28" src="https://user-images.githubusercontent.com/69136340/205557540-4bdeeb28-c6dc-43de-8f6e-24d92437b262.png">

## 👉 [widgetURL(_:)](https://developer.apple.com/documentation/swiftui/view/widgeturl(_:))

위젯을 클릭했을 때 containing app 에서 열릴 URL 을 설정합니다.

**Overview**

widgetURL modifier 는 view hierarchy 에서 하나만 지원합니다. 여러 뷰에 widgetURL 이 있는 경우 동작이 정의되지 않습니다. 

```swift
struct QRCodeEnytryView : View {
    var entry: QRCodeProvider.Entry
    
    var body: some View {
        Image("widgetQr")
            .resizable()
            .scaledToFill()
            // ✅
            .widgetURL(URL(string: "openQRCode"))
    }
}
```

## 👉 [scene(_:openURLContexts:)](https://developer.apple.com/documentation/uikit/uiscenedelegate/3238059-scene)

Asks the delegate to open one or more URLs.

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        let qrcodeURL: String = "openQRCode"
        
        guard let url = URLContexts.first?.url,
              let urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: true) else { return }
        
        if qrcodeURL == urlComponents.path {
            guard let nextViewController = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "NextViewController") as? NextViewController else { return }
            window?.rootViewController?.present(nextViewController, animated: true, completion: nil)
        }
    }
```

widgetURL 로 설정한 URL 이 들어오게 되면 containing app 의 특정 뷰로 화면전환하도록 하였습니다.

<img src="https://user-images.githubusercontent.com/69136340/205557627-463bcaa2-3bea-4553-a6d3-38b685018485.mp4" width ="250">


