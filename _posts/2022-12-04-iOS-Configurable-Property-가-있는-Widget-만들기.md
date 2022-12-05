---
title:  "iOS) Configurable Property 가 있는 Widget 만들기"
categories:
- iOS

date:   2022-12-04  16:54:00 +0900
author_profile: false
---
### 👉 Configurable Widget 만들기

- 첫 번째가 `StaticConfiguration` 이고, 두 번째가 `IntentConfiguration` 입니다. `위젯 편집` 을 통해서 세 번째처럼 사용자에게 위젯의 옵션을 설정하게 할 수 있습니다.

<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/205479593-de0c2964-53f9-4c01-bb37-027ad0e020ad.png">

**아래의 개발자 문서를 참고하여서 진행해 보겠습니다.**

[Apple Developer Documentation - Making a Configurable Widget](https://developer.apple.com/documentation/WidgetKit/Making-a-Configurable-Widget)

# 👉 [Making a Configurable Widget](https://developer.apple.com/documentation/WidgetKit/Making-a-Configurable-Widget)

프로젝트에 custom `SiriKit intent definition` 을 추가하여 사용자가 커스터마이즈 할 수 있는 옵션을 제공할 수 있습니다.

### Overview

사용자가 가장 관련성이 높은 정보에 쉽게 접근할 수 있도록 위젯은 customizable properties 를 제공합니다. 예를 들어, 사용자는 주식 시세 위젯의 특정 주식을 선택할 수 있거나 패키지 배송 위젯에 대한 추적 번호를 입력할 수 있습니다. 

위젯은 `custom intent definitions` 을 사용하여 customizable properties 를 정의합니다. `custom intent definitions` 을 사용하는 것은 Siri Suggestions(제안) 와 Siri Shortcuts(단축어)가 이러한 상호작용을 사용자 정의하는데 사용하는 것과 동일한 메커니즘입니다.

**configurable properties 를 위젯에 추가하기 위해서:**

1. Xcode 프로젝트에 configurable properties 를 정의하는 custom intent definition 을 추가합니다.
2. 위젯에서 [IntentTimelineProvider](https://developer.apple.com/documentation/widgetkit/intenttimelineprovider) 를 사용하여 사용자의 선택을 timeline entries 에 통합합니다.
3. 속성이 동적 데이터에 의존하는 경우, Intents extension 을 구현 합니다.

앱이 이미 Siri Suggestions 또는 Siri Shortcuts 를 지원하고 custom intent 를 가진 경우, 이미 대부분의 작업을 완료했을 것입니다.

그렇지 않으면, Siri Suggestions 또는 Siri Shortcuts 지원을 추가하는 것이 좋습니다. intent 를 최대로 활용하는 방법에 대한 자세한 내용은 [SiriKit](https://developer.apple.com/documentation/sirikit) 을 참조하세요.

다음의 섹션에서는 게임에서 캐릭터에 대한 정보를 표시하는 위젯에 configurable property 를 추가하는 방법을 안내합니다.

## ✅ Add a Custom Intent Definition to Your Project

Xcode 프로젝트에서 **File > New File > SiriKit Intent Definition File** 을 선택하고, `.intentdefinition` 파일을 생성하여 프로젝트에 추가합니다.

<img width="500" alt="2" src="https://user-images.githubusercontent.com/69136340/205479641-c3b17256-9a20-4185-8a3c-b824c0da2041.png">

target 에 파일의 코드를 사용하기 위해서 다음을 수행해주어야 합니다.

- intent definition files 을 target 의 멤버로 포함합니다.
- 특정 intent 를 나타내기 위해서 지원되는 intent 에 대해서 클래스 이름을 설정하여  target 에 추가해줍니다.

이 파일을 framework 에 추가할 때, containing app 의 타겟도 반드시 포함해야 합니다. 이 경우에, 앱과 프레임워크에서 타입이 중복되지 않도록 **File inspector** 에서 **Target Membership section** 에서 app target 에 대해 `No Generated Classes` 를 선택합니다.

<img width="250" alt="3" src="https://user-images.githubusercontent.com/69136340/205479669-407055a2-1901-47b7-a41a-5913cb831699.png">

**사용자가 게임에서 캐릭터를 선택할 수 있도록 하는 custom intent 를 추가하고 구성하기 위해서 :**

1. **Project Navigator** 에서 intent 파일을 선택하면 Xcode 가 비어있는 intent definition editor 를 보여줍니다.
2. **Editor > New Intent** 를 선택하고, Custom Intents 하위의 intent 를 선택합니다.
3. `SelectCharacter` 로 **custom intent** 의 이름을 변경해줍니다. **Attribute Inspector** 의 **Custom Class field** 에 코드에서 intent 를 참조할 때 사용하는 클래스 이름이 표시됩니다. 이 경우에는, `SelectCharacterIntent` 입니다.
4. **Category** 를 `View` 로 설정하고, 위젯이 intent 를 사용할 수 있음을 나타내기 위해서 **Intent is eligible for widgets** 체크박스를 선택합니다.
5. **Parameter** 에서 위젯의 configurable setting 이 되는 `character` 이름을 새로운 파라미터로 추가합니다.

<img src="https://user-images.githubusercontent.com/69136340/205479677-24af1563-546f-4d61-861a-3aa7fad5bec6.png" width ="700">

parameter 를 추가한 후, 세부정보를 구성합니다.

### 👉 정적/동적 선택 목록 만들기

 ✅ **만약 매개변수가 사용자에게 정적인 선택 목록을 제공하는 경우, pop-up menu 에서`Add Enum...` 를 선택해서 static enumeration 을 생성해주면 됩니다.**

<img width="600" alt="5" src="https://user-images.githubusercontent.com/69136340/205479701-535fe807-eb10-4031-9579-e517ea55febb.png">

<img width="700" alt="6" src="https://user-images.githubusercontent.com/69136340/205479706-631270f4-7f60-45cd-8efb-694a7b6cd26e.png">

예를 들어, parameter 가 캐릭터의 아바타를 지정하고, 가능한 아바타 목록이 변경되지 않는 상수 집합의 경우에 intent definition file 에서 static enumeration 을 사용할 수 있습니다.

✅ **가능한 아바타 목록이 다양하거나 동적으로 생성되는 경우, dynamic options 가 있는 type 을 대신 사용할 수 있습니다.**

**예를 들어, 캐릭터 속성은 앱에서 사용할 수 있는 캐릭터의 동적 목록에 의존합니다. 동적 데이터를 제공하기 위해서 새로운 type 을 만들어야 합니다. :**

1. Type pop-up menu 에서 **Add Type** 을 선택하면 Xcode 는 에디터의 Types 섹션에 새 type 을 추가합니다.

<img width="700" alt="7" src="https://user-images.githubusercontent.com/69136340/205479745-a1f176fc-798e-4f4e-95a7-cdb38aaffba3.png">

2. type 의 이름을 `GameCharacter` 로 변경해줍니다.

<img src="https://user-images.githubusercontent.com/69136340/205479753-0dfb6404-5938-44c6-97ee-82c16f7ec8fb.png" width ="700">

3. `name` property 를 새롭게 추가해주고, Type pop-up menu 로부터 `String` 을  선택해줍니다.

<img width="700" alt="9" src="https://user-images.githubusercontent.com/69136340/205479777-29bb457e-d0f3-4da0-b265-9c0bee717a27.png">

4. `SelectCharacter` intent 를 선택해줍니다.
5. intent 에디터에서 **Options are provided dynamically** 체크박스를 선택하여 코드가 해당 파라미터에 대해서 동적인 목록을 제공함을 나타냅니다.

<img width="700" alt="10" src="https://user-images.githubusercontent.com/69136340/205479799-79cf030b-9300-4e6b-a80a-84ac31cbc5f2.png">

`GameCharacter` 타입은 사용자가 선택할 수 있는 캐릭터를 설명합니다. 다음 섹션에서는 캐릭터 목록을 동적으로 제공하는 코드를 추가해보겠습니다.

### 🗞 Note

intent 의 parameter 의 순서는 사용자가 위젯을 편집할 때 표시는 되는 순서를 결정합니다. 항목을 드래그해서 재정렬할 수 있습니다.

## ✅ Add an Intents Extension to Your Project

동적인 캐릭터 목록을 제공하기 위해 Intents extension 을 앱에 추가합니다. 사용자가 위젯을 편집하면, WidgetKit 은 Intents extension 을 불러와 동적인 정보를 제공합니다. 

**Intents extension 을 추가하기 위해서 :**

1. **File > New > Target** 에서 **Intents extension** 을 선택해준다.
2. Intents extension 의 이름을 입력하고 **Starting Point** 를 `None` 을 설정한다.

<img width="600" alt="11" src="https://user-images.githubusercontent.com/69136340/205479843-381fe5c7-2713-4f40-ba2b-454748bc9ed6.png">

3. 마치면, Xcode 의 새로운 scheme 를 활성화하라는 메시지가 표시됩니다. Activate(활성화)해줍니다.
4. 새로운 target 속성의 **General** tab 에서 **Supported Intents** 섹션에 entry 를 추가하고 **Class Name** 을 `SelectCharacterIntent`**(Custom Class field 적혀있던)** 로 설정합니다.

<img width="700" alt="12" src="https://user-images.githubusercontent.com/69136340/205479858-84d63400-9a40-4403-9749-4c3bdbfa8fdc.png">

5. **Project navigator** 에서 이전에 추가한 custom intent definition file 을 선택합니다.
6. **File Inspector** 를 사용하여 **Intents extension target** 에 definition file 을 추가해줍니다.

(아래 예시와 같이 Target Membership 에서 containing app(포함하는 앱), widget extension, Intents extension 모두 체크해주면 됩니다.)

<img src="https://user-images.githubusercontent.com/69136340/205479869-0d5abe90-74e9-4435-b564-6a17555682d6.png" width ="700">

### ❗️ Important

File inspector 에서 **containing app(포함하는 앱), widget extension, Intents extension** 모두 intent definition file 을 포함하는지 확인합니다.

## ✅ Implement an Intent Handler to Provide Dynamic Values

사용자가 동적 값을 제공하는 custom intent 로 위젯 편집할 때, 시스템은 해당 값을 제공하는 개체가 필요합니다. intent 에 대한 handler 를 제공하도록 **Intents extension** 을 요청하여 개체를 식별하도록 합니다.

Xcode 가 **Intent extension** 을 만들 때, `IntentHandler` 클래스를 포함하는 `IntentHandler.swift` 파일을 프로젝트에 추가합니다. **이 클래스에는 handler 를 반환하는 메서드가 포함되어 있습니다. 이 핸들러를 확장해서 위젯의 사용자 정의 값을 제공할 수 있습니다.**

custom intent definition file 을 기반으로 Xcode 는 핸들러가 준수해야만 하는 `SelectCharacterIntentHandling` 프로토콜을 생성합니다.(`SelectCharacterIntent` 를 생성했기 때문)IntentHandler 클래스의 선언에 아래와 같이 추가해줍니다.

```swift
class IntentHandler: INExtension, SelectCharacterIntentHandling {
    ...
}
```

핸들러가 동적 옵션을 제공할 때(이전에 체크했던 **Options are provided dynamically** 옵션), `provide[Type]OptionsCollection(for:with:)` 메서드를 구현해야 합니다. 여기서 `[Type]` 은 intent definition file 의 custom type 이름입니다.

- ***준수하라는 오류 메시지의 fix 를 하면 stub 함수를 만들어주는데, 탈출 클로저를 사용하는 함수와  async/await 를 사용하는 함수 두 가지로 준수할 수 있다.***

### 🚨 트러블 슈팅 - resolveCharacter(for:with:)?

- 이때 `provide[Type]OptionsCollection(for:with:)` 외에도 `resolve[Type](for:with:)` 메서드를 구현하라고 합니다.

<img width="700" alt="14" src="https://user-images.githubusercontent.com/69136340/205479910-6bc8f28f-cd98-4749-992c-b6e3f9b4ffe0.png">

- 이는 아래의 Resolvable 를 체크해서 생기는 필수 구현 메서드입니다. 해제해주면 됩니다.

<img src="https://user-images.githubusercontent.com/69136340/205479922-6ccf9988-8099-4de9-98a3-9dad46da593b.png" width ="700">

**(문서로 돌아가서 살펴보겠습니다. 이하 설명은 탈출 클로저를 가진 함수에 대해서 이야기하고 있습니다.)**

이 메서드는 completion handler 를 포함하고 있고, `INObjectCollection<GameCharacter>` 를 전달합니다. `GameCharacter` 타입은 intent definition file 에서의 custom type 입니다. Xcode 는 다음과 같이 정의되는 코드를 생성합니다.

```swift
public class GameCharacter: INObject {
    @available(iOS 13.0, macOS 11.0, watchOS 6.0, *)
    @NSManaged public var name: String?
}
```

`name` 속성은 intent definition file 에서 사용자가 추가한 custom type 입니다.

**`provideCharacterOptionsCollection(for:with:)` 메서드를 구현하기 위해 위젯은 게임의 프로젝트에 있는 구조체를 사용합니다.(타겟은 containing app 과 Intent extension 을 체크합니다.)** 이 구조는 다음과 같이 사용 가능한 캐릭터 목록과 해당 세부사항을 정의합니다.

```swift
struct CharacterDetail {
    let name: String
    let avatar: String
    let healthLevel: Double
    let heroType: String

    static let availableCharacters = [
        CharacterDetail(name: "Power Panda", avatar: "🐼", healthLevel: 0.14, heroType: "Forest Dweller"),
        CharacterDetail(name: "Unipony", avatar: "🦄", healthLevel: 0.67, heroType: "Free Rangers"),
        CharacterDetail(name: "Spouty", avatar: "🐳", healthLevel: 0.83, heroType: "Deep Sea Goer")
    ]
}
```

intent handler 에서 코드는 `availableCharacters` 배열을 반복하여 각 캐릭터에 대한 `GameCharacter` 객체를 생성합니다. `GameCharacter` 의 **identity** 는 `name` 으로 하겠습니다.

```swift
class IntentHandler: INExtension, SelectCharacterIntentHandling {
    func provideCharacterOptionsCollection(for intent: SelectCharacterIntent, with completion: @escaping (INObjectCollection<GameCharacter>?, Error?) -> Void) {

        // Iterate the available characters, creating
        // a GameCharacter for each one.
        let characters: [GameCharacter] = CharacterDetail.availableCharacters.map { character in
            let gameCharacter = GameCharacter(
                identifier: character.name,
                display: character.name
            )
            // ✅ 이때 name 은 custom type 에서 추가해준 속성이다. 
            gameCharacter.name = character.name
            return gameCharacter
        }

        // 👉 Create a collection with the array of characters.
        let collection = INObjectCollection(items: characters)

        // 👉 Call the completion handler, passing the collection.
        completion(collection, nil)
    }
}
```

- 이때 `GameCharacter` 클래스의 convenience initializer 를 사용하여 인스턴스화 해줍니다.

<img width="400" alt="16" src="https://user-images.githubusercontent.com/69136340/205479968-e5686d32-974d-47a9-baa4-4e85386fbfc9.png">

- ✅ **주석 -** 동적인 목록을 만들어주기 위해 생성한 type 인 GameCharacter 을 기억해내보자.

<img width="600" alt="17" src="https://user-images.githubusercontent.com/69136340/205479978-861e0858-0790-4638-ab07-22f2a861f09b.png">

intent definition file 구성이 완료되고, Intents extension 이 앱에 추가되면, 사용자는 표시할 특정 캐릭터를 선택할 수 있습니다. **WidgetKit 은 intent definition file 의 정보를 사용하여 자동으로 위젯 편집을 위한 유저 인터페이스를 생성합니다.**

**사용자가 위젯을 편집하고 캐릭터를 선택하면, 다음 단계는 해당 선택을 위젯의 표시에 통합하는 것입니다.**

## ✅ Handle User-Customized Values

configurable properties 를 지원하기 위해서 위젯은 **IntentTimelineProvider** configuration 을 사용합니다. 예를 들어, CharacterDetail 위젯은 다음과 같이 configuration 을 정의합니다.

(**IntentTimelineProvider** 를 사용하기 위해서는 위젯을 만들 때 configurable 하게 만드는 체크박스를 선택하면 된다. 이를 선택함으로써 **TimelineProvider** 에서 **IntentTimelineProvider** 로 코드가 만들어진다.)

```swift
struct CharacterDetailWidget: Widget {
    var body: some WidgetConfiguration {
        IntentConfiguration(
            kind: "com.mygame.character-detail",
            // ✅ intent 설정.
            intent: SelectCharacterIntent.self,
            provider: CharacterDetailProvider(),
        ) { entry in
            CharacterDetailView(entry: entry)
        }
        .configurationDisplayName("Character Details")
        .description("Displays a character's health and other details")
        .supportedFamilies([.systemSmall, .systemMedium, .systemLarge])
    }
}
```

`SelectCharacterIntent` 파라미터는 위젯에 대한 사용자 정의 가능한 속성을 결정합니다. configuration 은 `CharacterDetailProvider` 를 사용하여 위젯의 timeline events 를 관리합니다. timeline providers 에 대한 자세한 내용은 [Keeping a Widget Up To Date](https://developer.apple.com/documentation/widgetkit/keeping-a-widget-up-to-date) 를 참조하세요.

사용자가 위젯 편집을 한 후, WidgetKit 은 timeline entries 를 요청할 때 사용자 정의 값을 제공자에게 전달합니다. 일반적으로 provider 가 생성하는 timeline entries 에 intent 의 관련 세부정보를 포함합니다.

예를 들어, provider 는 helper 메서드(이 예제에서는 `lookupCharacterDetail(for:)`)를 사용해서 intent 안의 캐릭터의 이름을 사용해서 CharacterDetail 을 조회한 다음 캐릭터의 세부정보를 포함하는 entry 가 있는 timeline 을 생성합니다.

```swift
struct CharacterDetailProvider: [IntentTimelineProvider](https://developer.apple.com/documentation/WidgetKit/Making-a-Configurable-Widget) {
    // ✅ 우리가 만들어준 intent definition file 의 custom class 타입의 configuration 을 사용한다.
    func getTimeline(for configuration: SelectCharacterIntent, in context: Context, completion: @escaping (Timeline<CharacterDetailEntry>) -> Void) {
        // ✅ Access the customized properties of the intent.
        let characterDetail = lookupCharacterDetail(for: configuration.character.name)

        // ✅ Construct a timeline entry for the current date, and include the character details.
        let entry = CharacterDetailEntry(date: Date(), detail: characterDetail)

        // ✅Create the timeline and call the completion handler. The .never reload 
        // policy indicates that the containing app will use WidgetCenter methods 
        // to reload the widget's timeline when the details change.
        let timeline = Timeline(entries: [entry], policy: .never)
        completion(timeline)
    }
}
```

### 👉 결과

지금까지 정적/동적 선택목록을 구현하는 방법을 살펴보았는데 프로젝트에 적용해서 확인해보겠습니다.

- 정적

static enumeration 을 다음과 같이 구현하였습니다.

<img width="700" alt="18" src="https://user-images.githubusercontent.com/69136340/205480068-94653533-5582-4f81-bfea-ca590a6cad21.png">

(unknown 은 지울 수 없는 기본값입니다. 그래서 아래 구현 화면에 보시면 맨 처음에 선택이 되지 않는데 이를 위한 case 같습니다.)

<img width="500" alt="19" src="https://user-images.githubusercontent.com/69136340/205480071-a3a3bcce-ecef-492f-a0a5-75052b7d73a5.png">

- 동적

containing app 에서 만든 모델을 통해 동적으로 생성할 수 있습니다.

<img width="500" alt="20" src="https://user-images.githubusercontent.com/69136340/205480075-4207b617-2a4f-4820-b100-7ab1d50dc77b.png">

이를 프로젝트에 적용하는 자세한 예제는 아래의 글에서 진행했습니다.

**_링크_**

## ✅ Offer Preconfigured Complications on Apple Watch

watchOS 9 및 iOS 16 부터 WidgetKit 을 사용하여 Apple Watch 컴플리케이션으로 표시되는 accessory family 를 구현할 수 있습니다. iOS 와 macOS 의 위젯과 마찬가지로, watch complications 는 user configurable data 를 표시하기 위해서 custom intent 를 사용합니다. 또한, watchOS 에서 구현한 configurable widgets 는 iOS 또는 macOS 에서 동일하게 작동합니다.

그러나 위에서 살펴본 것과 달리 watchOS 는 컴플리케이션 구성을 위한 전용 유저 인터페이스를 제공하지 않습니다. watch complication 에서 사용자와 관련된 데이터를 표시하려면, preconfigured complications 를 생성하고 사용 가능한 complications 의 목록을 사용자에게 추천할 수 있습니다.

**`TimelineProvder` 코드에서, [recommendations()](https://developer.apple.com/documentation/widgetkit/intenttimelineprovider/recommendations()-5ltr5) 를 구현하고 custom intents 를 사용하여 만든 [IntentRecommendation](https://developer.apple.com/documentation/widgetkit/intentrecommendation) 객체를 반환합니다.**

(그 과정은 아래와 같습니다.)

앱이 recommended widget configurations 와 관련된 새로운 데이터를 수신하면, [invalidateConfigurationRecommendations()](https://developer.apple.com/documentation/widgetkit/widgetcenter/invalidateconfigurationrecommendations()) 메서드를 호출해서 현재 오래된 recommendations 을 무효화 합니다. 즉, WidgetKit 에게 새로운 recommended preconfigured configurations 를 가져오도록 합니다.

이때 recommended configurations 를 무효화하는 경우, `recommendations()` callback 에서 업데이트된 `IntentRecommendation` 객체를 반환해야 합니다.

### 👉 추가

**WWDC22) Complications and widgets: Reloaded 세션에서 `recommendations` 에 대한 소개를 잠깐 했습니다. 세션을 요약한 글을 아래에 첨부합니다. 🙂** 

**[WWDC22) Complications and widgets: Reloaded](https://gyuios.tistory.com/226)**

### 👉 총정리

- Custom Intent Definition(SiriKit Intent Definition file) 을 추가하고, 정적인 목록(enums)/동적인 목록(types)을 생성해준다.
- Intents extension 을 앱에 추가해서 사용자가 위젯을 편집할 때 Intents extension 을 불러와 정보를 제공할 수 있도록 한다. 이때 Intents definition file 을 containing app, widget extension, intents extension 모두에 추가해준다.
- 정적인 선택 목록은 Enum 으로, 동적인 선택 목록은 Type 으로 intent 의 paramerter type 을 설정해준다.
- 정적) Enum 을 설정하면 선택목록을 정적으로 구현할 수 있다.
- 동적) Intent extension 의 IntentHandler 를 확장하여 intent 에 대한 핸들러를 제공할 수 있다. 이때 intent definition file 과 관련된 intent handling 프로토콜을 준수하여 확장한다. `provide[Type]OptionsCollection(for:with:)` 메소드를 구현하여 intent handling 프로토콜을 준수할 수 있다.
- configurable properties 를 지원하기 위해 IntentTimelineProvider 를 사용하여 IntentConfiguration 을 구성해준다.
