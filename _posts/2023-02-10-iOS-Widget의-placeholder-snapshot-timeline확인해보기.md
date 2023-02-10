---
title:  "iOS) Widget 의 placeholder, snapshot, timeline 확인해보기"
categories:
- iOS

date:   2023-02-10  13:50:00 +0900
author_profile: false
---
### 내용

- Widget 이 보여지는 세 가지 시점이 있습니다. placeholder, snapshot, timeline 를 확인해보겠습니다.
- 언제 보여지는지 UI 를 통해 확인해보자.

`TimelineProvider` 프로토콜은 다음 세 가지 메서드를 요구합니다.

```swift
struct Provider: TimelineProvider {
    
    typealias Entry = SampleEntry
    
    func placeholder(in context: Context) -> Entry {
        // Implementation here...
    }

    func getSnapshot(in context: Context, completion: @escaping (Entry) -> ()) {
        // Implementation here...
    }

    func getTimeline(in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        // Implementation here...
    }
}
```

위의 Entry 를 반환해야하는 세 가지 메서드를 구현해야 합니다. 다음의 코드로 EnytryView 에서 위젯이 어떻게 그려지는지 확인해보겠습니다.

```swift
struct SampleEntry: TimelineEntry {
    let date: Date
    let text: String
}

struct EnytryView : View {
    var entry: Provider.Entry
    
    var body: some View {
        Text(entry.text)
        ...
    }
}
```

### [plcaeholder(in:)](https://developer.apple.com/documentation/widgetkit/intenttimelineprovider/placeholder(in:))

WidgetKit 이 처음으로 위젯을 표시할 때, 위젯의 뷰를 placeholder 로 렌더링합니다.

WidgetKit 은 위젯의 placeholder configuration 을 요청하기 위해 `placeholder(in:)` 를 호출합니다. 

이를 확인해보기 위해서 다음과 같이 코드를 작성하였습니다.

```swift
func placeholder(in context: Context) -> Entry {
    // string 파라미터로 전달되는 문자열을 가지고 SwiftUI redaction effect 를 적용합니다. 실제 값은 중요하지 않습니다.
    return SampleEntry(date: Date(), text: "placeholder")
}
```

또한, WidgetKit 은 사용자가 Apple Watch 또는 iPhone 잠금 화면에서 민감한 정보를 숨기도록 선택한 경우, 위젯을 placeholder 로 렌더링할 수 있습니다.

`placeholder(in:)` 은 동기적이며 TimelineEntry 를 즉시 반환합니다. 가능한 빨리 `placeholder(in:)` 에서 돌아갑니다.

placeholder 의 길이와 크기만큼 다르게 렌더링되는 것을 확인할 수 있었습니다.

<img width="500" alt="스크린샷 2023-02-10 오후 1 45 23" src="https://user-images.githubusercontent.com/69136340/218003218-59a45d31-7fbb-4257-b344-ce8eee1c85fd.png">

### [getSnapshot(in:completion:)](https://developer.apple.com/documentation/widgetkit/timelineprovider/getsnapshot(in:completion:))

WidgetKit 은 위젯이 일시적인 상황에 나타날 때 `getSnapshot(in:completion:)` 을 호출합니다.

`context.isPreview` 가 true 일 경우에 위젯 갤러리에 나타납니다.(즉, 위젯 갤러리에서 `getSnapshot(in:completion:)` 호출되는데 이때 파라미터로 전달되는 context 의 isPreveiw 속성의 값이 true 입니다.)

이 경우 위젯의 현재 상태를 가져오거나 계산하는데 몇 초 이상 걸릴 수 있는 경우 샘플 데이터를 제공하여 가능한 빨리 completion handler 를 호출해야 합니다.

이를 확인해보기 위해서 다음과 같이 코드를 작성하였습니다.

```swift
func getSnapshot(in context: Context, completion: @escaping (Entry) -> ()) {
    let entry = SampleEntry(date: Date(), text: "snapshot")

    completion(entry)
}
```

### [getTimeline(in:completion:)](https://developer.apple.com/documentation/widgetkit/timelineprovider/gettimeline(in:completion:))

위젯을 업데이트하기 위해 현재 시간 및 선택적으로 미래 시간에 대한 timeline entries 를 제공합니다.

```swift
func getTimeline(in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
    let entry = SampleEntry(date: Date(), text: "timeline")
    let timeline = Timeline(entries: [entry], policy: .never)

    completion(timeline)
}
```

### 결과

좌측에서부터 placeholder, snapshot, timeline 입니다. 

<img width="700" alt="스크린샷 2023-02-10 오후 1 46 28" src="https://user-images.githubusercontent.com/69136340/218003255-43a0456c-016b-4925-bc33-e3edd94b92fe.png">

**참고:**

[Apple Developer Documentation - placeholder(in:)](https://developer.apple.com/documentation/widgetkit/intenttimelineprovider/placeholder(in:))

[[Apple Developer Documentation - getsnapshot(for:in:completion:)](https://developer.apple.com/documentation/widgetkit/intenttimelineprovider/getsnapshot(for:in:completion:))

[Apple Developer Documentation - gettimeline(for:in:completion:)](https://developer.apple.com/documentation/widgetkit/intenttimelineprovider/gettimeline(for:in:completion:))

[Getting Started With WidgetKit - Swift Senpai](https://swiftsenpai.com/development/getting-started-widgetkit/)
