---
title:  "iOS) containerBackgroundRemovable(_:) 알아보기(feat. iPad lock screen)"
categories:
- iOS

date:   2023-11-07  23:06:00 +0900
author_profile: false
---
아래 글에서 iOS 17부터 **containerBackground(for:)** modifier 를 통해서 container background 을 설정해주어야 한다고 했어요.

[iOS) iOS 17 Widget error- Please adopt containerBackground API 해결하기](https://gyuios.tistory.com/306)

**그렇다면 아래와 같이 딱히 background 가 필요하지 않는 경우에는 어떻게 할까요?**

<img width="250" alt="1" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/8adfc3cb-2301-4582-8ff0-18cea7ed7d83">


(출처: WWDC 23 > [Bring widgets to new places](https://developer.apple.com/videos/play/wwdc2023/10027/)

- 그때는 containerBackgroundRemovable(_:) 를 사용하면 되는데 코드를 살펴보겠습니다.

```swift
// ...
// iOS 17부터는 이 modifier 가 무조건적으로 들어가야 한다.
// StandBy 모드 등 사용 장소에 따라서 background 를 대응하기 위해서 iOS 17 부터 적용
    .containerBackground(for: .widget) {
        Color.white
    }
}

struct MyCardWidget: Widget {
    var body: some WidgetConfiguration {
        IntentConfiguration(kind: "...",
                            intent: MyCardIntent.self,
                            provider: MyCardProvider()) { entry in
            MyCardEntryView(entry: entry)
        }
        .configurationDisplayName("...")
        .description("...")
        .supportedFamilies([.systemSmall])
        // ✅ 지도, 사진 등 뚜렷한 foreground 콘텐츠가 없다면 제거할 background 가 없습니다.
        // 이런 경우에 false 로 설정해서 사용.
        .containerBackgroundRemovable(false)
    }
}
```

개발자 문서를 통해서 해당 modifier 를 알아보겠습니다.

### containerBackgroundRemovable(_:)

[containerBackgroundRemovable(_:) | Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/widgetconfiguration/containerbackgroundremovable(_:))

<img width="500" alt="2" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/71c0717a-2c51-4beb-8452-51dc78856c03">

isRemovable 가 true 인 경우 background 를 선호하지 않는 context 에서 container background 의 제거를 지원합니다. false 인 경우 시스템이 background 를 지우지 않습니다.

true 기본값 파라미터를 가집니다.

WidgetConfiguration 을 반환합니다.

**Discussion :**

대부분의 경우, 가능한 많은 context 에 위젯을 배치할 수 있도록 위젯의 background container 를 제거할 수 있도록 합니다. nonremovable 로 설정하면 다양한 상황에서 위젯이 부적격해집니다. **예를 들어, removable background 가 없는 small widget 은 iPad lock screen 에 표시되지 않습니다.(아래에서 확인해보겠습니다.)**

background 를 nonremovable 로 설정하면 시스템은 항상 위젯의 background container 를 표시합니다. 이때 background 는 색이 faded 또는 desaturated(채도가 낮다) 해보일 수 있습니다.

이 modifier 는 iOS 17, watchOS 10, macOS 14 이전의 버전에는 영향을 미치지 않습니다.

---

.**containerBackgroundRemovable(true)** 로 설정했을 때 대표적인 예시는 날씨 위젯입니다.

<img width="600" alt="3" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/a9beeac6-9434-4a4b-b1e1-74bf13ea4021">

(출처: WWDC 23 > [Bring widgets to new places](https://developer.apple.com/videos/play/wwdc2023/10027/)

iPhone lock screen 에서는 accessory widget 형태들만 잠금화면에서 노출되었습니다. 그런데 samll widget 에 removable background 를 설정하면 iPad lock screen 에 노출된다는데 확인해보겠습니다.

### 👉 iPad Lock Screen small widget 살펴보기

- 시뮬레이터를 iPad 로 설정하게되면 “Canvas Device Settings” 에서 Small Lock Screen 을 설정할 수 있었습니다.

<img width="350" alt="4" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/4173a11c-a737-457c-a119-f37de74bbc62">

containerBackground(for:) 로 background 를 설정하고 containerBackgroundRemovable() modifier 로 **removable background** 로 설정해보겠습니다.

```swift
struct EmptyMyCardView: View {

    // ...
    
    var body: some View {
// ✅ background 를 containerBackground(for:) 로 대체
//        ZStack {
//            Color.white
            VStack() {
                Text("아직 내 명함이 없어요😥")
                    // ...
                HStack {
                    Text("명함 만들러 가기")
                        // ...
                }
            }
//      }
            .containerBackground(for: .widget) {
                Color.white
            }
    }
}

struct EmptyMyCardView: Widget {
    var body: some WidgetConfiguration {
        IntentConfiguration(kind: "...",
                            intent: MyCardIntent.self,
                            provider: MyCardProvider()) { entry in
            EmptyMyCardView(entry: entry)
        }
        // ...
        // ✅
        .containerBackgroundRemovable(false)
    }
}
```

주석처럼 ZStack 을 사용해서 background 를 설정하면(containerBackground(for:) 에서 아무것도 설정하지 않음) 다음과 같이 부적합한 위젯이 만들어졌습니다.

<img width="200" alt="5" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/2e72239f-017c-4c31-bf50-b1578f07bda7">

containerBackground(for:) 로 background 을 적용해보았고, 적합한 위젯이 만들어졌습니다.

<img width="200" alt="6" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/86a13b5d-59eb-49c7-96dc-b8413ad476cf">
