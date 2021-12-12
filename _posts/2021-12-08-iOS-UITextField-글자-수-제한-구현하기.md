---
title:  "iOS) UITextField 글자 수 제한 구현하기"
categories:
- iOS

date:   2021-12-08  11:07:00 +0900
author_profile: false
---
### UITextField 에서 입력받을 글자 수를 제한해보자.

- 먼저 UITextField 의 텍스트가 변경될 때 옵저버에게 알릴 수 있도록 notification center 를 등록했다.

```swift
NotificationCenter.default.addObserver(self,
                                    selector: #selector(textFieldDidChange(_:)),
                                    name: UITextField.textDidChangeNotification,
                                    object: nil)
```

### UITextField.textDidChangeNotification

UITextField 의 텍스트가 변경될 때 observers 에게 알리는 notification

- textFieldDidChange(_:)

cardTitleTextField  의 텍스트가 변경될 때 maxLength(상수로 15를 지정해둔 상황) 보다 클 경우 더이상 입력되지 않도록 제한.

```swift
@objc
private func textFieldDidChange(_ notification: Notification) {
        if let textField = notification.object as? UITextField {
            switch textField {
            case cardTitleTextField:
                if cardTitleTextField.text?.count ?? 0 > maxLength {
                    cardTitleTextField.deleteBackward()
                }
            default:
                return
            }
        }
    }
```

그런데! 15자보다 긴 텍스트를 복사해서 붙여넣게 되면 아예 입력이 되지 않았다.

그래서 15자까지만 입력되도록 코드를 변경해보았다.

```swift
@objc
private func textFieldDidChange(_ notification: Notification) {
        if let textField = notification.object as? UITextField {
            switch textField {
            case cardTitleTextField:
                if let text = cardTitleTextField.text {
                    if text.count > maxLength {
                        // 🪓 주어진 인덱스에서 특정 거리만큼 떨어진 인덱스 반환
                        let maxIndex = text.index(text.startIndex, offsetBy: maxLength)
                        // 🪓 문자열 자르기
                        let newString = text.substring(to: maxIndex)
                        cardTitleTextField.text = newString
                    }
                }
            default:
                return
            }
        }
    }
```

<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/145136055-8123e729-6d14-4bf0-83a4-55dc478f8881.png">

substring(to:) 이 deprecated 됐으니까 partial range upto 연산자를 사용하라고 한다...

### PartialRangeUpTo

A partial half-open interval up to, but not including, an upper bound.

라고 개발자 문서는 설명했는데요... 그니까.. 우리가 아주 잘 알고있는 `..<` 입니다!

half-open range operator. 반개방 범위 연산자라고 불러지는데요.  범위에서 상위의 값을 포함하지 않는 범위를 의미합니다!

그러면 적용해볼까요?

```swift
let newString = text.substring(to: index)
// ->
let newString = String(text[text.startIndex..<maxIndex])
```
