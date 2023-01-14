---
title:  "iOS) Lock Screen Widget 만들기"
categories:
- iOS

date:   2023-01-14  11:40:00 +0900
author_profile: false
---
### 내용

- Lock Screen widget 의 UI 는 모두 불투명하게만 표현되어야 할까?
- Lock Screen 에 widget 을 만들어보자.

Lock Screen 의 위젯은 iOS 16 부터 새롭게 등장한 기술입니다. 아직 지원하는 앱은 많지 않지만 예시로 카카오톡과 카카오페이 위젯을 살펴보겠습니다.

![스크린샷 2023-01-13 오후 4.49.30.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eca2c5ed-d39e-41e6-8457-5382e0234ee7/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-01-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_4.49.30.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f6c8beea-e7b7-49a4-bd87-be478af9af8c/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3f4149f6-df72-43d9-a74f-3c8acaf50b09/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f42f0017-1941-4075-ad5a-acbef11ed2a8/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/04ea34a3-296e-4be7-a64b-050c749640c1/Untitled.png)

- 단순히 불투명한 듯한 UI 가 아닌듯한데 어떤 원리로 자연스럽게 배경과 어울러지는걸까요?
- 이는 적합한 이미지 에셋이 필요한걸까요? 함수로써 구현해줘야하는 걸까요? 아니면, 잠금화면에 맞는 렌더링 모드가 적용되는 걸까요?
- WidgetKit 에서 Lock Screen 이라는 환경을 설정하고, 대응해줘야하는 걸까요?

***여러가지 의문점이 있습니다! 이외에도 Lock Screen Widget 에 대해서 이제 알아보겠습니다.***

다음의 문서에서 렌더링 방식에 대해 확인해보았습니다.

[Apple Developer Documentation](https://developer.apple.com/documentation/widgetkit/creating-lock-screen-widgets-and-watch-complications)

**WidgetKit 은 세가지 다른 렌더링 모드를 사용합니다.**

- Lock Screen(잠금화면)과 complications 는 더 작고, 항상 표시되며, 플랫폼에 대해 tinted mode 를 지원하기 때문에 보다 제한된 색상을 사용하게 됩니다.

### 1️⃣vibrant

**잠금 화면 위젯의 경우,** iOS 텍스트, 이미지 및 게이지에 대해 monochrome(단색)으로 채도를 낮추고, 콘텐츠를 잠금 화면 배경에 맞게 색상을 지정하여 vibrant effect(생동감 있는 효과)를 생성하게 됩니다.

WWDC22) Complications and widgets: Reloaded 의 세션을 참고해보겠습니다.(아래는 세션을 시청하고 내용을 요약해놓은 글입니다.)

[WWDC22) Complications and widgets: Reloaded](https://gyuios.tistory.com/226)

**Vibrant** 에 대한 이야기를 시작해보겠습니다.

![스크린샷 2022-08-30 오전 1.11.42.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b61fa927-3d84-4460-bdab-39dc40ee20f9/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-08-30_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_1.11.42.png)

iOS 의 **Vibrant** 모드에서는 컨텐츠는 흐릿해진 다음 Lock Screen 배경에 맞게 적절한 색으로 바뀝니다.

시스템은 greyscale 의 컨텐츠를 주목할 수 있도록 그려냅니다. 그리고 이것은 환경에 적합하게 나타납니다.

추가적으로 Lock Screen 은 vibrant rendering mode 를 색상으로 물들이도록 설정될 수 있습니다. 밝은 컬러는 결국 주로 불투명해지고 더 밝아집니다. 반면, 어두운 컬러는 배경에서 약간의 광택만으로 흐릿하게 나타납니다.

![스크린샷 2022-08-30 오전 1.16.29.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7b3d57d8-2615-4f14-9c57-5b999588e6d4/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-08-30_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_1.16.29.png)

가독성을 보장하기 위해서 **Vibrant** 모드에서 투명 컬러는 사용을 피하고 대신 어두운색이나 검정을 사용하여 가독성은 유지하면서 콘텐츠는 눈에 덜 띄게 합니다.

이러한 미묘한 차이인 일관된 배경을 위젯에 주기 위해서 `AccessoryWidgetBackground` view 를 도입했습니다.

![스크린샷 2022-08-30 오전 1.18.45.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/94e20080-1e8d-498f-9859-ecfc24ba206d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-08-30_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_1.18.45.png)

background view 는 다양한 위젯 렌더링 모드에서 여러 모습을 가집니다.

---

나머지 두 가지 렌더링 모드에 대해서 알아보겠습니다.

### 2️⃣fullColor

WidgetKit 은 해당 렌더링 모드에서 complication 의 뷰 색성을 변경하지 않습니다. 그라데이션 및 full-color 이미지와 텍스트, 게이지를 사용합니다. **해당 렌더링 모드는 iPhone 잠금 화면 위젯에서 사용할 수 없습니다.**

### 3️⃣accented

유저들이 tint color 를 사용하도록 watch face 를 정의하거나 특정 watch face 를 선택하면 WidgetKit 이 accented 렌더링 모드에서 complication 을 렌더링합니다. watchOS 는 위젯의 뷰 계층 구조를 accent gorup 과 default group 으로 나눈 다음, 각 그룹에 단색을 적용합니다.

`.widgetAccentable()` 뷰 수정자를 사용하여 group views 를 accented group 으로 그룹화합니다. **이 렌더링 모드는 iOS 에서는 사용할 수 없습니다.**

**결론적으로, 잠금 화면에 대한 위젯의 렌더링은 시스템에 의해서 vibrant 렌드링 모드로 적용되게 됩니다.**

WidgetKit 을 다루는데 중간중간 watch 에 대한 이야기가 나옵니다. Lock Screen Widget 에 대한 정보가 부족한것 같습니다! 개발자 문서가 알려주는 overview 를 정리해보겠습니다.

### ✅ Overview

iOS 16 및 watchOS 9 부터 WidgetKit 의 범위를 iPhone Lock Screen 으로, Apple Watch 의 컴플리케이션인 watch face 로 확장할 수 있게되었습니다.(아하!)

지속적으로 표시되고, 앱의 가장 관련성 높고 한 눈에 볼 수 있는(glanceable) 콘텐츠를 표시하며 사람들이 자세한 내용을 위해서 앱에 빠르게 접근할 수 있게 해줍니다.

잠금 화면 위젯과 워치 컴플리케이션은 WidgetKit 과 SwiftUI views 를 사용하여 다음을 수행할 수 있습니다 :

- iPhone 의 잠금 화면 위젯을 지원하도록 기존 iOS Home Screen 과 **Today View(아래에서 다루겠습니다.)** widget 의 코드를 업데이트할 수 있습니다.
- watchOS 앱용 WidgetKit complications 를 제공하고, ClockKit complications 를 대체하고, iOS 와 watchOS 앱 간에 더 많은  코드를 공유할 수 있습니다.(이 내용은 WWDC22 세션 군데군데에서 얻을 수 있었습니다. 제가 정리한 관련 세션을 공유해드립니다.)

(ClockKit 을 대체하고 마이그레이션하는 내용에 대해서 다룹니다.)

[WWDC22) Complications and widgets: Reloaded](https://gyuios.tistory.com/226)

[WWDC22) Go further with Complications in WidgetKit](https://gyuios.tistory.com/247)

(아래 제가 작성한 글은 Apple Developer 에서 제공하는 튜토리얼을 통해 iOS 와 watchOS 앱 간에 코드를 공유하는 내용을 다룹니다.)

[WatchOS) Creating a watchOS App - SwiftUI Tutorials](https://gyuios.tistory.com/244)

- watch complications 와 Lock Screen widgets 을 동시에 생성합니다.
- iOS 또는 watchOS 에 대한 complications 와 widgets 의 지원을 제공합니다.

---

iOS 16 및 watchOS 9 부터 WidgetKit 의 범위가 확장되어 watchOS 앱용으로 WidgetKit complications 를 지원하기 때문에 설명들에 워치 이야기가 나오는 것을 알 수 있었습니다.

**이번 포스팅에서 다룰 내용은 기존의 프로젝트에서 WidgetKit 과 SwiftUI views 를 사용하여 iPhone Lock Screen widgets 을 지원해보는 것입니다.**

---

### 👉 Today View?

(위에서 언급된 Today View 는 Home Screen 또는 Lock Screen 에서 오른쪽으로 스와이프 하면 나타나는 뷰입니다.)

“Or you can use widgets from Today View by swiping right from the Home Screen or Lock Screen.”

**출처:**

[How to add and edit widgets on your iPhone](https://support.apple.com/en-us/HT207122)

---

***HIG 에서 Lock Screen Widget 에 대해서 얻을 정보가 있는지 살펴보겠습니다.***

### ✅ HIG - Widgets

iOS 16은 잠금 화면에 위젯을 도입하여 사람들이 Apple Watch face 에 컴플리케이션을 배치하는 것과 유사한 방식으로 iPhone 잠금 화면에 풍부하고 한눈에 볼 수 있는(glanceable) 콘텐츠를 배치할 수 있도록 합니다. 잠금 화면 위젯은 디자인과 구현 모두 컴플리케이션과 유사합니다.
가이드는 아래의 [Platform considerations](https://developer.apple.com/design/human-interface-guidelines/components/system-experiences/widgets#platform-considerations) 를 참조할 수 있습니다.

(👉 WidgetKit 의 범위를 iPhone Lock Screen 으로, Apple Watch 의 컴플리케이션인 watch face 로 확장했던 것은 HIG 적으로도 의도한 바임을 알 수 있었습니다.)

Today View, iPhone 의 잠금화면 그리고 iPadOS 의 Home Screen 에서는 앱 이름이 위젯 아래에 표시되지 않습니다.

(👉 생각해보니 아이폰의 Home Screen 과 달리 아이패드는 표시가 되지 않군요!)

### Platform considerations

**출처:**

[Widgets](https://developer.apple.com/design/human-interface-guidelines/components/system-experiences/widgets)

***이제 프로젝트에서 다음과 같이 잠금화면을 위한 위젯을 추가해보겠습니다.***

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2715bd71-a7f1-4a30-992a-7edb8e916308/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0e170719-8575-4729-b0b7-a2a00b3e7cc9/Untitled.png)

### 1️⃣ 프로젝트 세팅

아래 글에서 진행한 프로젝트에서 진행하였습니다. 아래의 글을 읽지 않아도 프로젝트에서 구현되는 부분은 별개이기 때문에 무관할 것 같습니다.🙂

- 두 가지 iPhone Lock Screen 위젯을 만들어보겠습니다.
- QR Code 를 인식할 수 있도록 앱 내의 카메라 뷰로 이동할 수 있는 위젯.
- 앱을 실행할 수 있는 위젯.

### 2️⃣ WidgetFamily

같은 기능을 하는 위젯인데 UI 를 위해서 따로 위젯을 만들어야 할까요? 그럴 필요는 없습니다. **왜냐면 다음과 같이 기존의 위젯 파일에서 진행할 수 있습니다.**(개발자 문서의 코드 예시입니다.)

```swift
struct EmojiRangerWidgetEntryView: View {
    var entry: Provider.Entry
    
    @Environment(\.widgetFamily) var family

    @ViewBuilder
    var body: some View {
        switch family {
        case .accessoryInline:
            // Code to construct the view for the inline widget or watch complication.
        case .accessoryRectangular:
            // Code to construct the view for the rectangular Lock Screen widget or watch complication.
        case .accessoryCircular:
            // Code to construct the view for the circular Lock Screen widget or watch complication.
        case .systemSmall:
            // Code to construct the view for the small widget.
        default:
            // Code to construct the view for other widgets; for example, the system medium or large widgets.
        }
    }
}
```

iOS 와 watchOS 는 다음과 같이 **WidgetFamily** 를 공유합니다. 아래의 widget family 들은 watchOS 9 에 새롭게 등장하였습니다.

![스크린샷 2023-01-13 오후 7.29.11.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1751424c-a759-4a6e-bfe7-e9dc47436bd0/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-01-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_7.29.11.png)

아래의 widget family 들은 위젯이 등장한 iOS 14부터 생겼습니다. **이하는 Lock Screen 에서 사용할 수 없습니다.**

![스크린샷 2023-01-13 오후 7.32.07.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/89158ba3-abee-415a-a046-e61228334e3f/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-01-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_7.32.07.png)

프로젝트에 적용해보겠습니다. 

### 3️⃣ widget 구현

- QRCodeWidget.swift

```swift
struct QRCodeEntryView : View {
    var entry: QRCodeProvider.Entry

    // ✅ 적용되는 widget family 에 따른 변수.
    @Environment(\.widgetFamily) var widgetFamily
    
    var body: some View {

        // ✅ switch 문을 사용하여 Lock Screen Widget 구현.
        switch widgetFamily {
        case .accessoryCircular:
                Image("widgetQr")
                    .resizable()
                    .widgetURL(URL(string: "openQRCode"))
        @unknown default:
            Image("widgetQr")
                .resizable()
                .scaledToFill()
                .widgetURL(URL(string: "openQRCode"))
        }
    }
}

struct QRCodeWidget: Widget {
    let kind: String = "QRCodeWidget"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind,
                            provider: QRCodeProvider()) { entry in
            QRCodeEntryView(entry: entry)
        }

        // ✅ Lock Screen Widget 을 추가할 때도 동일하게 적용됩니다.
        .configurationDisplayName("QR Code 위젯")
        .description("QR Code 를 인식할 수 있도록 카메라로 빠르게 접근합니다.")

        // ✅ accessoryCircular 추가로 지원.
        .supportedFamilies([.systemSmall, .accessoryCircular])
    }
}

struct QRCodeWidget_Previews: PreviewProvider {
    static var previews: some View {
        QRCodeEntryView(entry: QRCodeEntry(date: Date()))
            .previewContext(WidgetPreviewContext(family: .systemSmall))

        // ✅ 프리뷰를 통해 보기위해 widget family 를 적용하여 QRCodeEntryView 추가.
        QRCodeEntryView(entry: QRCodeEntry(date: Date()))
            .previewContext(WidgetPreviewContext(family: .accessoryCircular))
    }
}
```

위젯이 보여지는 형태에 따라 즉, widget family 에 따라 switch 문으로 구현할 수 있습니다. QRCodeWidget 을 탭했을 때 동일한 기능을 구현하고자 한다면 굳이 새로운 위젯을 만들지 않아도 됩니다.

**새로운 위젯을 프로젝트에 적용해봅시다!** 그저 앱을 실행시키는 위젯을 만들어 보겠습니다. 이는 Lock Screen 에서만 사용하겠습니다.

- OpenAppLockScreenWidget.swift

```swift
import WidgetKit
import SwiftUI

// 👉 StaticConfiguration 을 생성했을 때의 코드 그대로 사용하였습니다.
struct OpenAppLockScreenProvider: TimelineProvider {
    func placeholder(in context: Context) -> OpenAppLockScreenEntry {
        OpenAppLockScreenEntry(date: Date())
    }
    
    func getSnapshot(in context: Context, completion: @escaping (OpenAppLockScreenEntry) -> ()) {
        let entry = OpenAppLockScreenEntry(date: Date())
        completion(entry)
    }
    
    func getTimeline(in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        var entries: [OpenAppLockScreenEntry] = []
        
        let currentDate = Date()
        for hourOffset in 0 ..< 5 {
            let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
            let entry = OpenAppLockScreenEntry(date: entryDate)
            entries.append(entry)
        }
        
        let timeline = Timeline(entries: entries, policy: .never)
        completion(timeline)
    }
}

struct OpenAppLockScreenEntry: TimelineEntry {
    let date: Date
}

// ✅ widget family 변수를 사용하여 구현하였습니다.
struct OpenAppLockScreenEntryView : View {
    var entry: OpenAppLockScreenProvider.Entry
    @Environment(\.widgetFamily) var widgetFamily
    
    var body: some View {
        switch widgetFamily {
        case .accessoryCircular:
            ZStack {
                // ✅ 일관된 배경을 위젯에 주기 위해서 추가.
                AccessoryWidgetBackground()
                Image("logoNada")
                    .resizable()
                    // 원본 이미지의 색상을 그대로 사용하니 흐릿해서 진하게 표현하고자 다음의 코드를 사용.
                    .renderingMode(.template)
                    .tint(.white)
                    .padding(EdgeInsets(top: 8, leading: 8, bottom: 8, trailing: 8))
            }
        @unknown default:
            Image("logoNada")
        }
    }
}

struct OpenAppLockScreenWidget: Widget {
    let kind: String = "OpenAppLockScreen"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind,
                            provider: OpenAppLockScreenProvider()) { entry in
            OpenAppLockScreenEntryView(entry: entry)
        }

        // ✅ 추가할 때 설명을 작성하였습니다.
        .configurationDisplayName("나다 NADA")
        .description("나다 NADA를 실행합니다.")

        // ✅ accessoryCircular 만 지원.
        .supportedFamilies([.accessoryCircular])
    }
}

struct OpenAppLockScreenWidget_Previews: PreviewProvider {
    static var previews: some View {
        OpenAppLockScreenEntryView(entry: OpenAppLockScreenEntry(date: Date()))
            .previewContext(WidgetPreviewContext(family: .accessoryCircular))
    }
}
```

***개발자 문서에서 `AccessoryWidgetBackground` 에 대해서 언급한 부분을 가져와 보았습니다.***

### 👉 ****Set a Consistent Background Color****

widget 또는 complication 에 따라 일관된 배경을 위젯에 설정해야할 수 있습니다. 다음과 같이 `AccessoryWidgetBackground` 를 사용하여 위젯의 일관된 배경을 그릴 수 있습니다.

```swift
ZStack {
     AccessoryWidgetBackground()
     VStack {
        Text(“MON”)
        Text(“6”)
         .font(.title)
    }
}
```

Calendar app 에서 제공하는 원형의 Lock Screen widget 과 유사한 보기를 생성합니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/752569fb-ec63-418c-ba7c-ccf611cccdea/Untitled.jpeg)

**출처:**

[Apple Developer Documentation](https://developer.apple.com/documentation/widgetkit/creating-lock-screen-widgets-and-watch-complications)

***추가적으로, 잠금 화면이 비춰질 때 혹은 Always-On display 상태에서 민감한 정보를 숨기고 싶을 수 있습니다. 그에 대한 내용도 간단하게 소개하겠습니다.***

### 👉 Consider User Privacy

iOS 잠금 화면 위젯과 watchOS 컴플리케이션은 항상 표시됩니다. 위젯에 표시되는 정보를 검토해서, 기기가 잠겨 있거나 콘텐츠가 Apple Watch 및 iPhone 잠금 화면에 낮은 휘도로 나타날 때 민감한 정보를 숨길 수 있습니다.

추가적인 정보는 [Display a Placeholder Widget and Hide Sensitive Data](https://developer.apple.com/documentation/widgetkit/creating-a-widget-extension#Display-a-Placeholder-Widget-and-Hide-Sensitive-Data) 를 참조하면 됩니다.

**출처:**

[Apple Developer Documentation](https://developer.apple.com/documentation/widgetkit/creating-lock-screen-widgets-and-watch-complications)

### 🚨트러블 슈팅 - vibrant 렌더링을 위해서 사용해야하는 이미지

이미지에 따라서도 시스템이 반영하는 바가 달랐습니다. 이하 두가지 이미지를 사용하였습니다.

![스크린샷 2023-01-13 오후 5.25.40.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e0293853-5f71-4217-bfcd-5eb5463062fe/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-01-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.25.40.png)

![스크린샷 2023-01-13 오후 5.23.00.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9f6fe24-0579-419b-95d0-3210fa126c1f/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-01-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.23.00.png)

전자의 경우는 배경이 투명했기 때문에 `AccessoryWidgetBackground()` 뷰를 사용해주어야 했고, 후자의 경우는 배경이 존재해서 아래와 같이 렌더링 되었고, `.resizable()` 뷰 수정자를 사용하니 자연스럽게 반영되었습니다.

![스크린샷 2023-01-13 오후 5.27.35.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e1f4371a-3ce1-4d32-92dc-a5a1fc1db2f6/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-01-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.27.35.png)

![스크린샷 2023-01-13 오후 5.25.08.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/01695a82-5b25-4d4e-9645-82091f0867ff/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-01-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.25.08.png)

---

전체 코드는 다음 프로젝트에서 확인할 수 있습니다.

[https://github.com/hyun99999/WidgetsWithCoreDataTutorial-iOS](https://github.com/hyun99999/WidgetsWithCoreDataTutorial-iOS)

**출처:**

[Apple Developer Documentation](https://developer.apple.com/documentation/widgetkit/creating-lock-screen-widgets-and-watch-complications)

[Widgets](https://developer.apple.com/design/human-interface-guidelines/components/system-experiences/widgets)
