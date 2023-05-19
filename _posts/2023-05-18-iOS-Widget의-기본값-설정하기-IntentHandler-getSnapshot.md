 ---
title:  "iOS) Widget 의 기본값 설정하기(IntentHandler, getSnapshot)"
categories:
- iOS

date:   2023-05-18  18:32:00 +0900
author_profile: false
---
### 목표

- configurable widget 의 기본값을 설정해보겠습니다.
- 명함이 없다면 엠티뷰를, 명함이 있다면 첫 번째 명함을 기본값으로 설정해보겠습니다.
- 첫 번째 명함이 바뀌는 경우, 해당 명함이 삭제되는 경우, 로그아웃 및 회원탈퇴에 대응해서 기존의 기본값을 업데이트해보겠습니다.

두 가지 방법을 통해서 Widget 의 기본값을 설정해 보겠습니다.

첫 번째 방법은 IntentHandler 에서 서버통신을 통해 Intent 의 기본값을 설정하여서 Widget 의 `getSnapshot(for:in:completion:)` 에서 설정하는 것이고,

두 번째 방법은 Widget 의 `getSnapshot(for:in:completion:)` 에서 서버통신을 통해 설정하는 것입니다.

### 들어가기 전

두 가지 방법을 적용해보기 전에 구현하고자하는 엠티뷰와 위젯의 UI 에 대해서 살펴보겠습니다.

- 명함을 구분하는 identifier 가 되는 cardUUID
- 명함의 이름인 cardName
- 유저의 이름인 userName
- 명함 배경인 cardImage

를 데이터로 다룰 것입니다.

<img width="250" alt="1" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/923c7ed2-df9e-4130-92a4-a4e25a75048b">

### 1️⃣ IntentHandler 에서 default[[Parameter Name](for:)](for:) 메서드를 활용

```swift
// IntentHandler.swift

import Intents

class IntentHandler: INExtension {
   // ...
}

extension IntentHandler: MyCardIntentHandling {
    // 내 명함 목록 선택할 때 호출.
    func provideMyCardOptionsCollection(for intent: MyCardIntent, with completion: @escaping (INObjectCollection<MyCard>?, Error?) -> Void) {
        // ...
    }
    
    // ✅ 위젯 추가할때 호출. 기본값 설정.
    func defaultMyCard(for intent: MyCardIntent) -> MyCard? {
        var myCard: MyCard?
        
        let group = DispatchGroup()
        
        DispatchQueue.global().async(group: group) { [weak self] in
            group.enter()
            
            // ✅ 서버통신을 통해서 기본값이 되는 데이터 설정.
            self?.cardListFetchWithAPI { result in
                switch result {
                case .success(let result):
                    if let card = result?.data {
                        myCard = MyCard(identifier: card[0].cardUUID, display: card[0].cardName)
                        myCard?.userName = card[0].userName
                        myCard?.cardImage = card[0].cardImage
                    }
                case .failure(let err):
                    print(err)
                }
                group.leave()
            }
        }
        
        _ = group.wait(timeout: .now() + 60)
        
        return myCard
    }
}
```

pre-configuration 을 가지고 snapshot 을 만들게 되면 위젯을  추가하는 단계에서도 기본값으로 설정된 위젯을 확인할 수 있습니다.

```swift
// MyCardWidget.swift

import WidgetKit
import SwiftUI
import Intents

struct MyCardProvider: IntentTimelineProvider {
    func placeholder(in context: Context) -> MyCardEntry {
        // ...
    }

    func getSnapshot(for configuration: MyCardIntent, in context: Context, completion: @escaping (MyCardEntry) -> Void) {
        if let myCard = configuration.myCard {
            completion(MyCardEntry(date: Date(), widgetCard: WidgetCard(cardUUID: myCard.identifier ?? "",
                                                                        title: myCard.displayString,
                                                                        userName: myCard.userName ?? "",
                                                                        backgroundImage: fetchImage(myCard.cardImage ?? ""))))
        } else {
            completion(MyCardEntry(date: Date(), widgetCard: nil))
        }
    }

    func getTimeline(for configuration: MyCardIntent, in context: Context, completion: @escaping (Timeline<Entry>) -> Void) {
        // ...
    }

```

위와 같이 작성하면 다음의 목표를 만족할 수 있습니다.

- **명함이 없다면 엠티뷰를, 명함이 있다면 첫 번째 명함을 기본값으로 설정해보겠습니다.**

다음의 목표를 구현해보겠습니다.

- **첫 번째 명함이 바뀌는 경우, 해당 명함이 삭제되는 경우, 로그아웃 및 회원탈퇴에 대응해서 기존의 기본값을 갱신해보겠습니다.**

### 🚨 트러블 슈팅

현재까지는 위젯의 기본값이 위젯을 추가할때마다 매번 호출되는 것이 아닙니다. 최초에 호출되는 것을 확인하였습니다.

- 해결 방법

그래서 이를 IntentHandler 에서 다루는 것이 아니라 Widget 의 `getSnapshot(for:in:completion:)` 메서드에서 서버통신을 통해 설정해보겠습니다.

두 번째 화면이 snapshot 입니다. 이 위젯을 추가하는 snapshot 단계를 업데이트 해보기로 했습니다.

<img width="600" alt="2" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/8f790ad0-833a-4389-b854-5cfce38be475">

### 2️⃣ Widget 의 Provider 의 `getSnapshot(for:in:completion:)` 메서드를 활용

```swift
func getSnapshot(for configuration: MyCardIntent, in context: Context, completion: @escaping (MyCardEntry) -> Void) {
        // ✅ 서버통신을 통해서 명함 목록 가져오기
        cardListFetchWithAPI { result in
            switch result {
            case .success(let response):
                if let data = response?.data {
                    if !data.isEmpty {
                        // ✅ 첫 번째 명함을 기본값으로 설정.
                        let entry = MyCardEntry(date: Date(), widgetCard: WidgetCard(cardUUID: data[0].cardUUID,
                                                                                     title: data[0].cardName,
                                                                                     userName: data[0].userName,
                                                                                     backgroundImage: fetchImage(data[0].cardImage)))
                        completion(entry)
                    } else {
                        // ✅ 명함이 없음. 
                        completion(MyCardEntry(date: Date(), widgetCard: nil))
                    }
                }
            case .failure(let error):
                // ✅ 로그인 전.
                print(error)

                completion(MyCardEntry(date: Date(), widgetCard: nil))
            }
        }
    }
```

비록 바로바로 갱신된 snapshot 이 보이지는 않았습니다. 하지만, 첫 번째 방법보다는 확실하게 자주 갱신됨을 확인할 수 있었습니다.

하지만, 기본값이 되는 위젯의 선택 목록을 구성하는 pre-configuration 을 IntentHandler 에서 만들지 못하기 때문에 다음의 목표는 만족할 수 없습니다.

- **명함이 없다면 엠티뷰를, 명함이 있다면 첫 번째 명함을 기본값으로 설정해보겠습니다.**

### 결론

- 첫 번째 방법은 IntentHandler 에서 서버통신을 통해 Intent 의 기본값을 설정하여서 Widget 의 `getSnapshot(for:in:completion:)` 에서 설정
    - IntentHandler 에서 default[[Parameter Name](for:)](for:) 메서드를 사용하게되면 최초에 한번 호출되기 때문에 기본값을 정적으로 설정해줄 때 적합했습니다.
    - 또한, configuration 을 위젯에게 전달할 수 있기 때문에 snapshot 을 만들 때 사용할 수 있습니다.
- 두 번째 방법은 Widget 의 `getSnapshot(for:in:completion:)` 에서 서버통신을 통해 설정
    - 위젯을 추가하는 snapshot 을 얻기위한 단계에서 개입할 수 있기 때문에 위젯을 추가할때마다 기본값에 대해서 확인해야할 경우에는 해당 방법이 적합했습니다.
    - 하지만, 위젯을 추가할 때 snapshot 의 기본값이 전달되지 않기 때문에 pre-configuration 을 설정할 수 없고,  snapshot 을 만들 때도 서버통신을 해야합니다.

두 가지 방법에 대해서 알고 스펙에 맞게 적용하면 될 것 같습니다.
