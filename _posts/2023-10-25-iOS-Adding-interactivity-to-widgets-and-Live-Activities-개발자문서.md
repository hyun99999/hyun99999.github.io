 ---
title:  "iOS) Adding interactivity to widgets and Live Activities(개발자문서)"
categories:
- iOS

date:   2023-10-25  13:36:00 +0900
author_profile: false
---
### 내용

- interactive widget 에 대해서 알아보자
- 관련 개발자 문서, HIG 정리해보자

iOS 17과 iPad 17, macOS 14 부터 인터렉티브 위젯이 적용되었고 애플 공식 홈페이지에서 다음과 같이 소개하고 있습니다.

<img width="450" alt="1" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/730997d9-03f4-42b1-992b-19e6cf10d449">

<img width="450" alt="2" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/5ca369ef-f485-4d52-a05d-10a8de3ad5c9">

(출처: https://www.apple.com/kr/newsroom/2023/06/ipados-17-brings-new-levels-of-personalization-and-versatility-to-ipad/)

***개발자문서를 통해서 알아보겠습니다.***

[Adding interactivity to widgets and Live Activities](https://developer.apple.com/documentation/widgetkit/adding-interactivity-to-widgets-and-live-activities)

### Overview

iOS 17과 iPad 17, macOS 14 부터 Widget 과 Live Activities 에 특정 앱 기능을 제공하는 버튼과 토글이 포함될 수 있습니다. 예를 들어, 미리 알림 위젯을 사용하면 사람들이 토글을 사용하여 완료된 작업을 표시할 수 있습니다. locked device 에서 버튼과 토글들이 비활성화되어 잠금 해제되지 않는 이상 시스템이 작업을 수행하지 않습니다.

**Important**

버튼이나 토글과의 인터렉션은 앱을 여는 것 이상의 역할을 해야 합니다. 앱을 여는 인터렉션을 제공하기 위해서 [Link](https://developer.apple.com/documentation/SwiftUI/Link) 와 [widgetURL(_:)](https://developer.apple.com/documentation/SwiftUI/View/widgetURL(_:)) 을 사용할 수 있습니다. [Linking to specific app scenes from your widget or Live Activity](https://developer.apple.com/documentation/widgetkit/linking-to-specific-app-scenes-from-your-widget-or-live-activity)

다음의 위젯 크기에서 버튼과 토글이 포함될 수 있습니다.

- [WidgetFamily.systemSmall](https://developer.apple.com/documentation/widgetkit/widgetfamily/systemsmall)
- [WidgetFamily.systemMedium](https://developer.apple.com/documentation/widgetkit/widgetfamily/systemmedium)
- [WidgetFamily.systemLarge](https://developer.apple.com/documentation/widgetkit/widgetfamily/systemlarge)
- [WidgetFamily.systemExtraLarge](https://developer.apple.com/documentation/widgetkit/widgetfamily/systemextralarge)
- [WidgetFamily.accessoryCircular](https://developer.apple.com/documentation/widgetkit/widgetfamily/accessorycircular) on iPhone and iPad
- [WidgetFamily.accessoryRectangular](https://developer.apple.com/documentation/widgetkit/widgetfamily/accessoryrectangular) on iPhone and iPad

Live Activities 에는 확장된 그리고 Lock Screen presentation 에 버튼이나 토글을 추가할 수 있습니다.

디자인 지침은 다음을 참고할 수 있습니다. [Human Interface Guidelines > Widgets](https://developer.apple.com/design/human-interface-guidelines/components/system-experiences/widgets).

**Related session from WWDC23 :** [Session 10027: Bring widgets to new places](https://developer.apple.com/videos/play/wwdc2023/10027)

## WWDC23 Bring widgets to new places

올해부터 데스크탑과 아이패드의 잠금화면, 아이폰의 스탠바이 모드, 애플워치의 스마트 스택 총 4곳으로 확장됩니다.

<img width="500" alt="3" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/3776774d-834d-4728-bca0-7b7139eba065">

(출처: https://developer.apple.com/videos/play/wwdc2023/10027)

## HIG - Widgets

### [Adding interactivity to widgetsin page link](https://developer.apple.com/design/human-interface-guidelines/widgets#Adding-interactivity-to-widgets)

위젯을 탭하거나 클릭해서 앱을 실행할 수 있습니다. iOS 17, iPadOS 17, macOS 14 부터는 위젯은 버튼과 토글을 포함하여 앱의 실행없이 추가적인 기능을 제공할 수 있습니다.

예를 들어, 미리 알림 위젯을 사용해 작업을 완료로 표시할 수 있고, 일일 카페인 섭취를 기록하는 앱의 위젯에서 좋아하는 음료의 카페인 섭취를 늘리는 버튼이 포함될 수 있습니다.

<img width="450" alt="4" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/4702e92c-aad7-4eb6-ae41-353abdd5dc6c">

위젯은 위젯의 컨텐츠와 직접적으로 관련된 작업이나 동작을 완료하기 위해 집중적이고 구체적인 앱 기능을 제공합니다.

버튼이나 토글이 아닌 영역에서 위젯과 상호작용하면 앱이 실행됩니다. 위젯 콘텐츠와 관련된 세부 정보 및 작업을 제공하는 화면에 대한 deep link 를 사용하세요. 예를 들어, medium 주식 widget 을 클릭하거나 탭했을 때, 주식 앱은 해당 종목에 대한 정보를 표시하는 페이지가 열립니다.

한 눈에 알아볼 수 있고, 깔끔하게 유지하면서 상호작용을 위한 옵션을 제공하세요. 하지만, 앱과 유사한 레이아웃을 만드는 것은 피하세요. 대상의 크기에 주의를 기울이고, 사람들이 실수로 탭하거나 클릭할 수 있는지 확인해야 합니다. 

---

### Understand the role of app intents

위젯은 [App Intents](https://developer.apple.com/documentation/AppIntents) 프레임워크 와 SwiftUI 를 사용하여 앱과 직접적인 상호작용을 제공합니다. 예를 들어, 샘플 코드 프로젝트의 large widget 에는 버튼이 포함되어 있습니다.

위젯과 워치 complications 를 만들기 위해서 프로젝트에 widget extension 을 추가해야 하고, 앱과 독립적인 프로세스에서 실행됩니다. timeline entries 를 기반으로 시스템은 view representation 을 archive 하고 이러한 representations 을 사용하여 뷰만 렌더링합니다.

timeline 매커니즘과 별도의 프로세스에서의 렌더링으로 인해 시스템은 위젯을 렌더링할 때 코드를 실행하거나 데이터 바인딩을 업데이트할 수 없습니다.

App Intents 프레임워크를 사용해서 앱의 동작을 시스템에 노출하고, 필요할 때 작업을 수행할 수 있습니다.(위젯의 버튼 혹은 토글과 상호작용할 때)

**Note**

Live Activities 는 콘텐츠를 업데이트하기 위해 timeline 매커니즘을 사용하지 않지만, view archiving 과 decoding 주기가 유사한 WidgetKit 과 widget extension 을 사용합니다. 그래서 아래 방법의 위젯과 동일한 방식으로 Live Activity 에 버튼과 토글을 추가할 수 있습니다.

### Add an app intent that performs the action

widget 과 Live Activities 에 추가하는 버튼과 토글은 App Intents 프레임워크를 채택해서 시스템에 노출되는 기능을 사용합니다. 버튼이나 토글을 추가하기 전에 app intent 를 사용해서 기능을 사용할 수 있도록 설정할 수 있습니다.

1. 위젯의 경우, AppIntent 프로토콜을 채택하는 구조체를 생성하고 app target 에 추가합니다. Live Activity 같은 경우, LiveActivityIntent protocol 을 채택합니다.
2. 프로토콜의 요구사항을 구현합니다.
3. @Parameter 프로퍼티 래퍼를 사용하여 동작에 필요한 input parameter 를 정의하고 해당 타입이 [AppEntity](https://developer.apple.com/documentation/AppIntents/AppEntity) 프로토콜을 준수하는지 확인합니다. input parameter 에 값이 할당되어 있는지 확인해야 합니다. 왜냐하면, Siri 와 같은 시스템 기능에 대해 정의한 app intent 와 달리 위젯은 app intent 에 대한 매개변수를 해결하지 않습니다.
4. 프로토콜의 [preform()](https://developer.apple.com/documentation/AppIntents/AppIntent/perform()-2vmgc) 함수에서 위젯이 사용할 수 있도록 하려는 작업에 대한 코드를 추가합니다.

예를 들어, 사람들이 클릭 혹은 터치하여 hero 의 healing boost 를 주는 버튼이 large widget 에 있다고 해보겠습니다.

```swift
@available(iOS 16.0, macOS 13.0, watchOS 9.0, tvOS 16.0, *)
struct SuperCharge: AppIntent {
    
    static var title: LocalizedStringResource = "Emoji Ranger SuperCharger"
    static var description = IntentDescription("All heroes get instant 100% health.")
    
    func perform() async throws -> some IntentResult {
        EmojiRanger.superchargeHeros()
        return .result()
    }
}
```

LiveActivityIntent 또는 AudioPlaybackIntent 프로토콜을 채택하면 시스템은 앱 프로세스에서 app intent 를 실행합니다. app target 에 custom app intent 를 추가해야 합니다.

AppIntent 프로토콜을 채택하는 경우, widget extension target 과 app target 에 custom app intent 를 추가해야 합니다. 이를 통해 위젯에 추가한 버튼과 토글을 사용할 수 있습니다.

### Implement the perform function

사람들이 위젯의 버튼이나 토글과 상호작용하면 시스템은 앱 프로세스에서 **perform()** 함수를 실행합니다. 이 기능은 비동기식입니다. 결과적으로 Swift concurrency 를 최대한 활용할 수 있습니다. 

perform() 함수로부터 반환되면 시스템은 timeline provider 를 사용해서 위젯의 timeline 을 다시 로드합니다. 이를 인터렉션의 결과로 위젯을 업데이트하는 기회로 활용할 수 있습니다. perform() 반환되기 전에 timeline 업데이트에 필요한 코드가 실행되는지 확인해야 합니다.

예를 들어, 업데이트된 정보를 데이터베이스에 저장하는 동안 await 키워드를 사용하고 완료되면 perform() 에서 반환됩니다. 그런 다음 시스템은 위젯의 timeline 을 리로드하고 timeline provider 는 데이터베이스에서 업데이트된 데이터를 로드합니다.

**Note** : 버튼이나 토글은 항상 timeline 을 리로드하도록 보장합니다.

perfom() 함수는 throws 로 표시되어서 오류를 처리하여 앱, 위젯, 라이브 액티비티를 업데이트합니다. 예를 들어, 새로운 데이터를 로드할 수 없는 경우 오래된 정보를 표시한다고 사용자에게 말할 수 있습니다.

**Tip**

App Intents 프레임워크를 사용하여 Siri 와 Shorcuts 앱과 같은 시스템 수준의 서비스를 지원할 수 있습니다. 또한, app intents 를 사용하여 위젯을 configurable 하게 만들 수 있습니다. 위젯을 인터렉티브하게 만들면, 해당 인터렉션을 위한 시스템 서비스가 자동으로 지원됩니다. App Intents 프레임워크에 대해서 자세히 알아보려면 [App Intents](https://developer.apple.com/documentation/AppIntents) 와 [Providing your app’s capabilities to system services](https://developer.apple.com/documentation/AppIntents/Providing-your-app-s-capabilities-to-system-services)-services) 에서 볼 수 있습니다. configurable widgets 에 대해서는 [Making a configurable widget](https://developer.apple.com/documentation/widgetkit/making-a-configurable-widget) 에서 볼 수 있습니다.

### Add a button

app intent 를 구현한 후, [init(_:intent:)](https://developer.apple.com/documentation/SwiftUI/Button/init(_:intent:)-7urde) 와 같은 이니셜라이저를 사용해서 위젯에 버튼을 추가합니다.

**Tip**

위젯 또는 Live Activity 만을 위해서 버튼을 구현하지 않아도 됩니다. app intent implementation 을 포함하여 위젯에 버튼을 추가하는 코드를 앱에서 재사용할 수 있습니다. 

---

다음의 예시는 위젯이 hero 에게 healing boost 를 주는 버튼을 포함하고 있습니다. iOS 17이 있는 확인하는 방법에 유의해야 합니다.(if #available(iOS 17.0, *) 분기처리를 말함)

```swift
struct EmojiRangerWidgetEntryView: View {
    var entry: SimpleEntry
    
    @Environment(\.widgetFamily) var family
    
    @ViewBuilder
    var body: some View {
        switch family {
        case .systemLarge:
            VStack {
                HStack(alignment: .top) {
                    AvatarView(entry.hero)
                        .foregroundColor(.white)
                    Text(entry.hero.bio)
                        .foregroundColor(.white)
                }
                .padding()
                if #available(iOS 17.0, *) {
                    HStack(alignment: .top) {
                        // ✅ AppIntent 프로토콜을 채택하는 구조체 SuperCharge.
                        Button(intent: SuperCharge()) {
                            Image(systemName: "bolt.fill")
                        }
                    }
                    .tint(.white)
                    .padding()
                }
            }
            .containerBackground(for: .widget) {
                Color.gameBackgroundColor
            }
            .widgetURL(entry.hero.url)
            
        // Code for other widget sizes.
    }
}
```

버튼을 추가한 후에 인터렉션할 때 데이터를 변경한 뷰를 검토합니다. 예를 들어, 버튼을 누르면 몇가지 텍스트가 위젯에 리로드 될 수 있습니다. Mac 에서 iPhone 과 상호작용하는 경우 리로드에 시간이 걸릴 수 있습니다.

위젯이 콘텐츠의 업데이트를 기다리고 있음을 사용자에게 알리려면 업데이트 된 데이터를 수신하는 뷰에서 [invalidatableContent(_:)](https://developer.apple.com/documentation/SwiftUI/View/invalidatableContent(_:)) modifier 를 추가할 수 있습니다. 변경될 수 있는 모든 뷰에 사용하지 말고 중요한 뷰에 사용해야 합니다.

**Note :** 버튼은 사람이 상호작용했지만 누른 상태를 유지하지 않는 것을 의미합니다. Toggle 을 사용해서 on/off 기능을 나타낼 수 있습니다.

### Add a toggle

[init(isOn:intent:label:)](https://developer.apple.com/documentation/SwiftUI/Toggle/init(isOn:intent:label:)) 와 같은 이니셜라이저를 사용해서 뷰에 토글을 추가합니다.

다음 예시는 widget 에서 toggle 을 추가하는 것을 보여줍니다. 이니셜라이저의 **isOn** 파라미터는 Binding<Bool> 값 대신 Boolean 값을 받습니다.

```swift
struct TodoItemView: View {
    var todo: Todo

    var body: some View {
        Toggle(isOn: todo.complete, intent: ToggleTodoIntent(todo.id)) {
            Text(todo.body)
        }
        .toggleStyle(TodoToggleStyle())
    }
}
```

perform() 함수는 코드를 비동기적으로 실행하며 예를 들어, Mac 의 iPhone 위젯에서 상호작용을 수행하는 경우 시간이 걸릴 수 있습니다.

Toggle 은 appearance 를 수행된 작업의 결과를 기다리지 않고 즉시 새로운 상태를 나타냅니다.

변경된 상태를 정확하게 반영하려면 perform() 함수에서 실행한 코드 결과로 위젯을 업데이트하면 됩니다. 예를 들어, 사용자가 토글을 탭하는 경우 토글은 작업이 완료된 것으로 즉시 표시합니다. perform() 로직이 여전히 실행중이더라도.

위젯이 실제로 완료된 작업만 표시하도록 하기 위해 perform() 구현을 다음처럼 시작할 수 있습니다.

1. 위젯이 앱과 공유하는 데이터베이스에서 작업을 완료된 것으로 표시
2. 코드를 실행해서 데이터베이스를 서버와 동기화
3. 서버가 성공적인 동기화를 알려주면 위젯의 timeline 을 올바른 상태로 새로고침 합니다. Toggle 은 예측이 아닌 올바른 상태를 표시해야 합니다.
4. 오류 또는 서버에서 timeout 이 발생하는 경우 위젯이 작업이 완료되지 않음을 반영해야 합니다. 예를 들어, toggle 의 isOn 프로퍼티를 false 로 설정하고 위젯에 동기화 에러 텍스트를 보여줘야 합니다.

### Review interactions in iPhone widgets on Mac

Mac 의 iPhone 위젯과 관련하여 위젯에 버튼을 추가하는 경우 뷰에 invalidatableContent(_:) modifier 를 사용하여 Toggle 의 낙관적인 동작에 대해서 이해하는 것이 중요합니다.

iOS 17 과 macOS 14 부터 사용자는 Mac 데스크탑과 notification center 에 iPhone 위젯을 배치할 수 있습니다. 사람이 Mac 에서 인터렉션하면 시스템은 iPhone 으로 인터렉션을 보냅니다. iPhone 에서 시스템은 intent 를 수행하고 새로운 timeline 을 생성하고 업데이트된 timeline 을 Mac 의 iPhone 위젯으로 보냅니다.

이 프로세스는 추가적인 시간이 좀 걸릴 수 있습니다. 따라서 업데이트된 데이터를 기다리는 뷰를 표시하고 토글이 동기화된 데이터를 반영하는지 확인하는 것이 중요합니다.
