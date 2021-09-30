---
title:  "iOS) 클립보드(UIPasteboard)에 복사하기"
categories:
- iOS

date:   2021-09-29  15:56:00 +0900
author_profile: false
---
**내용**
- 버튼을 통해서 특정 문자열을 손쉽게 클립보드에 복사하기.

# 시작전

먼저 `UIPateboard` 에 대해서 개발자문서를 통해 알아보자.

## 🖇 [UIPasteboard](https://developer.apple.com/documentation/uikit/uipasteboard)

사용자가 앱 내의 한 위치에서 다른 위치로 데이터를 공유하는 것을 돕는 객체.

**Overview**

다른 앱과 데이터를 공유하려면 시스템 전체의 genaral pasteboard 를 사용해라.

같은 team ID 를 가지는 다른 앱으로 데이터를 공유하려면 pasteboards 의 이름을 사용해라. 

일반적으로 앱의 객체는 유저가 인터페이스의 선택항목에 대한 복사, 잘라내기, 복제 작업을 요청할 때 데이터를 쓴다. 그런 다음 같거나 다른 앱의 다른 객체가 pasteboard 에서 데이터를 읽고 새 위치에서 제공한다. 이것은 일반적으로 붙여넣기 작업을 요청할 때 발생한다.

**Note**

iOS 14 부터 시스템은 다른앱의 pasteboard 컨텐츠를 가져오면 사용자에게 알린다. 

### The General Pasteboard and Named Pasteboards

시스템 전체의 general pasteboard 는 [general](https://developer.apple.com/documentation/uikit/uipasteboard/1622106-general) 상수로 식별된다. 모든 유형의 데이터를 이를 통해서 paste board 를 얻어서 사용가능하다.

[init(name:create:)](https://developer.apple.com/documentation/uikit/uipasteboard/1622074-init) 및 [withUniqueName()](https://developer.apple.com/documentation/uikit/uipasteboard/1622087-withuniquename) 클래스 메서드로 pasteboards 를 생성하여 동일한 Team ID 를 가진 다른 앱으로 데이터를 공유할 수 있다.

### Using Pasteboards

UIPasteboard 클래스는 개별적인 pasteboard items 를 읽고 쓰는 방법과 여러가지 pasteboard items 를 한번에 읽고 쓰는 방법을 제공한다. 자세한 내용은 이 문서 하단의 `Getting and Setting pasteboard Items` 를 참조해라.

pasteboard 에 쓸 데이터는 다음 두 형식중 하나일 수 있다.

- 데이터가 NSString, NSArray, NSDictionary, NSDate, NSNumber, UIImage, or NSURL 과 같은 객체로 표현될 수 있는 경우 값으로 표시될 수 있다. [setValue(_:forPasteboardType:)](https://developer.apple.com/documentation/uikit/uipasteboard/1622079-setvalue) 와 같은 메서드를 사용해서 pastboard 에 쓴다.
- 만약 데이터가 binary 인 경우, [setData(_:forPasteboardType:)](https://developer.apple.com/documentation/uikit/uipasteboard/1622075-setdata) 메서드를 사용하여 pastboard 에 쓴다.

UIPasteboard 클래스는 단일 혹은 여러 pasteboard items 에 문자열, 이미지, URLs, colors 을 읽고 쓰기위한 편리한 메서드를 제공한다. (문서 하단의 `Getting and Setting Pasteboard Items of Standard Data Types` 참조)

<img src ="https://user-images.githubusercontent.com/69136340/135217531-c7a96469-6512-4326-b8cf-27a12146b42a.png" width ="500">

다음을 사용해서 UIPasteboard 에 데이터를 기록해보겠다. 꽤 간단하다!

# 시작하기

- 버튼에 이벤트를 부여해서 클립보드로 손쉽게 복사해보기로 했다.
- 데이터 공유하기 위해서 general pasteboard 를 얻어서 사용해보자

### 🖇 Main 뷰

<img src="https://user-images.githubusercontent.com/69136340/135217419-667cfd71-a612-40e5-bd1a-feaa7b49aa9c.png" width ="550">

### 🖇 UIPasteboard 이용

- MainViewController

```swift
import UIKit

class ViewController: UIViewController {

    @IBOutlet weak var idText: UILabel!

    override func viewDidLoad() {
        super.viewDidLoad()
    }

// ✅ 버튼 클릭시 idText 의 텍스트 pastebaord 에 복사.
    @IBAction func pasteText(_ sender: Any) {
        UIPasteboard.general.string = idText.text
    }
}
```

### 🖇 완성

<img src ="https://user-images.githubusercontent.com/69136340/135217448-89cabcc4-c7a6-4a15-bc24-1e3a2706938f.gif" width="250">

### 출처

[Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uipasteboard#1654138)

### 깃허브

[GitHub - hyun99999/UIPasteboardTutorial-iOS: 📎 클립보드로 복사하기 대작전](https://github.com/hyun99999/UIPasteboardTutorial-iOS)
