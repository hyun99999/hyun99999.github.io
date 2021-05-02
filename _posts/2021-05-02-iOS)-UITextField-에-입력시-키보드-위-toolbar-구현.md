---
title:  "iOS) UITextField 에 입력시 키보드 위 toolbar 구현"
categories:
- iOS

date:   2021-05-02 21:57:00 +0900
author_profile: false
---
UITextField 에 입력시 키보드 위 toolbar 구현

미리 알림에서 UITextField 에 입력을 할 때 키보드 위 toolbar 를 구현하려고 한다.

<img src ="https://user-images.githubusercontent.com/69136340/116815624-5d624780-ab99-11eb-934e-d7d4308377bd.jpeg" width ="250">

### 계획
UIToolbar 객체를 만들어서 textField 가 first responder 될 때 즉 focus 될 때 receiver 에 accessory view 를 보이게 할 것이다.

### inputAccessoryView
이 속성은 일반적으로 UITextField, UITextView 의 시스템제공 키보드에 악세사리 뷰를 연결하는데 사용. first responder 될 때 즉, focus 될 때 악세사리 뷰를 보여줄 수 있다.

```swift
//set UIToolbar 
private func setToolbar() -> UIToolbar {
    let toolbar = UIToolbar()
    //resize
    toolbar.sizeToFit()
    
    let flexibleSpace = UIBarButtonItem(barButtonSystemItem: .flexibleSpace, target: self, action: nil)
    
    let calendarBarBtn = UIBarButtonItem(image: UIImage(systemName: "calendar.badge.clock"), style: .plain, target: self, action: nil)
    
    let locationBarBtn = UIBarButtonItem(image: UIImage(systemName: "location"), style: .plain, target: self, action: nil)
    
    let flagBarBtn = UIBarButtonItem(image: UIImage(systemName: "flag"), style: .plain, target: self, action: nil)
    flagBarBtn.isEnabled = false
    
    let cameraBarBtn = UIBarButtonItem(image: UIImage(systemName: "camera"), style: .plain, target: self, action: nil)
    cameraBarBtn.isEnabled = false
    
    toolbar.items = [calendarBarBtn, flexibleSpace, locationBarBtn, flexibleSpace, flagBarBtn, flexibleSpace, cameraBarBtn]
    
    return toolbar
}

//inputAccessoryView
reminderField.inputAccessoryView = setToolbar()
```

### 결과
<img src ="https://user-images.githubusercontent.com/69136340/116816375-a2d44400-ab9c-11eb-9bb9-57f46064d1f0.png" width ="250">

### 출처

출처ㅣ[inputAccessoryView](https://developer.apple.com/documentation/uikit/uiresponder/1621119-inputaccessoryview)
