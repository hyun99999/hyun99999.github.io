---
title:  "iOS) 앱 내에서 Firebase 의 Dynamic Link(동적 링크) 생성하기"
categories:
- iOS

date:   2023-01-06  01:48:00 +0900
author_profile: false
---
## ❓왜 앱 내에서 동적 링크를 생성할까요?

- Firebase console 에서 만드는 동적 링크는 필요한 파라미터들로 웹에서 최초에 생성하기 때문에 전체 이벤트와 같은 유동적이지 않은 용도로 사용된다.
- iOS 에서 **FirebaseDynamicLinks** 프레임워크로 링크를 만들면 특정 아이템에 대한 유동적인 링크를 생성할 수 있다.(앱 미설치 시 앱스토어로 보낼 수도 있는 등의 기능도 동일하게 추가할 수 있다.)

## ❓무엇을 구현하고 싶은가요?

- 일반 카메라로 QR Code 인식 시 앱이 있다면 앱으로 연결하고 싶어요.
- 앱이 없다면 앱스토어로 연결하고 싶어요.
- 특정 아이템에 대해 유동적으로 링크를 생성하고 싶어요. 이를 통해 앱을 실행했을 때 특정 뷰를 보여주고 싶어요.

## 1️⃣ 프로젝트 세팅

### Swift Package Manager

```swift
https://github.com/firebase/firebase-ios-sdk
```

### CocoaPod install

```swift
pod 'FirebaseDynamicLinks'
```

### Firebase 에서 Dynamic Link 생성

- 우선, Firebase console 에서 프로젝트를 생성해서 iOS 앱을 등록해주어야 합니다.

<img src="https://user-images.githubusercontent.com/69136340/210833648-aef63662-25df-4964-bcd9-4a4f4d542fe2.png" width ="400">

- 그후에, Firebase console 에서 Dynamic LInks 섹션을 선택해서 URL 프리픽스를 생성해줍니다.

<img width="500" alt="2" src="https://user-images.githubusercontent.com/69136340/210833768-7de6ac66-acc3-47d1-bba0-d9bd40b3e04b.png">

- 원하는 도메인을 작성하면 추천 도메인을 설정할 수 있습니다.

<img width="500" alt="3" src="https://user-images.githubusercontent.com/69136340/210833808-5eaac72c-29fe-42ff-8f8f-69f69cb600ca.png">

## 2️⃣ DynamicLink 생성

### AppDelegate 에서 FirebaseApp 초기화

```swift
import Firebase 
    
// ...
    
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    FirebaseApp.configure()

    // ...
    
    return true
}
```

### Dynamic Link 생성 예시 코드 및 알아보기

- 특정 파라미터를 가지는 동적 링크를 만들어보겠습니다.
- 동적 링크를 만들려면 DynamicLinkComponents 객체를 만들고, 객체의 해당 속성을 설정하여 동적 링크 파라미터를 지정할 수 있습니다. 그런 다음 url 프로퍼티에서 long link 를 가져오거나, `shorten()` 메서드를 호출해서 short link 를 가져올 수 있습니다.
- 다음은 문서에서 예시로 보여주는 코드입니다. 살펴본 후에 프로젝트에 적용해보겠습니다.

[Create Dynamic Links on iOS | Firebase Dynamic Links](https://firebase.google.com/docs/dynamic-links/ios/create#use-the-ios-builder-api)

```swift
guard let link = URL(string: "https://www.example.com/my-page") else { return }
let dynamicLinksDomainURIPrefix = "https://example.com/link"
let linkBuilder = DynamicLinkComponents(link: link, domainURIPrefix: dynamicLinksDomainURIPRefix)
linkBuilder.iOSParameters = DynamicLinkIOSParameters(bundleID: "com.example.ios")
linkBuilder.androidParameters = DynamicLinkAndroidParameters(packageName: "com.example.android")

guard let longDynamicLink = linkBuilder.url else { return }
print("The long URL is: \(longDynamicLink)")

// short link
linkBuilder.shorten() { url, warnings, error in
  guard let url = url, error != nil else { return }
  print("The short URL is: \(url)")
}
```

기본적으로 짧은 동적 링크는 유효한 동적 링크를 추측할 수 있는 가능성이 낮은 17자의 link suffixes 로 이루어집니다.

짧은 동적 링크를 추측해도 문제가 없는 경우에 고유한 suffix 가 필요하다면 생성할 수도 있습니다. 이는 `DynamicLinkComponentsOptions` 로 설정할 수 있습니다.

```swift
linkBuilder.options = DynamicLinkComponentsOptions()
linkBuilder.options.pathLength = .short
linkBuilder.shorten() { url, warnings, error in
  guard let url = url, error != nil else { return }
  print("The short URL is: \(url)")
}

// ✅ short link 만 사용하게되면 다음과 같이 실행할때마다 suffix 가 바뀝니다. 
// https://(...).page.link/xZfXTYg6WxXrzu8X8
// https://(...).page.link/Z3qzVxWzNfpwuCwq5

// ✅ DynamicLinkComponentsOptions 로 설정하면 고유한 suffix 가 생성됩니다.
// https://(...).page.link/4Yif
```

### Dynamic Link 를 프로젝트에 적용하기

아래의 문서를 통해서 원하는 속성을 추가할 수 있습니다.

[Create Dynamic Links on iOS | Firebase Dynamic Links](https://firebase.google.com/docs/dynamic-links/ios/create#use-the-ios-builder-api)

프로젝트에서 필요로 한 내용은 다음과 같습니다.

- 앱 미설치 시 앱스토어로 연결할 수 있어야 한다.
- 앱 설치 시 특정 파라미터 값을 앱에 전달할 수 있어야 한다.
- 소셜 메타데이터는 건너띄기. (아래의 속성으로 app preview page 를 건너띄고 app 또는 app store 로 리다이렉트 될 수 있습니다.)

<img width="400" alt="4" src="https://user-images.githubusercontent.com/69136340/210833958-71e36c98-1b1f-40f1-86dd-dcaf306d396a.png">

다음과 같이 속성을 추가하겠습니다.

```swift
// QR Code 이미지 생성하는 메서드.
private func setQRImage() {
    let frame = CGRect(origin: .zero, size: qrImage.frame.size)
    let qrcode = QRCodeView(frame: frame)
    generateDynamicLink(with: cardDataModel?.cardID ?? "") { dynamicLink in
    
        // 🚨
        // 짧은 동적 링크를 만들때 네트워크 요청으로 인해 completion handler 를 사용하고 있다.
        // 이 작업이 끝나기를 기다린 후에 짧은 동적 링크를 사용할 수 있기 때문에,
        // @escaping closure 를 사용하여 generateDynamicLink(with:) 메서드의 작업이 끝난 후에 QR Code 이미지 생성 작업을 하도록 하였다.

        // 👉 QRCodeView 커스텀 클래스의 QR Code 이미지를 만드는 커스텀 메서드이다.
        qrcode.generateCode(dynamicLink)
        self.qrImage.addSubview(qrcode)
    }
}

private func generateDynamicLink(with cardID: String, completion: @escaping ((String) -> Void)) {

    // ✅ 링크에 cardID 를 파라미터로 추가. -> SceneDelegate 에서 cardID 를 받아와서 사용.
    guard let link = URL(string: "https://www.example.com/my-page" + "/?id=" + cardID),
          let bundleID = Bundle.main.infoDictionary?["CFBundleIdentifier"] as? String else { return }
    let domainURLPrefix = "https://(...).page.link"
    
    // ✅ Dynamic Link parameter 설정.
    
    // link : 앱이 열게될 링크. 데스크탑 웹 브라우저 환경에서 동적링크를 열게되면 해당 링크가 열립니다.
    // domainURLPrefix : Firebase 콘솔에서 설정한 Dynamic Link URL prefix 이다.
    let linkBuilder = DynamicLinkComponents(link: link, domainURIPrefix: domainURLPrefix)
    
    linkBuilder?.iOSParameters = DynamicLinkIOSParameters(bundleID: bundleID)
    
    // ✅ Apple Connect Store 에서 확인할 수 있는 ID 이다. 앱이 설치되지 않은 경우에 사용자를 App Store 로 보내는데 사용한다.
    linkBuilder?.iOSParameters?.appStoreID = "..."
    
    // ✅ true 로 설정하여 app preview page 를 건너띄고 app 또는 app store 로 리다이렉트.
    linkBuilder?.navigationInfoParameters = DynamicLinkNavigationInfoParameters()
    linkBuilder?.navigationInfoParameters?.isForcedRedirectEnabled = true
    
    var dynamicLink: String = ""
    
    // ✅ 고유한 suffix 생성.
    linkBuilder?.options = DynamicLinkComponentsOptions()
    linkBuilder?.options?.pathLength = .short
    
    // ✅ 짧은 DynamicLink 로 변경.
    // 짧은 동적 링크를 만들기 위해서는 네트워크 요청이 필요하므로 링크를 바로 반환하는 것이 아니라 요청이 완료된 후에 호출되는 completion handler 를 사용한다.
    linkBuilder?.shorten { url, warnings, error in
        if let url {
            dynamicLink = "\(url)"
        }

        // ✅ 현재 메서드의 작업이 끝난 후 즉, 짧은 동적 링크가 만들어진 후에 completion handler 실행.
        completion(dynamicLink)
    }
    
// ✅ 긴 동적 링크를 다음과 같이 그대로 사용할 수 있다. 하지만, 프로젝트에서는 QR Code 이미지가 너무 복잡해져서 보기 좋지 않았다. 그래서 짧은 동적 코드로 생성하였다.
//        if let url = linkBuilder?.url {
//            dynamicLink = "\(url)"
//        }
}
```

## 3️⃣ Dynamic Link 수신

### Associated Domains 추가

- target 에 Associated Domains 를 추가합니다.

<img width="500" alt="5" src="https://user-images.githubusercontent.com/69136340/210833991-a48d7bfd-77c1-4109-a6ab-5f1b67dc9a14.png">

- url prefix 중 `https://` 를 제외한 도메인 값을 추가합니다.

```swift
applinks:(...).page.link
```

<img width="500" alt="6" src="https://user-images.githubusercontent.com/69136340/210834031-0979268a-d981-493b-bf3b-c050906ae7c9.png">

### Identifiers 에서 해당 App ID 에 Associated Domains 추가

- Associated Domains  을 Xcode 에서 추가하게되면 **Signing & Capabilities tab** 이 다음과 같은 상태가 됩니다.

<img src="https://user-images.githubusercontent.com/69136340/210834107-acaeb347-0814-4e20-9169-0815e07e7386.png" width ="500">

- Apple Developer 에서 App ID 를 생성하고, **Associated Domains** Capabilites 를 추가해주면 됩니다.

<img width="700" alt="8" src="https://user-images.githubusercontent.com/69136340/210834155-ddf5fb04-5e5c-40e7-9195-521a75829286.png">

## 4️⃣ 앱에서 Dynamic Link 수신 및 처리

### FirebaseDynamicLinksCustomDomains key(선택사항)

Firebase console 에서 만든 URL Prefix 가 `page.link` 하위 도메인 대신 자체 도메인을 사용한다면 `info.plist` 에서 `FirebaseDynamicLinksCustomDomains` ****라는 키를 만들고 앱의 동적 링크 URL prefix 를 설정해주어야 합니다.

**진행한 프로젝트에서는 `page.link` 를 사용했기 때문에 별도로 설정해주지 않았습니다.**

### SceneDelegate 설정

아래의 문서를 확인하면 AppDelegate 에서 설정할 수 있습니다.

[iOS에서 동적 링크 수신 | Firebase Dynamic Links](https://firebase.google.com/docs/dynamic-links/ios/receive?hl=ko)

- `scene(_:continue:)` 메서드에서 동적 링크 수신 처리합니다.

```swift
import FirebaseDynamicLinks

func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    if let url = userActivity.webpageURL {
        let handled = DynamicLinks.dynamicLinks().handleUniversalLink(url) { dynamicLink, error in
            // 👉 동적링크에서 파라미터를 다루는 함수. 아래에서 살펴보겠습니다.
            if let cardID = self.handleDynamicLink(dynamicLink) {
 
                // TODO: - user defaults 로 cardID 저장하고, 홈 뷰에서 존재 유무로 명함 조회. 조회 시 user defaults 삭제.
                    
            }
        }
    }
}
```

- 앱이 종료된 상태(not running)에서 동적 링크를 통해 앱이 켜지는 경우도 핸들링 해주기 위해 `scene(_:willConnectTo:options:)` 함수도 수정하였습니다.

```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
    guard let _ = (scene as? UIWindowScene) else { return }

    if let userActivity = connectionOptions.userActivities.first {
        self.scene(scene, continue: userActivity)
    }
}
```

### Dynamic Link 에서 데이터 받기

- 동적 링크에서 `cardID` 파라미터를 받아서 반환하는 함수입니다.

```swift
// MARK: - Extensions

extension SceneDelegate {
    private func handleDynamicLink(_ dynamicLink: DynamicLink?) -> String? {
        // 앱으로 전달되는 url 을 얻을 수 있다.(딥링크)
        guard let dynamicLink = dynamicLink,
              let link = dynamicLink.url else { return nil }
        
        // resolvingAgainstBaseURL : URL 구문을 분석하기 전에 URl 에 대해 확인하는지 여부를 제어.
        // true 이고, url parameter 에 상대적인 URl 이 포함되어 있다면 absoluteURL 메서드를 호출해서 original URl 에 대해서 확인합니다. 그렇지 않으면, 문자열 부분이 자체적으로 사용됩니다.
        let queryItems = URLComponents(url: link, resolvingAgainstBaseURL: true)?.queryItems
        
        let cardID = queryItems?.filter { $0.name == "cardID" }.first?.value
        
        return cardID
    }
}
```

## 결과

- 앱 설치 시에 앱으로 연결.

<img src="https://user-images.githubusercontent.com/69136340/210834225-c0986fae-d409-4747-9c06-2a924a4af3ba.MP4" width ="250">

- 앱 미설 치에 앱스토어로 연결.

<img src="https://user-images.githubusercontent.com/69136340/210834273-8d4d56ac-fcbf-4faf-93b8-3cb05317b2c3.MP4" width ="250">

## 🚨 트러블 슈팅 - 동적 링크를 통해 앱이 열리지 않아요!

- **문제 :** 앱이 설치되어 있지만, 앱이 열리지 않고 App Store 로 연결해주었습니다.

Firebase 에서 동적 링크를 잘못 만들어준다고 판단하였습니다. 그래서 동적 링크가 유효한지 확인해보았습니다.

- 다음의 URL 을 열어 iOS 앱에서 동적 링크를 사용하도록 Firebase 프로젝트가 올바르게 구성되었는지 확인할 수 있습니다.

```swift
https://your_dynamic_links_domain/apple-app-site-association
```

앱이 정상적으로 연결된 경우 `apple-app-site-association` 파일에는 앱의 앱 ID 접두사 및 번들 ID에 대한 참조가 포함됩니다. 예를 들어:

```swift
{"applinks":{"apps":[],"details":[{"appID":"1234567890.com.example.ios","paths":["NOT /_/*","/*"]}]}}
```

하지만, 확인해보니 다음과 같이 details 이 비어있었습니다.

```swift
{"applinks":{"apps":[],"details":[]}}
```

**출처:** [Receive Dynamic Links on iOS | Firebase Dynamic Links](https://firebase.google.com/docs/dynamic-links/ios/receive#set-up-firebase-and-the-dynamic-links-sdk)

### 💫 **해결:**

그래서 Firebase iOS 앱의 설정을 다시 확인했습니다.

- iOS 앱의 설정을 확인해보니 팀 ID 를 입력해주지 않았습니다.

<img width="500" alt="11" src="https://user-images.githubusercontent.com/69136340/210834438-ec548858-3f0f-42d1-bc57-8e1bc61df705.png">

**팀 ID 를 설정해주고 아래의 URL 로 확인해보니 details 가 구성된 것을 확인할 수 있었습니다.**(바로는 적용이 안되고 조금 지나야 적용이 되었었습니다.)

```swift
https://your_dynamic_links_domain/apple-app-site-association
```

동적 링크로 유효한 URL 임을 확인하였고, 동적 링크를 사용해보니 유효하게 작동하였습니다.

**동적 링크를 적용하기 전에 먼저, 동적 링크가 유효한지 확인하는 단계가 필요할 것 같습니다.**

**참고:** 

[[iOS - swift] 4. DeepLink (딥 링크) - Dynamic Link (다이나믹 링크) 사용 방법 (Firebase, 공유하기 기능)](https://ios-development.tistory.com/726)

[Firebase DynamicLink에서 데이터 받기 - 뀔뀔(swieeft)의 개발새발기](https://swieeft.github.io/2020/09/02/DynamicLinkGetQueryData.html)

[iOS - 앱에서 Firebase Dynamic Link 생성하고 수신하기](https://medium.com/hcleedev/ios-%EC%95%B1%EC%97%90%EC%84%9C-firebase-dynamic-link-%EC%83%9D%EC%84%B1%ED%95%98%EA%B3%A0-%EC%88%98%EC%8B%A0%ED%95%98%EA%B8%B0-e357e95343c9)

[iOS에서 동적 링크 수신 | Firebase Dynamic Links](https://firebase.google.com/docs/dynamic-links/ios/receive?hl=ko)

[Create Dynamic Links on iOS | Firebase Dynamic Links](https://firebase.google.com/docs/dynamic-links/ios/create#use-the-ios-builder-api)
