---
title:  "iOS) Share and Action Extension 사용해보기(2) - Action Extension"
categories:
- iOS

date:   2021-11-30  03:00:00 +0900
author_profile: false
---
[Creating an iOS 10 Action Extension](https://www.techotopia.com/index.php/Creating_an_iOS_8_Action_Extension)

위의 튜토리얼을 큰틀로 잡고 따라가지만 코드는 수정하였습니다

## 1️⃣ Action Extension 추가

- [File] > [New] > [Target] 에서 Action Extension 을 선택해준다.


<img width="450" alt="1" src="https://user-images.githubusercontent.com/69136340/143911860-96421761-eecb-40f1-804e-f5ee7d335dc6.png">

<img width="450" alt="2" src="https://user-images.githubusercontent.com/69136340/143911863-e5eb0b3e-03c5-47a2-9a5d-8bf779af4e99.png">
<img width="200" alt="3" src="https://user-images.githubusercontent.com/69136340/143911871-4e239a5e-e3a2-400c-b09f-72244da1097e.png">

## 2️⃣ Action Extension 사용

- Action extension 템플릿은 principal view controller class (ActionViewController라고 함) 과 Info.plist 파일 및 인터페이스 파일(즉, 스토리보드 또는 xib 파일)에 대한 기본 소스 파일을 제공합니다.


<img width="300" alt="4" src="https://user-images.githubusercontent.com/69136340/143911977-292dd838-6ffc-4559-858b-c5de0449a731.png">

<aside>
👻 iOS. Action Extension 을 전체 화면으로 표시하려면 확장의 NSExtension 사전에 다음 키-값 쌍을 추가하세요.
`<key>NSExtensionActionWantsFullScreenPresentation</key>`
`<true/>`

</aside>

추가했으니 잘 나오는지 확인해봐야겠죠?

- 사진앱에서 acivity view 를 살펴보면 맨 아래에 ActionExtension 이 추가된 것을 볼 수 있다.

<img src="https://user-images.githubusercontent.com/69136340/143912023-6649133a-306e-4bbf-9873-eea572cd209b.jpeg" width ="250">

선택해볼까요?

- 기본적으로 Action Extension 은 다음과 같이 이미지를 가져오도록 구현되어 있습니다.

<img src="https://user-images.githubusercontent.com/69136340/143912155-cfd37a73-e7aa-4faa-986d-d385182ec503.jpeg" width="250">

### 🪓 activity view 띄우기

> 우리는 해당 Action Extension 을 선택하면 text view 의 텍스트를 바꿔주는 Action 을 만들어볼 예정입니다.
> 
- UITextView 를 추가해주고 텍스트를 "변경 전" 으로 설정한다. activity view 를 띄울 버튼도 추가한다.

<img src="https://user-images.githubusercontent.com/69136340/143912144-034e5740-180a-47ab-a0db-fbb568e1b848.jpeg" width="250">

- 버튼에 다음과 같은 화면전환 코드를 추가해준다.

```swift
// ✅ init(activityItems:applicationActivities:)
let activityViewController = UIActivityViewController(activityItems: [textView.text ?? ""], applicationActivities: nil)
self.present(activityViewController, animated: true, completion: nil)
```

### ✅ **init(activityItems:applicationActivities:)**

**activityItems**

activity 를 수행할 data 객체의 배열. 예를 들어, 데이터는 현재 선택된 콘텐츠를 나타내는 하나 이상의 문자열 또는 이미지 개체로 구성될 수 있다.

실제 데이터 객체 대신 UIActivityItemProvider 개체와 같이 UIActivityItemSource 프로토콜을 채택하는 개체일 수 있다.

이 배열은 nil 이 아니여야 하며 하나의 객체를 반드시 포함해야 한다.

**applicationActivities**

애플리케이션이 지원하는 커스텀 서비스를 나타내는 UIActivity 객체의 배열. 이 파라미터는 nil 일 수 있다.

activity view 가 정상적으로 등장하는지 확인해볼까요?

- 해당 액션을 가진 버튼을 탭하게되면 아래처럼 activity view 가 등장하고 추가한 ActionExtension 가 나온다. (현재 context 와 관련된 extension 이 등장한다. 텍스트 → Copy)

<img src="https://user-images.githubusercontent.com/69136340/143912273-65ba6048-1ea5-410a-bcc5-2c6e2eb8ecf8.jpeg" width ="250">


## 3️⃣ MainInterface 변경

> 우리는 해당 Action Extension 을 선택하면 text view 의 텍스트를 바꿔주는 Action 을 만들어볼 예정입니다.
> 
- 아래의 기본으로 있는 UIImageView 를 삭제한다. UITextView 를 추가해고 오토레이아웃을 잡아준다.

<img width="300" alt="10" src="https://user-images.githubusercontent.com/69136340/143912295-0d5b0e13-b5b5-4e9d-abf2-556a3eaf7c5f.png">

- 인터페이스 빌더에서 Behavior 의 Editable 을 체크 해제한다.

<img width="600" alt="11" src="https://user-images.githubusercontent.com/69136340/143912569-dcb8f22c-1bf0-4d87-998b-cd27ac2516e8.png">

## 4️⃣  host app 에서 컨텐츠 받기

> Action Extension 은 NSExtensionContext 메서드를 사용하여 host app 의 컨텐츠를 extesion 으로 보낼 수 있습니다.
> 
- Action Extension 에서 기존의 "이미지 뷰에 이미지를 추가하는 코드" 대신 다음의 코드로 수정해보자.

```swift
// ✅ extensionContext : Returns the extension context of the view controller. 자료형 NSExtensionContext
// ✅ inputitems : The list of input NSExtensionItem objects associated with the context. 자료형 NSExtensionItem
// ✅ attachments : An optional array of media data associated with the extension item. 자료형 [NSItemProvider]?
guard let textItem = self.✅extensionContext?.✅inputItems.first as? NSExtensionItem,
        let textItemProvider = textItem.✅attachments?.first else { return }
// ✅
if textItemProvider.hasItemConformingToTypeIdentifier(UTType.text.identifier) {
        // ✅
        textItemProvider.loadItem(forTypeIdentifier: UTType.text.identifier, options: nil) { item, error in
            guard let text = item as? String else { return }
            DispatchQueue.main.async {
                self.myTextView.text = text
            }
        }
}
```

### ✅ **hasItemConformingToTypeIdentifier(_:)**

item provider 에 지정된 UIT(파일 및 데이터 전송을 위한 유형 식별자) 를 준수하는 데이터 표현이 포함되어 있는지 여부를 나타내는 Boolean 값 리턴.

### ✅ **loadItem(forTypeIdentifier:options:completionHandler:)**

item 의 데이터를 가져오고 필요할 경우 지정된 유형으로 강제 변환.

- host app 의 text view 의 "변경 전" 이라는 텍스트를 extension 으로 가져온 모습

<img src="https://user-images.githubusercontent.com/69136340/143912648-84e6a6a8-9647-4b3a-a14d-d689d5f1063b.gif" width ="250">


## 5️⃣  host app 으로 컨텐츠 보내기

> Action Extension 은 NSExtensionContext 메서드를 사용하여 사용자의 편집 내용을 host app 으로 보낼 수 있습니다.
> 
- Action Extension 을 처음에 만들 때 부터 이미 기본적으로 `Done` 이라는 버튼이 연결이 되어 있습니다.

```swift
@IBAction func done() {
        // Return any edited content to the host app.
        // This template doesn't do anything, so we just echo the passed in items.
        self.extensionContext!.completeRequest(returningItems: self.extensionContext!.inputItems, completionHandler: nil)
    }
```

텍스트를 수정해서 host app 으로 보낼 것이기 때문에 수정해보겠습니다.

- 아래처럼 extension context 부터 접근해서 값을 가져왔기 때문에 내보낼때는 거꾸로 접근해야합니다!

```swift
// ✅ extensionContext 자료형 : NSExtensionContext
// ✅ inputitems 자료형 : NSExtensionItem
// ✅ attachments 자료형 : [NSProvider]?
guard let textItem = self.✅extensionContext?.✅inputItems.first as? NSExtensionItem,
        let textItemProvider = textItem.✅attachments?.first else { return }
```

> extension context > input items(extension item) > attachments(item provider)
> 

- item provider >  extension item > extension context

```swift
@IBAction func done() {
        let returnItemProvider = NSItemProvider(item: myTextView.text as NSSecureCoding?, typeIdentifier: UTType.text.identifier)
        let returnItem = NSExtensionItem()
        
        returnItem.attachments = [returnItemProvider]
        // ✅ 
        self.extensionContext?.completeRequest(returningItems: [returnItem], completionHandler: nil)
    }
```

### ✅ **completeRequest(returningItems:completionHandler:)**

host app 에 result itme 배열로 app extension 요청을 완료하도록 지시.

보내줬으니 host app 에서 받아야 겠죠?

## 6️⃣ host app 에서 받은 컨텐츠 활용

- 앞서 extension item 에서 컨텐츠를 가져왔듯이 completionWithItemsHandler 의 파라미터 items(`NSExtensionItem`) 을 사용해서 값을 가져오면 된다.

```swift
// ... 

// UTType 를 사용하기 위해서
import UniformTypeIdentifiers

// ...

@objc private func presentToActivityVC() {
        let activityViewController = UIActivityViewController(activityItems: [textView.text ?? ""], applicationActivities: nil)
        
        // ✅ 기본제공 서비스 중 사용하지 않을 ActivityType 제거할 수 있다.(에어드랍 제거)
//        activityViewController.excludedActivityTypes = [UIActivity.ActivityType.airDrop]

        // ✅ activity 가 완료되거나 activity view controller 가 해제되면 completion block 이 실행.
        activityViewController.completionWithItemsHandler = { activity, success, items, error in
            if success {
                guard let textItem = items?.first as? NSExtensionItem, let itemProvider = textItem.attachments?.first else { return }
                if itemProvider.hasItemConformingToTypeIdentifier(UTType.text.identifier) {
                    itemProvider.loadItem(forTypeIdentifier: UTType.text.identifier, options: nil) { item, error in
                        DispatchQueue.main.async {
                            self.textView.text = item as? String
                        }
                    }
                }
            } else {
                print(error)
           }
        }

        self.present(activityViewController, animated: true, completion: nil)
}
```

### ✅ **UIActivityViewController.CompletionWithItemsHandler**

Delclaration

```swift
typealias CompletionWithItemsHandler = ([UIActivity](https://developer.apple.com/documentation/uikit/uiactivity).[ActivityType](https://developer.apple.com/documentation/uikit/uiactivity/activitytype)?, [Bool](https://developer.apple.com/documentation/swift/bool), [Any]?, [Error](https://developer.apple.com/documentation/swift/error)?) -> [Void](https://developer.apple.com/documentation/swift/void)
```

activity 가 완료되거나 activity view controller 가 해제되면 completion block 이 실행된다. 이 block 을 사용해서 최종 코드를 실행할 수 있다. 매개변수는 다음과 같다.

**activityType**

사용자가 선택한 service 의 타입. 커스텀 서비스의 경우, UIActivity 객체의 activityType 메서드에서 반환된 값. 시스템 정의 activities 의 경우, UIActivity 의 내장 Activity Type 중 하나.(앞서 사용하지 않을 ActivityType 에서 살펴본 타입들 의미한다.)

**completed**

서비스가 수행된 경우 `true`. 수행되지 않은 경우 `false`. 이 매개변수는 사용자가 서비스를 선택하지 않고 뷰 컨트롤러를 닫을 때도 `false` 이다.

**returnedItems**

`NSExtensionItem` 객체 배열. 이 배열의 item 을 활용해서 original data 에서 extension 에 의해 변경된 사항을 가져온다. 수정된 항목이 없으면 `nil` 이다.

**activityError**

activity 가 완료되지 않은 경우 `error` 객체. activity 가 완료된 경우 `nil`.

### 결과

<img src="https://user-images.githubusercontent.com/69136340/143912694-d6f11051-9cf3-4e5e-b4eb-10f1fa92e662.gif" width ="250">

---

## 🪓 Declaring the Supported Content Type

진행하면서 다음과 같은 워닝이 뜨는것을 볼 수 있었을 거에요..

<img width="650" alt="00000" src="https://user-images.githubusercontent.com/69136340/143912800-6cc360b9-77f8-4b35-9804-15c3d2107981.png">

> 앱을 App Store에 제출하기 전에 TRUEPREDICATE의 모든 사용을 특정 술어 문 또는 NSExtensionActivationRule 키로 대체해야 합니다. 포함하는 앱의 확장 프로그램에 TRUEPREDICATE가 포함되어 있으면 앱이 거부됩니다.
> 

<img width="500" alt="1111" src="https://user-images.githubusercontent.com/69136340/143912841-55c1c76c-3ed8-4438-bb00-63186e3dd6dc.png">

info.plist 를 보면 진짜 TRUEPREDICATE 로 된 부분이 있네요..! 

### NSExtensionActivationRule

[Apple Developer Documentation](https://developer.apple.com/documentation/bundleresources/information_property_list/nsextension/nsextensionattributes/nsextensionactivationrule)

NSExtensionActivationRule 에 해당하는 value 를 넣어주면 됩니다! 저희는 text 를 다뤘기 때문에 **`NSExtensionActivationSupportsText`** 를 사용해주면 적합해요!

- **`NSExtensionActivationSupportsText` :** app extension 이 text 를 지원하는지의 여부.

`NSExtensionActivationRule` 을 `Dictionary` 으로 타입을 바꾸고 key-value 를 추가해주면 워닝이 없어집니다!

<img width="500" alt="22222" src="https://user-images.githubusercontent.com/69136340/143912866-dddcedcf-30bd-40ea-8f6b-d0d4b41f240e.png">

## 🪓 Changing the Extension Display Name

액션의 이름을 바꾸고싶다면 아래의 `Bundle display name` 에서 바꿀 수 있습니다!

<img width="650" alt="333" src="https://user-images.githubusercontent.com/69136340/143912992-9a59f4b1-ef71-45f9-b8f3-7ce0465cc729.png">

<img src="https://user-images.githubusercontent.com/69136340/143912995-ee5147b0-062f-4087-92cb-f24e8276394a.png" width="250">


