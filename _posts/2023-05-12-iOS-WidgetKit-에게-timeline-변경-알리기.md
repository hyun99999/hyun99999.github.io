---
title:  "iOS) WidgetKit 에게 timeline 변경 알리기"
categories:
- iOS

date:   2023-05-12  00:12:00 +0900
author_profile: false
---
아래 개발자 문서를 참고해서 timeline 이 변경되었을 때 WidgetKit 에 알리는 방법에 대해 알아보겠습니다.

[Keeping a widget up to date | Apple Developer Documentation](https://developer.apple.com/documentation/widgetkit/keeping-a-widget-up-to-date#Inform-WidgetKit-when-a-timeline-changes)

**그리고 다음의 목표를 구현해보겠습니다.**

- 회원탈퇴 or 로그아웃할 때는 위젯을 엠티뷰로 업데이트하고,
- 위젯이 표현하는 명함이 삭제될 때는 위젯을 대표명함(첫 번째 명함)으로 업데이트해보겠습니다.

## [Inform WidgetKit when a timeline changes](https://developer.apple.com/documentation/widgetkit/keeping-a-widget-up-to-date#Inform-WidgetKit-when-a-timeline-changes)

무언가 widget 의 현재 timeline 에 영향을 미칠 때, 앱이 WidgetKit 에게 새로운 timeline 을 요청할 수 있습니다.

예를 들어, 게임 위젯 예제에서 팀원이 캐릭터에게 치유 물약을 주었다는 푸시 알림을 앱이 수신하면 앱은 WidgetKit 에 timeline 을 다시 로드하고, widget 의 컨텐츠를 업데이트할 수 있습니다.

또한, 위젯에서 사용자가 계정에 로그아웃 한 경우 모든 위젯을 다시 로드할 수 있습니다.

```swift
// 특정 유형의 widget 을 로드하기 위햇 앱은 WidgetCenter 를 다음과 같이 사용할 수 있음.
// kind 는 위젯의 WidgetConfiguration 을 생성하는데 사용한 문자열과 동일함.
WidgetCenter.shared.reloadTimelines(ofKind: "com.mygame.character-detail")

// 앱에서 WidgetBundle 을 사용하여 여러 위젯을 지원한다면, 다음과 같이 모든 위젯의 timeline 을 다시 로드할 수 있음.
WidgetCenter.shared.reloadAllTimelines()
```

또한, 위젯이 user-configurable properties 가 있는 경우 WidgetCenter 를 사용해서 적절한 설정이 있는지 확인하여 불필요한 reload 를 피할 수 있습니다.

예를 들어, 치료 물략을 받는 캐릭터에 대한 푸시 알림을 받으면 timeline 을 다시 로드하기 전에 위젯이 해당 캐릭터를 표시하는지 확인할 수 있습니다.

이후에 widget 의 컨텐츠를 업데이트할 수 있습니다.

다음 코드에서 `getCurrentConfigurations(_:)` 를 호출해서 user-configured widget 의 목록을 검색합니다. 그런 다음 결과인 WidgetInfo 객체를 반복하여 치유 물약을 받은 캐릭터로 구성된 intent 객체를 찾습니다. 찾게되면 앱은 해당 widget’s kind 에 대해 `reloadTimelines(ofKind:)` 메서드를 호출합니다.

```swift
WidgetCenter.shared.getCurrentConfigurations { result in
    guard case .success(let widgets) = result else { return }

    // Iterate over the WidgetInfo elements to find one that matches
    // the character from the push notification.
    if let widget = widgets.first(
        where: { widget in
            let intent = widget.configuration as? SelectCharacterIntent
            return intent?.character == characterThatReceivedHealingPotion
        }
    ) {
        WidgetCenter.shared.reloadTimelines(ofKind: widget.kind)
    }
}
```

### 적용해보기

- 해당 명함이 삭제되었다 or 회원탈퇴 or 로그아웃했을 때 WidgetKit 에게 알려 위젯을 업데이트해보겠습니다.
    - 회원탈퇴 or 로그아웃할 때는 위젯을 엠티뷰로 업데이트하고,
    - 위젯이 표현하는 명함이 삭제될 때는 위젯을 대표명함(첫 번째 명함)으로 업데이트해보겠습니다.

해당 동작이 이루어질 때마다 `WidgetCenter.shared.reloadTimelines(ofKind: "MyCardWidget")` 메서드를 호출하여 위젯의 timeline 을 다시 로드하면 됩니다.

예를 들어, 다음과 같이 로그아웃 시에 호출하면 됩니다.

```swift
func setLogoutClicked() {
    // MyCardWidget timeline reload.
    WidgetCenter.shared.reloadTimelines(ofKind: "MyCardWidget")
    // 액세스 토큰 삭제.
    UserDefaults.standard.removeObject(forKey: "accessToken")
}
```

그렇다면 timeline 은 어떻게 구성하면 될까요?

- 현재로부터 5초뒤의 timeline entry 를 한 개 만들고, **TimelineReloadPolicy** 를 `atEnd` 로 설정해서 5초 뒤에 또 다시 timeline 을 생성하도록 하겠습니다.

```swift
    func getTimeline(for configuration: MyCardIntent, in context: Context, completion: @escaping (Timeline<Entry>) -> Void) {
        var entries: [MyCardEntry] = []

        let currentDate = Date()
        let entryDate = Calendar.current.date(byAdding: .second, value: 5, to: currentDate) ?? Date()
        
        if let card = configuration.myCard {
            // ✅ 서버통신
            cardListFetchWithAPI { result in
                switch result {
                case .success(let response):
                    if let data = response?.data {
                        // ✅ 해당 위젯의 명함(configuration.myCard.identifier)이 현재 로그인된 계정의 명함 목록(cardDataModel.cardUUID)에 있는가? 
                        if data.contains(where: { cardDataModel in
                            return cardDataModel.cardUUID == card.identifier
                        }) {
                            // ✅ 명함이 있음.
                            let entry = MyCardEntry(date: entryDate,
                                                    widgetCard: WidgetCard(cardUUID: card.identifier ?? "",
                                                                           title: card.displayString,
                                                                           userName: card.userName ?? "",
                                                                           // fetchImage: 이미지 다운로드 메서드.
                                                                           backgroundImage: fetchImage(card.cardImage ?? "")))
                            entries = [entry]
                            let timeline = Timeline(entries: entries, policy: .atEnd)
                            completion(timeline)
                        } else {
                            // ✅ 해당 명함이 삭제되어 없음. -> 대표 명함(첫 번째 명함)을 보여주도록 함
                            let entry = MyCardEntry(date: entryDate,
                                                   widgetCard: WidgetCard(cardUUID: data[0].cardUUID,
                                                                           title: data[0].cardName,
                                                                           userName: data[0].userName,
                                                                           backgroundImage: fetchImage(data[0].cardImage)))
                            entries = [entry]
                            let timeline = Timeline(entries: entries, policy: .atEnd)
                            completion(timeline)
                        }
                    }
                case .failure(let error):
                    print(error)
                    // ✅ 회원 탈퇴, 로그아웃
                    // Widget 에서 widgetCard 를 옵셔널 바이딩하여 nil 이면 엠티뷰가 설정되도록 분기처리하였습니다.
                    let entry = MyCardEntry(date: entryDate, widgetCard: nil)
                    entries = [entry]
                    let timeline = Timeline(entries: entries, policy: .atEnd)
                    completion(timeline)
                }
            }
        } else {
            // ✅ Configuration 로 부터 card 받지 못함.(=로그인 전 위젯 생성 시)
            let entry = MyCardEntry(date: entryDate, widgetCard: nil)
            entries = [entry]
            let timeline = Timeline(entries: entries, policy: .atEnd)
            completion(timeline)
        }
    }
```

### 결과

- 로그아웃 시에 엠티뷰로 위젯 업데이트(회원탈퇴도 동일한 결과)

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/bf43ad75-898d-428b-9a84-ca569082a799" width ="200">

- 명함 삭제 시에 대표 명함으로 위젯 업데이트(직장명함1 삭제. 덕질명함이 대표명함)

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/a65d2e65-416a-44c3-8b91-ed192451ba2e" width="200">

