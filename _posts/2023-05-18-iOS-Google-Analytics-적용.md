---
title:  "iOS) Google Analytics 적용"
categories:
- iOS

date:   2023-05-18  12:32:00 +0900
author_profile: false
---
### 내용

- iOS 앱에 화면 추적, 이벤트 추적을 위한 GA(Google Analytics) 를 적용해 보겠습니다.
- 잘 로깅되고 있는지 Xcode 에서 확인해 보겠습니다.

다음을 참고해서 정리하였습니다.

[Google 애널리틱스 시작하기  |  Firebase용 Google 애널리틱스](https://firebase.google.com/docs/analytics/get-started?hl=ko&platform=ios)

### 시작하기

- CocoaPods 을 활용한 SDK 추가

```swift
pod 'Firebase/Analytics'
```

- 앱 대리자의 `application(_:didFinishLaunchingWithOptions:)` 메서드에서 `FirebaseApp` 공유 인스턴스를 구성합니다.

```swift
// AppDelegate.swift

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Use Firebase library to configure APIs
    FirebaseApp.configure()
}
```

**우선, 간단하게 이벤트 로깅하는 방법에 대해서 알아보겠습니다.**

> [모든 앱에 권장](https://support.google.com/firebase/answer/6317498?hl=ko)되는 특정 이벤트가 있으며 특정 비즈니스 유형이나 카테고리에 권장되는 이벤트가 있습니다. 추천 이벤트를 사전 정의된 매개변수와 함께 전송해야 합니다. 이렇게 하면 보고서에 세부정보가 최대한 포함되고 향후 기능 및 통합을 즉시 사용할 수 있습니다.

출처: [https://firebase.google.com/docs/analytics/get-started?hl=ko&platform=ios](https://firebase.google.com/docs/analytics/get-started?hl=ko&platform=ios)
> 

```swift
Analytics.logEvent(AnalyticsEventSelectContent, parameters: [
  AnalyticsParameterItemID: "id-\(title!)",
  AnalyticsParameterItemName: title!,
  AnalyticsParameterContentType: "cont",
])
```

예를 들어, 대표적으로 화면을 보여주는 특정 이벤트는 `AnalyticsEventScreenView` 라는 문자열 값을 가진 static 변수가 있고 이를 사용해서 화면 추적을 하나의 이벤트에 모아서 보겠습니다.

## 1️⃣ 화면 추적을 위한 GA

화면 전환이 발생하면 애널리틱스는 새 화면을 식별하는 **screen_view** 이벤트를 로깅합니다.

이러한 화면에서 발생하는 이벤트에는 `firebase_screen_class` 매개변수(예: `menuViewController` 또는 `MenuActivity`(안드로이드)) 및 생성된 `firebase_screen_id`가 자동으로 태그됩니다.

그렇지 않은 경우에도 **screen_view** 이벤트를 수동으로 로깅하여 보고서를 생성할 수 있습니다.

### 자동 화면 트래킹

Info.plist에서 `FirebaseAutomaticScreenReportingEnabled`를 `NO`로 설정하여 iOS에서 자동 화면 조회수 보고를 중지할 수 있습니다.

### 수동 화면 트래킹

이러한 이벤트는 Apple 플랫폼의 경우 `onAppear` 또는 `viewDidAppear` 메서드에서 로깅할 수 있습니다.

파라미터로 `screen_class`를 설정하지 않으면 애널리틱스는 호출할 때 포커스에 있는 UIViewController 또는 Activity를 기반으로 기본값을 설정합니다.

```swift
Analytics.logEvent(AnalyticsEventScreenView,
                       parameters: [AnalyticsParameterScreenName: screenName,
                                    AnalyticsParameterScreenClass: screenClass])
```

### 수동 화면 트래킹 적용해보기

우선, 우리가 필요한 **screen_view** 정보를 얻기 위해서 자동으로 화면을 추적하고 조회수를 보고하는 것을 멈춰보겠습니다.

Info.plist에서 `FirebaseAutomaticScreenReportingEnabled`를 `NO` 설정하였습니다.

휴먼에러 방지 및 유지보수를 위해서 다음과 같이 관리하였습니다.

```swift
import Foundation

struct Tracking {
    // 사용할 때 불필요하게 인스턴스화 하지 않도록 하기 위함.
    private init() { }
    
    struct Screen {
        private init() { }

        static let splash = "A1_스플래시"
        static let onboarding = "A2_온보딩"
        static let login = "A3_로그인"
    }

    struct Event {
        private init() { }

        static let touchOnboardingStart = "A2_온보딩_시작"
        static let touchKakaoLogin = "A3_로그인_카카오"
        static let touchAppleLogin = "A3_로그인_애플"
    }
}
```

- SplashViewController.swift

```swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
        
    setTracking()
}

private func setTracking() {
    Analytics.logEvent(AnalyticsEventScreenView,
                       parameters: [
                        AnalyticsParameterScreenName: Tracking.Screen.splash
                       ])
}
```

## 2️⃣ 이벤트 추적을 위한 GA

FirebaseApp 인스턴스를 구성한 후에는 [logEvent()](https://firebase.google.com/docs/reference/swift/firebaseanalytics/api/reference/Classes/Analytics?hl=ko#logevent_:parameters:) 메서드를 사용하여 이벤트 로깅을 시작할 수 있습니다.

다음의 예시 코드를 통해서 쉽게 사용할 수 있습니다.

```swift
Analytics.logEvent(AnalyticsEventSelectContent, parameters: [
  AnalyticsParameterItemID: "id-\(title!)",
  AnalyticsParameterItemName: title!,
  AnalyticsParameterContentType: "cont",
])
```

코드에서 `AnalyticsEventSelectContent` 맞춤 이벤트를 확인할 수 있었는데요.

**제가 원했던 의도는 다음과 같습니다.**

> 해당 이벤트 자체만을 트래킹할 수 있는 맞춤 이벤트를 기록하고 싶습니다.
> 

### 맞춤 이벤트 트래킹 적용해보기

추천 이벤트로 해결되지 않을 수 있고, 혹은 저처럼 맞춤 이벤트를 기록하고 싶을 수 있습니다. 그럴때는 다음과 같이 사용할 수 있습니다.

다음의 예시 코드처럼 사용할 수 있습니다.

```swift
Analytics.logEvent("share_image",
                    parameters: [
                        "name": name as NSObject,
                        "full_text": text as NSObject,
                    ])
```

저는 다음과 같이 사용하였습니다.

- OnboardingViewController.swift

```swift
private func touchOnbardingStartEventTracking() {
    Analytics.logEvent(Tracking.Event.touchOnboardingStart,
                       parameters: nil)
}

// 해당 시작 버튼 클릭 시 이벤트를 트래킹하고 싶다.
```

### 기본 이벤트 매개변수 설정

또는, [setDefaultEventParameters](https://firebase.google.com/docs/reference/swift/firebaseanalytics/api/reference/Classes/Analytics?hl=ko#setdefaulteventparameters_:) 를 사용하면 여러 이벤트를 아울러 매개변수를 로깅할 수 있습니다.

```swift
Analytics.setDefaultEventParameters([
  "level_name": "Caverns01",
  "level_difficulty": 4
])
```

매개변수가 `logEvent()` 메서드에서 지정되었다면 해당 값이 위의 기본값 대신 사용됩니다.

기본값으로 설정한 매개변수를 삭제하려면 매개변수를 `nil`로 설정하고 `setDefaultEventParameters` 메서드를 호출하면 됩니다.

**올바르게 로깅되고 있는지 모니터링 해봐야겠죠?**

## 3️⃣ Xcode 디버그 콘솔에서 이벤트 보기

상세 로깅을 사용 설정하여 SDK의 이벤트 로깅을 모니터링하면 이벤트가 올바르게 로깅되는지 확인할 수 있습니다. 여기에는 자동으로 로깅되는 이벤트와 수동으로 로깅되는 이벤트가 모두 포함됩니다.

다음과 같이 상세 로깅을 사용 설정할 수 있습니다.

1. Xcode에서 Product(제품) > Scheme(스킴) > Edit scheme(스킴 수정)을 선택합니다.
2. 왼쪽 메뉴에서 Run(실행)을 선택합니다.
3. Arguments(인수) 탭을 선택합니다.
4. Arguments Passed On Launch(실행 시 전달 인수) 섹션에 `FIRAnalyticsVerboseLoggingEnabled`를 추가합니다. **(-FIRAnalyticsDebugEnabled 를 추가해야 합니다.)**

다음에 앱을 실행하면 Xcode 디버그 콘솔에 이벤트가 표시되어 이벤트 전송 여부를 즉시 확인할 수 있습니다.

<img width="700" alt="1" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/03e04cac-eb4e-40ba-935a-06dedde2768a">

**모든 자동, 수동 이벤트가 로깅되기 때문에 보기 불편할 수 있습니다. 비활성화하기 위해서 콘솔에서 다음과 같이 안내해줍니다.**

```swift
To disable debug logging set the following application argument: -noFIRAnalyticsDebugEnabled (see http://goo.gl/RfcP7r)
```

- `-noFIRAnalyticsDebugEnabled` 를 추가해서 실행해주시면 됩니다. 해당 매개변수를 실행 후 지워도 상관없습니다.
- 즉, 활성화할때는 `-FIRAnalyticsVerboseLoggingEnabled` , 비활성화할때는 `-noFIRAnalyticsDebugEnabled` 를 사용하고 유지하거나 삭제해야하는 것은 없습니다. 그저 실행 시 전달되는 매개변수이기 때문입니다.

<img width="700" alt="2" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/99762b4e-6f12-4d99-9d87-7d923302c03c">

### 🚨 트러블 슈팅

```swift
Event name must contain only letters, numbers, or underscores: A3.로그인_카카오
```

처음에는 온점을 포함했는데 다음과 같이 로그가 출력되서 온점을 ‘_’ 로 대체하였습니다.

### 결과

- 다음과 같이 자동, 수동 이벤트가 모두 로깅되는 것을 확인할 수 있었습니다.

정말 많이 로깅되기 때문에 확인할 때 사용할 수 있을 것 같습니다. 로깅이 되고 있음을 확인할 수 있는 일부를 가져왔습니다.

```swift
Logging screen view with screen name and screen class: A1.스플래시, SplashViewController

Event logged. Event name, event params: screen_view (_vs), {
    _mst = 1;
    ga_debug (_dbg) = 1;
    ga_event_origin (_o) = app;
    ga_realtime (_r) = 1;
    ga_screen (_sn) = A1.스플래시;
    ga_screen_class (_sc) = SplashViewController;
    ga_screen_id (_si) = -5531411540114619464;
}

Logging event: origin, name, params: app, A2_온보딩_시작, {
    ga_event_origin (_o) = app;
    ga_screen (_sn) = A2.온보딩;
    ga_screen_class (_sc) = SplashViewController;
    ga_screen_id (_si) = -5531411540114619463;
}
```

- 다음과 같이 screen_view 의 이벤트로 화면 트래킹을 확인할 수 있습니다.

<img width="700" alt="3" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/d1920326-0aa6-4582-b4c2-64b6253dc564">

- 다음과 같이 screen_view, 맞춤 이벤트로 만든 A2_온보딩_시작 등의 이벤트로 이벤트 트래킹을 확인할 수 있습니다.

<img width="700" alt="4" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/4b440fa0-dcfe-40de-8048-117c314f8601">
