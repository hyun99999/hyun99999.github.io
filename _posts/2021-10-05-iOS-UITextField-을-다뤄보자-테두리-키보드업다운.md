---
title:  "iOS) UITextField 을 다뤄보자 (테두리/키보드업다운)"
categories:
- iOS

date:   2021-10-05  13:47:00 +0900
author_profile: false
---
**내용**

- 텍스트필드 입력 시 테두리 색 변경
- 텍스트필드 입력 완료 후 키보드 다운 구현
- 텍스트가 채워지면 특정 버튼 활성화
- 텍스트필드 선택 후 뷰 터치하면 키보드 다운 구현
- 텍스트필드 선택 후 뷰 올리기(collectionview 에 inset 부여)

프로젝트를 진행하면서 위와 같이 UITextField 를 구현해야했다. 알아보자!

### 완성

<img src ="https://user-images.githubusercontent.com/69136340/135962043-a5cfb543-e6f2-4906-9dcb-2612d9d898ef.gif" width ="250">

## 🖍 테두리 색 변경 / 키보드 업다운 / 채워지면 특정 버튼 활성화

```swift
// cardNameTextField.layer.cornerRadius = 10
// 다음과 같이 cornerRadius 를 지정하지 않으면 borderSytle 프로퍼티에 roundedRect 를 주더라도 boderColor 를 주었을 때 각진 모서리를 가진다.

extension FrontCardCreationCollectionViewCell: UITextFieldDelegate {
// ✅ textField 에서 편집을 시작한 후
    func textFieldDidBeginEditing(_ textField: UITextField) {
// 키보드 업
        textField.becomeFirstResponder()
// 입력 시 textField 를 강조하기 위한 테두리 설정
        textField.borderWidth = 1
        textField.borderColor = Colors.white.color
    }

// ✅ textField 에서 편집이 끝난 후(first responder 를 resign 한 후)
    func textFieldDidEndEditing(_ textField: UITextField) {
// 입력 완료시 총 4개의 텍스트필드가 비어있는지 여부를 판단해서 notification 을 보냄.
// 비어있지 않다면 특정 버튼을 활성화 시키는 notification 을 보낸다.
        if cardNameTextField.hasText && userNameTextField.hasText && birthTextField.hasText && mbtiTextField.hasText {
            NotificationCenter.default.post(name: .frontCardtextFieldIsEmpty, object: false)
        } else {
            NotificationCenter.default.post(name: .frontCardtextFieldIsEmpty, object: true)
        }
// 입력 종료시 테두리 없애기
        textField.borderWidth = 0
    }

// ✅ 텍스트필드에 대한 return 버튼 누름처리를 할 것인지 묻는다.(true: 처리O, false: 처리X)
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
// 키보드 다운
// resignFirstResponder() 는 first responder 를 포기하는 메서드다. 리턴값이 Bool 이고 기본적으로 true 를 구현한다. 
        return textField.resignFirstResponder()
    }
}
```

## 🖍 뷰 터치하면 키보드 다운

```swift
// endEditing(_:) : cauese view to resign first responder
let tapGesture = UITapGestureRecognizer(target: self.view, action: #selector(self.view.endEditing(_:)))
self.view.addGestureRecognizer(tapGesture)
```

## 🖍 텍스트필드 선택시 키보드 크기만큼 뷰 올리기

```swift
// UIResponder.keyboardWillShowNotification : 키보드가 해제되기 직전에 post 된다.
NotificationCenter.default.addObserver(self, selector: #selector(setKeyboardShow(_:)), name: UIResponder.keyboardWillShowNotification, object: nil)
// UIResponder.keyboardWillHideNotificationdcdc : 키보드가 보여지기 직전에 post 된다.
NotificationCenter.default.addObserver(self, selector: #selector(setKeyboardHide(_:)), name: UIResponder.keyboardWillHideNotification, object: nil)

// ✅ 키보드 업
@objc
func setKeyboardShow(_ notification: Notification) {
    if let keyboardFrame = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue {
        let keyboardHeight = keyboardFrame.cgRectValue.height
        self.view.frame.origin.y -= keyboardHeight
    }
}

// ✅ 키보드 다운
@objc
private func setKeyboardHide(_ notification: Notification) {
    if let keyboardFrame = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue {
        let keyboardHeight = keyboardFrame.cgRectValue.height
        self.view.frame.origin.y += keyboardHeight
    }
}
```

UITextField 가 두개이상일 경우에는 경우의 수가 생긴다.

- 첫번째 텍스트 필드 선택 → 두번째 텍스트 필드 선택

이런 경우에는 텍스트 필드를 한번 더 선택해서 키보드가 내려가지 않은채로 `setKeyboardShow(_:)` 메서드가 한번 더 호출된다.

그래서 다음과 같이 구현해보았다.

```swift
// 뷰의 초기 y 값을 저장해서 뷰가 올라갔는지 내려왔는지에 대한 분기처리시 사용.
private var restoreFrameYValue = 0.0
restoreFrameYValue = self.view.frame.origin.y

// ✅ 키보드 업
    @objc
    func showKeyboard(_ notification: Notification) {
// 키보드가 내려왔을 때만 올린다.
        if self.view.frame.origin.y == restoreFrameYValue {
            if let keyboardFrame = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue {
                let keyboardHeight = keyboardFrame.cgRectValue.height
                self.view.frame.origin.y -= keyboardHeight
                print("show keyboard")
            }
        }
    }

// ✅ 키보드 다운
    @objc
    private func hideKeyboard(_ notification: Notification) {
// 키보드가 올라갔을 때만 내린다.
        if self.view.frame.origin.y != restoreFrameYValue {
            if let keyboardFrame = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue {
                let keyboardHeight = keyboardFrame.cgRectValue.height
                self.view.frame.origin.y += keyboardHeight
                print("hide keyboard")
            }
        }
    }
```
