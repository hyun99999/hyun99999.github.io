---
title:  "iOS) CoreData 를 사용하여 Configurable Widget 만들기 (3/3) - Configurable Widget"
categories:
- iOS

date:   2022-12-31  03:44:00 +0900
author_profile: false
---
1. 프로젝트 세팅
2. Widget 만들기
**✅ 3. Configurable Widget 만들기**

- **Configurable Widget 을 만들어 보겠습니다.**
    - **위젯 편집할 때 정직/동적 선택 목록을 구현.**
    - **처음 위젯을 추가할 때 기본이 되는 세팅을 구현.**
    - **여러가지 카드(명함이름, 이름을 가진)로 바꿀 수 있게 구현.(메모 위젯처럼 목록에서 선택할 수 있도록)**
- **MyCard Widget 은 Intent Configuration.**
- **QRCode Widget 은 Static Configuration.**
    - **위젯을 통해 앱의 특정 뷰로 이동.**

## 👉 ****Configurable Widget 만들기****

[iOS) Configurable Widget 만들기](https://gyuios.tistory.com/260)

위의 글을 참고해서 **Configurable Widget** 을 만들어보겠습니다.

- configurable properties 정의하는 **custom intent definition** 을 추가.
- **Intents extension** 을 구현하고 **IntentTimelineProvider** 를 사용해서 위젯이 동적 데이터를 반영.

### 1️⃣ SiriKit Intent Definition File 추가 및 설정

- custom intent 를 설정하기 위해서 프로젝트에 파일을 추가해줍니다.

<img width="500" alt="1" src="https://user-images.githubusercontent.com/69136340/210101528-9ef305b4-73cc-4be8-b713-6cf5a5c9f191.png">

- 이때, 사용하려는 타겟(widget extension)에 추가해줍니다.

<img width="602" alt="2" src="https://user-images.githubusercontent.com/69136340/210101641-faf134b8-ed2a-4ed5-b26a-fa8d41d20779.png">

- `Editor > New Intent` 를 선택해서 intent 를 추가해줍니다.
- 코드에서는 `SelectMyCardIntent` 라는 클래스 이름으로 참조할 수 있습니다.(**Attribute Inspector** 의 **Custom Class** 필드에서 확인할 수 있습니다.)
- `Intent is eligible for widgets` 체크박스를 체크합니다.(위젯에서 Intent 를 사용하겠다는 의미)

<img src="https://user-images.githubusercontent.com/69136340/210101677-2196f7be-2b09-4841-96b9-aef25a3523af.png" width ="700">

- configurable setting 이 되는 파라미터를 추가하겠습니다.
- **우리는 여러가지 명함 목록**에서 고르기 위해서 **Parameter** 에 `MyCard` 를 추가하였습니다.

<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/210101724-6e0f78fd-5075-4daf-bf16-e22c4cea71ac.png">

### 2️⃣ 우선, 정적인 선택 목록을 생성해보자.

- **매개변수가 사용자에게 정적인 선택 목록을 제공하는 경우 static enumeration 을 생성해주면 됩니다.**
- 좌측 하단의 + 를 선택해서 **New Enum** 을 선택하면 static enumeration 을 생성할 수 있습니다.

<img width="300" alt="5" src="https://user-images.githubusercontent.com/69136340/210101744-ef452129-441c-4a0d-8c13-3421130c4b08.png">

<img width="600" alt="6" src="https://user-images.githubusercontent.com/69136340/210101753-92d29fb3-8555-4eb5-9b72-a31b5580ebbd.png">

- 이렇게 하면 수동으로 파라미터의 **Type pop-up menu** 에서 설정해줘야 합니다. 혹은, **Type pop-up menu** 에서 **Add Enum…** 을 통해서 바로 생성하고 설정할 수도 있습니다.

<img width="700" alt="7" src="https://user-images.githubusercontent.com/69136340/210101775-a011756e-e657-4ab9-9f32-6965de8938e8.png">

- 다음과 같이 static enumeration 을 설정하겠습니다.

<img width="700" alt="8" src="https://user-images.githubusercontent.com/69136340/210101802-5cbf4ccb-c4bc-415e-8c4f-f32e880ad5c3.png">

(🚨 Case 에 한글로 작성하고, 코드의 정의를 따라가면 아래와 같이 enum 이 만들어집니다. **Case** 에서는 영어로 작성해주고, **Display Name** 에서 한글로 작성하면 의도한대로 됩니다!)

<img width="600" alt="9" src="https://user-images.githubusercontent.com/69136340/210101815-b140895e-a1ef-4dc1-8364-c699d098486d.png">

- 결과

<img width="500" alt="10" src="https://user-images.githubusercontent.com/69136340/210101835-bc95417f-51a5-4e6a-834b-7a621eec89a7.png">

### 3️⃣ 이제, 동적인 선택 목록을 생성해보자.

- **목록이 다양하거나 동적으로 생성되는 경우, dynamic options 가 있는 type 을 대신 사용할 수 있습니다.**
- **Type pop-up menu** 에서 **Add Type…** 을 통해 새로운 type 을 추가 및 설정할 수 있습니다. `Card` Enum 을 추가하겠습니다.

<img width="700" alt="11" src="https://user-images.githubusercontent.com/69136340/210101856-0dec86be-51ae-4694-b50e-dc95d3553bd1.png">

- `cardName` property 를 새롭게 추가해주고, **Type pop-up menu** 로부터 `String` 을  선택해줍니다.

<img width="700" alt="12" src="https://user-images.githubusercontent.com/69136340/210101876-e303b68e-485a-45cb-b13c-fdbf52891e69.png">

- **동적으로 만들 것이기 때문에** `Options are provided dynamically` 체크박스를 선택해 줍니다.

<img src="https://user-images.githubusercontent.com/69136340/210101904-0246f8c2-9bbf-4f48-b781-7d9b5e8b1335.png" width ="700">

### 4️⃣ 동적으로 선택 목록을 프로젝트에 제공해보자.(Intents extension)

- 프로젝트에 **Intent extension target** 을 추가합니다.

<img width="500" alt="14" src="https://user-images.githubusercontent.com/69136340/210102102-857bb311-ca40-4d41-9880-710a0b061c9d.png">

- Intents extension 의 이름을 입력하고 **Starting Point** 를 `None` 을 설정합니다.

<img width="500" alt="15" src="https://user-images.githubusercontent.com/69136340/210102113-3d3a962a-db33-4702-a3b9-3eb49b149c30.png">

- 이전에 만든 Intent definition file(SelectMyCardIntents) 을 선택하고, **Target Membership** 에서 containing app(extension 을 포함하는 앱), widget extension, Intents extension 모두 체크해주면 됩니다.

<img width="700" alt="16" src="https://user-images.githubusercontent.com/69136340/210102208-90078f40-6072-4aef-a940-9767c5be6ebb.png">

- 동적인 값을 제공하기 위해서 Intent 을 핸들링해야 하는데 이때 프로토콜을 채택해서 확장할 수 있습니다.

<img width="600" alt="17" src="https://user-images.githubusercontent.com/69136340/210102228-9b89586e-8f7c-4b5b-912b-7cd3feedbf74.png">

- 프로토콜을 준수하라는 오류 메시지의 fix 를 통해 아래의 stub 함수를 만들어주는데, **escaping closure를 사용하는 함수와  async/await 를 사용하는 함수 두 가지로 준수할 수 있습니다.**

### 🚨 트러블 슈팅 - resolveCharacter(for:with:)?

- 이때 `provide[Type]OptionsCollection(for:with:)` 외에도 `resolve[Type](for:with:)` 메서드를 구현하라고 합니다.

<img width="700" alt="18" src="https://user-images.githubusercontent.com/69136340/210102273-da35b33d-597e-4ce7-9737-b40e9b75b97b.png">

- 이는 아래의 Resolvable 를 체크해서 생기는 필수 구현 메서드입니다. 해제해주면 됩니다.
<img src="https://user-images.githubusercontent.com/69136340/210102287-c071d412-8f63-4dd8-b540-913c58fe59f4.png" width ="700">

---

이어서 진행해보겠습니다.

`provideMyCardOptionsCollection(for:with:)` 메서드를 구현하기 위해서 containing app 에서 구조체를 생성하겠습니다. 그렇기 때문에 타겟은 다음과 같이 설정합니다.

<img width="500" alt="20" src="https://user-images.githubusercontent.com/69136340/210102328-605058ed-4e11-48c9-9290-5f74ecbf947c.png">

- `MyCardDetail.swift` 를 사용해서 `provideMyCardOptionsCollection(for:with:)` 구현해보겠습니다.

```swift
import UIKit

struct MyCardDetail {
    let cardName: String
    let userName: String
    let cardImage: UIImage
    
    static let availableMyCards = [
        MyCardDetail(cardName: "첫번째 카드", userName: "첫현규", cardImage: UIImage(named: "background") ?? UIImage()),
        MyCardDetail(cardName: "두번째 카드", userName: "두현규", cardImage: UIImage(named: "imgCardWidget") ?? UIImage())
    ]
}
```

- `IntentHandler.swift`

```swift
import Intents

class IntentHandler: INExtension, SelectMyCardIntentHandling  {
    func provideMyCardOptionsCollection(for intent: SelectMyCardIntent, with completion: @escaping (INObjectCollection<Card>?, Error?) -> Void) {

        // ✅ intent defintion file 에서 만든 custom type 인 Card 를 사용.
        let myCards: [Card] = MyCardDetail.availableMyCards.map { card in

            // ✅ 구분자가 되는 identifier 로 card.cardName 프로퍼티 사용.
            let myCard = Card(identifier: card.cardName, display: card.cardName)

            // ✅ 위의 convenience initializer 말고도 새로 추가한 attribute 에 대해서 초기화.
            myCard.cardName = card.cardName
            return myCard
        }
        
        // ✅ completion handler 로 넘겨줄 INObjectCollection 생성.
        let collection = INObjectCollection(items: myCards)

        completion(collection, nil)
    }
    
    override func handler(for intent: INIntent) -> Any {
        // This is the default implementation.  If you want different objects to handle different intents,
        // you can override this and return the handler you want for that particular intent.
        
        return self
    }
}
```

- intent definition file 에서 설정한 intent 의 parameter 의 **Display Name** 이 좌측에 보여집니다.
- `IntentHandler` 에서 `MyCardDetail.availableMyCards` 로 설정한 completion handler 를 통해 우측에 동적인 목록이 보여집니다.

<img width="500" alt="21" src="https://user-images.githubusercontent.com/69136340/210102343-ad8251f2-2f1d-4935-9060-f2f761623eca.png">

## 🚨 트러블 슈팅 - 기본값으로 첫번째 카드가 선택되도록 하고싶어요!

- 기본값이 선택되지 않아서 아래와 같이 비어있게 되는데 선택되도록 해보겠습니다.

<img width="500" alt="22" src="https://user-images.githubusercontent.com/69136340/210102359-28ac88c5-c23d-4a20-8888-759de0db7f00.png">

- dynamic options 에서 parameter 의 기본값을 `default[Type](for:)` (설정한 Custom Type 이 들어감)메서드로 설정할 수 있습니다.

```swift
func defaultMyCard(for intent: SelectMyCardIntent) -> Card? {
    let defaultCard = MyCardDetail.availableMyCards[0]
    let card = Card(identifier: defaultCard.cardName, display: defaultCard.cardName)
    card.cardName = defaultCard.cardName
        
    return card
}
```

### 결과

- 기본값을 설정하였기 때문에 “첫번째 카드” 로 표시되고, 선택 목록 중에 체크되어 있는 것을 확인할 수 있습니다.

<img src="https://user-images.githubusercontent.com/69136340/210102379-f73d3fe2-2225-4b35-baa8-f47705cb01d7.mp4" width ="250">


***결과 화면 중에 위젯에 데이터가 반영되는 과정을 다음 단계에서 진행해보겠습니다.***

---

### 5️⃣ IntentHandler 수정

- 공유된 데이터를 **Configurable Widget** 에 반영해보겠습니다.
    - `IntentHandler` 에서 데이터 모델의 static property 를 사용하던 로직에서 CoreData 를 사용하여 조회하여 동적 선택 목록을 구현해보겠습니다.
    - `IntentTimelineProvider` 를 수정해서 **widget entry** 를 통해 데이터를 전달해보겠습니다.
- `IntentHandler.swift` 에서 `CoreDataManager` 클래스를 사용하기 위해서 `CoreDataManager` 의  **Target Membership** 도 수정해줍니다.

<img width="300" alt="24" src="https://user-images.githubusercontent.com/69136340/210102434-b456a5e4-b7c9-46bd-9289-bc6c9ec0895c.png">

- `IntentHandler.swift` 에서 **AppGroup** 을 활용한 `CoreDataManager` 를 사용하기 때문에 다음과 같이 **Capability** 를 추가해야한다.

<img width="600" alt="25" src="https://user-images.githubusercontent.com/69136340/210102468-eef03ae5-b719-4859-9829-e510883233b2.png">

- 기존에 static property 인 `MyCardDetail.availableMyCards` 을 사용하던 코드에서 CoreData 를 활용한 코드로 변경하였습니다.

```swift
import Intents
  
class IntentHandler: INExtension {
    override func handler(for intent: INIntent) -> Any {
        // This is the default implementation.  If you want different objects to handle different intents,
        // you can override this and return the handler you want for that particular intent.
        
        return self
    }
}

extension IntentHandler: SelectMyCardIntentHandling {
    func provideMyCardOptionsCollection(for intent: SelectMyCardIntent, with completion: @escaping (INObjectCollection<Card>?, Error?) -> Void) {

     // let myCards: [Card] = MyCardDetail.availableMyCards.map { card in
        // ✅ CoreData 조회.
        let cardDetail = CoreDataManager.shared.fetch(entityName: "CardDetail")
        let myCards: [Card] = cardDetail.map { card in
            let cardName = card.value(forKey: "cardName") as? String ?? ""
            let myCard = Card(identifier: cardName, display: cardName)
            
            myCard.cardName = cardName
            return myCard
        }
        
        let collection = INObjectCollection(items: myCards)

        completion(collection, nil)
    }
    
    func defaultMyCard(for intent: SelectMyCardIntent) -> Card? {

     // let defaultCard = MyCardDetail.availableMyCards[0]
        // ✅ CoreData 조회.
        let cardDetail = CoreDataManager.shared.fetch(entityName: "CardDetail")
        let defaultCardName = cardDetail[0].value(forKey: "cardName") as? String ?? ""        

        let card = Card(identifier: defaultCardName, display: defaultCardName)
        card.cardName = defaultCardName
        
        return card
    }
}
```

### 6️⃣ IntentTimelineProvider 과 위젯 수정

- core data 로 조회된 명함의 세부사항을 알려주는 `MyCardDetail` 데이터 모델을 widget 에서 사용할 것이기 때문에 다음과 같이 **Target Membership** 에서 `WidgetsExtension` 도 체크해줍니다.

<img width="300" alt="26" src="https://user-images.githubusercontent.com/69136340/210102507-ce3e754f-bb19-4ad2-bc48-a30de115751b.png">

- `MyCardWidget.swift` 에서 CoreData 를 통해 데이터를 조회하고, **Provider** 를 수정하여 `cardName` 이 동일한 `CardDetail` 을 결정하여 **widget view** 에서 보여주겠습니다. 그럼 최종적으로 다음과 같이 작성할 수 있습니다.

```swift
import WidgetKit
import SwiftUI
import Intents

// MARK: - Provider

// ✅ Configurable widget 을 만들기 위해서 IntentTimelineProvider 채택.
struct MyCardProvider: IntentTimelineProvider {
    
    // 👉 WidgetKit 이 처음으로 위젯을 표시할 때 placeholder 로 렌더링합니다.
    // placeholder view 는 사용자에게 위젯의 일반적인 표현을 표시하여 정보를 제공합니다.
    func placeholder(in context: Context) -> MyCardEntry {

        // ✅ CoreData 조회.
        // 가장 첫 번째 정보(대표가 되는 card)를 보여주겠습니다.
        let cardDetail = CoreDataManager.shared.fetch(entityName: "CardDetail")
        let myCardDetail = MyCardDetail(cardName: cardDetail[0].value(forKey: "cardName") as? String ?? "",
                                        userName: cardDetail[0].value(forKey: "userName") as? String ?? "",
                                        cardImage: UIImage(data: cardDetail[0].value(forKey: "cardImage") as? Data ?? Data()) ?? UIImage())
        return MyCardEntry(date: Date(), detail: myCardDetail)
    }
    
    // 👉 위젯을 추가할때와 같이 일시적인 상황에서 표시하기 위해서 snapshot 을 요청한다.
    // 위젯의 현재 상태를 가져오거나 계산하는데 몇 초가 걸릴 수 있는 경우에는 샘플 데이터를 제공하여 가능한 빨리 completion handler 를 호출해야 합니다.
    func getSnapshot(for configuration: SelectMyCardIntent, in context: Context, completion: @escaping (MyCardEntry) -> ()) {

        // ✅ CoreData 조회.
        // 가장 첫 번째 정보(대표가 되는 card)를 보여주겠습니다.
        let cardDetail = CoreDataManager.shared.fetch(entityName: "CardDetail")
        let myCardDetail = MyCardDetail(cardName: cardDetail[0].value(forKey: "cardName") as? String ?? "",
                                        userName: cardDetail[0].value(forKey: "userName") as? String ?? "",
                                        cardImage: UIImage(data: cardDetail[0].value(forKey: "cardImage") as? Data ?? Data()) ?? UIImage())
        let entry = MyCardEntry(date: Date(), detail: myCardDetail)
        completion(entry)
    }
    
    // 👉 위젯을 업데이트하기 위해서 현재 시간 및 선택적으로 미래 시간에 대한 timline 배열을 제공합니다.
    func getTimeline(for configuration: SelectMyCardIntent, in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        var entries: [MyCardEntry] = []
        
        let currentDate = Date()
        for hourOffset in 0 ..< 5 {
            let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
            
            // ✅ CoreData 조회.
            let cardDetail = CoreDataManager.shared.fetch(entityName: "CardDetail")
            
            // ✅ SelectMyCardIntent 에서 전달되는 cardName 을 cardDetail 과 대조해서 동일한 카드를 결정.
            let myCardName = configuration.MyCard?.cardName
            cardDetail.forEach { card in
                guard let cardName = cardDetail[0].value(forKey: "cardName") as? String else { return }
                
                if myCardName == cardName {
                    let myCardDetail = MyCardDetail(cardName: cardName,
                                                    userName: cardDetail[0].value(forKey: "userName") as? String ?? "",
                                                    cardImage: UIImage(data: cardDetail[0].value(forKey: "cardImage") as? Data ?? Data()) ?? UIImage())
                    let entry = MyCardEntry(date: entryDate, detail: myCardDetail)
                    entries.append(entry)
                }
            }
        }
        
        let timeline = Timeline(entries: entries, policy: .never)
        completion(timeline)
    }
}

// MARK: - TimelineEntry
struct MyCardEntry: TimelineEntry {
    let date: Date
    let detail: MyCardDetail
}

// MARK: - View
struct MyCardEnytryView : View {
    var entry: MyCardProvider.Entry
    @Environment(\.colorScheme) var colorScheme
    
    var body: some View {
        ZStack {
            Color.white
            GeometryReader { proxy in
                HStack(spacing: 0) {
                    // ✅ MyCardEntry timeline entry 을 사용해서 동적으로 위젯에서 표시할 수 있다.
                    Image(uiImage: entry.detail.cardImage)
                        .resizable()
                        .aspectRatio(contentMode: .fill)
                        .frame(width: proxy.size.height * (92 / 152), height: proxy.size.height)
                        .clipped()
                    Color.backgroundColor(for: colorScheme)
                }
            }
            VStack {
                HStack {
                    // ✅
                    Text(entry.detail.cardName)
                        .font(.system(size: 15))
                        .foregroundColor(.init(white: 1.0, opacity: 0.8))
                        .padding(EdgeInsets(top: 12, leading: 10, bottom: 0, trailing: 0))
                        .lineLimit(1)
                    Spacer()
                    Image("logoNada")
                        .padding(EdgeInsets(top: 10, leading: 0, bottom: 0, trailing: 10))
                }
                Spacer()
                HStack {
                    Spacer()
                    // ✅
                    Text(entry.detail.userName)
                        .font(.system(size: 15))
                        .foregroundColor(.userNameColor(for: colorScheme))
                        .padding(EdgeInsets(top: 0, leading: 10, bottom: 11, trailing: 10))
                        .lineLimit(1)
                        // .foregroundColor(colorScheme == .light ? Color(red: 19.0 / 255.0, green: 20.0 / 255.0, blue: 22.0 / 255.0) : Color(red: 255.0 / 255.0, green: 255.0 / 255.0, blue: 255.0 / 255.0))
                }
            }
        }
    }
}

// MARK: - Widget
struct MyCardWidget: Widget {
    let kind: String = "MyCardWidget"
    
    var body: some WidgetConfiguration {
        IntentConfiguration(kind: kind,
                            intent: SelectMyCardIntent.self,
                            provider: MyCardProvider()) { entry in
            MyCardEnytryView(entry: entry)
        }
        .configurationDisplayName("명함 위젯")
        .description("명함 이미지를 보여주고,\n내 명함으로 빠르게 접근합니다.")
        .supportedFamilies([.systemSmall])
    }
}

// MARK: - Preivews
struct MyCardWidget_Previews: PreviewProvider {
    static var previews: some View {
        MyCardEnytryView(entry: MyCardEntry(date: Date(), detail: MyCardDetail.availableMyCards[0]))
            .previewContext(WidgetPreviewContext(family: .systemSmall))
    }
} 
```

## 👉 QR Code 위젯은 static configuration

- QR Code 위젯은 위젯 편집이 필요하지 않은 static configuration 입니다. 이는 위젯을 처음에 만들 때 `Include Configuration Intent` 를 체크하지 않으면 자동으로 코드를 만들어줍니다.

<img width="500" alt="27" src="https://user-images.githubusercontent.com/69136340/210102523-18379634-0dcf-4711-b20a-b8f89305cd57.png">

- **Configuration Intent 를 포함하지 않는 코드가 어떻게 차이가 나는지 잠깐 살펴보겠습니다.**

```swift
import WidgetKit
import SwiftUI
// ✅import Intents

// ✅struct Provider: IntentTimelineProvider {
struct Provider: TimelineProvider {
    func placeholder(in context: Context) -> SimpleEntry {
//    ✅SimpleEntry(date: Date(), configuration: ConfigurationIntent())
        SimpleEntry(date: Date())
    }

// ✅func getSnapshot(for configuration: ConfigurationIntent, in context: Context, completion: @escaping (SimpleEntry) -> ()) {
    func getSnapshot(in context: Context, completion: @escaping (SimpleEntry) -> ()) {
//     ✅let entry = SimpleEntry(date: Date(), configuration: configuration)
        let entry = SimpleEntry(date: Date())
        completion(entry)
    }

// ✅func getTimeline(for configuration: ConfigurationIntent, in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
    func getTimeline(in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        var entries: [SimpleEntry] = []
        let currentDate = Date()
        for hourOffset in 0 ..< 5 {
            let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
//        ✅let entry = SimpleEntry(date: entryDate, configuration: configuration)
            let entry = SimpleEntry(date: entryDate)
            entries.append(entry)
        }

        let timeline = Timeline(entries: entries, policy: .never)
        completion(timeline)
    }
}

struct SimpleEntry: TimelineEntry {
    let date: Date
//  ✅let configuration: ConfigurationIntent
}

struct QRCodeEnytryView : View {
// 👉 이미지로 뷰를 채웠습니다.
    Image("widgetQr")
            .resizable()
            .scaledToFill()
}

struct QRCodeWidget: Widget {
    let kind: String = "QRCodeWidget"
    
    var body: some WidgetConfiguration {
//    ✅IntentConfiguration(kind: kind, intent: ConfigurationIntent.self, provider: Provider()) {
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            QRCodeEnytryView(entry: entry)
        }
        // ...
    }
}

struct QRCodeWidget_Previews: PreviewProvider {
    static var previews: some View {
//    ✅QRCodeEnytryView(entry: SimpleEntry(date: Date(), configuration: ConfigurationIntent()))
        QRCodeEnytryView(entry: SimpleEntry(date: Date()))
            .previewContext(WidgetPreviewContext(family: .systemSmall))
    }
}
```

QRCodeWidget 은 이미지를 사용하여서 뷰를 구현하였습니다.**(static configuration 이기 때문에 위젯 편집이 없습니다.)**

<img width="500" alt="28" src="https://user-images.githubusercontent.com/69136340/210102549-c4004ef8-45ae-44ec-a0dc-f9194b747327.png">

추가적으로 위젯을 통해 앱의 특정 뷰로 이동하도록 구현하겠습니다. 아래의 글을 작성하였습니다.

[iOS) 위젯으로 앱의 특정 뷰로 이동(widgetURL)](https://gyuios.tistory.com/261)

## 👉 GitHub

https://github.com/hyun99999/WidgetsWithCoreDataTutorial-iOS

**참고:**

[Add configuration and intelligence to your widgets - WWDC20 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2020/10194/)

[[WidgetKit] IntentConfiguration으로 Edit Widget 기능을 추가하기](https://eunjin3786.tistory.com/216)

[iOS) Configurable Property 가 있는 Widget 만들기](https://gyuios.tistory.com/260)
