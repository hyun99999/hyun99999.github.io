---
title:  "iOS) KakaoQRcode 클론코딩 - Widget(2/2)"
categories:
- iOS

date:   2021-09-04  22:16:00 +0900
author_profile: false
---
# 😇 본격적으로 클론코딩을 해보자

## 1️⃣ 앱 이름 변경

- 앱 이름 : 1번 결정. [General] → [Identity] → [Display Name] 에서 다음과 같이 설정해준다.

<img width="700" alt="19" src="https://user-images.githubusercontent.com/69136340/132096280-272a9516-090c-490b-9413-ab633eb726bb.png">

## 2️⃣ 여러가지 위젯 생성

위의 코드를 보면서 의문을 가졌다. 그러면 아래처럼 이름도 설명도 크기도 다른 위젯들을 어떻게 추가할 수 있을까?(애니메이션은 참 좋다 크-)

<img src ="https://user-images.githubusercontent.com/69136340/132096297-769b5c2b-383f-457a-b9eb-ef2deee40905.gif" width ="250">

## 📌 [WidgetBundle](https://developer.apple.com/documentation/swiftui/widgetbundle)

단일 widget extension 에서 여럿 위젯을 노출시키는데 사용되는 container.

여러 유형의 위젯을 지원하려면 `WidgetBundle` 을 채택하는 구조체에 `@main` 속성을 추가하십시오.

- apple developer's example code

```swift
@main
struct GameWidgets: WidgetBundle {
   var body: some Widget {
       GameStatusWidget()
       CharacterDetailWidget()
   }
}
```

## 📌 [IntentTimelineProvider](https://developer.apple.com/documentation/widgetkit/intenttimelineprovider) & [TimelineProvider](https://developer.apple.com/documentation/widgetkit/timelineprovider)

카카오톡 위젯을 살펴보면 전부 StaticConfiguration 처럼 위젯의 편집을 허용하지 않는다. 그래서 body 클로저 리턴값으로 StaticConfiguration 을 설정했다.

(target 를 만들 때 체크박스에 따라서 코드가 달라진다.)

- KakakoWidget_iOS_CloneCoding.swift

주석 처리 된 부분들이 IntentConfiguration 에 해당하는 경우들이다.

```swift
import WidgetKit
import SwiftUI
//import Intents

//struct Provider: IntentTimelineProvider {
struct Provider: TimelineProvider {
    func placeholder(in context: Context) -> SimpleEntry {
//        SimpleEntry(date: Date(), configuration: ConfigurationIntent())
        SimpleEntry(date: Date())
    }

//    func getSnapshot(for configuration: ConfigurationIntent, in context: Context, completion: @escaping (SimpleEntry) -> ()) {
    func getSnapshot(in context: Context, completion: @escaping (SimpleEntry) -> ()) {
//        let entry = SimpleEntry(date: Date(), configuration: configuration)
        let entry = SimpleEntry(date: Date())
        completion(entry)
    }

//    func getTimeline(for configuration: ConfigurationIntent, in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
    func getTimeline(in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        var entries: [SimpleEntry] = []

        // Generate a timeline consisting of five entries an hour apart, starting from the current date.
        let currentDate = Date()
        for hourOffset in 0 ..< 5 {
            let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
//            let entry = SimpleEntry(date: entryDate, configuration: configuration)
            let entry = SimpleEntry(date: entryDate)
            entries.append(entry)
        }

        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
}

struct SimpleEntry: TimelineEntry {
    let date: Date
//    let configuration: ConfigurationIntent
}

// ...

struct KakaoWidget_iOS_CloneCoding_Previews: PreviewProvider {
    static var previews: some View {
//        QRcodeWidgetEntryView(entry: SimpleEntry(date: Date(), configuration: ConfigurationIntent()))
        QRcodeWidgetEntryView(entry: SimpleEntry(date: Date()))
            .previewContext(WidgetPreviewContext(family: .systemSmall))
    }
}
```

### 🤨 여담

- 카카오톡이 의도한것인지 아닌지 모르겠지만 "즐겨찾기" 위젯의 경우는 같은 위젯에서 다른 크기를 사용했는데(위젯을 넘겨도 같은 설명은 그대로) "톡캘린더" 위젯에서는 systemSmall 과 systemMedium 을 다른 위젯 같은 설명을 사용한다.(위젯을 넘기면 같은 설명이 또 등장) 같은 설명을 사용할 것이라면 굳이 다른 위젯에 할 이유는 없었을 텐데 의아했다.

**위의 개념들에 대해서 이해해봤으니 구현해보자!**

## 📌 사이즈 별로 뷰 대응 + 여러 유형의 위젯 지원

카카오톡 위젯을 살펴보면 같은 위젯이지만 사이즈별로 뷰가 대응되는 것을 볼 수 있다.

또한, 여러 유형의 위젯을 볼 수 있다.(코드가 길어져서 2가지 위젯만 추가하는 코드를 남겼다. 전체코드는 깃허브에 있다.)

- KakakoWidget_iOS_CloneCoding.swift

```swift
// ...(다른 위젯들)

// ✅ Calender widget. 톡캘린더 역할을 하는 위젯 생성.
struct CalenderWidgetEntryView: View {
    // ✅ 위젯의 환경변수에 접근해서 사이즈별로 뷰를 대응할 수 있다.
    @Environment(\.widgetFamily) var family: WidgetFamily
    var entry: Provider.Entry

    var body: some View {
        sizeBody()
    }
    
    @ViewBuilder
    func sizeBody() -> some View {
        switch family {
        case .systemSmall:
            Text("톡캘린더 - small")
        case .systemMedium:
            Text("톡캘린더 - medium")
        default:
            EmptyView()
        }
    }
}

struct CalenderWidget: Widget {
    let kind: String = "CalenderWidget"

    var body: some WidgetConfiguration {
        // ✅ IntentConfiguration 의 경우.
        // IntentConfiguration(kind: kind, intent: ConfigurationIntent.self, provider: Provider()) { entry in
        //     CalenderWidgetEntryView(entry: entry)
        // }
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            CalenderWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("톡캘린더")
        .description("예정된 일정을 쉽게 확인하고 톡캘린더에\n빠르게 접근합니다.")
        // ✅ 배열에 담긴 순서 상관없이 small -> medium -> large 순서다.
        .supportedFamilies([.systemMedium, .systemSmall])
    }
}

// ✅ QRcode widget. QR코드 역할을 하는 위젯 생성.
struct QRcodeWidgetEntryView : View {
    var entry: Provider.Entry

    var body: some View {
        Text("QR코드")
    }
}

//@main
struct QRcodeWidget: Widget {
    let kind: String = "QRcodeWidget"

    var body: some WidgetConfiguration {
        // ✅ IntentConfiguration 의 경우.
        // IntentConfiguration(kind: kind, intent: ConfigurationIntent.self, provider: Provider()) { entry in
        //     QRcodeWidgetEntryView(entry: entry)
        // }
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            QRcodeWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("QR체크인")
        .description("홈 화면에서 QR체크인 페이지로\n 빠르게 접근합니다.")
        .supportedFamilies([.systemSmall])
        
    }
}

// ✅ 여러 종류의 위젯 추가
// ✅ 여기에 @main 추가
@main
struct KakaoWidget: WidgetBundle {
    var body: some Widget {
        // ...(다른 위젯들)
        CalenderWidget()
        QRcodeWidget()
    }
}
```

## 📌 결과

결과적으로 총 4개(내프로필, 즐겨찾기(2개), 톡캘린더(2개), QR체크인) 의 위젯을 만들어보았다. 

<img src ="https://user-images.githubusercontent.com/69136340/132096389-7698a2c8-36c2-4255-87b0-84791580f8c8.gif" width ="250">

## 3️⃣ 해당 위젯으로 앱 내의 특정 뷰 접근

이제 위젯을 통해서 앱 내의 특정 뷰로 접근하도록 해보겠다.

## 📌 구현

### ❗️[widgetURL(_:)](https://developer.apple.com/documentation/swiftui/view/widgeturl(_:))

사용자가 위젯을 클릭할 때 containing app 에서 열릴 URL 설정.

**overview**

위젯의 뷰계층에서 하나의 `widgetURL(_:)` 만 지원한다. 여러 뷰에 `widgetURL(_:)` 가 있는 경우 작동되지 않는다. 

- KakaoWidget_iOS_CloneCoding.swift

```swift
// QRcode widget
struct QRcodeWidgetEntryView : View {
    // ✅ AppDelegate 에서 확인할 URL
    let url = URL(string: "qrcode")
    var entry: Provider.Entry

    var body: some View {
        Text("QR코드")
            // ✅ 이 처럼 하나의 widgetURL(_:) 만 사용해야한다.
            .widgetURL(url)
    }
}
```

### ❗️ [application(_:open:options:)](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623112-application)

url 로 지정된 리소스를 열라고 요청하고 launch option 의 dictionary 제공.

**Return Value**

delegate 가 요청을 성공적으로 처리한 경우 true. URL 리소스 열기 시도가 실패한 경우 false.

- AppDelegate.swift

```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
        let qrcodeLinkPath = "qrcode"
        
        // ✅ URL 구문분석
        guard let components = NSURLComponents(url: url, resolvingAgainstBaseURL: true), let path = components.path else {
            return false
        }
        // ✅ 정의된 URL 요청 외의 시도는 false.
        if path == qrcodeLinkPath {
            // ✅ qr코드 화면전환
            let nextVC = QRCodeViewController()
            nextVC.modalPresentationStyle = .overFullScreen
            window?.rootViewController?.present(nextVC, animated: true, completion: nil)
            
            return true
        } else {
            return false
        }
    }
```

### 📌 결과

<img src ="https://user-images.githubusercontent.com/69136340/132096355-68acd21c-a01d-4551-b0e8-db9325749215.gif" width ="250">

### 참고 :

[Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/widget/)

[iOS 14+ ) Widget](https://zeddios.tistory.com/1077)

[[SwiftUI] Widget 위젯만들기](https://nsios.tistory.com/156)

## 깃허브

[GitHub - 28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu: 🧚 아요 미라클론코딩 김현규](https://github.com/28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu)
