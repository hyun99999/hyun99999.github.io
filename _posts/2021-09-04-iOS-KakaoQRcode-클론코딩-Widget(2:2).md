---
title:  "iOS) KakaoQRcode ν΄λ‘ μ½λ© - Widget(2/2)"
categories:
- iOS

date:   2021-09-04  22:16:00 +0900
author_profile: false
---
# π λ³Έκ²©μ μΌλ‘ ν΄λ‘ μ½λ©μ ν΄λ³΄μ

## 1οΈβ£ μ± μ΄λ¦ λ³κ²½

- μ± μ΄λ¦ : 1λ² κ²°μ . [General] β [Identity] β [Display Name] μμ λ€μκ³Ό κ°μ΄ μ€μ ν΄μ€λ€.

<img width="700" alt="19" src="https://user-images.githubusercontent.com/69136340/132096280-272a9516-090c-490b-9413-ab633eb726bb.png">

## 2οΈβ£ μ¬λ¬κ°μ§ μμ ― μμ±

μμ μ½λλ₯Ό λ³΄λ©΄μ μλ¬Έμ κ°μ‘λ€. κ·Έλ¬λ©΄ μλμ²λΌ μ΄λ¦λ μ€λͺλ ν¬κΈ°λ λ€λ₯Έ μμ ―λ€μ μ΄λ»κ² μΆκ°ν  μ μμκΉ?(μ λλ©μ΄μμ μ°Έ μ’λ€ ν¬-)

<img src ="https://user-images.githubusercontent.com/69136340/132096297-769b5c2b-383f-457a-b9eb-ef2deee40905.gif" width ="250">

## π [WidgetBundle](https://developer.apple.com/documentation/swiftui/widgetbundle)

λ¨μΌ widget extension μμ μ¬λΏ μμ ―μ λΈμΆμν€λλ° μ¬μ©λλ container.

μ¬λ¬ μ νμ μμ ―μ μ§μνλ €λ©΄ `WidgetBundle` μ μ±ννλ κ΅¬μ‘°μ²΄μ `@main` μμ±μ μΆκ°νμ­μμ€.

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

## π [IntentTimelineProvider](https://developer.apple.com/documentation/widgetkit/intenttimelineprovider) & [TimelineProvider](https://developer.apple.com/documentation/widgetkit/timelineprovider)

μΉ΄μΉ΄μ€ν‘ μμ ―μ μ΄ν΄λ³΄λ©΄ μ λΆ StaticConfiguration μ²λΌ μμ ―μ νΈμ§μ νμ©νμ§ μλλ€. κ·Έλμ body ν΄λ‘μ  λ¦¬ν΄κ°μΌλ‘ StaticConfiguration μ μ€μ νλ€.

(target λ₯Ό λ§λ€ λ μ²΄ν¬λ°μ€μ λ°λΌμ μ½λκ° λ¬λΌμ§λ€.)

- KakakoWidget_iOS_CloneCoding.swift

μ£Όμ μ²λ¦¬ λ λΆλΆλ€μ΄ IntentConfiguration μ ν΄λΉνλ κ²½μ°λ€μ΄λ€.

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

### π€¨ μ¬λ΄

- μΉ΄μΉ΄μ€ν‘μ΄ μλνκ²μΈμ§ μλμ§ λͺ¨λ₯΄κ² μ§λ§ "μ¦κ²¨μ°ΎκΈ°" μμ ―μ κ²½μ°λ κ°μ μμ ―μμ λ€λ₯Έ ν¬κΈ°λ₯Ό μ¬μ©νλλ°(μμ ―μ λκ²¨λ κ°μ μ€λͺμ κ·Έλλ‘) "ν‘μΊλ¦°λ" μμ ―μμλ systemSmall κ³Ό systemMedium μ λ€λ₯Έ μμ ― κ°μ μ€λͺμ μ¬μ©νλ€.(μμ ―μ λκΈ°λ©΄ κ°μ μ€λͺμ΄ λ λ±μ₯) κ°μ μ€λͺμ μ¬μ©ν  κ²μ΄λΌλ©΄ κ΅³μ΄ λ€λ₯Έ μμ ―μ ν  μ΄μ λ μμμ νλ° μμνλ€.

**μμ κ°λλ€μ λν΄μ μ΄ν΄ν΄λ΄€μΌλ κ΅¬νν΄λ³΄μ!**

## π μ¬μ΄μ¦ λ³λ‘ λ·° λμ + μ¬λ¬ μ νμ μμ ― μ§μ

μΉ΄μΉ΄μ€ν‘ μμ ―μ μ΄ν΄λ³΄λ©΄ κ°μ μμ ―μ΄μ§λ§ μ¬μ΄μ¦λ³λ‘ λ·°κ° λμλλ κ²μ λ³Ό μ μλ€.

λν, μ¬λ¬ μ νμ μμ ―μ λ³Ό μ μλ€.(μ½λκ° κΈΈμ΄μ Έμ 2κ°μ§ μμ ―λ§ μΆκ°νλ μ½λλ₯Ό λ¨κ²Όλ€. μ μ²΄μ½λλ κΉνλΈμ μλ€.)

- KakakoWidget_iOS_CloneCoding.swift

```swift
// ...(λ€λ₯Έ μμ ―λ€)

// β Calender widget. ν‘μΊλ¦°λ μ­ν μ νλ μμ ― μμ±.
struct CalenderWidgetEntryView: View {
    // β μμ ―μ νκ²½λ³μμ μ κ·Όν΄μ μ¬μ΄μ¦λ³λ‘ λ·°λ₯Ό λμν  μ μλ€.
    @Environment(\.widgetFamily) var family: WidgetFamily
    var entry: Provider.Entry

    var body: some View {
        sizeBody()
    }
    
    @ViewBuilder
    func sizeBody() -> some View {
        switch family {
        case .systemSmall:
            Text("ν‘μΊλ¦°λ - small")
        case .systemMedium:
            Text("ν‘μΊλ¦°λ - medium")
        default:
            EmptyView()
        }
    }
}

struct CalenderWidget: Widget {
    let kind: String = "CalenderWidget"

    var body: some WidgetConfiguration {
        // β IntentConfiguration μ κ²½μ°.
        // IntentConfiguration(kind: kind, intent: ConfigurationIntent.self, provider: Provider()) { entry in
        //     CalenderWidgetEntryView(entry: entry)
        // }
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            CalenderWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("ν‘μΊλ¦°λ")
        .description("μμ λ μΌμ μ μ½κ² νμΈνκ³  ν‘μΊλ¦°λμ\nλΉ λ₯΄κ² μ κ·Όν©λλ€.")
        // β λ°°μ΄μ λ΄κΈ΄ μμ μκ΄μμ΄ small -> medium -> large μμλ€.
        .supportedFamilies([.systemMedium, .systemSmall])
    }
}

// β QRcode widget. QRμ½λ μ­ν μ νλ μμ ― μμ±.
struct QRcodeWidgetEntryView : View {
    var entry: Provider.Entry

    var body: some View {
        Text("QRμ½λ")
    }
}

//@main
struct QRcodeWidget: Widget {
    let kind: String = "QRcodeWidget"

    var body: some WidgetConfiguration {
        // β IntentConfiguration μ κ²½μ°.
        // IntentConfiguration(kind: kind, intent: ConfigurationIntent.self, provider: Provider()) { entry in
        //     QRcodeWidgetEntryView(entry: entry)
        // }
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            QRcodeWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("QRμ²΄ν¬μΈ")
        .description("ν νλ©΄μμ QRμ²΄ν¬μΈ νμ΄μ§λ‘\n λΉ λ₯΄κ² μ κ·Όν©λλ€.")
        .supportedFamilies([.systemSmall])
        
    }
}

// β μ¬λ¬ μ’λ₯μ μμ ― μΆκ°
// β μ¬κΈ°μ @main μΆκ°
@main
struct KakaoWidget: WidgetBundle {
    var body: some Widget {
        // ...(λ€λ₯Έ μμ ―λ€)
        CalenderWidget()
        QRcodeWidget()
    }
}
```

## π κ²°κ³Ό

κ²°κ³Όμ μΌλ‘ μ΄ 4κ°(λ΄νλ‘ν, μ¦κ²¨μ°ΎκΈ°(2κ°), ν‘μΊλ¦°λ(2κ°), QRμ²΄ν¬μΈ) μ μμ ―μ λ§λ€μ΄λ³΄μλ€. 

<img src ="https://user-images.githubusercontent.com/69136340/132096389-7698a2c8-36c2-4255-87b0-84791580f8c8.gif" width ="250">

## 3οΈβ£ ν΄λΉ μμ ―μΌλ‘ μ± λ΄μ νΉμ  λ·° μ κ·Ό

μ΄μ  μμ ―μ ν΅ν΄μ μ± λ΄μ νΉμ  λ·°λ‘ μ κ·Όνλλ‘ ν΄λ³΄κ² λ€.

## π κ΅¬ν

### βοΈ[widgetURL(_:)](https://developer.apple.com/documentation/swiftui/view/widgeturl(_:))

μ¬μ©μκ° μμ ―μ ν΄λ¦­ν  λ containing app μμ μ΄λ¦΄ URL μ€μ .

**overview**

μμ ―μ λ·°κ³μΈ΅μμ νλμ `widgetURL(_:)` λ§ μ§μνλ€. μ¬λ¬ λ·°μ `widgetURL(_:)` κ° μλ κ²½μ° μλλμ§ μλλ€. 

- KakaoWidget_iOS_CloneCoding.swift

```swift
// QRcode widget
struct QRcodeWidgetEntryView : View {
    // β AppDelegate μμ νμΈν  URL
    let url = URL(string: "qrcode")
    var entry: Provider.Entry

    var body: some View {
        Text("QRμ½λ")
            // β μ΄ μ²λΌ νλμ widgetURL(_:) λ§ μ¬μ©ν΄μΌνλ€.
            .widgetURL(url)
    }
}
```

### βοΈ [application(_:open:options:)](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623112-application)

url λ‘ μ§μ λ λ¦¬μμ€λ₯Ό μ΄λΌκ³  μμ²­νκ³  launch option μ dictionary μ κ³΅.

**Return Value**

delegate κ° μμ²­μ μ±κ³΅μ μΌλ‘ μ²λ¦¬ν κ²½μ° true. URL λ¦¬μμ€ μ΄κΈ° μλκ° μ€ν¨ν κ²½μ° false.

- AppDelegate.swift

```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
        let qrcodeLinkPath = "qrcode"
        
        // β URL κ΅¬λ¬ΈλΆμ
        guard let components = NSURLComponents(url: url, resolvingAgainstBaseURL: true), let path = components.path else {
            return false
        }
        // β μ μλ URL μμ²­ μΈμ μλλ false.
        if path == qrcodeLinkPath {
            // β qrμ½λ νλ©΄μ ν
            let nextVC = QRCodeViewController()
            nextVC.modalPresentationStyle = .overFullScreen
            window?.rootViewController?.present(nextVC, animated: true, completion: nil)
            
            return true
        } else {
            return false
        }
    }
```

### π κ²°κ³Ό

<img src ="https://user-images.githubusercontent.com/69136340/132096355-68acd21c-a01d-4551-b0e8-db9325749215.gif" width ="250">

### μ°Έκ³  :

[Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/widget/)

[iOS 14+ ) Widget](https://zeddios.tistory.com/1077)

[[SwiftUI] Widget μμ ―λ§λ€κΈ°](https://nsios.tistory.com/156)

## κΉνλΈ

[GitHub - 28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu: π§ μμ λ―ΈλΌν΄λ‘ μ½λ© κΉνκ·](https://github.com/28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu)
