---
title:  "iOS) 저장하지 않고 dismiss 할 때 ActionSheet 띄우기"
categories:
- iOS

date:   2021-04-22 01:16:00 +0900
author_profile: false
---
저장하지 않고 dismiss 할 때 ActionSheet 띄우기
### UIAdaptivePresentationControllerDelegate

- UIAdaptivePresentationControllerDelegate

<img src = "https://user-images.githubusercontent.com/69136340/115586758-e1017600-a307-11eb-8627-8635003378eb.png" width ="500">

4개의 메서드가 있다.
> `func presentationControllerDidAttemptToDismiss(UIPresentationController)`
: Notifies the delegate that a user-initiated attempt to dismiss a view was prevented.

>`func presentationControllerShouldDismiss(UIPresentationController) -> Bool`
: Asks the delegate for permission to dismiss the presentation.

> `func presentationControllerDidDismiss(UIPresentationController)`
: Notifies the delegate after a presentation is dismissed.

> `func presentationControllerWillDismiss(UIPresentationController)`
: Notifies the delegate before a presentation is dismissed.

- presentationControllerDidAttemptToDismiss(_:)

dismiss 를 시도할 때 ActionSheet 를 띄울 것이기 때문에 이 메서드를 사용한다.

<img src = "https://user-images.githubusercontent.com/69136340/115587054-3178d380-a308-11eb-8687-7ded02e357c5.png" width ="550">

- UIKit 은  `isModalInPresentation` 이 `true` 일 경우, dismiss 거부를 지원한다. 그래서 이 메서드를 사용해서 `UIAertController` 인스턴스를 표시하는 등 dismiss 할 수 없는 이유를 사용자에게 알릴 수 있다.

### isModalInPresentation

- isModalInPresentation

<img src = "https://user-images.githubusercontent.com/69136340/115586521-a4357f00-a307-11eb-8e1f-eaaa4d83780e.png" width ="500">

뷰컨이 modal behavior 를 적용하는지 여부를 나타내는 boolean 값이다. 즉, `true` 로  설정하면 interactive dismissal 를 막는 상태를 의미한다.

### ActionSheet
presentationControllerDidAttemptToDismiss(_:) 로 ActionSheet 를 띄워보자.

뷰에서 변경사항이 없는대도 dismiss 를 거부하게되면 안되니까 hasChanges 라는 변수를 만들어서 여부를 설정하자.

```swift
//
override func viewWillLayoutSubviews() {
    
    if #available(iOS 13.0, *) {
        self.isModalInPresentation = self.hasChanges
    } else {
        //Fallback on earlier versions
    }
}
```
뷰의 상태변화를 감지하기 위한 대표적인 메서드는 `viewWillLayoutSubviews` 와 `viewDidLayoutSubViews` 이다. 뷰가 바뀔 때마다 호출된다.

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    navigationController?.presentationController?.delegate = self
}
```

```swift
extension AddListViewController: UIAdaptivePresentationControllerDelegate {
    func presentationControllerDidAttemptToDismiss(_ presentationController: UIPresentationController) {
    if hasChanges {
        let alert = UIAlertController(title: nil, message: nil, preferredStyle: .actionSheet)
        let dismiss = UIAlertAction(title: "변경 사항 폐기", style: .destructive) { _ in
            //Hide keyboard
            self.resignFirstResponder()
            self.dismiss(animated: true, completion: nil)
        }
        let cancel = UIAlertAction(title: "취소", style: .cancel, handler: nil)
        alert.addAction(dismiss)
        alert.addAction(cancel)
        present(alert, animated: true, completion: nil)
    } else {
        dismiss(animated: true, completion: nil)
    }
}
```

### 출처
출처ㅣhttps://developer.apple.com/documentation/uikit/uiviewcontroller/3229894-ismodalinpresentation/

출처ㅣhttps://developer.apple.com/documentation/uikit/uiadaptivepresentationcontrollerdelegate

출처ㅣhttps://zeddios.tistory.com/831?category=682195
