---
title:  "iOS) KakaoQRcode 클론코딩 - Widget(1/2)"
categories:
- iOS

date:   2021-09-04  16:44:00 +0900
author_profile: false
---
### 내용

- 카카오톡에서 qrcode 를 위젯으로 제공하고 있다. 물론 홈으로 가져올 수도 있다. 귀엽다... 구현해보자

<img src ="https://user-images.githubusercontent.com/69136340/132086628-19a4b126-1d56-41a4-9298-3a265b16eabd.jpeg" width ="250">

# 😇Widget?!

Widget 은 프로토콜이다.

<img width="300" alt="스크린샷 2021-09-01 오후 5 05 20" src="https://user-images.githubusercontent.com/69136340/132086633-aedc70a3-668d-4304-b8fa-13360ab95fb7.png">

(히익!)

Home 화면(iOS)이나 Notification Center(macOS) 에 표시할 위젯의 구성 및 내용이다.

### Overview

위젯은 바로 앱의 관련 콘텐츠를 한눈에 볼 수 있도록 표시합니다. 사용자는 개별 요구 사항에 맞게 추가, 구성 및 정렬할 수 있다. 여러 유형의 위젯을 제공할 수 있다. 

위젯에는 세가지 주요 구성요소가 있다.

- configuration : 위젯이 구성할 수 있는지 여부를 결정하고, 위젯을 식별하고, SwiftUI 뷰를 정의한다.
- timeline provider : 시간이 지남에 따라 위젯의 보기를 업데이트하는 프로세스를 주도한다.
- SwiftUI View : 위젯을 보여주기 위해서 WidgetKit 을 사용한다.

앱에 widget extension 을 추가하고 최신상태로 유지하는 방법에 대해서 [Creating a Widget Extension](https://developer.apple.com/documentation/WidgetKit/Creating-a-Widget-Extension) 과 [Keeping a Widget Up To Date](https://developer.apple.com/documentation/WidgetKit/Keeping-a-Widget-Up-To-Date) 를 각각 참조해라.

커스텀 SiriKit 의 intent definition 을 추가하여 사용자가 가장 관련성이 높은 정보를 표시하도록 커스텀하게 할 수 있다. Siri 또는 Shortcuts에 대한 지원을 이미 추가했다면 사용자 지정 가능한 위젯을 지원하는 데 순조롭게 진행되고 있는 것입니다. 자세한 내용은 [Making a Configurable Widget](https://developer.apple.com/documentation/WidgetKit/Making-a-Configurable-Widget) 를 참조하십시오.

(**무슨 말...?** user-configurable 옵션을 제공해서 위젯을 커스텀할 수 있는 IntentConfiguration 위젯을 만드는 내용이다.)

# 😇 프로젝트 설정

### 1️⃣ Wdget Extension 추가

- 프로젝트 내에서 [File] → [New] → [Target]

<img width="600" alt="1" src="https://user-images.githubusercontent.com/69136340/132086743-63e85276-896c-4204-b013-4d7e5514099a.png">

- Widget Extenstion 추가

<img width="600" alt="2" src="https://user-images.githubusercontent.com/69136340/132086750-e70ec7a7-bff9-4873-9201-31164fed6b6d.png">

- target 의 이름을 설정해주면된다.

체크박스 체크 시 Intent Configuration 위젯을 생성한다. 내용에 대해서는 아래에서 더 알아보자

<img width="600" alt="3" src="https://user-images.githubusercontent.com/69136340/132086772-2511e769-8c74-4584-a5cd-d30c856fdcbe.png">

- Activate 해준다.

<img width="250" alt="3-1" src="https://user-images.githubusercontent.com/69136340/132086774-d27be32e-ceb1-42ad-a1ad-db78672129cd.png">


- 다음과 같은 폴더가 생성된다. 스위프트 파일에서 위젯에 대한 코드를 짜면 된다.

(Include Configuration Intent 체크박스 체크 시)

<img width="350" alt="3-2" src="https://user-images.githubusercontent.com/69136340/132086788-634f8b21-a9b0-44da-8cf5-adb53d8d216c.png">

(Include Configuration Intent 체크박스 체크 해제 시) 

.intentdefinition 파일이 없다.

<img width="275" alt="3-3" src="https://user-images.githubusercontent.com/69136340/132086796-2fb41a63-9452-46dd-849c-c2a7c1e340b4.png">

### 2️⃣ target 를 추가했으니 build 해서 확인해보자

- 사용가능한 Widget 으로 설정되었다.

<img src ="https://user-images.githubusercontent.com/69136340/132086814-fa289c54-840c-412e-9b94-56d7b0ed137e.png" width ="250">

- size 별로 확인해보자. 지금은 아무런 추가 설정하지 않았기 때문에 시간이 등장한다.

<img width="700" alt="스크린샷 2021-09-04 오후 4 33 51" src="https://user-images.githubusercontent.com/69136340/132086830-7dc06002-2596-4fa0-9de4-96635421fc07.png">

- 홈에도 추가해보겠다.

<img src ="https://user-images.githubusercontent.com/69136340/132086842-15722a41-d7e3-4467-add5-ff6c974c48a2.png" width ="250">

- Widget 은 SwiftUI 로 코드가 짜져있다. 그래서 canvas 를 통해서 프리뷰를 볼 수 있다.

<img width="800" alt="10" src="https://user-images.githubusercontent.com/69136340/132086847-57cda844-636d-4c7e-9097-edab0ed1584c.png">

# 😇 시작해보자!

앞서 Widget 은 프로토콜이라고 했다. 그렇다면 분명 Widget 을 채택하는 것이 있을 것이고 Required 프로퍼티가 있을 것이다.  바로 body 가 Required 프로퍼티에 해당한다.

## 📌 body

Widget 의 content 와 behavior 를 나타낸다.

그렇다면 기본적으로 구현된 body 를 뜯어보자

```swift
@main
struct KakakoWidget_iOS_CloneCoding: Widget {
    let kind: String = "KakakoWidget_iOS_CloneCoding"

    // ✅ Required 프로퍼티인 body
    var body: some WidgetConfiguration {
        IntentConfiguration(kind: kind, intent: ConfigurationIntent.self, provider: Provider()) { entry in
            KakakoWidget_iOS_CloneCodingEntryView(entry: entry)
        }
        // ✅ 타이틀
        .configurationDisplayName("My Widget")
        // ✅ 서브타이틀
        .description("This is an example widget.")
        // ✅ 위젯 크기(기본값은 세개 모두를 가진다.)
        .supportedFamilies([.systemSmall, .systemMedium, .systemLarge])

    }
}
```

아래의 사진과 비교해보면 어떤 항목에 해당하는지 알 수 있다.

- 앱 이름 : 1번 결정.
- kind : widget 의 indentity.
- configurationDisplayName : 2번 결정. 위젯을 추가하거나 편집할 때 표시되는 이름.
- description : 3번 결정. 위젯을 추가하거나 편집할 때 표시되는 설명.
- supportedFamilies : 좌측부터 .systemSmall / .systemMedium / .systemLarge

<img width="700" alt="11" src="https://user-images.githubusercontent.com/69136340/132086869-2eafbef1-5b98-4065-99a9-6831997db03e.png">

## ❓ some

```swift
var body: some WidgetConfiguration
```

아직 명확하지 않은 타입(opaque type)이라서 컴파일 과정에서 오류가 발생하게 되는데 그 오류를 없애기 위해서는 타입을 명확하게 만들어 줘야 하는데 그 역할을 해주는 것이 some 키워드이다. 라고 한다. 즉, 명확하지 않은 타입을 리턴해야하는 경우 사용한다.

**참고 :** 

[SwiftUI - some 키워드에 대한 정리](https://peachberry0318.tistory.com/18)

그래서 body 가 가지는 클로저의 리턴값은 IntentConfiguration 과 StaticConfiguration 모두 사용할 수 있다.

## ❗️[StaticConfiguration](https://developer.apple.com/documentation/widgetkit/staticconfiguration) & [IntentConfiguration](https://developer.apple.com/documentation/widgetkit/intentconfiguration)

- StaticConfiguration

: 사용자가 구성할 수 있는 옵션이 없는(no user-configurable) 위젯의 콘텐츠를 설명하는 개체.

예를 들어 아래처럼 위젯을 편집할 수 없는 경우.

<img src ="https://user-images.githubusercontent.com/69136340/132086901-2498ca35-90e8-4f18-8b65-597df28d286e.png" width ="250">

- IntentConfiguration

: 사용자 지정 의도 정의를 사용하여 사용자 구성 가능한 옵션을(user-configurable) 제공하는 위젯의 콘텐츠를 설명하는 개체.

예를 들어 아래처럼 위젯 편집 할 수 있는 경우.

<img src ="https://user-images.githubusercontent.com/69136340/132086905-6d761bc3-6979-467d-842a-62d98ef6d879.png" width = "250">

다음과 같이 위젯 편집해서 사용자가 구성 가능한 옵션을 제공할 수 있는 경우이다.

<img src ="https://user-images.githubusercontent.com/69136340/132086909-107b98f7-12bc-4e05-ac89-3b6ad4a78e94.png" width = "250">

처음 여기에 체크박스를 체크한 경우 자동으로 IntentConfiguration 으로 설정된다.

<img width="600" alt="16" src="https://user-images.githubusercontent.com/69136340/132086929-1bbc08c5-5bbd-40d8-a1bc-a97873f5428e.png">

자 이제 나머지 코드도 살펴보자

## 📌 [IntentTimelineProvider](https://developer.apple.com/documentation/widgetkit/intenttimelineprovider) & [TimelineProvider](https://developer.apple.com/documentation/widgetkit/timelineprovider)

위의 체크박스 설정 시 user-configurable 위젯을 위한 코드로 구성된다. 

- IntentTimelineProvider : user-configurable 위젯의 표시를 업데이트할 때 WidgetKit 에 조언하는 유형.
- TimelineProvider : 위젯의 표시를 업데이트할 때 WidgetKit 에 조언하는 유형.

**위의 프로토콜을 채택하는 Provider 로 선언된 구조체에 대해서 알아보자.**

**(Include Configuration Intent 체크박스 체크해서 IntetnTimelineProvider 로 구조체가 정의되었다.)**

```swift
import WidgetKit
import SwiftUI
import Intents

// ✅ Widget 의 업데이트할 시기를 WidgetKit 에게 알려주는 프로토콜.
struct Provider: IntentTimelineProvider {
    // ✅ (Required)
    func placeholder(in context: Context) -> SimpleEntry {
        SimpleEntry(date: Date(), configuration: ConfigurationIntent())
    }

    // ✅ (Required)위젯의 현재시간과 상태를 보여주는 timeline entry 를 제공한다.
    // ✅ 위젯을 추가할때와 같이 일시적인 상황에서 나타낼 때 호출.
    // ✅ WidgetKit 이 timeline 을 요청하는 방법 첫번째(아래 확인)
    func getSnapshot(for configuration: ConfigurationIntent, in context: Context, completion: @escaping (SimpleEntry) -> ()) {
        let entry = SimpleEntry(date: Date(), configuration: configuration)
        completion(entry)
    }
    // ✅ (Required)현재 시간 및 선택적으로 위젯을 업데이트할 미래 시간에 대한 timeline entry 배열을 제공합니다.
    // ✅ 처음에 위젯을 랜더링하기 위해서 Provider 에게 timeline 을 요청할 때 호출.
    // ✅ WidgetKit 이 timeline 을 요청하는 방법 두번째(아래 확인)
    func getTimeline(for configuration: ConfigurationIntent, in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        var entries: [SimpleEntry] = []

        // Generate a timeline consisting of five entries an hour apart, starting from the current date.
        let currentDate = Date()
        // ✅ 1시간 간격으로 업데이트. 시스템 시간 기준이 아닌 currentDate 를 만든 시간 기준이다. (ex.현재시간 1시. entry 는 [1시,2시,3시,4시,5시]) 
        for hourOffset in 0 ..< 5 {
            let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
            let entry = SimpleEntry(date: entryDate, configuration: configuration)
            entries.append(entry)
        }
        // ✅ policy 가 .atEnd 이기 때문에 마지막 날짜 즉, 5시가 지나면 WidgetKit 이 새 타임라인을 요청한다.
        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
}

// timeline entry 구조체
struct SimpleEntry: TimelineEntry {
    let date: Date
    let configuration: ConfigurationIntent
}
```

## 📌 WidgetKit 이 timeline 을 요청하는 방법 2가지

### 1️⃣ 현재 시간과 상태를 나타내는 스냅샷 줘

<img src ="https://user-images.githubusercontent.com/69136340/132086951-511b26bb-9228-4167-92b5-32cb5f858b57.PNG" width = "250">

위처럼 위젯을 추가할 때 이미 위젯이 그려진다. 위젯을  추가할 때와 같이 일시적인 상황에서 표시하기 위해서 snapshot 요청을 하고 그 메서드가 바로 `getSnapshot(for:in:completion:)` 이다.

### 2️⃣ 현재 시간 및 미래 시간을 포함한 timeline entry 배열 줘

다음은 WidgetKit 이 timeline provider 에게 timline 을 요청하는 과정이다.

Widget Extension 이 항상 실행되는 것은 아니기 때문에 WidgetKit 은 언제 실행해야할지 timeline 을 알아야 하기 때문에 `getTimeline(for:in:completion:)` 메서드로 요청한다.

<img width="450" alt="18" src="https://user-images.githubusercontent.com/69136340/132086955-4ef34d27-162e-4e63-8c3b-0f848cbd0d58.png">

`getTimeline(for:in:completion:)` 메서드는 다음과 같이 탈출 클로저를 가진다.

```swift
let timeline = Timeline(entries: entries, policy: .atEnd)
completion(timeline)
```

policy 파라미터를 살펴보자. timeline 의 [TimelineReloadPolicy](https://developer.apple.com/documentation/widgetkit/timelinereloadpolicy) 를 정하는 역할이다.

- .atEnd : timeline 의 마지막 날짜가 지난 후 WidgetKit 이 새 timeline 을 요청하도록 지정.
- .after(Date) : WidgetKit이 새 timeline 요청할 미래 날짜를 지정.
- .never : 새 timeline 을 사용할 수 있을 때 앱이 WidgetKit에 알림

다음과 같이 사용될 수 있다.(다음은 애플 예제)

<img width="419" alt="22" src="https://user-images.githubusercontent.com/69136340/132086977-4500a37a-c9ef-4660-9350-333949354606.png">

### 참고 :

[Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/widget/)

[Apple Developer Documentation](https://developer.apple.com/documentation/WidgetKit/Keeping-a-Widget-Up-To-Date)

[iOS 14+ ) Widget](https://zeddios.tistory.com/1077)

[WidgetKit (2) - TimelineEntry / TimelineProvider / TimelineReloadPolicy](https://zeddios.tistory.com/1089)

## 깃허브

[GitHub - 28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu: 🧚 아요 미라클론코딩 김현규](https://github.com/28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu)
