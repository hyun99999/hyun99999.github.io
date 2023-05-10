---
title:  "iOS) 잠금 화면 위젯을 위해서 최소버전 대응하기"
categories:
- iOS

date:   2023-05-10  23:30:00 +0900
author_profile: false
---
### 내용

- 최소버전과 관련하여 위젯이 등장하지 않던 문제 원인 파악 및 해결
- iOS deployment target 15.0 으로 설정된 상태에서 잠금 화면 위젯(16.0+) 대응해보기

### 🚨 문제 상황 인식

메인 앱 타겟의 최소 버전은 15.0 이었습니다. 16.0 버전의 아이폰에서 앱은 설치할 수 있지만, 위젯을 설치할 수 없었습니다.

**그 이유는 widget extension 의 최소 버전이 16.2 로 설정되어 있었기 때문이었습니다.**

### **왜 16.2 였었나?**

PreviewProvider 프로토콜을 채택하여 preview 하는 struct 에서 Live Activity 의 미리보기를 만들 수 있는 메서드가 있는데 해당 메서드의 지원 버전이 16.2+ 이기 때문이었습니다.

[previewContext(_:isStale:viewKind:) | Apple Developer Documentation](https://developer.apple.com/documentation/activitykit/activityattributes/previewcontext(_:isstale:viewkind:)?changes=latest_ma_10__4_3&language=objc)

widget extension 을 만들 때 Live Activity 를 포함하는 옵션을 선택하게 되었고 자연스레 widget extension target 의 deployment target 이 16.2 로 설정되었습니다.

**추가적으로,** 아래 링크에서는 현재 iOS 사용 현황에 대해서 살펴볼 수 있습니다. 현재 iOS 15 미만의 기기들은 전체에서 8% 정도 됩니다.(iOS 16 은 72%, iOS 15 는 20%)

[App Store - 지원 - Apple Developer](https://developer.apple.com/kr/support/app-store/)

### ✅ 문제 해결

현재 서비스는 메인 앱이 최소 버전 15.0, 홈 위젯은 14.0+, 잠금 화면 위젯은 16.0+, 추가적으로 프로젝트 내에는 Live Acitivty 지원하는 위젯(16.1+)이 있지만 아직 서비스를 제공하지 않습니다.

그렇기 때문에 운영체제의 버전별로 분기처리를 해주지 않으면 다음과 같이 에러가 등장합니다.

<img width="600" alt="1" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/b0492d30-f0df-4417-85cb-f2101628da45">

대표적으로 잠금 화면 위젯을 지원하기 위한 widget family 의 case 로 accessoryCircular 가 있습니다. 해당 case 를 사용하기 위해서 16.0 이상에서만 가능합니다.

- 다음과 같이 코드에 분기처리를 해주겠습니다.

```swift
struct QRCodeWidget: Widget {
    // ✅ 지원하는 WidgetFamily 대응.
    private let supportedFamilies: [WidgetFamily] = {
        if #available(iOSApplicationExtension 16.0, *) {
            return [.systemSmall, .accessoryCircular]
        } else {
            return [.systemSmall]
        }
    }()
    
    let kind: String = "QRCodeWidget"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: QRCodeProvider()) { entry in
            QRCodeEntryView(entry: entry)
        }
        .configurationDisplayName("QR Code 위젯")
        .description("QR Code 를 인식할 수 있도록 카메라로 빠르게 접근합니다.")
        // ✅ supportedFamilies([.systemSall, .accessoryCircular]) 에서 분기처리하기 위해서 수정.
        .supportedFamilies(supportedFamilies)
    }
}

// 출처: https://stackoverflow.com/questions/72819402/lockscreen-swift-widget-only-available-to-ios-16-users

struct QRCodeWidget_Previews: PreviewProvider {
    static var previews: some View {
        QRCodeEntryView(entry: QRCodeEntry(date: Date()))
            .previewContext(WidgetPreviewContext(family: .systemSmall))
        // ✅ 지원하는 WidgetFamily 대응.
        if #available(iOSApplicationExtension 16.0, *) {
            QRCodeEntryView(entry: QRCodeEntry(date: Date()))
                .previewContext(WidgetPreviewContext(family: .accessoryCircular))
        }
    }
}
```

iOS 15.0 시뮬레이터에서 정상 실행되었습니다.

### 🚨 문제 상황 인식 - WidgetBundel 에서 분기처리

iOS 16.0 시뮬레이터에서는 실행 시 다음과 같이 `Fatal error: Unavailable` 가 등장했습니다.

<img width="600" alt="2" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/9363094d-8d83-4458-bc55-635c35842800">

다음과 같이 분기처리를 하는데도 다음과 같이 에러가 등장했습니다.

`Closure containing control flow statement cannot be used with result builder 'WidgetBundleBuilder’`

<img width="600" alt="21" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/5f704d7d-fd6c-420e-8cf9-f252bad4272c">

**WidgetBundleBuilder** 인 body 에서 분기 처리는 할 수 없다는 에러였습니다.

### ✅ 문제 해결

**WidgetBundle** 프로토콜을 살펴보면 아래와 같이 구성되어 있습니다.

```swift
public protocol WidgetBundle {
    associatedtype Body : Widget
    /// Creates a widget bundle using the bundle's body as its content.
    init()

    /// Declares the group of widgets that an app supports.
    ///
    /// The order that the widgets appear in this property determines the order
    /// they are shown to the user when adding a widget. The following example
    /// shows how to use a widget bundle builder to define a body showing
    /// a game status widget first and a character detail widget second:
    ///
    ///     @main
    ///     struct GameWidgets: WidgetBundle {
    ///        var body: some Widget {
    ///            GameStatusWidget()
    ///            CharacterDetailWidget()
    ///        }
    ///     }
    ///
    @WidgetBundleBuilder var body: Self.Body { get }
}
```

Body 는 Widget 자료형을 가지기 때문에 분기 처리 과정의 결과로 Widget 을 반환해주어야 합니다.

그래서 직접 `WidgetBundleBuilder.buildBock()` 를 사용해서 하나의 단일 Widget 을 반환하였습니다.

(참고로 10개까지 한번에 Widget 으로 반환해줄 수 있습니다.)

<img width="400" alt="3" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/2625ec47-ffc5-41be-8545-d3510939e11c">

```swift
import WidgetKit
import SwiftUI

@main
struct WidgetsBundle: WidgetBundle {
    var body: some Widget {
        // ✅ OpenAppLockScreenWidget 이 잠금화면 위젯이기 때문에 다음과 같이 분기처리.
        if #available(iOSApplicationExtension 16.0, *) {
            return WidgetBundleBuilder.buildBlock(MyCardWidget(), OpenAppLockScreenWidget(), QRCodeWidget())
        } else {
            return WidgetBundleBuilder.buildBlock(MyCardWidget(), QRCodeWidget())
        }
    }
}

// 출처: [https://developer.apple.com/forums/thread/711441](https://developer.apple.com/forums/thread/711441)
```

### 결과

iOS 15.0, 16.0 에서 정상작동을 확인할 수 있습니다.

<img width="700" alt="4" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/dfb023af-26d6-47da-8fd1-fdfedf8ddba3">

### 관련 이슈들 출처:

[What is the proper way of adding A… | Apple Developer Forums](https://developer.apple.com/forums/thread/711441)

[Variable WidgetBundle configuration based on conditions](https://www.avanderlee.com/swiftui/variable-widgetbundle-configuration/)
