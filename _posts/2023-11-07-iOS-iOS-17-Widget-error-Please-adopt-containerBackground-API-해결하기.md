 ---
title:  "iOS) iOS 17 Widget error- Please adopt containerBackground API 해결하기"
categories:
- iOS

date:   2023-11-07  23:06:00 +0900
author_profile: false
---
## 👉 내용

iOS 17 containerBackground API error 와 extra padding 이 생기는 문제를 해결해 보겠습니다.

- iOS 17 적용 후에 잠금화면 위젯이 제대로 보이지 않는 이슈가 발생했습니다. Xcode 에서 한 번 살펴보겠습니다.

<img width="450" alt="1" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/746797f1-3f45-4fbf-bb07-fa8d3f032e9f">

Xcode 에서 preview 로 확인해보려했더니 해당 경고가 등장했습니다.

<img width="450" alt="2" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/c0fbfcba-3f39-4700-9013-92e5aae70040">

**“Ensure that you have called the containerBackground(for: .widget) {…} modifier in your widget view.”**

## 👉 containerBackground(for:alignment:content:)

개발자 문서를 살펴보겠습니다.

[containerBackground(for:alignment:content:) | Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/view/containerbackground(for:alignment:content:))

<img width="500" alt="3" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/b86f565e-b575-4dad-ab96-bf01e6870fa7">

뷰를 사용해서 둘러싸는 container 의 container background 를 설정합니다.

***Parameters***

- **alignment :**The alignment that the modifier uses to position the implicit ZStack that groups the background views. The default is center.
- **container :**The container that will use the background.
- **content :** The view to use as the background of the container.

***Discussion***

해당 modifier 는 전체 parent container 를 자동으로 채우는 점에서 [background(_:ignoresSafeAreaEdges:)](https://developer.apple.com/documentation/swiftui/view/background(_:ignoressafeareaedges:)) 와 다릅니다.

- 적용해 보겠습니다!

```swift
if #available(iOSApplicationExtension 17.0, *) {
    Image("widgetEmpty")
         .resizable()
         .scaledToFill()
         .containerBackground(for: .widget) {
             // ...    
         }
} else {
     Image("widgetEmpty")
         .resizable()
         .scaledToFill()
}
```

이제야 프리뷰에서 에러 없이 확인할 수 있게 되었습니다.

**background 를 사용하지 않는 경우라도 containerBackground(for:) modifier 를 사용해야 에러를 해결할 수 있었습니다.**

<img width="250" alt="4" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/887dd517-a025-4cca-a6b9-1c7042a2666e">

🫢 ??? 저 여백은 뭐지?

## 🚨 트러블 슈팅

- **그런데 아직 완성은 아닙니다! 에러가 나지 않을 뿐이지 전에는 없던 padding 이 생겼습니다.**

많은 커뮤니티에서도 iOS 17 widget extra padding 에 대한 질문들이 올라와 있었습니다.

<img width="400" alt="5" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/5827ef77-0519-48eb-92b1-125632949f66">

## 👉 contentMarginsDisabled

[contentMarginsDisabled() | Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/widgetconfiguration/contentmarginsdisabled())

WWDC 23 > [Bring widgets to new places](https://developer.apple.com/videos/play/wwdc2023/10027/) 에서 **contentMarginsDisabled()** 를 사용해서 safe area 를 비활성화할 수 있다고 안내하고 있습니다.

개발자 문서를 살펴보겠습니다.

<img width="500" alt="6" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/7858f3bd-3d6a-4e4f-950e-859f93125881">

default content margins 이라고 표현하는 것을 봐서 기본적으로 설정되는 값을 비활성화하는 것 같습니다.

**Return Value :** 

default content margins 를 사용하지 않는 widget configuration 을 반환합니다.

**Discussion :**

content margins 을 비활성화하면 위젯의 content 주위에 마진을 자동으로 추가하지 않습니다. custom margins 을 지정하기 위해서 [widgetContentMargins](https://developer.apple.com/documentation/swiftui/environmentvalues/widgetcontentmargins) 를 padding() 과 함께 사용할 수 있습니다.

해당 modifier 는 iOS 17, watchOS 10, macOS 14 이전의 버전에서는 영향을 미치지 않습니다.

*(즉, 별도의 if #available 구문을 사용하지 않아도 됩니다!)*

---

**반환 값이 WidgetConfiguration 입니다.** 잠시 코드를 살펴보겠습니다.

containerBackground(for:) 처럼 View 를 반환하는 것이 아닌 **WidgetConfiguration** 을 반환하기 때문에 사용하는 위치가 다릅니다.

```swift
struct ExampleEntryView: View {
    // ...

    var body: some View {
        if #available(iOSApplicationExtension 17.0, *) {
            Text("Example")
                // ...
                // ✅
                .containerBackground(for: .widget) { }
        } else {
            Text("Example")
                // ...
    }
}

struct ExampleWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind,
                            provider: Provider()) { entry in
            ExampleEntryView(entry: entry)
        }
        .configurationDisplayName("...")
        .description("...")
        .supportedFamilies([.systemSmall])
        // ✅
        .contentMarginsDisabled()
        .containerBackgroundRemovable(false)
    }
}
// 🚨 preview 에서는 아직도 content margin 이 있는데요..???

// 👉 preview 에서는 `contentMarginsDisabled()` 를 사용하지 않았기 때문에 그런 것 같습니다.
struct ExampleWidget_Previews: PreviewProvider {
    static var previews: some View {
        // View 를 사용
        ExampleEntryView(entry: ExampleEntry(date: Date()))
            .previewContext(WidgetPreviewContext(family: .systemSmall))
    }
}
```

preview 에서 content margin 이 보이더라도 실제로 빌드해보면 content margin 이 사라짐을 확인 할 수 있습니다!

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/3cd5f56d-4461-4a8d-9aaa-e2556336e0fc" width ="200">
