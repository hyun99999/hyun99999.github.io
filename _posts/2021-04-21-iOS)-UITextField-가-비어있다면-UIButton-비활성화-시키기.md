---
title:  "iOS) UITextField 가 비어있다면 UIButton 비활성화 시키기"
categories:
- iOS

date:   2021-04-22  14:44:00 +0900
author_profile: false
---
### UITextField 가 비어있다면 UIButton 비활성화 시키기

```swift
@IBAction func textFieldValueChanged(_ sender: UITextField) {
        if textField.text?.isEmpty == true {
            saveBtn.isEnabled = true
        } else {
            saveBtn.isEnabled = false
        }
    }
```

- UITextFieldDelegate 를 활용해서 구현했다.

```swift
@IBOutlet weak var textField: UITextField!

override func viewDidLoad() {
        super.viewDidLoad()
     
        textField.delegate = self
    }

extension AddListViewController: UITextFieldDelegate {
    func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange, replacementString string: String) -> Bool {
        
        let text = (textField.text! as NSString).replacingCharacters(in: range, with: string)
        
        if !text.isEmpty {
            saveBtn.isEnabled = true
        } else {
            saveBtn.isEnabled = false
        }
        return true
    }
}
```

위의 메서드는 지정된 텍스트가 변경될 때 요청하는 것이다. 즉 text 로 설정한 값이 바뀌면 호출된다는 메서드이다.

이 메서드를 활용해서 text 의 값이 바뀔때마다 text 가 비었다면 saveBtn 를 비활성화시켜주었다.

### 출처
출처ㅣ[Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uitextfielddelegate/1619599-textfield)
